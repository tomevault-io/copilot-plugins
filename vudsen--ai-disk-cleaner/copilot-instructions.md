## ai-disk-cleaner

> 该项目是一个 wails 项目，前端代码在 frontend，后端代码在 backend。

# AGENTS.md

该项目是一个 wails 项目，前端代码在 frontend，后端代码在 backend。

本项目的目标是，完成一个 AI 集成的磁盘清理器。

架构：

1. 使用 gdu 分析磁盘占用， 加载到内存中
2. 应用暴露工具给 LLM 调用，渐进式披露文件占用，让 AI 找出可以删除的文件
3. 列出可以删除的文件，交给用户决定是否删除

## 后端代码结构

- 禁止在根目录创建新的 `go` 文件，`app.go` 只能写暴露给前端的函数，其它业务逻辑全部写到 backend 中。
- `app.go` 中暴露给前端的函数，必须通过调用 `backend/service` 中的某个函数来实现，禁止写长段代码
- 对于一个新服务，必须创建对应的文件夹存放，禁止在 `backend/service` 目录中直接创建 go 文件编写，并且命名强制为 `{服务名称}/{服务名称}.go`，
  `{服务名称}.go` 中，专门写对外暴露的方法，其它地方不要暴露，至于其它内部逻辑，在同一目录另外创建文件进行编写。
- 对于服务中可能出现循环引用的接口，可以放到 `backend/service/types.go`

## 其它 AGENTS.md 索引

当你修改对应目录内部的文件时，请务必阅读相关 AGENTS.md

- frontend/AGENTS.md
- backend/data/models/AGENTS.md
- backend/service/AGENTS.md

---
> Source: [vudsen/ai-disk-cleaner](https://github.com/vudsen/ai-disk-cleaner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
