## deep-research-agent

> 本项目是基于 OpenAI Agents SDK 的 Deep Research / Deer Research 风格应用。所有 Codex 相关修改请优先遵守本文件。

# Agent Development Guide

本项目是基于 OpenAI Agents SDK 的 Deep Research / Deer Research 风格应用。所有 Codex 相关修改请优先遵守本文件。

## 目标

- 保持 agent 框架高扩展、低耦合、容易测试。
- 后端代码以清晰分层为主，不把 prompt、编排、API、工具、数据模型混在一起。
- 新功能先贴近现有结构扩展，避免为了短期实现引入全局状态、隐式依赖或大型重构。

## 后端目录边界

目录布局对齐 `agent-framework`（DeerFlow）的 `src/` 分包方式：

```text
src/
  agents/     OpenAI Agents SDK 注册与 Agent 实例
  config/     配置、环境变量
  models/     Pydantic/dataclass 研究模型，不依赖外部服务
  prompts/    Agent system prompt 文案
  server/     FastAPI、HTTP schema、SSE 边界
  tools/      agent 可调用工具与外部 API 封装
  workflow/   研究流程编排、状态存储接口
```

`src/manager.py`、`src/schemas.py`、`src/sse.py` 是兼容入口。新代码应直接导入分层包，例如 `src.workflow.research_manager` 或 `src.models`。

## 扩展规则

- 新 agent：在 `prompts/` 增加 instruction，在 `agents/agents.py` 注册 agent，并显式声明 `output_type`。
- 新工具：放在 `tools/` 下，外部 API 访问和 agent tool wrapper 分开写；工具返回结构化模型或明确的 dict。
- 新流程：放在 `workflow/` 下，优先依赖 `models/` 和抽象接口，不直接依赖 FastAPI request/response。
- 新状态存储：实现 `workflow/run_store.py` 的 `ResearchRunStore`，不要把数据库逻辑写进 `ResearchManager`。
- 新 API：放在 `server/` 下，API 层只做校验、转换和错误边界，不写研究循环逻辑。
- **每次修改代码后必须同步更新 `docs/` 中对应的文档**，确保文档与代码保持一致。

## 代码风格

- Python 使用 3.12 类型语法，保持 `from __future__ import annotations`。
- 单个模块保持一个清晰职责；超过 250-300 行时优先拆分。
- Prompt 文案集中在 `prompts/`，不要散落在 workflow 层。
- 不在业务代码里硬编码 secret、模型名、URL；新增配置放入 `config/configuration.py` 并通过环境变量读取。
- 保持函数返回值显式、可测试；少用隐式全局状态。
- 修改后至少运行：

```bash
uv run python -m compileall src
```

有测试后优先运行对应测试，再考虑全量测试。

## 研究产品原则

- 默认上下文不足，研究型回答要基于可引用证据。
- 不伪造 URL、引用、数据、案例或搜索结果。
- 报告写作必须清楚区分事实、证据、分析和限制。
- 对时间敏感信息必须联网或使用外部搜索工具确认。

## 前端约定

- 前端在 `web/`，保持 Next.js app router 结构。
- API 调用集中在 `web/src/lib/`，页面组件不要散写 fetch 细节。
- UI 优先服务研究工作流：输入、计划、检索过程、证据、报告输出要清楚可追踪。

---
> Source: [futurehafuture/deep-research-agent](https://github.com/futurehafuture/deep-research-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
