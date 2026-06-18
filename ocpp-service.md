---
title: ocpp-service
tags:
  - project
  - cpp
  - overseas
  - claude-md
aliases:
  - 海外C++服务
---

# ocpp-service — 海外 C++ 服务

> [!info] 所属平台
> [[平台架构总览|医疗/药房 SaaS 平台]] · 海外版 · C++ 端
> 配对 Go 仓库：[[ogo-service]]
> 国内原版：[[cpp-service]]

## 基本信息

| 项 | 值 |
|----|-----|
| 本地路径 | `C:\Users\PC\Documents\cpp-code\code\ocpp-service` |
| 测试服务器 | `ssh -p 36000 guanbin@172.30.12.165` |
| 日志 CGI | `/data/c2c_logs/*_debug*.log` |
| 日志 DAO/AO | `/data/applog/*_debug*.log` |
| 构建方式 | Makefile（Linux 服务器上执行） |
| C++ 标准 | C++03 兼容 |

## 与国内版的关系

- 架构、构建方式、分层结构、IDL 代码生成流程与 [[cpp-service]] 完全一致
- 是 [[cpp-service]] 的克隆体，业务不同
- 代码可参考但不能直接混用

## 架构

与 [[cpp-service]] 相同：

```
*_proj/
├── idl/    → dao/    → ao/    → web/
├── comm/   → daemon/ → script/
```

CGI 通过 HTTP 调用 [[ogo-service]] 的 API 接口。

## CLAUDE.md 内容概要

项目级 `CLAUDE.md` 与 [[cpp-service]] 结构一致，额外说明：
- 海外版身份标识
- 关联 Go 仓库指向 [[ogo-service]]

全局上下文见 [[平台架构总览]]（`~/.claude/CLAUDE.md` 自动加载）。
