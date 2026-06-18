## eveningstudystatistics

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

晚自习统计系统 — 用于导入学校晚自习日报（xls/xlsx），解析报表数据入库，按教师维度统计出勤情况。FastAPI 后端 + 原生 HTML/JS/CSS 前端，SQLite 本地存储。

## 常用命令

```bash
# 激活虚拟环境并安装依赖
source .venv/bin/activate
pip install -r requirements.txt

# 启动开发服务器
uvicorn app.main:app --reload

# 运行全部测试
python -m unittest discover -s tests

# 运行单个测试文件
python -m unittest tests.test_teacher_stats_presenter
```

## 架构

三层结构：路由层 → 服务层 → 数据层。

- `app/main.py` — FastAPI 入口，注册路由和静态文件挂载
- `app/db.py` — SQLAlchemy 引擎/会话配置，SQLite 存储在 `data/app.db`，启动时自动建表
- `app/models.py` — 四张 ORM 模型：`ImportBatch`、`Teacher`、`SchoolClass`、`StudyReportRecord`
- `app/api/routes/` — 路由层
  - `imports.py` — 文件上传解析 (`/api/v1/import-batches/parse`) 和导入入库 (`/api/v1/import-batches`)
  - `query.py` — 查询接口：批次列表/详情、明细记录、教师统计
  - `pages.py` — 前端页面路由，返回 `frontend/index.html`
- `app/services/` — 业务逻辑层
  - `report_parser.py` — `EveningStudyReportParser`，解析 xls/xlsx 报表，提取标题日期、部门、汇总、明细记录
  - `import_service.py` — `ImportBatchService`，文件去重（SHA-256）、解析入库、自动创建教师/班级主数据
  - `query_service.py` — `QueryService`，分页查询批次/明细/教师统计，教师统计通过分组子查询聚合
  - `teacher_stats_presenter.py` — 教师统计展示层，构建 display/summary/attention 结构
- `frontend/` — 原生前端，无构建工具，通过 FastAPI StaticFiles 挂载 `/assets`

## 关键业务规则

- 报表通过文件 SHA-256 哈希去重，同一文件不会重复导入
- `is_roll_called = false` 时，出勤数字段（应到/实到/迟到等）置为 null，不参与统计
- 教师统计口径：点名覆盖率 = 已点名次数/排班次数；到课率 = sum(实到)/sum(应到)，仅统计已点名记录
- 报表标题格式：`{日期}{部门}自修课情况统计表`，日期也可从文件名中提取

## 数据库

开发环境使用 SQLite（`data/app.db`），生产 schema 设计面向 MySQL（见 `sql/schema.sql`）。ORM 模型与 SQL schema 保持对应。数据库设计文档在 `docs/database-design.md`。

## 测试

测试使用 unittest，位于 `tests/`。集成测试通过临时 SQLite 数据库隔离，无需外部依赖。

---
> Source: [alastairxhy/EveningStudyStatistics](https://github.com/alastairxhy/EveningStudyStatistics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
