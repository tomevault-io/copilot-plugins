## forge-admin

> 本文件为 Claude Code（claude.ai/code）在本仓库中开发时提供指导。

# CLAUDE.md

本文件为 Claude Code（claude.ai/code）在本仓库中开发时提供指导。

## 仓库概述

Forge Admin 是一套企业级中后台框架：前端 **Vue 3 + Naive UI**，后端 **Java 17 + Spring Boot 3** 微内核插件化架构。内置 RBAC + 多租户 + 数据权限、AI 代码生成、Flowable 工作流、AI 大屏报表、H5/uni-app 移动端，以及较新的 **统一能力开放平台（capability）** 与 **企业微信（WeCom）协作** 能力。

后端位于 **`forge-server/`**（不是 `forge/` —— README.md 与 AGENTS.md 仍用过时的 `forge/` 路径；其中 AGENTS.md 第 16 行写了 `forge-server/`，但下方完整模块树、启动命令、配置路径仍全是 `forge/`，前后矛盾）。前端共**三个端**：`forge-admin-ui/`（管理端，dev :3000）、`forge-report-ui/`（AI 大屏设计器，dev :3021）、`forge-h5-ui/`（uni-app 移动端 H5，dev :3001）。文档：`forge-docs/`（VitePress）。各端位置、端口、代理细节见下方"常用命令 → 前端"。

## 常用命令

### 后端（`forge-server/`）
```bash
cd forge-server && mvn clean install -DskipTests   # 全量构建（快速）
cd forge-server && mvn test -P enable-tests        # 运行测试（默认跳过测试！）
cd forge-server/forge-admin-server && mvn spring-boot:run        # 主后台服务，端口 8580
cd forge-server/forge-report-server && mvn spring-boot:run       # 大屏服务，端口 8581
cd forge-server/forge-app-server && mvn spring-boot:run          # App 接口服务，端口 8583
cd forge-server/forge-flow/forge-flow-server && mvn spring-boot:run  # 独立流程服务，端口 8081
```
Maven Profile（默认 `dev`）：`local`、`dev`、`prod`。测试执行受 `enable-tests` Profile 门控 —— 直接 `mvn test` 会跳过测试。构建需要 Java 17、MySQL 8+、Redis 6+。登录账号：`admin` / `123456`。

**后端各服务端口与前端代理的对应关系**：`forge-admin-server`(8580) ↔ 管理端 `/dev-api`、`forge-app-server`(8583) ↔ H5 `/dev-api`、`forge-report-server`(8581) ↔ 大屏 `/forge-report-api`、`forge-flow-server`(8081) ↔ 管理端与 H5 的 `/api/flow`。`forge-business`、`forge-flow-client` 为业务/客户端模块，非独立服务。

本地配置：把 `forge-server/forge-admin-server/src/main/resources/application-dev.example.yml` 复制为 `application-dev.yml`（已 gitignore），填入 MySQL、Redis、AI 供应商等配置。

### 前端（三个端）

| 端 | 目录 | 技术栈 | 配置文件 | dev 端口 | dev 访问地址 | dev 命令 |
|----|------|--------|----------|----------|--------------|----------|
| **管理端** | `forge-admin-ui/` | Vue 3 + Naive UI + Vite | `vite.config.js` + `.env.development` | `3000`（`VITE_HTTP_PORT`） | `http://localhost:3000/` | `pnpm dev` |
| **大屏设计器** | `forge-report-ui/` | Vue 3 + Vite（TS） | `vite.config.ts` | `3021`（硬编码） | `http://localhost:3021/forge-report` | `npm run dev` |
| **H5 移动端** | `forge-h5-ui/` | uni-app 3 + Vue 3 | `vite.config.js` + `.env.development` | `3001`（`VITE_HTTP_PORT`） | `http://localhost:3001/` | `pnpm dev:h5` |

**管理端 `forge-admin-ui`**：接口前缀 `/dev-api`。代理规则（`vite.config.js`）：
- `/dev-api/api/flow`、`/dev-api/api/workspace` → 流程服务 `8081`
- 其余 `/dev-api` → 主后台服务 `http://127.0.0.1:8580/`
- `/ws` → WebSocket 转发到主后台服务

**大屏设计器 `forge-report-ui`**：接口前缀 `/forge-report-api` → `http://localhost:8581`（`vite.config.ts`）；另有 `/llm` 直连阿里百炼 OpenAI 兼容接口的 SSE 代理（禁用上游 gzip 避免缓冲流式响应）。

**H5 `forge-h5-ui`**：接口前缀 `/dev-api`。代理规则（`vite.config.js` + `.env.development`）：
- `VITE_HTTP_PROXY_TARGET=http://127.0.0.1:8583/`（**app-server**，不是 8580）
- `VITE_FLOW_PROXY_TARGET=http://127.0.0.1:8081/`
- `/dev-api/api/flow`、`/dev-api/ai/business/flow` → flow-server（8081），其余 `/dev-api` → app-server（8583）

改各端网络行为前，先读对应项目的 `vite.config.*` 与 `.env*`。要求 `Node >= 20.19`、`pnpm >= 8`。构建：管理端 `pnpm build`（`NODE_OPTIONS=--max-old-space-size=40961`）、大屏 `npm run build`、H5 `pnpm build:h5`；测试仅管理端有（`pnpm test`，Vitest）。

### 数据库 / Flyway
Flyway 迁移只在 **`forge-admin-server` 启动时**执行（`forge-report-server` 单独启动不会执行）。扫描位置默认 `filesystem:./db/migration,filesystem:../db/migration,filesystem:forge-server/db/migration`；可用 `FORGE_FLYWAY_LOCATIONS` / `FORGE_FLYWAY_ENABLED` 覆盖。

- `forge-server/db/migration/` — 版本化迁移脚本 `V<版本>__<小写蛇形描述>.sql`（版本必须大于 `1.0.0`、单调递增；已执行脚本禁止修改 —— 修正时新增下一个版本脚本）。
- `forge-server/db/seed/required/R__*.sql` — 系统必需初始化数据；`forge-server/db/seed/demo/D__*.sql` — 演示数据（默认不导入）；`forge-server/db/seed/optional/O__*.sql` — 可选模块数据。
- SQL 必须幂等或有防重复保护（`CREATE TABLE IF NOT EXISTS`、`INSERT ... SELECT ... WHERE NOT EXISTS`、新增列/索引前查 `information_schema`）。业务内置数据 `tenant_id` 必须为 `1`（禁止 `0`）。`sys_resource` / `sys_role_resource` 等权限资源脚本必须做 `NOT EXISTS` 防重复。

## 架构

### 后端分层
`forge-server/` 是 Maven 多模块工程，分三层：

1. **`forge-framework/forge-starter-parent/`** — 23 个技术 Starter（纯技术能力，无业务逻辑）：`forge-starter-core`（统一响应 `RespInfo` 位于 `.../starter/core/domain/RespInfo.java`，全局异常 `GlobalExceptionHandler` 位于 `.../starter/core/exception/`，以及 `@ApiEncrypt`/`@ApiDecrypt`/`@OperationLog` 注解）、`forge-starter-web`（Undertow 嵌入式服务器）、`forge-starter-auth`（Sa-Token）、`forge-starter-orm`（MyBatis-Plus + 动态数据源）、`forge-starter-tenant`（租户拦截，追加 `WHERE tenant_id = ?`）、`forge-starter-datascope`（`DataScopeInterceptor` 位于 `.../datascope/handler/DataScopeInterceptor.java`，配置前缀 `forge.datascope`，`enabled` 默认 `true`）、`forge-starter-crypto`（API 加解密）、`forge-starter-excel`、`forge-starter-file`、`forge-starter-log`（操作日志）、`forge-starter-idempotent`（`@Idempotent`）、`forge-starter-cache`、`forge-starter-config`、`forge-starter-trans`、`forge-starter-social`、`forge-starter-websocket`、`forge-starter-message`、`forge-starter-job`、`forge-starter-api-config`、`forge-starter-openapi-security`、`forge-starter-outbound`、`forge-starter-collaboration`、`forge-starter-id`。

2. **`forge-framework/forge-plugin-parent/`** — 业务插件。当前有 11 个一级聚合模块，其中 `forge-plugin-capability-parent` 再聚合 4 个能力平台子模块；其它包括 `forge-plugin-system`（用户/角色/菜单/部门/租户/字典）、`forge-plugin-generator`（AI 代码生成）、`forge-plugin-job`、`forge-plugin-message`、`forge-plugin-flow`（Flowable）、`forge-plugin-ai`（模型供应商）、`forge-plugin-collaboration`（企业微信协作）、`forge-plugin-mcp`、`forge-plugin-external`、`forge-plugin-data`。

3. **应用服务** —— 聚合插件并对外提供接口：`forge-admin-server`（主应用，基础包 `com.mdframe.forge.admin`）、`forge-report-server`、`forge-app-server`、`forge-flow`、`forge-business`。

各插件/模块内部标准分层：`controller` → `service`/`impl` → `mapper` → `entity`/`dto`/`vo`/`constant`/`listener`。统一基础包为 `com.mdframe.forge`。`forge-dependencies` 是 BOM（在根 `pom.xml` 的 dependencyManagement 中声明）。

### 新增子系统（近期工作）

- **统一能力开放平台（capability）**（`forge-plugin-capability-parent`）：基于能力模型与 SPI（`CapabilitySource`、`CapabilityExecutor`、`CapabilityAuthorizationPolicy`、`CapabilityInvocationObserver`）及运行时 `registry`/`schema`。父模块下固定为 4 个子模块：`forge-plugin-capability-core`（协议无关模型/SPI）、`forge-plugin-capability-platform`（控制面 + Identity + Open Gateway）、`forge-plugin-capability-actions`（Secure Actions + Flow Actions）、`forge-plugin-capability-high-risk-approval`（版本化 KEK 载荷加密）。通过 `application.yml` 中 `forge.capability.*` 配置开关（`FORGE_CAPABILITY_OPEN_GATEWAY_ENABLED`、`FORGE_MCP_ENABLED` 等）。**开放网关本身默认开启**（`FORGE_CAPABILITY_OPEN_GATEWAY_ENABLED:true`）；`identity`、`secure-actions`、`flow-actions` 默认跟随 `FORGE_CAPABILITY_OPEN_GATEWAY_ENABLED || FORGE_MCP_ENABLED`（即默认关闭），`high-risk` 默认关闭且需显式提供版本化 KEK。
- **企业微信协作**（`forge-plugin-collaboration`）：WeCom 集成位于 `.../collaboration/provider/wecom/`（access token、回调加解密、通讯录连接器）。管理端 UI：`forge-admin-ui/src/views/system/collaboration/`（连接、同步、映射、回调事件、投递、问题）。

### 前端（`forge-admin-ui`，管理端）
> 三个前端端点的位置与端口见上方"常用命令 → 前端"对照表：管理端 `forge-admin-ui`（:3000）、大屏设计器 `forge-report-ui`（:3021）、H5 移动端 `forge-h5-ui`（:3001）。以下小节均针对**管理端**。
- **路由自动生成**：`unplugin-vue-router` 扫描 `src/views/`（排除 `components/**`、`api/**`、`workspace/**`），因此 `src/views/xxx/index.vue` 会自动生成路由；动态参数用文件名 `[param]`（如 `job-config.editor.[id].vue`）。守卫逻辑在 `src/router/guards/`（权限、tab、页面标题、页面加载）。
- **API 层**：`@/utils/http`（axios 封装目录，`index.js` 导出 `request`/`noPrefixRequest`/`mockRequest`，`baseURL` = `VITE_REQUEST_PREFIX`；拦截器在 `utils/http/interceptors.js`）、`@/utils/encrypt-request`（`postEncrypt` ↔ 后端 `@ApiDecrypt`）、`@/utils/file`（`getFileUrl(fileId)`）。**注意**：`@/utils/request` 是 AGENTS.md 里的旧路径，已废弃 —— 当前代码只有 `api/system/region.js` 还在用，其余 13+ 处都用 `@/utils/http`。按模块拆分的接口定义在 `src/api/`。
- **状态管理**：Pinia 模块在 `src/store/modules/`（`user`、`permission`、`app`、`tab`、`router`、`tenant`、`auth`）。
- **常用组件**：`AiCrudPage`（位于 `src/components/ai-form/AiCrudPage.vue`，由 api-config + JSON schema 驱动的零代码 CRUD 页面，含 `AiCrudRowExpand`/`AiCrudImportModal`/`AiCrudFlowDetail` 等配套）、`AiForm`/`AiFormItem`、`DictSelect`/`DictTag`（`useDict('<类型>')`）、`RegionTreeSelect`、`AuthImage`（鉴权图片组件；文件按 **fileId** 渲染，不是 URL）、`IconSelector`、`src/components/bpmn/` 与 `flow-designer/` 下的 BPMN 设计器。
- **样式**：UnoCSS 语义化颜色类区分操作 —— `text-primary`（编辑/查看）、`text-info`（详情/统计）、`text-warning`（重置/刷新）、`text-error`（删除/强制下线）、`text-success`（启用/发布/通过）。全局样式在 `src/styles/`。
- **前端测试**：Vitest，spec 与源码同目录（`src/**/__tests__/`），用 `pnpm test` 运行。

## 关键约定（违反会直接出问题）

- **查询 SQL 必须写在 Mapper XML 中**，禁止在 Service 层用 `LambdaQueryWrapper` 构建。`DataScopeInterceptor` 在 `beforeQuery` 里按 `MappedStatement` id（即 `命名空间.方法名`）查 `sys_data_scope_config` 配置并精确改写 SQL（分页 count 查询会去掉 `_mpCount`/`_COUNT` 后缀匹配原方法）。用 Wrapper 构建的 SQL 没有可匹配的 XML 方法，会绕过数据权限/租户过滤。例外：MyBatis-Plus 内置方法（`selectById`、`insert`、`updateById`、`deleteById`）。
- **租户 ID**：业务数据 `tenant_id` 必须为 `1`；`TenantLineInnerInterceptor` 自动追加 `WHERE tenant_id = ?`，`0` 的数据对所有租户不可见。`sys_resource` 不受租户拦截。
- **分页参数**：前端传 `pageNum`/`pageSize`；后端 Controller 必须用 `@RequestParam(defaultValue = "1") Integer pageNum`（不能用 `page`）。主流是 `Integer`（20 处），个别控制器用 `Long pageNum`（2 处）也能工作（有 `PageParamResolver.resolve(page, pageNum)` 兜底解析）。
- **`AiCrudPage` 占位符用冒号 `:id`，禁止花括号 `{id}`** —— 组件只识别 `includes(':id')`。
- **图片/文件展示**：`imageUpload` 存的是 **fileId**；渲染用 `AuthImage`（自动带 Token），禁止直接 `NAvatar` src。下载链接用 `getFileUrl(fileId)`。
- **禁止 Service 互相注入**，跨 Service 协调逻辑上提到 Controller 层。
- **字典禁止硬编码**在前端 options 或标签映射里 —— 必须用 `DictSelect`/`DictTag`/`useDict`。新增内置字典通过 Flyway 迁移写入（带 `NOT EXISTS` 防重复，`tenant_id = 1`）。
- **逻辑删除为默认**：业务/配置/设计态表加 `del_flag`（0 = 未删除；数值主键表用 `@TableLogic(value = "0", delval = "<主键列>")`）。Mapper XML 查询必须显式过滤（`AND del_flag = 0`）。仅框架表/关联重建表/临时表或留存清理任务允许物理删除（需在变更说明中写明理由）。禁止只给实体加 `@TableLogic` 但数据库缺字段。
- **接口风格**：`RespInfo.success(data)` / `RespInfo.error(msg)`；REST 路径 `GET /page`、`GET /:id`、`POST /`、`PUT /`、`DELETE /:id`；敏感接口用 `@ApiDecrypt`/`@ApiEncrypt`。
- **安全红线**：禁止硬编码密钥/AK/SK/数据库密码；日志中禁止打印手机号、身份证、银行卡；返回前端的 API Key/Secret 必须脱敏（保留前4后4，中间 `****`）。`application-dev.yml` / `.env.local` 属本地配置（已 gitignore），通用配置同步到 `application-dev.example.yml` / `.env.example` 并用占位符。
- **必备字段**：业务表必须含 `id`、`tenant_id`、`create_by`、`create_time`、`create_dept`、`update_by`、`update_time`；字符集 `utf8mb4`、引擎 `InnoDB`；金额用 `long` 单位分；时间用 `LocalDateTime`。

## 行政区划查询规则（含 `region_code` 字段的业务表）

虚拟组织 code 以 `ALL` 结尾（如 `150000ALL`），表示"本级 + 下级"。区划数据表为 `sys_region_code`（实体 `SysRegion`，`forge-plugin-system` 的 `entity/SysRegion.java`，`@TableName("sys_region_code")`）。Mapper XML 写法：
```xml
<if test="regionCode != null and regionCode != '' and regionCode.contains('ALL')">
    AND (region_code = REPLACE(#{regionCode},'ALL','')
         OR region_code IN (SELECT code FROM sys_region_code WHERE parent_code = REPLACE(#{regionCode},'ALL','')))
</if>
<if test="regionCode != null and regionCode != '' and !regionCode.contains('ALL')">
    AND region_code = #{regionCode}
</if>
```

## 参考文档

- `README.md` — 产品介绍、架构图、快速开始（注意：命令写的是 `forge/`，真实路径是 `forge-server/`）。
- `AGENTS.md` — 详细约定（本文件总结了其中的关键规则；完整编码规范、安全、自动化测试标准请阅读它）。
- `code-copilot/rules/` — `coding-style.md`、`domain-rules.md`、`security.md`、`project-context.md`、`button-style-guide.md`、`automated-testing-standard.md`。
- `code-copilot/memory/` — `pitfalls.md`（新任务开始时先读）、`decisions.md`、`preferences.md`。
- `.agents/skills/` — 项目级技能：`forge-codegen-crud`（CRUD 代码生成/审查）、`forge-business-flow-development`（流程业务开发）。做 CRUD/流程任务前先读对应 `SKILL.md`。
- `NGINX_CONFIG.md` — 生产环境 Nginx 配置。

## 企业微信 / 协作平台开发说明

WeCom 工作台免登由**后端连接配置**驱动：`sys_social_config.sso_workbench_enabled` + `connection_code`（在协作配置中维护），不在前端环境变量。改动 `forge-plugin-collaboration` 的 provider 行为前，先确认集成绑定的是哪个连接/租户。

---
> Source: [yaomindong1996/forge-admin](https://github.com/yaomindong1996/forge-admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
