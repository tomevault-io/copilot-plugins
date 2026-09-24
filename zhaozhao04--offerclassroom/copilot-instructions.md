## offerclassroom

> OfferClassroom — 一站式 AI 求职辅助平台

# CLAUDE.md

## 项目名称

OfferClassroom — 一站式 AI 求职辅助平台

## 项目概述

帮助求职者完成：上传简历 → 针对目标岗位 AI 优化 → AI 模拟面试（10/20/40 分钟）→ 五维评分报告 → 历史记录回看与两次面试雷达图对比。数据按用户隔离，AI 输出以流式（SSE）逐字呈现。

需求见 docs/PRD.md，技术规格与开发阶段见 docs/SPEC.md —— 动任何功能前先读这两个文档，特别是 SPEC 第 8 节的阶段 TODO。

## 核心技术栈

- 后端：Python 3.12 + FastAPI + SQLAlchemy 2.x 异步 ORM + aiomysql（MySQL 8）
- AI：LangChain init_chat_model（LLM）+ Chroma（知识库切片向量检索）
- 前端：Vue 3 + Element Plus + Vite（frontend/ 目录）
- 测试：Pytest（全项目唯一测试框架）
- 包管理：uv

## 项目结构

```text
OfferClassroom/
├─ docs/                     # PRD.md、SPEC.md
├─ .env                      # 环境变量（不提交 git）
├─ pyproject.toml
├─ src/offerclassroom/
│  ├─ main.py                # FastAPI 入口，启动时自动建表
│  ├─ config.py              # 读 .env 的配置（统一从这里读，不要到处 os.getenv）
│  ├─ db.py                  # SQLAlchemy 异步引擎与会话（get_db 依赖注入）
│  ├─ models/                # ORM 模型类（base.py + 按资源分文件，create_all 自动建表）
│  ├─ schemas/               # Pydantic 请求/响应数据类
│  ├─ crud/                  # 数据库增删改查封装（按 id 查询强制带 user_id）
│  ├─ routers/               # 接口层：参数校验、鉴权、拼响应（阶段 1-6 逐步创建）
│  └─ services/              # 服务层：业务逻辑，一个职责一个文件（阶段 1-6 逐步创建）
├─ tests/                    # Pytest 测试
├─ frontend/                 # Vue 3 前端（阶段 1 创建）
└─ data/                     # 运行时生成：上传文件、Chroma 向量（不提交 git）
```

## 约定规范

- 分层：routers/ 只做参数校验、鉴权、拼响应；数据库增删改查统一走 crud/；业务逻辑全部放 services/，一个职责一个文件
- 数据访问统一走 SQLAlchemy ORM（模型查询 + `Depends(get_db)` 会话依赖注入），不手写 SQL；确需原生 SQL 的场景（如建库）必须先过白名单校验再拼接（见 db.py 的 create_database）
- 所有按 id 操作的查询必须带 user_id 条件 —— 数据按用户隔离是硬性要求
- LLM 统一走 services/agent.py 封装（模型单例 + agent 构建 / 事件流），别处不要自己 new 模型；提示词集中在各 service 文件顶部常量
- 英文命名（snake_case / PascalCase），中文注释只解释”为什么”，不写废话；同一资源的 models/crud/schemas/routers 文件名保持一致且用复数（users.py、resumes.py、knowledge_files.py），auth.py 等无资源概念的除外
- 后续新增资源统一照此结构：models/xxx.py（ORM）+ schemas/xxx.py（Pydantic）+ crud/xxx.py（增删改查）+ routers/xxx.py（子路由）
- 前端：不引入 vue-router / Pinia / ECharts；页面切换用 store.js 状态 + v-if；一个页面一个文件放 views/
- 测试：仅 Pytest；模型调用通过依赖注入 mock，核心逻辑写成纯函数

## 常用命令

```powershell
uv sync                          # 装依赖
uv run uvicorn offerclassroom.main:app --reload   # 后端（启动时自动建表）
cd frontend; npm install; npm run dev             # 前端
uv run pytest                    # 测试
```

## 注意事项

- .env、data/、node_modules、.venv、frontend/dist 不提交 git；密钥只放 .env
- 运行需要本机 MySQL 8（或用 Docker 只跑 MySQL）；.env 里“待填”字段填好才能跑通 AI 功能
- 上传文件白名单 .pdf / .docx / .txt / .md，单文件 ≤ 10MB；磁盘用 UUID 文件名防路径穿越
- 面试状态机：running / finished；报告生成失败（重试一次仍失败）保持 running 可重试（见 SPEC 5.3）
- 面试官是挂载知识库检索工具的 agent（llm.build_interview_agent + interview_tools.build_search_tool）：模型主动决定何时检索（规划考点/验证细节/设计追问），检索词由模型写、user_id 与勾选文件由后端固定注入；检索期间发 tool_status SSE 事件（前端显示“查阅资料”）。人格 skill 走 system prompt 首轮在场，不走 tool 自选加载（见 SPEC 关键数据流 3）
- 修改 .env、删除文件、git push 等操作先征求用户同意
- git commit的描述请使用中文，合作者不加CLAUDE

---
> Source: [zhaozhao04/OfferClassroom](https://github.com/zhaozhao04/OfferClassroom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
