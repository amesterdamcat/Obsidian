# 接口文档：外送检验项目明细列表

## 基本信息

| 项目 | 说明 |
|---|---|
| 接口名 | QueryOutsendItemList |
| 请求方式 | GET |
| 说明 | 查询指定诊所在时间范围内的所有外送检验项目明细，按开单时间正序排列，支持分页 |

---

## 请求参数

| 参数名 | 类型 | 是否必填 | 说明 |
|---|---|---|---|
| `clinic_id` | uint32 | **必填** | 诊所 ID |
| `start_time` | int64 | 选填 | 开单时间范围起始，Unix 秒。不传默认取当月1号 00:00:00 |
| `end_time` | int64 | 选填 | 开单时间范围截止，Unix 秒。不传默认取今日 23:59:59 |
| `item_name` | string | 选填 | 项目名称，模糊匹配 |
| `item_code` | string | 选填 | 项目编码，精确匹配 |
| `user_name` | string | 选填 | 患者姓名，精确匹配 |
| `page_no` | int32 | 选填 | 页码，从 1 开始，不传默认 1 |
| `page_size` | int32 | 选填 | 每页条数，不传默认 20 |

### 注意事项

- `start_time` 和 `end_time` 同时不传时，后端自动取当月范围，**前端应避免清空时间条件**
- `start_time` 不能大于 `end_time`，否则返回参数错误
- `item_name` 支持输入部分名称模糊搜索，如输入"血常"可匹配"血常规"
- `item_code` 需输入完整编码才能匹配

### 请求示例

```
GET /inspect/outsend/item/list?clinic_id=311&start_time=1777564800&end_time=1778860799&page_no=1&page_size=20
```

---

## 响应参数

### 外层结构

| 参数名 | 类型 | 说明 |
|---|---|---|
| `code` | int | 状态码，200 为成功 |
| `cur_page` | int32 | 当前页码 |
| `total_page` | int32 | 总页数 |
| `total_count` | int32 | 总条数 |
| `data` | array | 项目明细列表，见下表 |

### `data` 数组元素字段

| 参数名 | 类型 | 说明 |
|---|---|---|
| `create_time` | string（int64） | 开单时间，Unix 秒 |
| `bar_code` | string | 条码号 |
| `user_name` | string | 患者姓名 |
| `item_name` | string | 检验项目名称 |
| `item_code` | string | 检验项目编码 |
| `sell_price` | string（int64） | 项目价格，单位为**分**，前端展示时除以 100 |
| `doctor_name` | string | 开单医生姓名 |
| `department` | string | 开单科室 |
| `document_id` | string（int64） | 检验申请单 ID |
| `deal_id` | string（int64） | 订单 ID |

### 响应示例

```json
{
  "code": 200,
  "cur_page": 1,
  "total_page": 1,
  "total_count": 4,
  "data": [
    {
      "create_time": "1778828128",
      "bar_code": "1100000218020",
      "user_name": "周小北",
      "item_name": "真菌药敏试验",
      "item_code": "0602012000162",
      "sell_price": "700000",
      "doctor_name": "docstg专家工作室",
      "department": "小儿推拿科",
      "document_id": "218020",
      "deal_id": "18286029"
    },
    {
      "create_time": "1778828128",
      "bar_code": "1100000218020",
      "user_name": "周小北",
      "item_name": "葡萄糖耐量试验（测5次血糖）",
      "item_code": "0602010000070",
      "sell_price": "500000",
      "doctor_name": "docstg专家工作室",
      "department": "小儿推拿科",
      "document_id": "218020",
      "deal_id": "18286029"
    }
  ]
}
```

---

## 错误响应

| 场景 | code | 说明 |
|---|---|---|
| 缺少 `clinic_id` | 400 | 参数缺失 |
| `start_time` > `end_time` | 500 | 参数错误，时间范围非法 |
| 服务内部错误 | 500 | 联系后端排查 |

---

## 前端使用说明

1. **`sell_price` 展示**：接口返回单位为分，展示金额时需除以 100，如 `700000` → `7000.00 元`
2. **时间字段**：`create_time` 为 Unix 秒时间戳，前端自行格式化为年月日
3. **int64 字段**：`create_time`、`sell_price`、`document_id`、`deal_id` 在 JSON 中以字符串形式返回，避免 JavaScript 大整数精度丢失问题，使用时注意类型转换
4. **同一条码多个项目**：同一张申请单（相同 `bar_code` / `document_id`）可能有多个外送项目，在列表中会分别展示为独立的行
5. **默认时间范围**：页面初始化时可以不传时间参数，后端自动返回当月数据；建议前端 UI 显示默认值为当月1号到今日，与后端行为保持一致