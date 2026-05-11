## ainovel

> ﻿# AInovel 协作约定（AGENTS.md）

﻿# AInovel 协作约定（AGENTS.md）
编码一定要使用UTF-8!!!!!

> 永远用中文回答问题；本文件适用于仓库根目录及其所有子目录。
> 重要说明：当前仓库未集成 LangGraph；文档中涉及 LangGraph 的内容为后续规划，不影响现有 FastAPI 实现与测试。

## 概览
- 项目：AI 自动小说创作平台（specs/001-ai-novel-creation-platform）。
- 目标：用户给出灵感→生成大纲→逐章创作；维护“世界线/人物线/感情线”等全局线数据库，允许用户编辑并驱动创作调整。

## 技术栈与环境
- 后端：Python 3.11，FastAPI，SQLite 持久化（可选 Gemini 2.5 Flash，需要 `GEMINI_API_KEY`）。
- LangGraph：可选（当前未集成，后续再评估）。
- 前端：React 18 + Vite，TypeScript 5+，Node.js 18+。
- 测试：pytest；Jest + React Testing Library；（如需）LangSmith 追踪。
- 文档编码：统一使用 UTF-8。

## 推荐目录结构（Option 2：Web 应用）
```
backend/
  src/{models,services,api,...}
  tests/{contract,integration,unit,performance}
frontend/
  src/{components,pages,services,store,...}
  tests/
```

## 数据模型要点（详见 `specs/.../data-model.md`）
- 核心实体：`Inspiration`、`Outline(+ChapterOutline[])`、`WorldEvent`、`Character(+Relationship, +Development)`、`Relationship`、`Chapter(+ChapterReference[])`。
- 状态 `NovelCreationState`（内部实现，兼容未来图式编排）：`current_chapter`、`total_chapters`、`status`；追加型 reducer 管理 `worldline/characterline/relationshipline`；`outline/chapters`；`user_feedback`；`cached_context`、`api_call_count` 等。

## API 合约（详见 `specs/.../contracts/api-spec.yaml`）
- POST `/api/inspiration` 创建灵感
- POST `/api/outline` 生成大纲
- POST `/api/novel/{novel_id}/chapter` 生成章节
- GET/PUT `/api/novel/{novel_id}/global-lines` 全局线读写

## 工程化约束（Constitution 摘要）
- 简单优先：仅前端与后端两个项目；直连 SQLite；避免过度模式；统一数据模型（DTO 仅作 API 序列化）。
- 架构：创作引擎可抽为独立库；文档采用标准注释/TypeDoc。
- 可观测性：结构化日志；错误上下文充分；前后端日志可汇聚。
- 版本：语义化版本从 1.0.0 起；每次变更递增 build。

## 任务与顺序（摘自 `specs/.../tasks.md`）
- Phase 3.1 初始化：目录、依赖、代码质量（pre-commit/ESLint/Prettier）。
- Phase 3.2 测试先行（必须先写且失败）：
  - 合约测试：T006–T009（四个端点）。
  - 集成测试：T010–T013（用户故事）。
- Phase 3.3 核心实现：
  - 数据模型 T014–T019；状态与 reducer T020–T021（内部实现；LangGraph 集成延后）；
  - 服务层 T022–T026；API T027–T030。
- Phase 3.4 前端：组件 T031–T035；状态管理与 API 客户端 T036–T038。
- Phase 3.5 集成与优化：DB/检查点/监控；批处理/缓存/条件执行/Token 统计；前端路由/主题/错误边界/加载态。
- Phase 3.6 完善与发布：单元与性能测试；文档与部署（docker-compose/Dockerfile）；依 quickstart 验证。
- 依赖约束：测试先于实现；模型→服务→API→UI；前端可在 API 之后并行。
- 并行规则：用 `[P]` 标注可并行，但不得有多个 `[P]` 任务修改同一文件。

## TDD 与测试准则
- 顺序：Contract → Integration → Unit；严格 RED→GREEN→REFACTOR；实现绝不先于测试。
- 覆盖面：四个端点合约测试；灵感→大纲、章节创作、全局线、用户修改影响创作的集成测试；组件/Hook 单测；性能基准测试。

## 性能目标（来自 plan/quickstart）
- 大纲生成 < 60 秒；章节生成 < 30 秒；全局线更新 < 5 秒；支持 100+ 章节长篇。

## 运行与验证（摘自 `quickstart.md`）
- 后端：
  1) `python -m venv venv` 并激活；2) `pip install -r backend/requirements.txt`；
  3) 配置 `.env` 中 `GEMINI_API_KEY`；4) `uvicorn src.main:app --reload --port 8000`。
- 前端：`cd frontend && npm install && npm run dev`；访问 `http://localhost:3000`。
- 基础 API 验证：参考 quickstart 中 `curl` 示例。

## 代码质量与提交
- 提交历史应体现“先测试、后实现”；小步提交，任务完成即提交。
- 统一格式化；保持前后端分离；避免引入非必要新依赖。

## 验证清单（完成判定）
- 合约均有对应测试；核心实体全部实现；并行任务独立；任务均落到具体文件路径；`quickstart` 全部通过。

## 注意事项
- 优先减少 Gemini API 调用：缓存、批处理、条件执行；统计 token 用量。
- 文档与代码一一对应，新文档放在 `specs/001-ai-novel-creation-platform/` 下。

—— 更新日期：2025-09-13

---
> Source: [a1594834522-coder/AINovel](https://github.com/a1594834522-coder/AINovel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
