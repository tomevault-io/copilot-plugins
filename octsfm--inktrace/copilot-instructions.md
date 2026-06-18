## inktrace

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

InkTrace 是一个面向长篇小说创作的作者智能体工作系统——"作者与智能体协作写小说的 IDE"。后端 Python/FastAPI，前端 Vue 3 + Vite + Pinia + Element Plus。

核心设计：**Kimi（DeepSeek 系）负责理解/分析/规划/控制，DeepSeek（非 Kimi）负责写作/续写/改写/润色**。Kimi 决定故事往哪走，DeepSeek 把它写出来。

## Commands

### Backend
```bash
pip install -r requirements.txt     # 安装 Python 依赖
python main.py                      # 启动后端服务 (127.0.0.1:9527)
pytest tests/                       # 运行全部后端测试
pytest tests/test_v1_xxx.py -v      # 运行单个测试文件
pytest tests/test_v1_xxx.py::test_name -v  # 运行单个测试用例
pytest -k "keyword" -v              # 按关键词筛选测试
```

### Frontend
```bash
cd frontend && npm install           # 安装前端依赖
cd frontend && npm run dev           # 启动开发服务器 (localhost:3000)
cd frontend && npm run build         # 生产构建
cd frontend && npm test              # 运行全部前端测试 (vitest)
cd frontend && npx vitest run src/xxx.spec.js  # 运行单个测试文件
cd frontend && npx vitest run -t "test name"   # 按名称运行单个测试
```

### Full Stack
```powershell
.\start-all.bat                      # 一键启动前后端
.\stop.bat                           # 停止前后端
.\scripts\build_and_smoke.bat        # 构建 + 冒烟测试
.\scripts\stage5_acceptance.bat      # Stage 5 验收测试
.\scripts\stage6_acceptance.bat      # Stage 6 验收测试
```

### Desktop Build
```bash
npm run build:win      # Windows 桌面打包 (electron-builder)
npm run build:mac      # macOS 桌面打包
```

### Database Migrations
```bash
python migrations/migrate_structured_story_data.py
```

## Architecture

### 分层架构（Clean Architecture / DDD）

后端遵循领域驱动设计，四层职责分明：

```
presentation/    → API 层（FastAPI 路由、中间件、异常处理、依赖注入）
application/     → 应用服务层（编排用例、prompt 构建、workflow 调度）
domain/          → 领域层（实体、值对象、仓储接口、领域服务、常量/枚举）
infrastructure/  → 基础设施层（SQLite 持久化、文件 I/O、ChromaDB 向量存储）
```

数据流向：`presentation → application → domain ← infrastructure`

**V1 接口**（当前主力）位于 `presentation/api/routers/v1/`，对应 `application/services/v1/`。  
**重构版 DDD 接口**（正在演进）位于 `backend/`，包含 `backend/src/domain/`（纯 DDD 领域模型）和 `backend/src/presentation/`。

### 核心文档目录

```
docs/
├── 01_requirements/   # 需求
├── 02_architecture/   # 架构设计
├── 03_design/         # 详细设计
├── 04_plan/           # 开发计划
├── 05_tasks/          # 任务追踪
├── 06_validation/     # 验收记录
└── 07_overview/       # 总览
```

### V1 API 路由

所有 V1 路由注册在 `presentation/api/routers/v1/`，公共前缀 `/api/v1`：
- `works` — 作品 CRUD
- `chapters` — 章节 CRUD、排序、发布
- `sessions` — 编辑会话（乐观锁 409 冲突）
- `io` — 导入/导出（TXT、大纲导入）
- `outlines` — 大纲资产管理
- `timeline` — 时间线事件管理
- `foreshadows` — 伏笔管理
- `characters` — 人物管理
- `health` — 健康检查

### V1 服务层

`application/services/v1/` 包含核心业务逻辑：
- `work_service.py` / `chapter_service.py` / `session_service.py` — 作品-章节-会话主链路
- `io_service.py` — 导入/导出
- `writing_asset_service.py` — 写作资产（人物/伏笔/大纲/时间线）
- `text_metrics.py` — 字数/文本度量
- `content_tree_schema.py` — 内容树结构定义

### Domain 领域层

`domain/entities/` — 核心实体：`Work` (作品)、`Chapter` (章节)、`EditSession` (编辑会话)、`Novel` (小说)、`Outline` (大纲)、`WritingAsset` (写作资产)  
`domain/repositories/` — 仓储接口  
`domain/services/` — 领域服务：`WritingEngine` (写作引擎)、`ConsistencyChecker` (一致性检查)、`PlotAnalyzer` (剧情分析)、`StyleAnalyzer` (风格分析)  
`domain/value_objects/` — 值对象：`CharacterState`、`Embedding`、`StyleProfile`、`WritingConfig`  
`domain/constants/` — 枚举与常量

### Infrastructure 基础设施

`infrastructure/database/` — SQLite 数据库会话与仓储实现  
`infrastructure/persistence/` — 新版 SQLite 持久化（`sqlite_chapter_repo.py` 等，基于 aiosqlite）  
`infrastructure/file/` — 文件导入导出（`txt_parser.py`、`txt_exporter.py`、`markdown_exporter.py`）

### Frontend 架构

```
frontend/src/
├── api/              # Axios HTTP 请求封装
├── components/       # Vue 组件
│   ├── works/        # 作品列表相关（卡片、创建、导入、导出弹窗）
│   └── workspace/    # 写作工作区（编辑器、侧栏、资产面板、时间线等）
├── composables/      # 组合式函数
├── constants/        # 常量与枚举标签
├── layouts/          # 布局组件
├── router/           # Vue Router 路由配置
├── stores/           # Pinia 状态管理
├── utils/            # 工具函数
└── views/            # 页面视图
    ├── novel/        # 小说导入
    └── works/        # 作品列表 & 写作工作室
```

### 关键设计约束

- **乐观锁并发**: Chapter/Session 通过 `version` 字段实现乐观锁，并发更新返回 409 冲突
- **排序原子性**: Timeline/Chapter 排序通过完整映射提交 + 单事务写入保证原子性
- **双模型协同**: 系统固定使用两类模型（Kimi 负责理解/规划/控制，DeepSeek 负责生成/改写）
- **Local-First 保存**: 前端本地草稿 + 服务端持久化，冲突弹窗处理版本分歧
- **Story Model**: 人物图谱、世界观规则、PlotArc、风格画像等构成小说的可持续结构模型

### 测试分布

- **后端测试**: `tests/test_v1_*.py`（pytest），`conftest.py` 提供隔离数据库 fixture
- **前端测试**: 散布在 `src/**/__tests__/*.spec.js`（vitest + jsdom + @vue/test-utils）

---
> Source: [octsfm/InkTrace](https://github.com/octsfm/InkTrace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
