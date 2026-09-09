## panelagent

> 流式细胞多色 panel 设计工具。LLM + 确定性算法混合架构，共享 SQLite 内核，提供 CLI + Skills、MCP、WebUI 三种入口。

# AGENTS.md — PanelAgent

流式细胞多色 panel 设计工具。LLM + 确定性算法混合架构，共享 SQLite 内核，提供 CLI + Skills、MCP、WebUI 三种入口。

## 必读（按顺序）

1. `docs/panelagent-core.md` — 内核包公开 API、schema、CLI/MCP 用法（权威文档）
2. 对应任务书 `docs/refactor/` — 本次任务范围与验收要求

项目约束直接维护在本文件和内核文档中，不再依赖项目专用 skill。

## 快速事实

- **架构**：`panelagent/`（SQLite 内核包：CLI `pa` + MCP server + Python API）← FastAPI backend← Next.js 前端
- **实验工作台**：`/exp-design` → Next 服务端 `/api/workbench/*` → stdio MCP → SQLite；可选模型负责工具编排。其他页面经 FastAPI 读写同一 SQLite 库，接入配置见 `docs/web-workbench.md`。
- **DB**：SQLite WAL，路径 `--db` > `PANELAGENT_DB` > `~/.local/share/panelagent/panelagent.db`；一个 DB = 一个实验室（多抗体库 + 多仪器）
- **切面分离**：backend 可 import panelagent 内核模块，禁止 import panelagent.cli/mcp；反向亦然
- **验证基线**：`PYTHONPATH=. python3 -m pytest tests/ -q`（在项目虚拟环境运行；当前 314 passed）+ `ruff check`
- **git 政策**：不主动 commit，等用户点头

## 当前状态（2026-09-07）

- v0.1–v0.1.4 内核已提交：`panelagent/` + `tests/core/`
- v0.2.0：CLI/MCP 契约对齐、用户 Skill、发行构建、Web/后端 SQLite 整合及 CLI 安装包指南已实现。
- Phase 2 任务书：`docs/refactor/phase2-backend.md`；迁移说明 `docs/sqlite-migration.md`。
- 历史任务书：`docs/refactor/`

## 常用命令

```bash
make test-backend          # pytest
make lint-backend          # ruff
make typecheck-frontend    # tsc
make generate-client && make check-drift   # OpenAPI 客户端同步检查
pa init --config-dir config --inventory-dir antibody_vault
pa panel generate --library Mouse --markers CD3,CD4,CD8 --json
```

## 注意

- frontend 用 Next.js 16（较新），写代码前查 `frontend/node_modules/next/dist/docs/`
- 抗体 target / fluorochrome 可以为 NULL，搜索和计算必须正确处理。
- 亮度只采用有来源的数据；缺失时保持未知，不猜测补值。
- `config/` 是静态数据编辑源；修改后同步 `panelagent/data/seed/`，运行时使用 SQLite。
- MCP 使用声明支持的官方 SDK 版本；工具只读或纯计算，协议输出独占 stdout。

---
> Source: [PanCodeInventory/PanelAgent](https://github.com/PanCodeInventory/PanelAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
