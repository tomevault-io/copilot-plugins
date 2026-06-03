## seagull

> > 适用范围：所有 Seagull 项目开发工作

# Seagull 量化研究平台 - 开发规范

> 适用范围：所有 Seagull 项目开发工作
> 版本：v1.3
> 更新日期：2026-05-19
> 专业等级：机构级 ■■■■■ | 深度：深度交叉验证 | 参考标准：MSCI Barra, Axioma Qontigo, Northfield, BlackRock Aladdin
>
> 作为MSCI Barra, Axioma Qontigo, Northfield, BlackRock Aladdin，Two-Sigma专业量化专家，你认为那种方式最合适最专业，容易管理和维护，也容易输出各种对比结果

---

## ⚠️ MANDATORY IMPORT RULE (HIGHEST PRIORITY)

**此规则覆盖所有其他代码模式，必须严格遵守：**

- ✅ **正确写法:** `from seagull.module import X`
- ❌ **禁止写法:** `from src.seagull.module import X`

**重要提示：**
- 即使你在代码库中看到 `from src.seagull.` 的旧模式，也 **不要** 复制或使用它
- 所有新代码和修改的代码 **必须** 使用 `from seagull.` 格式
- 违反此规则的提交将被自动拒绝
- 如果你发现还有文件使用错误格式，请先修复后再提交

---

## ⚠️ MANDATORY TECH STACK RULE (HIGH PRIORITY)

**此规则覆盖所有数据处理代码，必须严格遵守：**

- ✅ **首选技术栈：Polars + Parquet**
  - 所有新的 ETL、因子计算、数据处理代码 **必须** 使用 Polars 作为计算引擎
  - 所有持久化数据存储 **必须** 使用 Parquet 格式（列式存储 + 压缩）
  - 性能目标：比 Pandas + CSV 架构提升 **10-20 倍**

- ❌ **禁止默认使用 Pandas + CSV**
  - 除非有明确的兼容性理由（且需在 PR 中说明），否则禁止新代码使用 Pandas + CSV
  - 现有 Pandas 代码应逐步迁移到 Polars

### Polars 最佳实践（强制）

| 规则 | 原因 |
|------|------|
| ✅ 优先使用惰性执行 (`scan_parquet`) | 支持谓词下推、投影下推，全链路优化 |
| ✅ 使用 `over()` 做分组窗口计算 | 原生支持分组窗口，比 Pandas `groupby + join` 简洁 10 倍 |
| ✅ 表达式链式调用，避免中间变量 | 优化器可以做全局优化 |
| ✅ 优先使用内置 `rolling_*` 函数 | 向量化实现，比自定义 UDF 快 100x |
| ❌ 不要逐行 `apply` | 这是 Pandas 的坏习惯，Polars 中慢 100 倍 |
| ❌ 不要频繁 `to_pandas()` | 会触发完整内存复制，尽量全链路 Polars |

### Parquet 最佳实践（强制）

| 规则 | 原因 |
|------|------|
| ✅ Row Group 大小设为 50-100 万行 | 太小元数据开销大，太大谓词下推效果差 |
| ✅ 按日期分区存储 | `factor.parquet/date=2020-01-01/file.parquet`，按日期查询跳过整个目录 |
| ✅ 热数据用 Snappy 压缩，冷数据用 ZSTD | Snappy 速度快，ZSTD 压缩比高 30% |
| ✅ 按查询模式排序 | 经常按日期查询就按 date 排序，经常按股票查询就按 stock 排序 |
| ❌ 不要存成一个超大文件 | 最好控制在 100-500MB / 文件，利于并行读取 |
| ❌ 不要频繁小批量追加 | Parquet 不可变，追加 = 重写，用 Delta Lake 解决 |

### 标准工作流示例

```python
import polars as pl

# ✅ 惰性扫描 + 谓词/投影下推（关键优化）
result = (
    pl.scan_parquet("market_data.parquet")
      .select(["date", "stock", "close", "volume"])  # 只读需要的列
      .filter(pl.col("date") >= "2020-01-01")         # 只读需要的行
      .with_columns([
          pl.col("close").pct_change().over("stock").alias("return"),
          pl.col("close").rolling_mean(20).over("stock").alias("ma20"),
          (pl.col("close") / pl.col("close").rolling_mean(5).over("stock") - 1)
              .rank("average").over("date").alias("factor")
      ])
      .collect()  # 最终触发执行
)

# ✅ 结果写回 Parquet
result.write_parquet("factor_results.parquet", compression="snappy")
```

---

## 0. 项目结构与常用路径

### 0.1 目录结构

```
seagull/
├── src/seagull/              # 核心源码目录
│   ├── attribution/            # 归因与ablation研究 
│   ├── backtesting/            # 回测引擎
│   │   └── model/              # 回测模型
│   ├── core/                   # 核心工具与通用组件
│   ├── data/                   # 数据层
│   │   └── etl/                # ETL脚本
│   ├── execution/              # 交易执行
│   ├── factor/                 # 因子开发
│   └── risk/                   # 风险管理
├── tests/                      # 测试套件
├── docs/                       # 文档与设计
├── nas/doc/                    # 完整量化知识库（15个分类）
├── factor/                     # 因子相关文件
├── scripts/                    # 工具脚本
├── .venv/                      # 虚拟环境
├── pyproject.toml              # 项目配置
├── README.md                   # 项目文档
└── CLAUDE.md                   # 本文件 - 开发规范
```

### 0.2 常用路径速查表

| 路径 | 说明 | 典型使用场景 |
|------|------|--------------|
| `tests/` | 测试根目录 | 运行所有测试: `pytest tests/ -v` |
| `src/seagull/backtesting/model/` | 回测模型 | 添加新的策略模型、绩效计算 |
| `src/seagull/data/etl/` | 数据ETL | 新增数据源、数据清洗逻辑 |
| `src/seagull/factor/` | 因子开发 | 新因子实现、因子工具函数 |
| `src/seagull/attribution/` | 归因分析 | 收益归因、ablation研究 |
| `nas/doc/` | 完整知识库 | 架构文档、操作手册、SOP |
| `nas/doc/README.md` | **文档系统首选索引入口** | 查阅、整理、合并或新增文档前优先读取 |
| `nas/doc/GUIDE.md` | Seagull 文档编写指南 | 编写、重构、拆分、合并项目文档 |
| nas/doc/3_01_数据获取与处理/02-表结构/02-REF-表命名与分层枚举.md |  | ddl命名规范，data/etf文件命名规范 |

**Core 核心组件子目录：**

| 路径 | 说明 |
|------|------|
| `src/seagull/core/config/` | 配置管理、全局常量、环境配置 |
| `src/seagull/core/dag/` | Dagster 资产定义、Graph 定义 |
| `src/seagull/core/frontend/` | 前端页面、可视化组件 |
| `src/seagull/core/pipeline/` | Pipeline 基类、编排逻辑、状态管理 |
| `src/seagull/core/route/` | 路由配置、API 路由 |
| `src/seagull/core/services/` | 服务层、外部服务集成 |
| `src/seagull/core/utils/` | **最常用** - 通用工具函数、数学工具、日期处理 |
| `src/seagull/core/workflow/` | 工作流定义、任务调度 |
| `log/{module_name}.log` | **日志文件** - 每个模块独立日志，异常排查首选 |

### 0.2 Pipeline 数据 IO 统一入口

**推荐使用 `facade_data.py` 作为 Pipeline 数据读写统一入口：**

| 函数 | 说明 |
|------|------|
| `load_data` / `write_data` | DB/Parquet 统一读写 |
| `read_parquet_artifact` / `write_parquet_artifact` | Artifact 读写（Pipeline 中间结果） |
| `stable_artifact_hash` | 文件 SHA256 哈希（数据一致性验证） |
| `write_metadata_json` | 元数据 JSON 写入 |

```python
# ✅ 推荐：统一入口
from seagull.core.utils.facade_data import load_data, write_data, write_parquet_artifact

# 读取数据
df = load_data("dwd_stock_factor_preprocess_incr", input_source="parquet")

# 写入 Artifact
write_parquet_artifact(result_df, "output/result.parquet")
```

**兼容模式**：`facade_artifacts.py` 仍可使用，内部委托 `facade_data.py`。建议新代码直接使用 `facade_data.py`。

### 0.3 常用命令

```bash
# 运行测试
pytest tests/ -v
pytest tests/factor/test_turnover.py -v

# 运行 Pipeline
python src/seagull/main.py --pipeline factor --factor turnover

# Dagster UI
dagster dev -f src/seagull/main.py

# 代码格式化
black src/seagull/
isort src/seagull/

# 验证 Polars + Parquet 性能
polars check --show-stats factor_results.parquet
```

### 0.4 异常排查：日志使用

遇到异常时，优先查看对应模块的日志文件：

- **日志位置**: `{PATH}/log/{模块名}.log` (如 `utils_database.py` → `log/utils_database.log`)
- **快速排查**: 无需读取全部文件，直接搜索 `ERROR` / `CRITICAL` 或按时间筛选最近日志
- **链路追踪**: 按时间戳排序查看，还原完整调用链路

```bash
# 查看最近 50 行日志
tail -n 50 log/utils_database.log

# 只看错误日志 (Windows)
findstr "ERROR CRITICAL" log/ods_stock_quote_daily_incr_baostock.log

# 按时间筛选 (指定日期之后)
grep "26-05-14 10:[2-5][0-9]" log/factor_rolling_pipeline.log
```

---

## 1. 新增 ETL 表快速指南（高频需求）

> 完整文档：`nas/doc/15_操作手册/Seagull_新增ETL表操作手册.md`

### 1.1 核心流程

```
1. 需求确认 → 2. 实现 ETL → 3. 接入 Pipeline → 4. 接入 Dagster → 5. 验证
```

### 1.2 命名规范（强制）

```
格式：{layer}_{asset_type}_{category}_{name}_{type}.py
示例：ods_stock_quote_daily_incr_baostock.py
```

| 层 | 含义 |
|----|------|
| `ods` | 原始数据层 |
| `dwd` | 清洗明细层 |
| `dws` | 汇总层 |
| `ads` | 应用层 |

### 1.3 ETL 文件位置

```
src/seagull/data/etl/{layer}/{asset_type}/{category}/

示例：src/seagull/data/etl/ods/stock/finance/
```

### 1.4 ETL 标准结构

```python
# 导入 → 常量配置 → fetch_data() → clean_data() → write_to_db() → pipeline()
# ✅ 必须：可独立运行 python xxx.py 测试
```

### 1.5 Pipeline 接入

| 数据类型 | Pipeline 文件 |
|---------|--------------|
| 原始数据、行情 | `01_data_pipeline.py` |
| 因子计算 | `02_factor_pipeline.py` |

添加 Step → 在 run() 中注册 → 保存 checkpoint

### 1.6 Dagster Asset 接入

| 层 | Asset 文件 | IO Manager |
|----|-----------|-----------|
| ODS / DIM | `ods_assets.py` / `dim_assets.py` | `io_manager`（只读） |
| DWD / DWS | `dwd_factor_assets.py` | `parquet` / `postgres` |

**⚠️ 重要：Asset 是薄封装，只调用 ETL，不写业务逻辑！**

### 1.7 避坑指南

| 坑 | 正确做法 |
|---|---------|
| Asset 中写业务逻辑 | ❌ 禁止 → ✅ 只调用 ETL/Pipeline 函数 |
| table_name 带 schema 前缀 | ❌ `stg.ods_xxx` → ✅ `ods_xxx`（自动读取环境配置） |
| IO Manager 配置错误 | 只读 → `io_manager`，持久化 → `parquet`，写库 → `postgres` |

### 1.8 新增表结构快捷 DDL 建表

**⚡ 快速建表工具：** `bin/create_table.py`

遇到新增表结构需求时，按以下步骤操作：

1. **添加 DDL SQL 文件** 到 `ddl/` 目录：
   - 命名规范：`{table_name}.sql`
   - 示例：`ddl/factor_validation_tables.sql`

2. **使用快捷命令建表**：
   ```bash
   # 创建所有表（自动识别当前环境 schema：STG 或 public）
   python bin/create_table.py
   
   # 创建指定表
   python bin/create_table.py --table factor_validation_tables
   
   # 强制指定 schema（覆盖环境配置）
   python bin/create_table.py --schema public
   ```

3. **也可在 Python 代码中调用**：
   ```python
   from seagull.core.utils.utils_database import create_table_from_ddl, create_all_tables
   
   # 创建单个表
   create_table_from_ddl('factor_validation_tables')
   
   # 批量创建所有表
   create_all_tables()
   ```

**自动 schema 检测：**
- 优先读取 `settings.POSTGRES_SCHEMA` 配置
- STG 测试环境 → 建表到 `stg.` schema
- 开发环境 → 建表到 `public` schema
- 可通过 `--schema` 参数强制覆盖

---

### 1.9 Pipeline / ETL 脚本测试规范

#### 1.9.1 测试环境：ENV 必须为 STG

**所有测试必须使用 STG 环境**，禁止在生产环境（public schema）直接测试。

| 环境 | schema | 用途 |
|------|--------|------|
| STG | `stg.*` | 开发、测试、验证 |
| PROD | `public` | 生产数据 |

**ENV 切换方式：**
```python
# 方式 1：通过环境变量（推荐）
import os
os.environ["ENV"] = "STG"  # 或 "PROD"

# 方式 2：直接修改 settings
from seagull.core.config import settings
settings.POSTGRES_SCHEMA = "stg"  # 测试时
settings.POSTGRES_SCHEMA = "public"  # 正式部署时
```

**验证当前环境：**
```python
from seagull.core.utils.utils_database import get_postgres_schema
print(get_postgres_schema())  # 应输出 "stg" 或 "public"
```

#### 1.9.2 `if __name__ == "__main__":` 必须用真实数据 E2E 验证

每个 Pipeline / ETL 脚本的 `if __name__ == "__main__":` 块是**开发者的第一道 E2E 门禁**。必须满足：

1. **使用真实数据**（从 DB 或 parquet 文件），不能用 mock 数据
2. **完整流程**（从输入 → 处理 → 输出），不能只打印 "hello world"
3. **ENV=STG** 运行，写入 `stg.` schema
4. **验证输出**：检查返回数据的 schema、行数、日志无 ERROR

**标准模板：**

```python
if __name__ == "__main__":
    # =========================================================================
    # 调试入口 - 直接运行脚本进行测试（ENV=STG）
    # =========================================================================
    # 真实数据配置（从 DB registry 或文档中获取）
    # 必须替换为实际存在的 reasoning_run_id / artifact_id / features_path
    # =========================================================================
    import os
    os.environ["ENV"] = "STG"  # 强制使用 STG 环境

    # Registry 模式（推荐）：从 DB 读取真实输入
    DEBUG_REGISTRY_MODE: bool = True
    DEBUG_REASONING_RUN_ID: str | None = "pred_a2dfad79992ae367f30a0f75"  # 替换为真实 ID
    DEBUG_ARTIFACT_ID: str | None = "artifact_train_20260525_140409_ded7b4d83ec4"
    DEBUG_BYPASS_CONTRACT: bool = True   # 测试时跳过契约校验（可选）
    DEBUG_DRY_RUN: bool = False
    DEBUG_RERUN: bool = True

    # 或直接模式：不走 registry，直接指定输入路径
    # DEBUG_REGISTRY_MODE = False
    # DEBUG_REGISTRATION_PATH = "nas/train_results/exp_xxx_model_registration.json"
    # DEBUG_FEATURES_PATH = "data/features/oos/2025-04.parquet"

    extra_args = []
    if DEBUG_REGISTRY_MODE and DEBUG_REASONING_RUN_ID:
        extra_args.extend(["--reasoning_run_id", DEBUG_REASONING_RUN_ID])
        if DEBUG_ARTIFACT_ID:
            extra_args.extend(["--artifact_id", DEBUG_ARTIFACT_ID])
    else:
        extra_args.extend(["--registration_path", DEBUG_REGISTRATION_PATH])
        extra_args.extend(["--features_path", DEBUG_FEATURES_PATH])

    if DEBUG_BYPASS_CONTRACT:
        extra_args.append("--bypass_contract_check")
    if DEBUG_DRY_RUN:
        extra_args.append("--debug_dry_run")
    if DEBUG_RERUN:
        extra_args.append("--rerun")

    result = main(extra_args)
    print("✅ Pipeline 运行完成")
    print(f"   输出行数: {result.height if hasattr(result, 'height') else len(result)}")
    print(f"   输出列: {result.columns if hasattr(result, 'columns') else 'N/A'}")
```

**快速定位真实数据的方法：**

```python
# 从 DB registry 查询可用的 reasoning_run_id
from seagull.core.utils.facade_db import create_psycopg2_connection
conn = create_psycopg2_connection()
with conn.cursor() as cur:
    cur.execute("""
        SELECT reasoning_run_id, dataset_role, as_of_date, publish_status,
               selection_policy, features_uri, requested_artifact_id
        FROM reasoning_input_registry
        WHERE publish_status = 'published'
        ORDER BY created_at DESC LIMIT 5
    """)
    for r in cur.fetchall():
        print(r)
```

**验收标准：**

| 检查项 | 合格标准 |
|--------|---------|
| ENV=STG | `stg.` schema 可写 |
| 真实数据 | 从 DB parquet 读取，不用 mock |
| 完整流程 | 输入 → 计算 → 输出全部经过 |
| 输出可查 | 数据写入 DB 或 parquet，日志无 ERROR |
| schema 正确 | 输出列名、行数与预期一致 |

**禁止的行为：**

```python
# ❌ 禁止：假数据、空跑、不验证输出
if __name__ == "__main__":
    print("hello world")  # 什么也没测
    df = pl.DataFrame({"a": [1]})  # mock 数据，没有真实场景
    main()  # 不传参数，不知道跑了什么

# ✅ 正确：真实 ID + 完整流程 + 输出验证
if __name__ == "__main__":
    os.environ["ENV"] = "STG"
    result = main(["--reasoning_run_id", "pred_xxx", "--artifact_id", "artifact_xxx", "--rerun"])
    assert result.height > 0, "输出为空"
```

---

## 一、开发准则（Two Sigma 标准）

### 1.1 Git 提交规范

**提交格式：** `type(scope): message`

| type | 说明 |
|------|------|
| `feat` | 新增功能（新因子、新Pipeline、新工具） |
| `fix` | 修复bug |
| `docs` | 文档更新 |
| `refactor` | 代码重构（无功能变更） |
| `test` | 测试相关 |
| `chore` | 构建/工具链/依赖更新 |

**scope 取值：**
`data`, `factor`, `validation`, `backtest`, `execution`, `risk`, `attribution`, `core`

**示例：**
```bash
git commit -m "feat(factor): add turnover factor, L1 test pass"
git commit -m "fix(validation): fix turnover percentage calculation"
git commit -m "docs(readme): add 5-minute quick start guide"
```

**要求：** 每次提交一个完整功能模块，多个模块分多次提交。

---

### 1.2 验证成功标准

完成任何功能后，必须提供验证证据：
- **因子开发：** 提供 L1 单元测试报告 + 计算正确性验证
- **Pipeline 开发：** 提供 E2E 运行日志 + 输出数据 schema 验证
- **Bug 修复：** 提供修复前后对比数据 + 回归测试通过
- **可视化：** 提供图表截图 + 数据来源说明

---

### 1.3 开发环境

- **操作系统：** Windows
- **Python 版本：** 3.10+
- **包管理：** uv
- **虚拟环境：** .venv

**环境配置：**
```bash
uv venv .venv
.venv\Scripts\activate
uv pip install -e ".[dev]"
```

---

### 1.4 调试方法论

遇到编译/运行问题时，采用**分步骤模块化验证：**

1. **环境加载验证** - Python 版本、依赖版本、路径配置
2. **简单编译验证** - 单独 import 模块，无语法错误
3. **数据计算验证** - 小样本数据跑通核心逻辑
4. **完整 Pipeline 验证** - 全量数据运行
5. **端到端验证** - 从数据到报告完整流程

每一步确认通过后再进入下一步。

---

### 1.5 代码复用原则

- 优先复用 `src/seagull/core/` 中的基础组件
- 因子计算的通用逻辑放到 `factor/utils.py`
- Pipeline 的编排逻辑放到 `core/pipeline.py`
- 不要重复实现相同的数学计算逻辑

---

### 1.6 交互规范

所有命令行工具的选项和提示使用**中文**展示。

---

## 二、量化因子开发 SOP

所有因子必须通过四层验证，才能进入回测：

### L1 单元测试（Unit Test）
- ✅ 字段对齐检查（datetime, instrument 维度正确）
- ✅ 滞后检查（无未来函数泄漏）
- ✅ 缺失值处理逻辑正确
- ✅ 越界防护（无 Inf/-Inf）
- ✅ 数学稳定性（极端输入下不崩溃）

### L2 集成测试（Integration Test）
- ✅ Pipeline 端到端跑通
- ✅ 数据血缘正确（输入 → 输出链条完整）
- ✅ 横截面分布稳定
- ✅ Rank IC 连续性合理
- ✅ 暴露漂移在可接受范围

### L3 OOS 测试（Out-of-Sample Validation）
- ✅ Walk Forward 验证通过
- ✅ Regime 稳定性（牛熊/震荡市表现一致）
- ✅ 子市场稳定性（主板/创业板/科创板均有效）
- ✅ 因子衰减曲线正常（单调递减，无突变）
- ✅ 参数鲁棒性（参数变化 20% 内结果稳定）

### L4 实盘前检查（Pre-Live Validation）
- ✅ 回测 vs 仿真一致性验证
- ✅ 延迟检查（计算时间在可接受范围）
- ✅ 实时异常监控规则定义
- ✅ 风控规则联调通过
- ✅ Shadow Mode 影子交易验证完成

---

## 三、Superpowers 技能矩阵

### 3.1 核心技能应用场景

| 技能 | 触发时机 | 典型场景 |
|------|----------|----------|
| `/superpowers:brainstorming` | 任何创造性工作开始前 | 新因子方案设计、新Pipeline架构、重构方案 |
| `/superpowers:writing-plans` | 有需求规格，做多文件实现任务 | 大型功能开发、重构、跨模块变更 |
| `/superpowers:executing-plans` | 已有实现计划，开始实施 | 按计划执行编码 |
| `/superpowers:systematic-debugging` | 遇到bug/测试失败/异常 | 因子计算错误、Pipeline失败、性能问题 |
| `/superpowers:test-driven-development` | 实现功能或修复bug前 | 核心算法、关键路径开发 |
| `/superpowers:requesting-code-review` | 完成主要实现 | 任何超过 100 行的代码变更 |
| `/superpowers:verification-before-completion` | 即将声称工作完成前 | 交付前最终验证 |
| `/superpowers:dispatching-parallel-agents` | 面对 2+ 个独立任务 | 多个独立bug修复、多个并行功能开发 |
| `/quantitative-finance-expert` | 任何量化金融相关问题 | 因子定义、指标解释、金融术语确认 |
| `/code-refactor` | 代码重构优化 | 清理技术债务、提升可维护性 |
| `/naming-refactor` | 命名规范化 | 因子命名统一、变量命名规范 |
| `/markdown-document-organizer` | 整理文档 | 笔记整理、文档索引生成 |

---

### 3.2 标准工作流组合

**新因子开发工作流：**
```
1. brainstorming → 因子方案设计、Alpha逻辑验证
2. writing-plans → 实现计划、测试用例设计
3. test-driven-development → 先写 L1/L2 测试
4. executing-plans → 实现因子计算逻辑
5. systematic-debugging → （如果有问题）排查修复
6. requesting-code-review → 代码审查
7. verification-before-completion → L3/L4 验证
```

**Bug 修复工作流：**
```
1. systematic-debugging → 系统化排查根因
2. test-driven-development → 写复现测试用例
3. executing-plans → 修复代码
4. verification-before-completion → 回归测试验证
```

**文档编写工作流：**
```
1. brainstorming → 文档结构设计
2. executing-plans → 编写文档
3. verification-before-completion → 验证README复制粘贴即可运行
```

---

### 3.3 Agent 团队协作模式

对于 Complex 级别的任务，使用多 Agent 分工协作：

| Agent 角色 | 职责 |
|------------|------|
| **架构师 Agent** | 方案设计、技术选型、架构决策 |
| **开发者 Agent** | 代码实现、单元测试 |
| **测试工程师 Agent** | 集成测试、E2E测试、边界条件覆盖 |
| **代码审查者 Agent** | 代码质量、规范检查、最佳实践应用 |

---
### 3.4 开发原则
1.不需要询问 Subagent-Driven (recommended)还是Inline Execution，直接选择Subagent-Driven

2.用中文回复

## 四、代码审查 Checklist

每次提交代码前自查：

- [ ] 数学逻辑正确，无未来函数泄漏
- [ ] 边界条件处理：NaN、Inf、空集、全相同值
- [ ] 关键路径有单元测试覆盖
- [ ] 输入输出契约在 docstring 中声明（@input, @output）
- [ ] 关键步骤有日志输出（使用 loguru）
- [ ] 没有硬编码的魔法数字
- [ ] 没有重复代码，可复用部分已抽取
- [ ] README/文档对应部分已更新
- [ ] 提供了验证证据（测试报告、运行日志、截图等）
- [ ] **数据处理使用 Polars + Parquet**，而非 Pandas + CSV（除非有明确理由）
- [ ] 优先使用 Polars 惰性执行 (`scan_parquet`) 而非立即加载
- [ ] 没有逐行 `apply`，使用 Polars 内置向量化函数
- [ ] Parquet 存储配置正确：Row Group 大小、压缩算法合理

---

## 五、顶级量化公司参考标准

本项目参考以下顶级量化公司实践，详见 [机构级量化开发准则](./nas/doc/5_03_开发规范/15-REF-机构级量化开发准则.md)：

| 公司 | 参考领域 |
|------|----------|
| Two Sigma | 代码规范、Git 提交、测试分层 |
| MSCI Barra | 因子正交化、风险因子模型 |
| BlackRock Aladdin | 风险验证、风控门禁 |
| Northfield | 组合优化、质量标准 |
| Citadel | 执行标准、延迟控制 |

### 5.1 量化正确性检查

| 检查项 | 标准 |
|--------|------|
| 未来函数 | T 日因子不得预测 T 日收益，应为 T 日因子 → T+1 交易 → T+2 收益 |
| 数据泄漏 | 训练/测试集严格分离，时间切片 |
| 涨跌停 | 回测必须剔除涨跌停、停牌、退市股票 |
| IC 显著性 | 同时报告 p 值、t-stat（p < 0.05, t > 2） |

### 5.2 Barra 正交化标准

> **注意**：以下为 Barra 正交化的完整标准，当前 Seagull 实现状态见 [机构级量化开发准则](./nas/doc/5_03_开发规范/15-REF-机构级量化开发准则.md) Section 3.3。

| 步骤 | 要求 | 当前状态 |
|------|------|----------|
| 市值中性化 | WLS 加权最小二乘回归 | ⚠️ 等权 OLS（待升级） |
| 行业控制 | 申万一级行业哑变量 | ❌ 缺失（待实现） |
| 风格正交 | Beta、Momentum、Volatility、Liquidity、Size | ⚠️ 部分实现 |
| 标准误调整 | Newey-West 调整 | ❌ 缺失（待实现） |

### 5.3 测试分层标准（L1-L4）

| 层级 | 核心验证内容 |
|------|--------------|
| L1 单元测试 | 字段对齐、滞后检查、NaN/Inf、数学稳定性 |
| L2 集成测试 | Pipeline E2E、数据血缘、横截面分布、Rank IC 连续性 |
| L3 OOS 测试 | Walk Forward、Regime 稳定性、参数鲁棒性 |
| L4 Pre-live | 回测 vs 仿真一致、Shadow Mode、延迟检查 |

---
> Source: [StarlightDamian/seagull](https://github.com/StarlightDamian/seagull) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
