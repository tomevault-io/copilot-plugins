## polardbx-sql

> SQL 优化与执行计划生成模块，将 SQL AST（SqlNode）转换为分布式执行计划（RelNode），通过 RBO + CBO 生成最优计划，并提供 SPM 计划缓存与基线管理。

# polardbx-optimizer

SQL 优化与执行计划生成模块，将 SQL AST（SqlNode）转换为分布式执行计划（RelNode），通过 RBO + CBO 生成最优计划，并提供 SPM 计划缓存与基线管理。

## Key Classes

| Class | Role |
|-------|------|
| Planner | 优化器核心入口，SQL → RelNode → RBO/CBO → ExecutionPlan |
| PlannerContext | 单次优化的动态上下文（参数值、SPM 基线、统计追踪） |
| OptimizerContext | 全局上下文单例（SchemaManager、RuleManager、Partitioner） |
| ExecutionContext | 请求级执行上下文（事务、连接、参数、内存池） |
| PlanManager | SPM 计划管理器（缓存、基线、计划演化） |

## Optimization Pipeline

```
SQL String
  → [1] Parser: FastsqlParser.parse() → SqlNode
  → [2] Validator: SqlConverter.validate()
  → [3] SqlToRel: SqlConverter.toRel() → Logical RelNode
  → [4] ToDrdsRel: ToDrdsRelVisitor → Drds RelNode
  → [5] RBO: HepPlanner (子查询去嵌套、谓词下推、投影裁剪)
  → [6] CBO: VolcanoPlanner (Join 重排序、索引选择、聚合下推)
  → [7] 执行模式选择: SMP / MPP / Columnar
  → [8] Final RBO + constructExecutionPlan()
  → [9] SPM 缓存/基线匹配
```

## Do NOT

- **执行阶段禁止访问 RelMetadataQuery**：SPM 共享 Cluster 对象，优化结束后 mq 失效，执行时访问会 NPE
- **执行计划必须为静态对象**：SPM 缓存的 RelNode 不可变，动态属性放 ExecutionContext/PlannerContext
- **禁止在执行阶段修改 SqlNode/RelNode**：会导致 Union Order By 别名丢失等难查 bug
- **OptimizerContext 中不要放静态属性或组件对象**：应从 gms 模块动态获取

## Context Layering

| Context | Scope | Example |
|---------|-------|---------|
| OptimizerContext | 全局（per schema） | SchemaManager, RuleManager |
| PlannerContext | 单次优化 | params, baselineInfo |
| ExecutionContext | 单次执行 | 事务状态, traceId, 内存池 |

## Optimizer UT Mechanism

### 类继承结构

```
BasePlannerTest (核心基类，YML解析/环境初始化/plan比较)
  ↑
PlanTestCommon
  ↑
ParameterizedTestCommon (SQL参数化 + Hint处理)
  ↑
具体测试类 (如 PaginationForceTest，只写构造器和loadSqls)
```

### YML 四段式结构 (以 PaginationForceTest.yml 为例)

```yaml
SQL:          # 测试用例列表，每条含 sql + plan(expected)
  - sql: |
      select ... FROM ... WHERE ...
    plan: |
      mergesort(...)
      logicalview(...)
DDL:          # 建表语句 → 内存 SchemaManager
  table_name:
    CREATE TABLE ... PARTITION BY ...
STATISTICS:   # Mock 统计信息 → MockStatisticDatasource → StatisticManager
  table_name:
    100       # 行数
CONFIG:       # 连接级参数 → executionContext.extraCmds，覆盖 ConnectionProperties 默认值
  ENABLE_AUTO_FORCE_INDEX:
    true
```

### 执行流程

```
类初始化 (initTestEnv, 每个测试类一次)
  ├─ loadConfig()        → configMaps (CONFIG段)
  ├─ loadDdl()           → ddlMaps (DDL段)
  ├─ loadStatistic()     → MockStatisticDatasource (STATISTICS段)
  └─ prepareSchemaByDdl() → 解析DDL → TableMeta + PartitionInfo → 内存SchemaManager

每条SQL执行 (testSql → doPlanTest → execSqlAndVerifyPlan)
  ├─ getPlan(testSql)
  │   ├─ parameterize()              字面量 → ?(RexDynamicParam)
  │   ├─ FastsqlParser.parse()       SqlNode
  │   ├─ HintPlanner.collectAndPreExecute()  处理 /*+TDDL:cmd_extra(...)*/
  │   ├─ Planner.getPlan()           RBO + CBO → RelNode
  │   ├─ PostPlanner.optimize()      后处理
  │   └─ RelUtils.toString()         RelNode → plan字符串
  │
  └─ 比较 (归一化后字符串匹配)
      ├─ trim + 统一\r\n + 转小写
      ├─ 去除 _$abcd 哈希后缀
      └─ $corN → $cor 归一化
```

### 关键机制

1. **全内存 Mock**：不连真实数据库。DDL 在内存中构建 TableMeta/PartitionInfo/StatisticManager，优化器在 mock 数据上运行。
2. **参数化**：SQL 中字面量被替换为 `?`，plan 输出显示为 `?N`。
3. **Hint 处理**：`/*+TDDL:cmd_extra(KEY=VALUE)*/` 被 HintPlanner 解析后写入 `executionContext.getExtraCmds()`，覆盖 CONFIG 默认值。
4. **fixFlag 机制**（BasePlannerTest line 165）：
   - `fixFlag = false`（默认）：执行 assertEquals，比较 actual vs expected plan
   - `fixFlag = true`：跳过断言，收集 actual plan，全部用例跑完后通过 `printFullCases` → `insertExpectPlan` 将 actual plan 回写到 YML 内容，最终写入 `new File(caseName)`（即模块根目录下的 YML 文件名）。不是写回 `src/test/resources/` 源文件，需手动复制到对应包路径。

### Maven 跑 UT 的依赖要求

必须先 install 所有上游模块到本地 `~/.m2/repository`，否则 Maven 从远程拉旧版 SNAPSHOT JAR，行为与 IntelliJ 不一致：

```bash
mvn install -pl polardbx-common,polardbx-calcite,polardbx-parser,polardbx-gms,polardbx-rpc,polardbx-rule,polardbx-net -DskipTests -q
mvn test -pl polardbx-optimizer -Dtest=<TestClass>
```

---
> Source: [polardb/polardbx-sql](https://github.com/polardb/polardbx-sql) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
