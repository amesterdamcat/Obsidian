# 包材收费与快递费计算交接

## 背景

需求文档：`C:\Users\PC\Documents\Obsidian Vault\包材收费配置需求文档.md`

当前寄付下单计算快递费时，C++ 侧先根据处方计算药品重量，再按门店白名单决定是否调用 Go 运费接口。新需求要求：

- 包材费由我方收取，不进入顺丰计费。
- 包材重量并入药品重量，作为传给顺丰/Go 运费接口的计费重量。
- 患者侧费用需要分列展示：快递费 + 包材费。
- 退费和下单收费链路后续再看，本交接只覆盖计算部分。

## 当前 C++ 入口

主入口：

- `recipe_proj/web/recipecache/src/recipecache.cpp`
- `int CRecipeCache::CalExpressFee(...)`

当前返回渲染：

- `int CRecipeCache::RenderCalExpressFee(...)`
- 目前只返回 `express_fee`。

Go 运费调用：

- `uint32_t CRecipeCache::GoCalExpressFee(...)`
- 请求：

```text
{GO_URL}/express/handler/ComputeShippingFee?clinic_id=...&dest_province=...&dest_city=...&weight=...
```

Go 分流配置：

```cpp
string strGoCalExpressClinic = CEnvironment::GetProperty("GO_CAL_EXPRESS_CLINIC");
CStringUtils::SplitToSet(strGoCalExpressClinic, ",", m_setGoCalExpressClinic);
```

只有 `clinic_id` 命中 `GO_CAL_EXPRESS_CLINIC` 时，才走 Go 运费计算；否则继续走旧 C++ 运费逻辑。

## 当前 Go 分支数据流

代码位置：`recipecache.cpp` 中 `CalExpressFee` 附近。

当前逻辑：

```cpp
if (m_setGoCalExpressClinic.find(expressInfo.dwClinicId) != m_setGoCalExpressClinic.end()) {
    uint32_t dwWeight = 0;
    dwRet = calExpressFee.GetTotalAmont(oCntlInfo, expressInfo, dwWeight);
    if (dwRet != 0) {
        SetError(dwRet, "计算快递总重失败，请稍后重试");
        return dwRet;
    }

    dwRet = GoCalExpressFee(
        oCntlInfo,
        expressInfo.dwClinicId,
        expressInfo.strProvince,
        expressInfo.strCity,
        dwWeight,
        m_dwExpressFee
    );
    if (dwRet != 0) {
        SetError(dwRet, "计算快递费失败，请稍后重试");
        return dwRet;
    }
    return dwRet;
}
```

也就是：

```text
请求 CalExpressFee
-> 解析 clinic_id/province/city/recipe_list
-> 命中 GO_CAL_EXPRESS_CLINIC
-> C++ GetTotalAmont 算药品重量
-> GoCalExpressFee 把药品重量传给 Go ComputeShippingFee
-> Go 返回 amount
-> C++ 返回 express_fee
```

## 药品重量算法

实现位置：

- `recipe_proj/web/recipecache/src/calexpresfee.cpp`
- `uint32_t CCalExprssFee::GetTotalAmont(...)`

核心规则：

1. 根据 `recipe_list` 中的 `register_id` 查处方详情。
2. 如果 `is_down_recipe == 1`，走落地处方查询：`QueryRecipeList + QueryItemList`。
3. 否则走缓存详情：`CRecipeCacheDAOStub4Web::Detail`。
4. 过滤不参与运费计算的处方类型：检查、检验、治疗、手术、其他、协议等。
5. 遍历处方累加重量：

```text
西药：固定加 10，最后除以 10，相当于 1g
中药且代煎：4000 * 剂数，最后除以 10，相当于 400g * 剂数
其他：累加 item.total_amount，最后除以 10
```

最终：

```cpp
dwTotalAmount = dwTotalAmount / 10;
```

传给 Go 的 `weight` 按 Go proto 注释是克。

## Go 运费接口

Go API handler：

- `C:\Users\PC\Documents\go-code\code\go-service\express-service\api\handler\shippingconfig_handler.go`
- `ComputeShippingFee`

Go srv handler：

- `C:\Users\PC\Documents\go-code\code\go-service\express-service\srv\handler\shippingconfig_handler.go`
- `ComputeShippingFee`

Go 侧大致逻辑：

- 用 `clinic_id` 查询门店城市。
- 根据 `dest_province/dest_city` 匹配区域 ID。
- 查城市配置，决定走官方接口还是本地 `t_shipping_config` 配置。
- 按重量计算快递费。
- 最终 `resp.Amount = respAmount * 100` 返回。

## 包材计算改造建议

计算部分推荐数据流：

```text
CalExpressFee
-> C++ GetTotalAmont 算药品重量 drug_weight
-> C++ 整理包材匹配参数
-> 调 Go 包材配置/匹配接口
-> Go 返回包材规则、包材重量、包材费
-> charge_weight = drug_weight + total_package_weight
-> GoCalExpressFee(... charge_weight ...)
-> 返回 express_fee + package_fee
```

最小 C++ 改动点：

1. 新增成员字段保存包材费，例如 `m_dwPackageFee`。
2. `RenderCalExpressFee` 增加返回字段 `package_fee`。
3. Go 分支中，在 `GetTotalAmont` 后、`GoCalExpressFee` 前，调用包材 Go 接口。
4. 用药品重量 + 包材重量后的计费重量调用现有 `GoCalExpressFee`。

建议不要把包材逻辑塞进 `GoCalExpressFee`，因为 `GoCalExpressFee` 现在职责很清晰：只调用运费接口。包材规则匹配应该是另一个函数，例如：

```cpp
uint32_t CRecipeCache::GoQueryPackageFeeConfig(...);
```

## C++ 需要给 Go 包材配置接口的值

按文档，匹配维度是：

```text
区域 + 类型 + 对应剂数
```

C++ 可以给 Go：

```json
{
  "clinic_id": 123,
  "province": "广东省",
  "city": "广州市",
  "district": "天河区",
  "decoct_type": 1,
  "dose_count": 16
}
```

字段映射：

```text
clinic_id    -> expressInfo.dwClinicId
province     -> expressInfo.strProvince
city         -> expressInfo.strCity
district     -> expressInfo.strDistrict
decoct_type  -> recipe_list[register_id].is_decoct，建议约定 1=代煎，2=自煎
dose_count   -> CRecipeInfo::GetAmount()
```

如果存在多处方合并寄付，最好让 Go 接口支持 recipes 列表：

```json
{
  "clinic_id": 123,
  "province": "广东省",
  "city": "广州市",
  "district": "天河区",
  "recipes": [
    {
      "register_id": 111,
      "decoct_type": 1,
      "dose_count": 16
    }
  ]
}
```

这样 Go 可以按处方独立匹配包材规则并汇总费用/重量。

## Go 包材接口建议返回

建议 Go 直接返回已经计算好的总值，避免 C++ 和 Go 重复实现规则：

```json
{
  "rule_id": 10,
  "sf_material_code": "430300308905",
  "package_name": "F3纸箱",
  "unit_price": 2000,
  "package_count": 2,
  "single_package_weight": 500,
  "total_package_weight": 1000,
  "package_fee": 4000
}
```

C++ 使用：

```text
charge_weight = drug_weight + total_package_weight
package_fee = package_fee
```

如果 Go 只返回单个包材重量和单价，那么 C++ 需要计算：

```text
package_count = dose_count > 15 ? 2 : 1
total_package_weight = package_weight * package_count
package_fee = unit_price * package_count
```

但更推荐 Go 返回最终结果，规则归属更清晰。

## 15 剂以上规则

文档写法：

```text
15 剂以上拆 2 个包装，单项收费，即包材费 = 单价 * 2
```

当前理解：

```text
15 剂以上 -> 使用两个包材 -> 收两份包材费
```

需要确认：

1. “15 剂以上”到底是 `> 15` 还是 `>= 15`。
2. 包材重量是否也乘以 2。

按业务直觉，如果真实使用两个包材，则计费重量也应：

```text
drug_weight + package_weight * 2
```

## 计算接口返回变化

当前返回：

```json
{
  "status": 0,
  "message": "",
  "express_fee": 120000
}
```

新需求最小返回：

```json
{
  "status": 0,
  "message": "",
  "express_fee": 120000,
  "package_fee": 4000
}
```

如果后续下单要落快照，建议同时返回：

```json
{
  "express_fee": 120000,
  "package_fee": 4000,
  "drug_weight": 1300,
  "charge_weight": 2300,
  "package_count": 2,
  "package_rule_id": 10,
  "sf_material_code": "430300308905"
}
```

下单收费、入账科目、快照落库和退款规则不是当前计算部分的重点，后续单独看。

## 当前代码风险顺手记录

这些不是包材需求本身，但后续改这块时建议一起修：

1. `GoCalExpressFee` 拼 URL 时没有 URL encode，中文省市可能有风险。
2. `province/city` 在 C++ 中不是必填，但 Go 分支实际依赖它们。
3. `amount` 解析使用 `jsonData["amount"].asCString()`，如果 Go 返回 JSON number，建议改成数值读取。
4. JSON parse 失败时当前没有明确返回错误。

