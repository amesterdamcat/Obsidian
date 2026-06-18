---
title: go-service
tags:
  - project
  - go
  - domestic
  - claude-md
aliases:
  - 国内Go服务
---

# go-service — 国内 Go 微服务

> [!info] 所属平台
> [[平台架构总览|医疗/药房 SaaS 平台]] · 国内版 · Go 端
> 配对 C++ 仓库：[[cpp-service]]
> 海外克隆体：[[ogo-service]]

## 基本信息

| 项 | 值 |
|----|-----|
| 本地路径 | `C:\Users\PC\Documents\go-code\code\go-service` |
| 测试服务器 | `ssh -p 36000 guanbin@172.30.12.14` |
| 日志 srv | `/data/srv/` |
| 日志 api | `/data/api/` |
| 框架 | go-micro v1.18 |
| ORM | gorm v1（jinzhu/gorm） |
| 服务数量 | ~120+ 微服务 |

## 架构

```
<name>-service/
├── api/          # HTTP 网关（go-micro API 服务）
│   ├── handler/  # HTTP handler，通过 gRPC 调用 srv
│   └── main.go
└── srv/          # gRPC 后端（业务逻辑 + 数据库）
    ├── handler/  # *_handler.go 业务逻辑，*_dbhandler.go 数据库
    └── models/
        ├── model/   # gorm 模型
        ├── api/     # 请求/响应视图
        ├── biz/     # 业务逻辑模型
        └── rdao/    # Redis DAO
```

### 请求调用链

```
HTTP → API handler → gRPC → SRV handler → MySQL (gorm) / Redis / 跨服务 gRPC
```

被 [[cpp-service]] 的 CGI/AO 通过 HTTP 调用。

## 核心技术栈

- go-micro v1.18（服务注册/发现、RPC）
- gorm v1（MySQL ORM）
- go-redis v8
- protobuf + micro 插件
- Consul（配置管理）

## CLAUDE.md 内容概要

项目级 `CLAUDE.md` 包含：
- 构建部署方式（make build/up/k8s）
- Proto 编译流程
- 服务目录结构（api/srv 双层）
- 公共包 comm/ 子包说明
- Consul 配置管理
- 服务间通信（gRPC、Pub/Sub、Cron）
- 代码生成工具 autogen

全局上下文见 [[平台架构总览]]（`~/.claude/CLAUDE.md` 自动加载）。
