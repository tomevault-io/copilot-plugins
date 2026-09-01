## easypoints

> 本项目是“企业微信小程序易积分系统”的生产版骨架，当前目录从需求文档出发，已经搭建出一个 monorepo：

# AGENTS.md - EasyPoints 项目协作说明

## 1. 项目概览

本项目是“企业微信小程序易积分系统”的生产版骨架，当前目录从需求文档出发，已经搭建出一个 monorepo：

- 后端 API：`apps/api`
- PC 管理后台：`apps/admin-web`
- 企业微信小程序/uni-app 员工端：`apps/miniapp`
- 基础设施与部署：`infra`
- 项目文档：`docs`

项目目标是实现企业内部积分管理闭环：后台人工录入积分、员工查看积分明细和备注、员工申诉、积分商城兑换、订单审核核销、数据报表导出、全程操作留痕。

## 2. 当前技术栈

- 后端：Node.js，当前核心 API 使用 Node 原生 `http` 实现，可在没有 npm 的环境下运行和测试。
- 管理后台：Vue 3 + Vite + Element Plus。
- 员工端：uni-app，目标编译到企业微信小程序。
- 数据库：MySQL 8。
- 缓存/队列：Redis，计划用于会话、短期状态和消息重试。
- 部署：Docker Compose + Nginx，默认单服务器部署。

## 3. 关键业务规则

### 积分备注是核心需求

- 所有积分变动都必须和备注绑定。
- 单笔奖惩积分必须填写备注，否则后端拒绝提交。
- 月度批量录分支持统一备注，也支持单个员工备注覆盖统一备注。
- 员工端积分明细必须展示每条流水的完整备注。
- 报表导出需要包含每条流水的备注。

### 积分流水不可篡改

- 后端不提供积分流水删除接口。
- 后端不提供积分流水备注修改接口。
- `point_records` 创建后，`employeeId`、`pointsDelta`、`remark` 等核心字段不可改。
- 如需纠错，必须新增一条反向“冲正流水”，不能修改原流水。
- 数据库 schema 中对 `point_records` 设置了禁止 `UPDATE` 和 `DELETE` 的触发器。

### 权限隔离

- 超级管理员和人事管理员拥有全局权限。
- 部门管理员只能管理本部门员工。
- 员工只能查看自己的积分流水。
- 员工只能针对自己的积分流水提交申诉。

### 商城与订单

- 员工兑换礼品时扣减积分并生成订单。
- 订单驳回或取消时，通过退分流水返还积分。
- 订单状态变化会写入操作日志，并进入消息 outbox。

## 4. 重要目录说明

### `apps/api`

后端核心代码。

- `src/app.js`：HTTP 路由入口，包含管理端和员工端主要 API。
- `src/server.js`：API 启动入口。
- `src/data/store.js`：当前内存数据仓库，用于骨架和测试。
- `src/data/seed.js`：示例部门、员工、管理员、礼品。
- `src/services/points.js`：积分录入、批量录分、流水查询、冲正等核心规则。
- `src/services/permissions.js`：部门权限判断。
- `src/services/appeals.js`：申诉创建和审核。
- `src/services/orders.js`：商城兑换、订单状态、退分。
- `src/services/wecom.js`：企业微信通讯录同步和消息发送 mock。
- `tests/points.test.js`：核心业务测试。

### `apps/admin-web`

Vue 管理后台。

- `src/App.vue`：后台整体布局。
- `src/api.js`：管理端 API 客户端。
- `src/views/Points.vue`：单笔加减分、月度批量录分。
- `src/views/Reports.vue`：带备注积分明细报表。
- `src/views/Appeals.vue`：申诉审核页面。
- `src/views/Orders.vue`：订单审核、发货、驳回退分。
- `src/views/Mall.vue`：礼品管理骨架。
- `src/views/Logs.vue`：操作日志。

### `apps/miniapp`

uni-app 员工端。

- `pages/home/index.vue`：首页积分、本月变动、通知数量。
- `pages/points/index.vue`：按月份展示积分流水和完整备注。
- `pages/appeal/index.vue`：基于具体积分流水提交申诉。
- `pages/mall/index.vue`：积分商城。
- `pages/orders/index.vue`：我的订单。
- `api.js`：员工端 API 请求封装。

### `infra`

部署和数据库。

- `docker-compose.yml`：MySQL、Redis、API、Nginx 单机部署编排。
- `mysql/schema.sql`：数据库结构，包括积分流水不可改删触发器。
- `mysql/seed.sql`：初始化部门、员工、管理员、礼品。
- `nginx/default.conf`：Nginx 反向代理配置。

### `docs`

- `api.md`：接口摘要。
- `deployment.md`：部署说明。
- `admin-guide.md`：管理员操作说明。

## 5. 常用命令

当前环境可直接使用 Node 运行后端测试：

```bash
node --test apps/api/tests/*.test.js
```

启动 API：

```bash
node apps/api/src/server.js
```

根目录 `package.json` 中也声明了：

```bash
npm run dev:api
npm test
```

注意：当前机器曾出现 `npm` 不可用的情况，因此不要假设 npm 已安装或可直接调用。若要安装前端依赖或执行 Vite/uni-app 构建，需要先确认 npm/pnpm/yarn 环境。

### 5.1 常用环境变量（生产/联调）

- `NODE_ENV=production`：启用更严格的鉴权（禁止用 header 冒充身份）。
- `JWT_SECRET`：JWT 签名密钥，生产必须替换默认值。
- `API_PORT`：后端端口（默认 3000）。

### 5.2 MySQL 持久化开关

后端 store 支持 memory/mysql 双模式：

- 默认：未配置 MySQL 时使用内存 store（便于本地快速跑通与测试）。
- 配置 `MYSQL_HOST` 且未设置 `FORCE_MEMORY_STORE` 时自动切到 MySQL store。

常用 MySQL 环境变量：

- `MYSQL_HOST`
- `MYSQL_PORT`
- `MYSQL_USER`
- `MYSQL_PASSWORD`
- `MYSQL_DATABASE`
- `FORCE_MEMORY_STORE=1`：强制使用内存 store（用于测试/排障）。

### 5.3 Outbox/消息派发相关

- `ENABLE_OUTBOX_WORKER=1`：启动时开启定时派发（可选）。
- `OUTBOX_INTERVAL_MS`：worker 轮询间隔（默认 5000ms）。
- `OUTBOX_BATCH_SIZE`：单次认领/派发条数（默认 50）。
- `OUTBOX_MAX_RETRIES`：失败重试上限（默认 3）。
- `OUTBOX_PROCESSING_TIMEOUT_SEC`：processing 超时回收（默认 60 秒）。
- `WECOM_MOCK_FAIL=1`：模拟派发失败（用于验证重试链路）。

## 6. 已有测试覆盖

`apps/api/tests/points.test.js` 当前覆盖：

- 单笔加减分必须填写备注。
- 月度批量录分支持统一备注和单人备注覆盖。
- 部门管理员不能操作非本部门员工。
- 员工只能申诉自己的积分流水。
- 积分流水创建后不可修改。
- 员工积分明细返回完整备注。
- 订单驳回退回积分并生成退分流水。
- outbox：积分变动入队、派发后状态更新、失败重试与超限失败、processing 超时回收、批量重试失败消息。
- 权限：部门管理员不能审核其它部门申诉、不能处理其它部门订单。

## 7. 当前实现状态与后续重点

本项目从“生产版骨架”推进到了“可上线形态的最小闭环”，但仍有若干生产化缺口需要按优先级补齐。

### 7.1 已落地（当前代码已具备）

- 后端持久化：`apps/api/src/data/store.js` 支持 memory/mysql 双模式，并在 MySQL 模式下提供事务边界（积分/订单写入可原子化）。
- 管理端鉴权：`/admin/auth/login` 返回 JWT；生产环境下 `/admin/*` 禁止用 header 冒充身份，必须 `Authorization: Bearer <token>`。
- 员工端鉴权：`/miniapp/auth/login` 返回员工 JWT；miniapp 请求层支持 401 自动重登并重试一次。
- 权限隔离（RBAC）：部门管理员在读接口（员工/订单/申诉/报表）已做部门范围裁剪；写接口（订单处理/申诉审核）已做权限校验。
- 报表导出：支持 `GET /admin/reports/point-records.xlsx`（包含备注等关键字段）。
- Outbox：消息入队、派发、失败重试、processing 超时回收；管理后台提供 outbox 列表、筛选、单条重试与批量重试。
- 分页：管理端主要列表接口支持 `page/pageSize` 返回 `{ data, meta }`（向后兼容不传分页时返回全量数组）。

### 7.2 关键接口速查（后端）

- 管理端登录：`POST /admin/auth/login`
- 员工端登录：`POST /miniapp/auth/login`
- 健康检查：`GET /health`
- 积分报表（分页/筛选）：`GET /admin/reports/point-records?page=&pageSize=&month=&employeeId=`
- 积分报表导出：`GET /admin/reports/point-records.xlsx`
- Outbox 列表：`GET /admin/outbox?page=&pageSize=&status=&type=&employeeId=`
- Outbox 元信息：`GET /admin/outbox/meta`（types + 运维配置）
- Outbox 单条重试：`POST /admin/outbox/{id}/retry`
- Outbox 批量重试 failed：`POST /admin/outbox/retry-failed`（body: `{ type?, employeeId? }`）
- 立即派发 pending：`POST /admin/wecom/dispatch-messages`

### 7.3 管理后台（admin-web）要点

- 登录态：token 过期/401 会自动清理登录态并回到登录页；右上角展示管理员姓名与角色。
- 报表页：支持分页/筛选，并可下载 Excel（下载同样走统一鉴权处理）。
- 日志页：包含 outbox 运维面板（筛选/分页/派发/重试/批量重试，均带二次确认）。

### 7.4 小程序端（miniapp）要点

- API_BASE 可配置：读取 `uni storage: apiBase`，默认 `http://localhost:3000`。
- 登录重试：请求遇到 401 会自动重新登录并重试一次。
- 调试设置入口收口：首页设置区默认隐藏，需要连续点击“展开设置”区域 7 次才会启用（`uni storage: enableDebug=1`）。

### 7.5 后续重点（仍建议继续补齐）

生产仍有明显缺口，建议按优先级推进：

- 企业微信真实接入：免登、通讯录同步、消息发送（当前是 mock + outbox 骨架）。
- 安全与运维：JWT_SECRET、数据库密码等密钥管理；日志采集与审计；限流/防刷策略。
- Outbox 生产化：引入真正的 sent/failed 语义、失败原因、DLQ（死信）与可观测指标。
- 数据规模：更完善的筛选条件、索引与 SQL 优化；导出大表的流式导出/异步任务化。
- 部署：Docker Compose/Nginx 配置按生产拓扑调整，补齐环境配置模板与启动校验脚本。

## 8. 操作安全要求

禁止批量删除文件或目录。

不要使用：

- `del /s`
- `rd /s`
- `rmdir /s`
- `Remove-Item -Recirse`
- `Remove-Item -Recurse`

需要删除文件时，只能一次删除一个明确路径的文件，例如：

```powershell
Remove-Item "C:\path\to\file.txt"
```

如果需要批量删除文件，应停止操作，并请求用户手动删除。

## 9. 协作注意事项

- 修改积分相关逻辑时，优先查看 `apps/api/src/services/points.js` 和 `apps/api/tests/points.test.js`。
- 不要新增能修改或删除积分流水备注的接口。
- 不要绕过 `remark` 必填校验。
- 新增积分变动类型时，应同步考虑员工端展示、报表导出、操作日志和消息通知。
- 数据库层的 `point_records` 不可改删约束是业务底线，不应移除。
- 文档和代码中中文内容应按 UTF-8 保存；PowerShell 读取中文时建议使用 `Get-Content -Encoding UTF8`。

---
> Source: [wjk1120-learning/EasyPoints](https://github.com/wjk1120-learning/EasyPoints) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
