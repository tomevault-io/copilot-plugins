## agentic-rag-platform

> <!-- Shared Codex/Kimi guidance converted from the Claude rules for this repository. -->

<!-- Shared Codex/Kimi guidance converted from the Claude rules for this repository. -->

## 语言与基础协作

面向用户的自然语言输出一律使用简体中文。代码、命令、路径、标识符、API/字段/变量名、错误日志和 commit message 保持原文或仓库既有风格。

如果需要理解或定位代码，先看对应功能目录的 `README.md`。如果仓库根目录存在 `.codegraph/`，在 grep/find/read 文件前优先使用 CodeGraph；不存在时直接跳过。

`AGENTS.md` 是 Codex 与 Kimi 共用的项目规则入口，已将 `.claude/rules/*.md` 中会影响实施的要点压缩到本文件，以避免超过两端默认的指令大小上限。`.claude/rules/*.md` 与根 `CLAUDE.md` 保留为 Claude Code 的详细兼容指引，前端细则在 `web/CLAUDE.md`。修改持久规则时，同步更新对应细分文件与本文件的精简表述，不要只改其中一份。项目共享 skill 放在 `.agents/skills/`，不要再在 `.claude/skills/` 复制同名 skill。

## 项目概览

`agentic-rag-platform` 是 ES Agentic RAG 控制台，覆盖索引建模、Excel/数据库/API/推送接入、term/match/向量/混合检索、ES 到 DB 降级、自然语言查询计划、MCP 暴露、逐索引 RBAC 与行级权限、模型接入和全链路审计。

本仓库当前保留 `common`/`auth`/`captcha`/`system`/`file`/`audit`/`gateway` 基座，以及 `rag`、`agent` 业务模块；`task`、`pay`、`open` 三条产品线代码已移除，不再回流。

后端是 Kotlin + Spring Cloud 微服务，Gradle 多模块，Nacos 服务发现，MySQL + Redis + RabbitMQ + MinIO + Elasticsearch。前端在 `web/`，是 Vue 3 管理端；改动 `web/` 前必须读 `web/CLAUDE.md`。

## 文档导航

定位功能源码时先查这些文档：

- `gateway/README.md`：路由、真实 IP、内部路径拦截、OpenAPI 聚合。
- `modules/common/README.md`：鉴权注解、JWT、Jimmer 基类、MVC 基础设施。
- `modules/auth/README.md`：登录/注册、JWT 签发与刷新、登录日志。
- `modules/captcha/README.md`：图形验证码、reCAPTCHA、`@RequireCaptcha`。
- `modules/system/README.md`：用户、角色、权限、菜单、通知、登录日志、邮件。
- `modules/file/README.md`：MinIO 预签名上传/下载、秒传去重、业务配置。
- `modules/audit/README.md`：`AuditLogger`、MQ 消费落库、字段级 diff。
- `modules/rag/README.md`：索引建模、检索、内置 MCP、权限矩阵、embedding、查询计划、导入长任务、非结构化文件接入。
- `docs/agent/README.md`：Agent 平台设计/开发文档索引，非源码位置映射。
- `web/README.md`：前端页面功能地图；规范见 `web/CLAUDE.md`。
- `deploy/README.md`：docker-compose、环境变量、首次部署。

## 技术栈与模块

版本由 `gradle/libs.versions.toml` 管理：Kotlin 2.2.0（JVM 21）、Spring Boot 3.5.0、Spring Cloud 2025.0.1、Spring Cloud Alibaba 2025.0.0.0、Jimmer 0.10.6、MySQL Connector 9.6.0、Flyway 11.15.0、MinIO 8.6.0、springdoc-openapi 2.7.0、java-jwt 4.4.0。

所有后端代码包名根为 `v.boot`，Gradle group 为 `v.boot`。`modules/common` 和 `*-api` 是纯库；`auth`、`captcha`、`system-impl`、`file-impl`、`audit-impl`、`rag`、`agent` 和 `gateway` 是可运行 Spring Boot 应用。

服务端口：gateway 8000（管理端口 8001）、system 10000、file 10001、auth 10002、captcha 10003、agent 10005、audit 10008、rag 10009。基础设施端口见 `deploy/docker-compose.yml`，均绑定本机或 compose 内网。

## 构建与测试

常用命令：

```bash
./gradlew build
./gradlew test
./gradlew :modules:rag:test
./gradlew :modules:rag:test --tests "v.boot.queryplan.QueryPlanCompilerTest"
./gradlew :modules:system:system-impl:build
./gradlew :modules:rag:build
./gradlew :gateway:build
```

可运行模块的 `bootJar` 产物统一命名为 `app.jar`。新增 Jimmer 实体后如 IDE 报红，先跑对应模块 `kspKotlin` 或整体 build。新增查询编译规则、权限判定分支、ES DSL 形状时补对应单测。

## 请求链路与服务边界

网关路由统一为 `/{服务名}/**` 加 `StripPrefix=1` 转发到 `lb://{服务名}`；新增服务需同步更新网关路由。`/internal/**` 和 `/actuator/**` 外部访问由网关 404，内部 Feign 接口路径必须以 `/internal/` 开头。

跨模块调用走 `*-api` 子模块 Feign 接口。若同一下游服务有多个客户端接口，必须为每个 `@FeignClient` 显式设置唯一 `contextId`，否则 OpenFeign bean 名冲突。

基座服务 `common`/`auth`/`captcha`/`system`/`file`/`audit`/`gateway` 不得依赖产品线模块；产品依赖基座正常。需要产品数据时由前端编排或产品侧主动登记，不要让基座 Feign 拉产品。

平台内部服务之间不签调用方 JWT，内部身份通过 `X-Mcp-Caller-User-Id` 等头在 `/internal/**` 信任边界内传递。

MCP 入站支持 OAuth Bearer、接入方注册表私有 header + 自签 JWT、内部通道 `/rag/internal/mcp`。出站连接器按 `identity_mode` 选择 `none`、`header`、`jwt` 或 `oauth`；禁止 token passthrough，资源令牌不得当登录态。

认证鉴权由 `PermInterceptor` 按方法级优先顺序处理：`@Public`、`@SuperAdmin`、`@HasRole`、`@HasPerm`、`@Authenticated`。用户上下文用 `LoginUtils` 获取。

微服务中禁止自行解析 `X-Forwarded-For` 或 `RemoteAddr`，统一调用 `ServletContextUtil.getRequestIp()`，网关已经写入 `X-Real-Client-IP`。

HTTP 响应默认由 `GlobalResponseBodyAdvice` 包装为 `ResultData(code, msg, data)`。裸文本接口在共享 `-api` 接口方法上声明 `produces = [MediaType.TEXT_PLAIN_VALUE]`。Feign 错误由全局 `ErrorDecoder` 将下游 `ResultData` 还原为 `BusinessException`。

## 异步、配置与运维

MQ：审计经 `audit.topic` 和死信队列落库；登录日志由 auth 发、system 消费；管理员告警用 `platform.admin.alert.exchange`；定向站内信用 `platform.user.notify.exchange`，`kind` 用字符串。

调度：Spring 默认 `TaskScheduler` 单线程，对延迟敏感的周期任务不要挂普通 `@Scheduled`。多副本下 `@Scheduled` 每副本都会触发，Redis 锁不足以保证正确性，重复执行有副作用的任务必须用 DB 条件更新/CAS 做最终保证。

配置：各模块只有 `application.yml`，环境相关值用 `${ENV_VAR}` 注入，禁止新增 `application-local.yml` 或 `application-prod.yml`。真实 `.env*` 不入库。

Flyway：每个 impl 模块管理自己的迁移，新增表结构必须加新脚本，禁止改已发布迁移。命名为 `V<yyyyMMdd>_<三位序号>__<描述>.sql`，描述前必须是双下划线。

日志使用各服务自带 `logback-spring.xml`，带 `[requestId]`，滚动写 `logs/{服务名}.log` 和 `logs/err-{服务名}.log`。

新增服务需同步接入 Actuator + Prometheus 配置，并更新 `deploy/docker/prometheus.yml` 抓取目标。

## 后端编码规范

用户已选定的方案可直接写入，无需二次询问。分析第三方依赖源码时先找 Gradle 缓存中的 `*-sources.jar`，没有源码再 `jar tf` + `javap -c -p`，不要以无法访问依赖源码为由拒绝。

Kotlin `data class` 用类型可空性表达必填/可选，禁止在非空字段上写 `@field:NotNull`；字符串非空用 `@field:NotBlank`。

Feign 接口入参/出参必须使用 Kotlin `data class`，禁止使用 Jimmer `@Entity` 接口或 KSP 生成 DTO。

ByteArray 转十六进制用 `toHexString()`，禁止 `joinToString` 手动格式化。Logger 用 `LoggerFactory.getLogger(javaClass)`，禁止硬编码类名。

Controller 路径必须与前端 `BaseApi` 方法对应，使用显式动作路径：`POST /list`、`POST /page`、`POST /add`、`POST /batchAdd`、`PUT /update`、`DELETE /{id}`、批量删除 `DELETE` 无后缀、`PUT /resetOrder`、`POST /getEntityList`。禁止纯 RESTful 风格。内部 Feign 接口不受此表约束，但必须 `/internal/` 开头。

Spring Boot 3.x 下 `@SpringBootApplication` 会扫描同包及子包的 `@ConfigurationProperties`，通常无需额外 `@ConfigurationPropertiesScan` 或 `@EnableConfigurationProperties`。若配置 data class 所有字段都有默认值，需用 `var` 或去掉默认值以启用正确绑定。

Jimmer 实体是 Kotlin 接口，继承 `BaseEntity`、`BaseUpdateEntity` 等基础类型。创建实体实例用 KSP 生成的同名顶层函数 `Entity { this.field = value }`，禁止 `org.babyfish.jimmer.kt.new<T> { }` 泛型写法。

禁止使用 Jimmer DTO；控制器入参/出参和 Service 层传输用 Kotlin `data class`。

Repository 查询中 `page` 接收 `PageInput<QueryClass>`，`list` 接收 `QueryInput<QueryClass>`，共享过滤逻辑抽为 `private fun EntityQuery<E>.applyFilter(condition: QueryClass)`，排序统一 `applyOrderBy(input.orderBy)`。固定维度参数作为独立形参传入，不放进 Query 类。

Jimmer 查询字段和操作符必须显式导入，如 `v.boot.entity.createdAt`、`org.babyfish.jimmer.sql.kt.ast.expression.eq`。缺字段或操作符 import 会导致类型推断或 unresolved reference 问题。

项目枚举统一实现 `BasicEnum`，通过 `@EnumKey` 和 ScalarProvider 做 JSON/DB 映射，新增枚举不要裸用 Kotlin enum。

Kotlin 字符串含多个 `$` 时使用多美元符字面量 `$$"..."` 或 `$$"""..."""`，禁止逐个 `\$`、字符串拼接或 `${'$'}` 绕开。

## 前端规则

改动 `web/` 前先读 `web/CLAUDE.md`，并优先沿用其现行模式。

前端技术栈：Vue 3.5 `<script setup>` + TypeScript，Vite+ (`vp`) + Vite 8/Rolldown/Oxc，shadcn-vue（reka-ui + Tailwind v4），TanStack Table 封装 `@/components/data-table`，Pinia，Vue Router hash 模式，图标 `@lucide/vue`。

颜色只用 `src/styles/tailwind.css` 中定义的语义 token：`primary`、`info`、`success`、`warning`、`destructive`、`muted`、`accent`、`secondary` 及其 foreground。没有 `danger` token，危险色用 `destructive`。不要引入 `--chart-*`。

Element Plus、AG Grid、VTable 已清退。看到 `el-*`、`ElMessage`、`ElMessageBox`、`gridOptions`、`headerSlot`、`TableColumnDefinition`、`cellRender`、`useTableHook` 等视为过时写法。

常用命令在 `web/` 下执行：`pnpm install`、`npm run dev`、`npm run front`、`npm run build`、`npm run lint`、`npm run fmt`、`npm run check`、`npm run typecheck`。类型门禁走本地 `npm run typecheck`，不要用 `npx vue-tsc`。不要把 `vite.config.ts` 放回 `tsconfig.json` 的 `include`。

API 层用 `src/api/request.ts` 拆 `ResultData`，资源 API 继承 `src/api/base-api.ts` 的 `BaseApi`。新增后端接口时同步加前端 API 类。

新列表页统一用 `@/components/data-table`。列定义用 `createColumnHelper<T>()` 和工厂函数，数组后置 `as ColumnDef<T>[]` 收口。优先全服务端分页/排序/筛选，绑定 `row-count` 进入服务端模式；筛选放表头筛选行，不要额外做一片搜索表单。

操作列用 `TableActions`，删除等危险操作用 `type: "danger"` 和 `confirm`。行号用 `features.rowNumbers`，不要手写 index 列。

CRUD 弹窗用 `openFormModal`，提示用 `notify`，确认用 `confirm` 或 `TableActions.confirm`。不存在 `useTableHook` / `grid.hook.ts` 这类 CRUD 组合式。

Vue 双向绑定优先 `defineModel()`。`<Input type="number">` 的 v-model 可能返回 `number` 或空字符串，字段类型写 `string | number`，取值先 `String(x).trim()`。

真实页面验证 UI 时按 `.agents/skills/verify/SKILL.md` 的流程：先探测已有 Vite dev server 端口，能复用就不新起；真实登录优先，后端不可用时用离线 API mock；临时脚本/截图写 scratchpad，不放进 `web/`。复用用户已开的 dev server 时不要停掉。

## 部署与安全

本地/单机 compose 在 `deploy/docker-compose.yml`。首次部署按 `deploy/README.md` 从模板复制 `.env*` 并配置真实值；真实密码、JWT 密钥、S3 凭据等只走不入库的 `.env*`。

单 Dockerfile 多阶段构建，服务 target 输出按 `app.jar` 分层运行。`deploy/deploy.sh` 支持增量镜像部署和 `--sync-config`。

Gitea Actions：push 到 `dev` 自动检测受影响服务部署测试机；`main` 上 `workflow_dispatch` 手动部署正式机。变更传播规则见 `.gitea/workflows/deploy.yml`。

新增内部接口必须以 `/internal/` 开头。新增管理端写操作按既有模式接入 `AuditLogger`。

## Git 协作

建议每个 clone 开启：

```bash
git config --local rerere.enabled true && git config --local rerere.autoupdate true
git config --local merge.conflictStyle zdiff3
```

长期分支日常对齐主干优先 `merge main`，收尾需要线性历史时再 rebase。新增规范先放入已有主题文件；确实不属于任何现有文件才新建，并同步更新根 `CLAUDE.md` 导航/导入清单与根 `AGENTS.md` 的共享摘要。

---
> Source: [L1yp/agentic-rag-platform](https://github.com/L1yp/agentic-rag-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-05 -->
