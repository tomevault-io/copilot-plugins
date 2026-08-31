## wyg-dolai

> DolphinScheduler 项目知识库：把 Dolphin 元数据（项目/工作流/任务/依赖/审计）与资源 SQL 单向同步到自有知识库 `dolphin_meta`，提供向量检索 + LLM 问答 + 表级/任务级血缘 + 表结构字段问答 + SQL 自动修复。独立项目，不复用 LanchainL / hive_ai。

# dolp-ai 项目记忆（AGENTS.md）

## 项目定位
DolphinScheduler 项目知识库：把 Dolphin 元数据（项目/工作流/任务/依赖/审计）与资源 SQL 单向同步到自有知识库 `dolphin_meta`，提供向量检索 + LLM 问答 + 表级/任务级血缘 + 表结构字段问答 + SQL 自动修复。独立项目，不复用 LanchainL / hive_ai。

## 技术栈与版本
- Java 17 + Spring Boot 3.3.5（Web + JDBC）；LangChain4j 1.1.0（DeepSeek 对话 + 本地量化 Embedding all-MiniLM-L6-v2，384 维）
- Flink CDC 3.1.1（`flink-connector-mysql-cdc`）+ Flink 1.18.1（需显式带 `flink-table-common`/`flink-connector-base`/`flink-streaming-java`/`flink-clients`，否则运行期 `ClassNotFoundException`）
- Milvus SDK 2.4.0；MySQL 8（mysql-connector-j 8.4.0）；Druid 1.1.16（SQL 血缘解析，支持 Hive 方言）

## 数据源与服务
| 项 | 值 |
|---|---|
| 知识库目标库 | MySQL `dolphin_meta`（本地 127.0.0.1:3306，root/your-password；环境变量 `DOLP_META_URL`/`DOLP_META_USER`/`DOLP_META_PASSWORD` 可覆盖） |
| Dolphin 源库 | MySQL `dolphinscheduler`（生产 bigdata3，dolphinscheduler 账号；CDC `server-id` 5400-5404；`startup` 默认 `initial`） |
| StarRocks 镜像 | `dolphin_sr`（生产 FE `starrockscluster-fe-service.starrocks.svc.cluster.local:9030`，账号 wyg_flink） |
| Milvus 集合 | `dolphin_doc`（384 维 COSINE，127.0.0.1:19530） |
| LLM | AI Hub `https://aihub.i.wygtech.com/v1`，模型 `deepseek-v4-flash`，temperature 0.2，300s 超时 |
| Web 端口 | 8082（前端 `static/index.html`：问答 + 反馈 + 修复中心） |

## 知识库表（dolphin_meta，`SchemaInitializer` 启动自动建库建表）
- `meta_project` / `meta_workflow` / `meta_task` / `meta_task_relation`（`pre_task_code=0` 为 DAG 起点）/ `meta_audit`（只增不改不删）/ `meta_resource`（SQL 内容 + 目标/源表血缘）
- P1+：`meta_task_resource`（任务↔资源关联）、`meta_table_lineage`（Druid 表级血缘）、`meta_column`（解析 CREATE TABLE 得到的表结构字段，供“XX 表有哪些字段”问答）、`meta_repair`（修复审计）

## Job 入口（`DolpAiApplication --job=xxx`）
| job | 说明 |
|---|---|
| `cdc` | Flink CDC 常驻同步（**必须用 `bin/run-cdc.sh`（java -cp 直跑），`mvn exec:java` 会报 `ClassNotFoundException: ExecutionConfig`**） |
| `scan` | 扫描资源 SQL：本地默认 `/home/cyh/work/wyg/dolp_ai/sql`（跳过 docker cp）；生产容器 `dolphinscheduler-standalone`:`/dolphinscheduler/default/resources` |
| `build` | 读 `dolphin_meta` → 切片 → Embedding → 重建 Milvus `dolphin_doc`（幂等） |
| `serve`（默认） | 启动 Web 问答服务 8082 |
| `ask` | 命令行问答验证 |
| `repair` | 失败任务 → LLM 诊断 → 自动修复 SQL 写回 Dolphin 资源 → 重配并触发重跑（`meta_repair` 审计） |
| `sr-sync` | 生产不可直连 Dolphin MySQL 时，从 StarRocks 镜像库同步元数据到 `dolphin_meta`（幂等 upsert + 删除收敛） |

## 常用命令
```bash
cd /home/cyh/work/wyg/dolp_ai
mvn -q compile -DskipTests
mvn -q test                       # 58 个单测（解析器/血缘 BFS/SR 同步/修复/意图/切片/表结构）
./bin/run-cdc.sh                  # CDC 常驻（另开终端）
mvn -q exec:java -Dexec.mainClass=com.wyg.dolpai.DolpAiApplication -Dexec.args="--job=scan"
mvn -q exec:java -Dexec.mainClass=com.wyg.dolpai.DolpAiApplication -Dexec.args="--job=build"
mvn -q spring-boot:run            # 8082
curl -s http://localhost:8082/api/health
curl -X POST http://localhost:8082/api/ask -H "Content-Type: application/json" \
  -d '{"question":"01-test 工作流包含哪些任务，任务之间是什么依赖关系？"}'
```

## REST API
- `POST /api/ask`（RAG 问答；血缘问题自动附带 `LINEAGE_TREE` 证据；“XX 表有哪些字段”命中 `meta_column` 直接直答）
- `GET /api/lineage/tree`（表级 BFS 血缘）、`GET /api/task-lineage/tree`（任务级 BFS）、`GET /api/tasks/{code}/resources`
- `POST /api/sync/sr`（SR 同步）、`POST /api/repair`（自动修复）、`FeedbackController`（提问反馈）
- `GET /api/health`

## 代码结构
```
src/main/java/com/wyg/dolpai/
├── DolpAiApplication.java  # 主类 + --job 分发
├── config/                 # DolpProperties / SchemaInitializer / MetaTables / JobDispatcher / JobRunner / StartupInitializer
├── meta/                   # MetaDao / MetaRows / LineageService（BFS）
├── cdc/                    # CdcSyncJob / CdcRecordDeserializer / MetaSink / CdcMetaWriter / JdbcUrlParser
├── resource/               # ResourceScanJob / SqlTableRefParser / HiveLineageParser / TaskResourceResolver
├── index/                  # BuildJob / ChunkBuilder / EmbeddingService / MilvusService
├── ask/                    # AskService / AskController / AskCliJob / FeedbackController
├── repair/                 # RepairJob / RepairService / RepairController
└── sync/                   # SrSyncJob / SrTableReader / MySqlToSrSyncJob / MySqlToSrMirrorMain / SyncController / SyncScope
```
测试：`src/test/java/com/wyg/dolpai/` 下 9 个测试类（AskServiceIntent / ChunkBuilder / LineageService / RepairService / HiveLineageParser / SqlTableRefParser / TaskResourceResolver / SrSyncJob / SrTableReader）

## tools/sr-mirror（Java 8 独立工具）
- 生产 JDK 1.8 跑不了主项目（Spring Boot 3.3.5 需 Java 17+），抽成独立 main：只依赖 mysql-connector-j 8.0.33，单 jar 可直接部署
- 逻辑与 `MySqlToSrSyncJob` 一致：源库 5 张表全量 → SR Unique Key 模型自动建表 + 批量 INSERT（同键整行替换）→ 删除收敛（源为空时不删）
- 构建/运行：`cd tools/sr-mirror && mvn -q package` → `java -jar target/sr-mirror-java8.jar [--workflow=<code|名称>]`（连接信息用 `DOLP_*` 环境变量覆盖，默认即生产）
- 产物需 Java 8 字节码（major 52），构建脚本 `build.sh`

## 资源 SQL（sql/ 目录，约 273 个文件）
- 生产数仓 Hive SQL 按分层组织：ods / dwd / dwm / dws / dim / app，含企业/人员/招投标/政策文档等主题
- 本地扫描直接用该目录，可做血缘解析与修复验证的素材

## 文档（docs/）
00 总体架构 / 01 配置与初始化 / 02 CDC 同步 / 03 资源扫描 / 04 向量构建 / 05 RAG 问答 / 06 运行手册 / 07 前端页面 / 08 P1 血缘增强 / 09 检索增强 / 10 SR 数据接入 / 11 SQL 自动修复

## 注意事项
- 本目录不是 git 仓库（无 .git）
- 沙箱 bwrap 偶发启动失败（`loopback: Failed RTM_NEWADDR`），命令失败时需提权执行
- CDC 首次 `initial` 会全量快照；`server-id` 用 5400-5404 区间
- 本机开发环境全局记忆（MySQL/Milvus/LLM 等）见 Codex 全局 AGENTS.md

---
> Source: [ChenYunHey/wyg_dolai](https://github.com/ChenYunHey/wyg_dolai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
