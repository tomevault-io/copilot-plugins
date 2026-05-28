## gc-ems-dngl-fastapi

> - 本项目基于 [需求文档.md](D:/研究生/课题组/小龙虾/OpenClaw/GC-EMS-DNGL-FastAPI/需求文档.md) 和 [开发文档.md](D:/研究生/课题组/小龙虾/OpenClaw/GC-EMS-DNGL-FastAPI/开发文档.md) 重新实现 Python/FastAPI 电能管理系统。

# GC-EMS-DNGL-FastAPI AGENTS

本文件定义当前项目范围内的补充规则。
未特别说明时，继承更外层全局默认规则。

## Project Context

- 本项目基于 [需求文档.md](D:/研究生/课题组/小龙虾/OpenClaw/GC-EMS-DNGL-FastAPI/需求文档.md) 和 [开发文档.md](D:/研究生/课题组/小龙虾/OpenClaw/GC-EMS-DNGL-FastAPI/开发文档.md) 重新实现 Python/FastAPI 电能管理系统。
- 首期目标是独立完成电能数据展示、管理、统计和基础 CRUD，不追求完整复刻原 Java/Vue 工业 EMS。
- 原 `GC-EMS-DNGL` Java/Vue 目录仅作为业务参考，不作为运行时代码复用来源。
- 生产目标数据库为 MySQL 8.0+，本地开发默认使用 SQLite。
- 依赖安装、运行和测试默认通过 `uv` 执行。

## Preferred Skills

优先使用项目级 skills：

- `gc-ems-fastapi-dev`：FastAPI 开发、接口、服务、脚手架、联调
- `gc-ems-data-modeling`：SQLAlchemy 模型、MySQL/SQLite 兼容、统计查询
- `gc-ems-ui-builder`：静态管理端页面、看板、浏览器端 API 对接
- `gc-ems-testing`：pytest、接口测试、兼容性验证、UI 冒烟
- `gc-ems-ops`：`uv`、环境变量、数据库切换、启动和部署说明

仅加载完成当前任务所需的最小范围内容；不要一次性展开所有 skill 说明。

## Preferred MCP Usage

- 涉及 OpenAI/Codex 官方能力、skills、subagents、API 文档时，优先使用官方 OpenAI 文档来源。
- 涉及 FastAPI、SQLAlchemy、Pydantic、`uv`、MySQL 等第三方技术，且本地文档不足时，优先查官方文档。
- 若已有本地代码、项目文档或 skill 足以可靠回答，优先使用本地上下文，不为低风险问题额外联网。

## Preferred Subagents

本项目允许自主启动 subagent，无需逐次确认，但必须优先复用 [`.codex/agents`](D:/研究生/课题组/小龙虾/OpenClaw/GC-EMS-DNGL-FastAPI/.codex/agents) 下已有项目级配置。
仅当现有项目级 subagent 的职责均不匹配时，才允许创建临时会话级 subagent；若创建，必须先说明原因。

推荐分工：

- `ems_backend_worker`：后端 API、认证、服务层、接口测试
- `ems_database_designer`：数据模型、索引、统计 SQL、兼容性设计
- `ems_ui_worker`：静态管理端页面、图表、前端 API 对接
- `ems_test_engineer`：pytest、回归验证、失败诊断、验收检查
- `ems_reviewer`：需求映射、代码审查、测试缺口
- `ems_security_reviewer`：认证授权、JWT、密码、CORS、数据库访问安全
- `ems_ops_engineer`：`uv`、环境变量、启动脚本、部署和运行检查
- `ems_docs_researcher`：官方文档调研、技术决策背景

## Implementation Rules

- 默认先实现 P0：认证、设备、点位、模拟能耗数据、首页看板。
- P0 开发必须围绕数据展示、标准 CRUD API、管理端列表和看板展开，并保留后续对接外部平台的扩展能力。
- 运维、光伏、复杂权限、消息通知等模块属于后续扩展，不能阻塞首期可运行版本。
- API 统一响应格式：`{"code": 0, "message": "success", "data": ...}`。
- 数据表和字段命名使用小写下划线。
- 统计接口必须带时间范围、分页或聚合边界，避免无边界全表扫描。
- 新增代码必须能在 SQLite 开发环境启动，并保留 MySQL 8.0+ 兼容性。
- 优先复用现有 `app`、`tests`、`scripts` 下结构，不随意引入与当前阶段无关的复杂抽象。

## Verification

完成改动后，优先执行与改动范围匹配的检查；至少考虑以下命令：

- `uv run pytest`

若任务涉及启动、接口联调或页面改动，再补充执行对应的本地运行或冒烟验证。

---
> Source: [mikewang68/GC-EMS-DNGL-FastAPI](https://github.com/mikewang68/GC-EMS-DNGL-FastAPI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
