## geo-saas

> GEO (Generative Engine Optimization) SaaS — 面向中国 AI 搜索引擎（豆包 + 通义千问）的商户可见度优化工具。

# GEO SaaS — 项目规范

## 项目简介

GEO (Generative Engine Optimization) SaaS — 面向中国 AI 搜索引擎（豆包 + 通义千问）的商户可见度优化工具。

## 技术栈

- **后端**: Python 3.12 + FastAPI + SQLAlchemy 2.0 (async) + Pydantic v2 + Celery 5
- **前端**: Next.js 14 (App Router) + TypeScript + TailwindCSS + ECharts
- **数据库**: PostgreSQL 16 (asyncpg) + Redis 7
- **采集**: Playwright (浏览器自动化)
- **LLM**: OpenAI 兼容模式 (用户自定义 URL+密钥)
- **迁移**: Alembic (async)

## 架构

模块化单体，8 个业务模块:

```
⓪ 基础设施层（认证/DB/事件总线/Celery）
① AI搜索采集器  ② 信源分析  ③ 逆向分析引擎
④ 诊断评分      ⑤ 竞对分析  ⑥ 优化任务引擎
⑦ AI内容生成    ⑧ 客户报告门户
```

## 目录约定

```
backend/app/
  main.py          — FastAPI 入口
  config.py        — Settings(BaseSettings)
  database.py      — AsyncSession 工厂
  events.py        — EventBus (PostgreSQL + Celery)
  deps.py          — 依赖注入 (认证/多租户)
  auth/            — JWT 认证
  models/          — SQLAlchemy 模型
  schemas/         — Pydantic 模型
  core/            — 跨模块共享服务 (LLM客户端/成本控制/质量关卡/知识图谱)
  modules/         — 8个业务模块，每个含 routes/service/tasks
```

## 编码规范

- 语言: Python 代码英文，注释/commit/文档中文
- 异步优先: 所有数据库操作使用 async/await
- 不可变性: 优先创建新对象，避免就地修改
- 多租户: 所有业务表含 `client_id`，查询层统一过滤
- 错误处理: 每层显式处理，不静默吞掉
- 输入验证: Pydantic 模型验证所有外部输入
- 测试覆盖: ≥ 80%，先写测试

## 关键决策

| 决策 | 选择 | 理由 |
|------|------|------|
| 事件总线 | PostgreSQL events表 + Celery | 5-20客户规模，Kafka过度设计 |
| 共享服务 | `core/` 目录 | 与模块内 `service.py` 区分 |
| 采集并发 | Semaphore=4 | 防止 Playwright 内存溢出 |
| 信源优先级 | 百度地图/大众点评/百度百科 | Must Have 3平台，其余Phase 4 |
| 前端并行 | 第3周起 | 不压缩到第9周 |

## 重要文档

- 设计规格: `C:\Users\zouma\Desktop\GEO\docs\GEO-SaaS-设计规格文档-v1.md`
- 实施计划: `docs/implementation-plan.md`

## 常用命令

```bash
# 启动开发环境
docker compose up -d

# 后端开发
cd backend && uvicorn app.main:app --reload --port 8000

# 数据库迁移
cd backend && alembic upgrade head

# Celery Worker
cd backend && celery -A celery_app worker -l info -c 4

# 前端开发
cd frontend && npm run dev

# 测试
cd backend && pytest -v --cov=app --cov-report=term-missing
```

---
> Source: [halawwadjo-sketch/geo-saas](https://github.com/halawwadjo-sketch/geo-saas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
