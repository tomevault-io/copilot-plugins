## asksql

> > 项目速查手册，供 Claude Code 快速了解项目结构。不要全文读出，按需要查阅。

# NL2SQL Project Guide

> 项目速查手册，供 Claude Code 快速了解项目结构。不要全文读出，按需要查阅。

## 项目概述

**AskSql** — 一个自然语言转 SQL 的 AI 数据查询工具。用户用自然语言提问，系统自动生成 SQL、执行查询、并用自然语言 + 图表展示结果。

**技术栈**：
- **后端**：Python 3.12 + FastAPI + LangGraph + 多 LLM 供应商（Claude / OpenAI 兼容）
- **前端**：React 18 + TypeScript + Vite + Recharts + 玻璃拟态 UI
- **数据库**：SQLite（默认）+ 支持 MySQL/PostgreSQL 数据源接入
- **嵌入模型**：Qwen3-Embedding-4B（本地，用于记忆召回）

**架构模式**：LangGraph 多 Agent 编排 + ReAct 反思循环 + 记忆系统。

---

## 目录结构速查

```
nl2sql/
├── backend/                    # 后端（Python）
│   ├── app/                    # FastAPI 应用层
│   │   ├── api/                # API 路由（chat/datasources/schema/memories...）
│   │   ├── core/               # 核心配置、数据库
│   │   ├── services/           # 业务服务（chat/schema/memory/profiling...）
│   │   └── main.py             # FastAPI 入口
│   ├── nl2sql/                 # 核心 nl2sql 引擎库
│   │   ├── agent/              # Agent 逻辑（LangGraph 图 + 节点）
│   │   │   ├── graph.py        # NL2SQLAgent 主图定义
│   │   │   ├── dispatcher.py   # 分发 Agent（意图分类 + 路由）
│   │   │   ├── nodes/          # 各个节点（intent/generate/reflect/summarize...）
│   │   │   ├── tools/          # 工具定义（schema_tools/probe_tools）
│   │   │   └── state.py        # AgentState 状态定义
│   │   ├── llm/                # LLM 客户端抽象（claude/openai 兼容）
│   │   ├── schema/             # Schema 模型 + 匹配器 + Profiler
│   │   ├── executor/           # SQL 执行器
│   │   └── config.py           # 全局配置（Settings）
│   ├── tests/                  # 测试（pytest）
│   └── pyproject.toml          # 项目配置（uv）
├── frontend/                   # 前端（React + TypeScript）
│   └── src/
│       ├── components/         # UI 组件
│       ├── hooks/              # 自定义 hooks
│       ├── i18n/               # 国际化
│       └── lib/                # 工具函数
├── docs/                       # 设计文档和计划（很大，非必要不要读）
│   └── superpowers/
│       ├── plans/              # 实施计划（每个 4-5K 行，慎读）
│       └── specs/              # 设计规格
├── models/                     # 本地嵌入模型文件（.safetensors，不要读）
├── Makefile                    # 开发命令
├── run.sh                      # 一键部署脚本
└── docker-compose.yml          # Docker 编排
```

---

## 关键文件索引

### Agent 核心
- `backend/nl2sql/agent/dispatcher.py` — 顶层分发 Agent，意图分类后路由到子 Agent
- `backend/nl2sql/agent/graph.py` — NL2SQLAgent，LangGraph 主图
- `backend/nl2sql/agent/state.py` — AgentState 状态定义
- `backend/nl2sql/agent/schema_explorer.py` — Schema 探索 Agent（tool calling 循环）

### Agent 节点
- `backend/nl2sql/agent/nodes/intent.py` — 意图分析节点
- `backend/nl2sql/agent/nodes/generate.py` — SQL 生成节点
- `backend/nl2sql/agent/nodes/reflect.py` — 反思节点
- `backend/nl2sql/agent/nodes/execute.py` — SQL 执行节点
- `backend/nl2sql/agent/nodes/summarize.py` — 结果总结节点
- `backend/nl2sql/agent/nodes/visualize.py` — 可视化节点
- `backend/nl2sql/agent/nodes/clarify.py` — 澄清节点
- `backend/nl2sql/agent/nodes/rewrite.py` — 查询改写节点
- `backend/nl2sql/agent/nodes/_schema_context.py` — Schema 上下文构建工具

### LLM 抽象
- `backend/nl2sql/llm/factory.py` — LLM 客户端工厂
- `backend/nl2sql/llm/base.py` — 基类定义
- `backend/nl2sql/llm/claude_client.py` — Claude 客户端
- `backend/nl2sql/llm/openai_client.py` — OpenAI 兼容客户端
- `backend/nl2sql/config.py` — 全局配置

### Schema 系统
- `backend/nl2sql/schema/models.py` — Table/Column 数据模型
- `backend/nl2sql/schema/profiler.py` — Schema 数据剖析（统计信息、样例数据）
- `backend/nl2sql/schema/matcher.py` — Schema 匹配器（文本匹配表/列）

### 应用服务层
- `backend/app/services/chat_service.py` — 聊天服务（SSE 流式）
- `backend/app/services/schema_service.py` — Schema 服务
- `backend/app/services/memory_service.py` — 记忆服务
- `backend/app/services/profiling_service.py` — 数据剖析服务
- `backend/app/services/correction_detector.py` — 用户纠错检测
- `backend/app/services/schema_import.py` — Schema 导入

### API 层
- `backend/app/api/chat.py` — 聊天接口
- `backend/app/api/schema.py` — Schema 接口
- `backend/app/api/memories.py` — 记忆接口
- `backend/app/api/datasources.py` — 数据源接口

---

## 开发常用命令

### 后端
```bash
# 启动后端开发服务器（在 backend/ 目录下）
cd backend && uvicorn app.main:app --reload --port 8000

# 运行测试
cd backend && pytest tests/ -v
cd backend && pytest tests/ -v -k "test_name"  # 运行指定测试

# 依赖管理（uv）
cd backend && uv sync
cd backend && uv add <package>
```

### 前端
```bash
# 启动前端开发服务器
cd frontend && npm run dev

# 构建
cd frontend && npm run build
```

### Make 命令（项目根目录）
```bash
make dev          # 启动前后端开发环境
make backend      # 只启动后端
make frontend     # 只启动前端
make test         # 运行所有测试
make test-backend # 运行后端测试
```

---

## ⚠️ 不要读取的大文件

以下文件体积大且不需要完整阅读，除非明确需要具体内容：

| 文件 | 大小 | 原因 |
|------|------|------|
| `docs/superpowers/plans/*.md` | 每个 50-150K | 计划文档，按需读取特定章节 |
| `docs/superpowers/specs/*.md` | 每个 20-50K | 规格文档 |
| `backend/uv.lock` | 347K | Python 锁文件 |
| `frontend/package-lock.json` | 105K | npm 锁文件 |
| `backend/models/**/*.safetensors` | 多 GB | 嵌入模型二进制文件 |
| `backend/data/*.db` | 几百 KB | SQLite 数据库文件 |

**原则**：
- 不要用 `Read` 读整个 docs/superpowers/ 下的计划文档，用 offset/limit 读特定章节
- 不要读 lock 文件
- 不要读任何二进制文件
- 探索项目结构用 `find` + `ls`，不要靠 Read

---

## 当前进度（2026-08-24）

项目正在进行 **schema enrichment + memory** 特性开发：
- ✅ Phase 1: Schema 丰富化（Table/Column 扩展模型 + SchemaProfiler）
- ✅ Phase 2: Memory 系统（服务层 + API + 前端 UI）
- 🔄 Phase 2 后续：纠错检测、剖析服务等

相关 git 分支：`main`（所有改动已合入）

---

## 测试约定

- 后端测试框架：pytest
- 测试文件放在 `backend/tests/` 下，按模块组织
- 运行测试：`cd backend && pytest tests/ -v`
- 新增功能必须带测试

---
> Source: [Reiner-Liu/AskSql](https://github.com/Reiner-Liu/AskSql) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
