---
title: ogo-service
tags:
  - project
  - go
  - overseas
  - claude-md
aliases:
  - 海外Go服务
---

# ogo-service — 海外 Go 微服务

> [!info] 所属平台
> [[平台架构总览|医疗/药房 SaaS 平台]] · 海外版 · Go 端
> 配对 C++ 仓库：[[ocpp-service]]
> 国内原版：[[go-service]]

## 基本信息

| 项 | 值 |
|----|-----|
| 本地路径 | `C:\Users\PC\Documents\go-code\code\ogo-service` |
| 测试服务器 | `ssh -p 36000 guanbin@172.30.12.166` |
| 日志 srv | `/data/srv/` |
| 日志 api | `/data/api/` |
| 框架 | go-micro v1.18 |
| ORM | gorm v1 |
| 服务数量 | ~120+ 微服务 |

## 与国内版的关系

- 架构、服务结构、构建方式与 [[go-service]] 完全一致
- 是 [[go-service]] 的克隆体，业务不同
- 海外版额外支持：多语言 i18n（`comm/errors`、`comm/multilingual`）

## 架构

与 [[go-service]] 相同：

```
<name>-service/
├── api/    → HTTP 网关
└── srv/    → gRPC 后端 + 数据库
```

被 [[ocpp-service]] 的 CGI/AO 通过 HTTP 调用。

## CLAUDE.md 内容概要

项目级 `CLAUDE.md` 与 [[go-service]] 结构一致，额外说明：
- 海外版身份标识
- 关联 C++ 仓库指向 [[ocpp-service]]
- i18n 多语言支持细节

全局上下文见 [[平台架构总览]]（`~/.claude/CLAUDE.md` 自动加载）。
