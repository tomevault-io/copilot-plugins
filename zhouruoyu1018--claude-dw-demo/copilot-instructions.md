## claude-dw-demo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a data warehouse project using the Hadoop ecosystem with three query engines:
- **Hive (Tez)**: Batch processing and complex ETL transformations
- **Impala**: Low-latency interactive queries
- **Doris**: Real-time OLAP analytics

## Project Structure

```
sql/
├── hive/          # Hive SQL scripts (executed via Tez)
│   ├── ddl/       # Table definitions, schema changes
│   ├── dml/       # Data manipulation, inserts, updates
│   └── etl/       # ETL transformation jobs
├── impala/        # Impala SQL scripts
│   ├── ddl/       # Table definitions (Impala-specific syntax)
│   ├── dml/       # Query and data operations
│   └── etl/       # Incremental processing jobs
├── doris/         # Doris SQL scripts
│   ├── ddl/       # Doris table models (Aggregate, Unique, Duplicate)
│   ├── dml/       # Stream load and data operations
│   └── etl/       # Real-time sync jobs
└── common/        # Shared SQL snippets and utilities
scripts/           # Shell scripts for job orchestration
config/            # Connection configs and environment settings
```

## Common Commands

### Hive (Tez)
```bash
# Execute Hive script
hive -f sql/hive/etl/script.sql

# Execute with variables
hive -f script.sql -hivevar stat_date=2024-01-01

# Beeline connection
beeline -u "jdbc:hive2://host:10000/default" -f script.sql
```

### Impala
```bash
# Execute Impala script
impala-shell -f sql/impala/etl/script.sql

# With database and variables
impala-shell -i host:21000 -d database_name -f script.sql --var=stat_date=2024-01-01

# Refresh metadata after Hive changes
impala-shell -q "INVALIDATE METADATA table_name"
```

### Doris
```bash
# MySQL client connection
mysql -h host -P 9030 -u user -p < sql/doris/ddl/script.sql

# Stream Load
curl -u user:password -H "label:load_label" -T data.csv \
  http://fe_host:8030/api/db/table/_stream_load
```

## 业务领域

本数仓服务于**贷款业务**：
- **贷款销售**: 进件、授信、签约、放款
- **贷后管理**: 还款、逾期、催收、核销

## 数仓分层

```
da  (数据应用层)  ← 应用数据，直接支撑报表/接口     ← 你的工作
dm  (数据集市层)  ← 业务指标宽表、数据预聚合       ← 你的工作
dws (汇总层)      ← 通用汇总指标
dwm (中间层)      ← 明细宽表
dwd (明细层)      ← 清洗后明细数据
ods (原始层)      ← 原始数据接入
```

**你的工作范围**: 仅操作 **dm** 和 **da** 层

### 物理库名映射

逻辑分层与物理库名的对应关系（Hive/Impala 共享 Metastore，Doris 独立库）：

| 逻辑分层 | Hive/Impala 库名 | Doris 库名 | 表名前缀 |
|---------|-----------------|-----------|---------|
| **dm** | `ph_sac_dmm` | `ph_dm_sac_drs` | `dmm_sac_` |
| **da** | `ph_sac_da` | `ph_dm_sac_drs` | `da_sac_` |

> **重要**: DDL 和 ETL SQL 中必须使用**物理库名**（如 `ph_sac_dmm.dmm_sac_loan_prod_daily`），不能使用逻辑分层名（如 `dm.dmm_sac_loan_prod_daily`）。上游源表的库名从元数据获取，不在此映射范围内。

## SQL Conventions

- **Naming**: Use snake_case for tables and columns (e.g., `dm_loan_daily`, `da_overdue_report`)
- **Layer prefixes**: `ods_` (raw), `dwd_` (detail), `dwm_` (middle), `dws_` (summary), `dm_` (mart), `da_` (application)
- **Partitions**:
  - Hive/Impala: 日分区 `stat_date` (格式 `YYYY-MM-DD`), 月分区 `stat_month` (格式 `YYYY-MM`)
  - Doris: 统一使用 `partition_key`
- **File headers**: Include author, create date, description, and changelog

## Engine-Specific Notes

### Hive vs Impala Compatibility
- Use `INVALIDATE METADATA` or `REFRESH` in Impala after Hive DDL changes
- Impala does not support all Hive UDFs; check compatibility before using
- Avoid Hive-specific syntax (e.g., `SORT BY`, `CLUSTER BY`) in shared scripts

### Doris Table Models
- **Aggregate Model**: Pre-aggregation for metrics (SUM, MAX, MIN, etc.)
- **Unique Model**: Upsert semantics with primary key
- **Duplicate Model**: Append-only for detailed logs

## 数据开发决策流程 (复用优先)

收到指标/报表需求时，严格遵守以下步骤：

1. **Check (查)**: 先调用 `search_existing_indicators` 搜索指标库，检查指标是否已计算过
2. **Verify (验)**: 找到后展示口径描述，让用户确认是否与需求一致
3. **Reuse (用)**: 用户确认一致则直接 SELECT 该字段，**禁止重写计算逻辑**
4. **Build (造)**: 仅当指标不存在或口径不符时，在 dm/da 层开发

**核心原则**: 复用现有资产，避免重复造轮子，确保数据口径一致性。

**分层选择**:
- **dm**: 业务指标宽表、可复用的预聚合数据
- **da**: 直接面向特定报表/接口的应用数据

### 相关 Skills

本项目提供以下自定义 Skills（位于项目根目录的 .skill 文件）：

| Skill | 文件 | 用途 |
|-------|------|------|
| `dw-requirement-triage` | `dw-requirement-triage.skill` | 需求拆解，自动触发指标复用检索 |
| `search-hive-metadata` | `search-hive-metadata.skill` | 元数据搜索，提供 MCP 工具 |
| `generate-standard-ddl` | `generate-standard-ddl.skill` | 模型设计，生成符合规范的 DDL 语句 |
| `generate-etl-sql` | `generate-etl-sql.skill` | ETL 代码生成，编写 Hive/Impala/Doris SQL |
| `generate-qa-suite` | `generate-qa-suite.skill` | 测试与 DQC 生成，冒烟测试 + 质量规则 + Doris 性能分析 |
| `review-sql` | `.claude/skills/review-sql/SKILL.md` | SQL 审查，检查现有 DDL/ETL 脚本规范合规性 |
| `refactor-sql` | `.claude/skills/refactor-sql/SKILL.md` | SQL 重构与自动改写，支持 code/perf/compliance/migrate |
| `dw-dev-workflow` | `dw-dev-workflow.skill` | **主控工作流**，串联以上所有 skill |

## Skills 配置

```yaml
skills:
  - path: ./dw-dev-workflow.skill
    name: dw-dev-workflow
    description: 数仓开发全流程工作流
  - path: ./dw-requirement-triage.skill
    name: dw-requirement-triage
    description: 需求拆解
  - path: ./generate-standard-ddl.skill
    name: generate-standard-ddl
    description: DDL 生成
  - path: ./generate-etl-sql.skill
    name: generate-etl-sql
    description: ETL SQL 生成
  - path: ./generate-qa-suite.skill
    name: generate-qa-suite
    description: 测试套件生成
  - path: ./.claude/skills/review-sql/SKILL.md
    name: review-sql
    description: SQL 审查
```

### 使用方式

1. **完整工作流**: `/dw-dev-workflow` - 从需求到测试脚本一站式生成
2. **单独调用**: `/generate-standard-ddl`, `/generate-etl-sql` 等

## Doris 跨引擎关联规范

### Catalog 命名（固定，所有环境统一）
- 外表(Hive): `hive_bdsp.{hive库名}.{表名}`，例: `hive_bdsp.ph_sac_dmm.dim_ds_dept_relation`
- 内表(Doris): `internal.{doris库名}.{表名}`，例: `internal.ph_dm_sac_drs.dmm_sac_xxx`
- **混用内外表时，内表也必须加 `internal.` 前缀**

### 数据同步策略（二选一）

| 策略 | 适用场景 | 实现方式 |
|------|---------|---------|
| **Catalog 直查** | Hive 表 <100w 行（维度表等） | Doris SQL 中直接 `JOIN hive_bdsp.库.表` |
| **INSERT 同步** | Hive 表 ≥100w 行 | 定时调度将 Hive 数据 INSERT INTO Doris 本地表 |

> 特大表同步时，必须在 SELECT 中过滤有效数据再导入，禁止全量同步。

### Doris SQL 强制规范
- 禁止 `SELECT *`
- JOIN/比较/计算必须统一数据类型
- 分区表必须用 `partition_key` 提前过滤
- 所有作业需设置 `set exec_mem_limit=1024*1024*1024*15;`（15GB 内存限制）
- JOIN 必须显式指定 hint：大表 JOIN `[broadcast]` 小表，大表 JOIN `[shuffle]` 大表
- 关联 Hive 外表：≥100w 加 `[shuffle]`，<100w 放右表加 `[broadcast]`

### Doris 元数据获取
Doris 表无 MCP 元数据服务，需用户提供建表 DDL 语句。

## 建模决策策略 (Schema Evolution Policy)

当用户提出新指标需求时，不要急着生成 CREATE TABLE。请执行以下检查：

1. **维度匹配**: 提取新指标所需的维度（Group By Keys）。
2. **候选搜索**: 调用 `search_hive_metadata` 寻找是否存在具有相同维度的现有表。
3. **表角色分级**: 对候选表按 标签表 > 维度/引用表 > 指标宽表 > 普通业务表 顺序判定角色。标签表/指标宽表/普通业务表为高优先级（进入 CASE A），维度/引用表（表名/注释含 dim 等关键词）为低优先级（走 CASE E 询问用户）。
4. **决策逻辑**（两阶段：先独立判断 D，再在 A → C → E → B 中首条命中即执行）:
   - **CASE D (同步表)**: Doris ETL 需关联 Hive 大表（≥100w）-> 生成同步表 DDL（独立于 A/B/C/E，可同时产出）。
   - **CASE A (扩充)**: 高优先级候选（指标宽表/标签表/普通业务表），粒度匹配，业务主题相近 -> 生成 `ALTER TABLE ADD COLUMN`。
   - **CASE C (冲突)**: 存在粒度匹配的高优先级候选但全部跨域 -> 询问用户。
   - **CASE E (维度表扩列确认)**: 低优先级候选（维度/引用表），粒度匹配，主题相近，且无高优先级候选命中 A -> 询问用户是否确认扩列，否则新建表。
   - **CASE B (新建)**: 以上均不命中 -> 生成 `CREATE TABLE`。

5. **输出要求**: 在你的回复中，明确告知用户你的决定理由。
   例如："检测到现有表 `dmm_sac_loan_dtl` 粒度与新指标一致，建议在该表中新增字段，而不是新建表。"

---
> Source: [zhouruoyu1018/claude_dw_demo](https://github.com/zhouruoyu1018/claude_dw_demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
