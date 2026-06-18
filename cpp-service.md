---
title: cpp-service
tags:
  - project
  - cpp
  - domestic
  - claude-md
aliases:
  - 国内C++服务
---

# cpp-service — 国内 C++ 服务

> [!info] 所属平台
> [[平台架构总览|医疗/药房 SaaS 平台]] · 国内版 · C++ 端
> 配对 Go 仓库：[[go-service]]
> 海外克隆体：[[ocpp-service]]

## 基本信息

| 项 | 值 |
|----|-----|
| 本地路径 | `C:\Users\PC\Documents\cpp-code\code\cpp-service` |
| 测试服务器 | `ssh -p 36000 guanbin@172.30.12.124` |
| 日志 CGI | `/data/c2c_logs/*_debug*.log` |
| 日志 DAO/AO | `/data/applog/*_debug*.log` |
| 构建方式 | Makefile（Linux 服务器上执行） |
| C++ 标准 | C++03 兼容，不假设 C++11 |

## 架构

```
*_proj/
├── idl/          # 接口定义（Java 注解 → 代码生成 stub/ddo）
├── dao/          # 数据访问层（.so 共享库）
├── ao/           # 应用逻辑层（.so，编排多个 DAO 调用）
├── web/          # CGI 入口层（HTTP 请求 → 业务调用）
├── comm/         # 模块内公共代码
├── daemon/       # 后台守护进程
└── script/       # 部署/数据脚本
```

### 请求调用链

```
HTTP → CGI (web/) → DAO stub → DAO (.so) → MySQL
                  → AO stub  → AO (.so)  → 多个 DAO stub
```

CGI 通过 HTTP 调用 [[go-service]] 的 API 接口。

## 业务模块

| 模块 | 职责 |
|------|------|
| deal_proj | 订单（挂号、诊断、处方关联） |
| recipe_proj | 处方（开方、缓存、拆分） |
| pharmacy_proj | 药房（库存、锁库、发药） |
| schedule_proj | 排班（医生排班、同步） |
| workstation_proj | 工作台（模板、医嘱本、打印） |
| inspect_proj | 检验检查 |
| external_proj | 外部对接 |
| pay_center_proj | 支付中心 |

## CLAUDE.md 内容概要

项目级 `CLAUDE.md` 包含：
- Makefile 构建方式、构建输出路径
- 分层架构（IDL → DAO → AO → Web）详细说明
- IDL 代码生成流程
- 跨模块 stub 调用规范
- PM 协作规则、日志规范（析构日志优先）
- C++03 兼容约束

全局上下文见 [[平台架构总览]]（`~/.claude/CLAUDE.md` 自动加载）。
