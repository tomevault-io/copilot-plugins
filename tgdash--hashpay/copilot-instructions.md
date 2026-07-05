## hashpay

> 本文件帮助快速理解 HashPay 的项目结构、运行原理和开发约定。

# AGENTS.md

本文件帮助快速理解 HashPay 的项目结构、运行原理和开发约定。
修改代码需严格遵守。

## 项目概述

HashPay 是一个运行在 Cloudflare Workers 环境上的加密货币收款系统，项目同时包含后端 Worker 服务和 Vue 管理/收银台前端。

主要能力包括：

- 管理员后台：配置商户、收款通道、订单、系统设置、Banner 和汇率微调。
- 商户 API：通过 RSA 签名创建订单、查询订单，并返回托管收银台链接。
- 收银台：选择资产和网络、展示地址/二维码、触发服务端查账、超时和支付成功状态处理。
- Telegram 支持：绑定管理员、Mini App 后台入口、PIN 登录、Bot 内创建收款订单、获得订单通知。
- 支付检测：支持自动链上检查、交易所/钱包 API 查账、用户触发服务端查账和人工确认。
- 回调通知：订单支付成功后发送回调消息，利用队列系统确保投递稳定。
- 汇率管理：定时同步法币兑 USD 汇率，USD / USDT / USDC 按 1:1 处理，并支持后台微调。

## 技术栈

- 语言：TypeScript，ESM，严格类型检查。
- 前端：Vue 3 + Vue Router + Naive UI + Vite + SCSS。
- 后端：Cloudflare Workers + Hono。
- 数据库：Cloudflare D1，SQL 迁移文件位于 `src/server/db/d1/migrations/`。
- Cloudflare：Workers Assets、D1、Queues、Cron Triggers。
- Telegram：grammY + Telegram Bot API。
- 测试：Vitest。

## 常用命令

```bash
npm install
npm run dev
npm run dev:worker
npm run check
npm run test
npm run build
npm run deploy:dry
npm run cf-typegen
```

- `npm run dev` 启动 Vite + Cloudflare Vite 插件，本地前端端口见 `vite.config.ts`。
- `npm run dev:worker` / `npm start` 直接启动 `wrangler dev`，端口见 `wrangler.jsonc`。
- `npm run db:migrate:local` / `npm run db:migrate:remote` 分别应用本地/远程 D1 迁移。
- 修改 Cloudflare bindings 后运行 `npm run cf-typegen`，并同步更新 `src/server/types/env.ts`。
- 不要提交 `.dev.vars`、真实 Bot Token、APP_SECRET、商户私钥或其他密钥。

## 目录结构

```text
HashPay/
├── src/index.ts                         # Worker 入口：fetch、scheduled、queue
├── src/server/http/                     # Hono 应用、中间件、API envelope、路由
│   └── routes/                          # public、auth、admin 路由
├── src/server/services/                 # 后端业务服务
│   ├── app/                             # 系统状态、设置、汇率、定时任务
│   ├── auth/                            # Session、Telegram initData、PIN 登录
│   ├── images/                          # Banner 和订单二维码
│   ├── merchants/                       # 商户、RSA 密钥、商户签名校验
│   ├── orders/                          # 建单、收银台、订单管理、通知
│   └── telegram/                        # Bot API、Webhook、初始化绑定
├── src/server/payments/                 # 支付通道模型、快照生成、查账驱动
│   └── providers/                       # 付款渠道的查询实现
├── src/server/db/                       # D1 helper、配置 helper、迁移执行器
├── src/server/types/                    # Worker 环境类型和 Hono 变量
├── src/shared/                          # 前后端共享类型
├── src/app/                             # Vue 前端应用
│   ├── api/                             # 前端 API client，复用 shared/types/api.ts
│   ├── components/                      # 前端组件
│   ├── pages/                           # 前端页面
│   ├── payments/                        # 前端支付展示和浏览器侧查账能力
│   ├── utils/                           # 前端格式化、剪贴板、收银台状态机等工具
│   ├── i18n.ts                          # 前端轻量 i18n store
│   └── styles.scss                      # 全局样式
├── test/                                # Vitest 单元测试
├── wrangler.jsonc                       # Cloudflare Workers 配置
├── vite.config.ts                       # Vite + Cloudflare 插件配置
└── vitest.config.ts                     # Vitest 配置
```

## Worker 运行方式

- `src/index.ts` 是唯一 Worker 入口，导出 `fetch`、`scheduled`、`queue`。
- `fetch` 进入 `createApp()`，Hono 在 `src/server/http/app.ts` 装配中间件、路由和 Assets 回退。
- API 成功响应直接返回业务 JSON，不包 `{ data }`；错误统一走 `{ error: { key, params } }`，前端负责按当前语言翻译。
- `/api/*` 请求会先执行 `migrateD1(c.env)`，但 `/api/state` 例外；`appState()` 自行尝试迁移并把 DB 错误转成状态字段。
- 未命中后端路由时回落到 `ASSETS.fetch()`，用于服务 SPA。
- Cron 每分钟执行 `runJobs()`：迁移、整点同步汇率、过期订单、检查待支付订单、把到期通知放入队列。
- Queue 消费 `QUEUE_NOTIFY`，消息体需要包含 `notifyId`，失败时延迟重试。

## Cloudflare 绑定

当前 `wrangler.jsonc` 绑定：

- `ASSETS`：Workers Assets，目录 `dist`，SPA fallback。
- `DB`：D1 数据库，迁移目录 `src/server/db/d1/migrations`。
- `QUEUE_NOTIFY`：通知投递队列。
- Cron：`* * * * *`。

环境变量：

- `APP_SECRET`：JWT Session、PIN 登录挑战签名等服务端签名用途。
- `TGBOT_TOKEN`：Telegram Bot API。

改动绑定或环境变量时必须同步检查：

- `wrangler.jsonc`
- `src/server/types/env.ts`
- 依赖这些字段的服务和测试
- `npm run cf-typegen` 生成结果

## 数据模型

D1 初始 schema 在 `src/server/db/d1/migrations/0001_init.sql`：

- `configs`：系统配置、Bot 状态、管理员、Banner blob、汇率缓存等。
- `merchants`：商户、公钥、回调 URL、状态和类型。
- `payments`：收款通道、driver、地址、支持资产、凭据和状态。
- `orders`：订单主体，`merchant_no` 是商户侧幂等单号，`payment` 是支付快照 JSON。
- `notify`：商户回调任务、重试状态、payload 和错误信息。
- `review`：用户提交的人工审核答案、截图和审核状态。
- `d1_migrations`：由迁移执行器自动创建，作为迁移是否已应用的事实来源。

数据库字段使用 snake_case；服务和 API DTO 使用 camelCase。`orders.merchant_no` 对应公开字段 `merchantNo`，不要重新引入旧字段名或兼容分支，除非需求明确要求接入旧外部协议。

## 核心链路

### 初始化

1. 前端 `/setup` 调用 `/api/state` 检查 DB、Queue、Bot。
2. 环境就绪后，前端提交公网 HTTPS 域名到 `/api/admin/setup`。
3. `startTelegramSetup()` 写入 domain、bot_secret，刷新 Bot 信息并设置 webhook。
4. 管理员访问 Bot 并发送任意消息，`bindSetupAdmin()` 写入默认设置、默认 Banner、Mini App 菜单，同步一次汇率，然后写入 `admin_id` 和 `admin_user`。
5. `admin_id` 是实例是否已安装的唯一指示；写入后再次提交 `/api/admin/setup` 会直接返回已初始化错误。
6. 安装完成后进入后台走 Telegram Mini App 登录或浏览器 PIN 登录，不再通过 `/api/admin/setup` 签发 Session Cookie。

初始化完成是内部状态变化，不要新增公开 finalize/status 流程，除非需求明确要求。

### 登录

- Telegram Mini App 登录走 `/api/admin/session/telegram`，使用 `validateWebAppInitData()` 校验 initData。
- 浏览器 PIN 登录走 `/api/admin/session/pin` 创建 challenge，再由管理员在 Bot 中发送 `/login <pin>` 确认。
- Session Cookie 名称为 `hashpay_session`，JWT 由 `APP_SECRET` 签名，有效期 7 天。
- Admin 路由在 setup/session 之后统一 `requireAdmin()` 保护。

### 商户 API 和建单

1. 商户请求 `/api/merchant/new`，使用 `X-Merchant-Id`、`X-Signature`、`X-Timestamp`。
2. 签名原文是 `METHOD\npathname+search\ntimestamp\nbody`。
3. `requireSignedMerchant()` 校验时间窗口、公钥、商户状态和 RSA-SHA256 签名。
4. `createMerchantOrder()` 读取 `merchantNo`、金额、币种、callback、return_url。
5. `createOrder()` 以 `(merchant, merchantNo)` 做幂等，已有订单直接返回 `reused: true`。
6. 响应包含 `checkoutUrl` 和 `order` 摘要。

### 收银台支付

1. `/api/checkout/:orderId` 读取订单、商户、启用的支付通道和当前汇率缓存。
2. `checkoutData()` 用一次 `rateContext()` 为所有通道生成候选金额。
3. 用户选择资产/网络后，`selectCheckoutPayment()` 随机选择可用通道，并写入 `orders.payment` 快照。
4. 已写入的 `payment.amount` 是固定应付金额；除非用户重新选择网络/币种，不应随汇率变化。
5. `uniqueAmount()` 会避开同一通道、同一地址、同一资产网络下其他未过期订单的相同金额。
6. 前端 `Pay.vue` 展示订单状态、二维码/地址、倒计时、支付成功返回和人工审核入口。

不要在打开收银台或生成候选项时重新请求外部汇率。汇率同步归 `syncMarketRates()`，收银台读取归 `currentMarketRates()` / `rateContext()`。

### 支付确认

- 自动检查入口：`checkOrderPayment()` 和 Cron 扫描待支付订单。
- 浏览器和 Telegram 的“我已完成付款/检查付款”只触发一次服务端查账，不提交或信任浏览器交易候选。
- 服务端查账：`server/payments/driver.ts` 根据 `snapshot.driver` 派发 provider，并只使用 provider 从链上、交易所或钱包 API 拉取的数据匹配订单。
- 当前自动查账 provider 包括 TRC20、EVM、TON、Aptos、Binance Pay、OKX；OKPay 主要走回调/创建返回流程。
- 交易所查账以收款通道的 `address` 作为 Binance ID / OKX UID，API Key、Secret、Passphrase 等细节放在 `PaymentData` / `credentials` 内。
- 确认支付统一调用 `markPaid()`，写入 `payment.tx`，更新订单为 `paid`，并创建 notify。
- 人工确认入口在后台订单详情，`confirmedBy` 为 `admin`。

### 商户通知

1. `markPaid()` 调用 `createNotify()`。
2. notify payload 包含订单号、商户单号、金额、币种、状态和 payment 快照。
3. 如果订单没有 callback，则不创建通知。
4. `deliverNotify()` 使用商户公钥将 `{ timestamp, payload }` 加密为 JSON 信封，POST 到 callback，并附带 `X-HashPay-Merchant`、`X-HashPay-Timestamp`、`X-HashPay-Encryption`；失败最多重试 8 次，`next_run_at` 按 attempts 分钟递增。
5. 后台可对已支付订单手动重发通知。

### Telegram 收款

- Telegram 只接受 inline query 下单，通过 `createTelegramOrder()` 创建 `INLINE` 内部商户订单；DM `/pay` 命令不创建订单。
- 未初始化时不响应 inline query；初始化后只有管理员账号的 inline query 会创建订单。
- Telegram 内选择资产和网络复用 `checkoutData()` 与 `selectTelegramPayment()`。
- Telegram 订单重新选择网络时会刷新支付窗口；普通网页收银台不会自动刷新窗口。
- Telegram 消息中的付款说明、检查付款按钮、审核入口和交易链接由 `server/services/telegram/bot.ts` 维护。

## 模块职责

- `src/server/http/app.ts`：只负责 Hono 装配、中间件、迁移触发、路由挂载和 Assets 回退。
- `src/server/http/routes/*`：只做 HTTP 参数解析、鉴权边界和调用 service；复杂业务放到 services。
- `src/server/db/index.ts`：D1 原语、配置读写和 JSON helper；不要把业务查询堆到这里。
- `src/server/db/migrations.ts`：D1 SQL 文件加载和迁移状态；新增迁移必须放在 `d1/migrations/` 并保持文件名排序。
- `src/server/services/app/*`：系统状态、设置、汇率缓存和后台统计。
- `src/server/services/auth/*`：Session、PIN、Telegram initData 校验；不要在页面组件里复制认证逻辑。
- `src/server/services/merchants/index.ts`：商户 CRUD、密钥生成、签名认证。
- `src/server/services/orders/create.ts`：建单和删除订单。
- `src/server/services/orders/repository.ts`：订单行和 DTO 映射、列表查询、支付快照写入。
- `src/server/services/orders/checkout.ts`：收银台数据、支付选择、查账、人工确认、审核提交。
- `src/server/services/orders/notifications.ts`：notify 创建、投递和重试。
- `src/server/payments/driver.ts`：支付定义校验、候选项、支付快照、查账 provider 分派。
- `src/server/payments/channels.ts`：收款通道 CRUD、通道健康和查账结果状态回写；服务端内部称为 `PaymentChannel`。
- `src/shared/payments.ts`：支付网络、资产、展示名、图标、地址规则、explorer URL 的事实来源。
- `src/shared/types/api.ts`：前后端共享 API DTO。
- `src/shared/types/domain.ts`：领域状态和支付快照类型。
- `src/app/api/*`：前端请求封装和 API 模块；不要在页面里拼重复 fetch 逻辑。
- `src/app/pages/Admin.vue`：后台应用壳、会话初始化和导航。
- `src/app/pages/Pay.vue` + `src/app/utils/checkout.ts`：公开收银台 UI 和轮询/查账状态机。
- `src/app/payments/index.ts`：前端支付展示、EVM 组合选择和通道展示元数据。

## 代码结构和命名约定

- 命名优先表达当前模块语境，不把实现细节堆进函数名。例：收银台选择付款用 `selectCheckoutPayment()`，Telegram 选择付款用 `selectTelegramPayment()`，不要写成长串 `selectOrderPaymentByAssetNetwork()`。
- Server 内部把 `payments` 表的业务对象称为 `PaymentChannel`；前端 API DTO 使用 `Payment` 作为展示层接口名，不要把旧 method 命名重新带回 server。
- 每个收款通道只保留一个基础 `address`；API Key、Secret、Passphrase 等附加字段放在 `data` / `credentials`，不要为单个交易所新增平行字段。
- 支付通道函数统一使用 `listPayments()`、`savePayment()`、`deletePayment()`、`recordCheck()`、`paymentHealth()`。
- 订单确认统一走 `markPaid()`；只有后台手动确认时写入 `confirmedBy: "admin"`，自动检查、用户触发查账、Cron 命中都默认是 system。
- 不新增只做换名或一行转发的 helper。允许保留的短函数必须承担明确边界：DB helper、row -> DTO 映射、领域公式、加密/SQL 转义等。
- 如果一个 helper 只是为了“看起来分层”而没有隐藏复杂度、没有统一规则、没有隔离边界，应直接内联。
- 不做兼容旧函数名、旧字段名的导出别名；破坏性修改后同步更新调用方和测试。

## 支付通道扩展约定

新增或调整支付网络/资产时，按这个顺序检查：

1. `src/shared/payments.ts`：新增 Payment 定义、资产、图标 id、地址规则、`data` 凭据字段和 explorer。
2. `src/server/payments/driver.ts`：确认 `validateChannel()`、`paymentOptions()`、`assignPayment()`、`validateData()`、`checkPayment()` 是否能覆盖。
3. `src/server/payments/providers/`：如果要自动查账，新增 provider 并注册到 `src/server/payments/driver.ts` 的 `providers`。
4. `src/app/payments/index.ts`：前端网络/资产选择和 EVM 聚合展示。
5. `public/icons.svg`：需要展示的新图标。
6. `test/payment-model.test.ts`、`test/payment-channels.test.ts`、`test/payment-providers.test.ts`、`test/app-payments.test.ts`：补充模型、通道保存、查账和前端展示行为。

不要把网络名当资产别名。例：TON 是网络，GRAM 是资产；EVM 兼容网络在前端可以合并展示，但后端仍是具体 driver。
当前交易所通道只保留 Binance Pay 和 OKX；Huobi 已移除，不要重新引入相关 key、图标、翻译或文档。

## 汇率约定

- `systemSettings()` 读取基础货币、超时时间、汇率微调和快速确认。
- `syncMarketRates()` 是唯一会请求外部汇率 API 的同步路径。
- `currentMarketRates()` 只读取缓存或默认值，并带内存 60 秒缓存。
- `rateContext()` 将设置和汇率快照合并，供一次 checkout 或统计计算复用。
- `marketAmount()` 使用市场汇率做统计/展示换算，不使用汇率微调。
- `payAmount()` 使用汇率微调并向上保留两位，用于收银台应付金额，避免用户少付。
- Cron 每分钟运行，但汇率只在每个整点同步；初始化完成时也同步一次。

收银台性能敏感。不要在每个支付选项、每次打开收银台或每次选择网络时重复外部汇率请求。

## 前端约定

- 路由集中在 `src/app/main.ts`。
- 全局样式在 `src/app/styles.scss`，页面优先复用现有 panel、grid、section-title、orders-table 等样式。
- UI 组件使用 Naive UI；支付/网络图标通过 `AppIcon` 和 `public/icons.svg`。
- 后台页面由 `Admin.vue` 统一承载，页面组件在 `src/app/pages/`。
- 公开收银台独立在 `Pay.vue`，不要把后台登录态、管理菜单或内部设置泄漏到收银台。
- 前端接口类型从 `@/app/api` 导出，源头是 `src/shared/types/api.ts`。
- 可见文案走轻量 i18n：共享字典在 `src/shared/i18n.ts`，前端语言状态在 `src/app/i18n.ts`；不要在页面、Toast、Modal 或 Telegram 消息里硬编码新增文案。

## API 和错误约定

- 服务端业务错误使用 `AppError(status, key, params?)`。
- 前端 `request()` 直接返回成功响应 JSON，并把 `{ error: { key, params } }` 翻译成当前语言的 `ApiError.message`。
- 轮询、后台刷新、用户触发查账等非阻塞请求使用 `{ silent: true }`。
- Admin API 在 `src/server/http/routes/admin.ts`，公开收银台和状态接口在 `public.ts`，商户签名 API 在 `auth.ts`。
- 新增 API 时同步更新 `src/shared/types/api.ts`、`src/app/api/index.ts`；新增错误 key 时同步更新 `src/shared/i18n.ts`。

## 测试约定

当前测试覆盖重点：

- `test/d1-migrations.test.ts`：迁移加载、迁移状态、并发迁移。
- `test/merchant-auth.test.ts`、`test/merchant-api.test.ts`：RSA 签名和商户 API。
- `test/payment-model.test.ts`、`test/payment-channels.test.ts`、`test/payment-providers.test.ts`、`test/app-payments.test.ts`：支付模型、通道保存、查账和前端展示。
- `test/checkout-payments.test.ts`、`test/jobs.test.ts`、`test/notify.test.ts`：收银台选择、定时任务和通知投递。
- `test/rates.test.ts`、`test/settings.test.ts`：汇率和系统设置。
- `test/auth.test.ts`：Session 和 PIN 登录。
- `test/app-state.test.ts`：系统状态和 Bot 用户名缓存。
- `test/api-http.test.ts`、`test/amount-format.test.ts`、`test/i18n.test.ts`：前端 API client、金额展示和共享文案。

涉及以下改动时优先补测试：

- 建单、幂等、订单状态流转、通知投递。
- 支付快照、查账、汇率、金额格式或金额避让。
- 商户签名、Session、Telegram 登录或 PIN 登录。
- D1 migration、Cloudflare bindings、Cron/Queue 行为。
- 前后端共享 DTO 字段名。

## 开发注意事项

- 默认围绕用户明确目标做最小完整改动，不引入无关业务路径。
- 不做补丁式兼容分支来保留旧结构；如果字段、函数或模块边界已经明确，应直接改到目标结构。
- 修改前按输入、处理流程、状态变化、输出、上下游影响检查一遍。
- 无法从代码或运行结果确认的内容，必须标注为假设，不要写成事实。
- 不要回滚或覆盖未说明的工作区修改；这个项目经常存在正在进行的未提交重构。
- 涉及数据库字段时同步检查 SQL migration、repository 映射、API DTO、前端展示和测试。
- 涉及 Cloudflare 绑定时同步检查 `wrangler.jsonc`、`AppEnv`、Worker 入口和测试环境 mock。
- 涉及支付金额时保持应付金额快照语义：选择后固定，重新选择才重新计算。
- 涉及 setup/status 时保持内部完成流程，不额外公开 finalize/status 表面。
- 涉及 Telegram Bot 时注意 webhook 入口会先迁移 D1，并且 Bot 初始化/菜单配置依赖公网 HTTPS domain。
- 生产敏感数据只通过环境变量、Workers bindings、D1 配置或一次性响应流转，不写入源码、测试快照或文档示例。

## 关键文件

- `README.md`：项目对外说明、部署、本地开发、初始化和支付接入文档。
- `src/index.ts`：Worker 总入口。
- `src/server/http/app.ts`：Hono 应用装配和中间件。
- `src/server/http/routes/public.ts`：健康检查、收银台、状态、Telegram webhook。
- `src/server/http/routes/auth.ts`：商户签名 API。
- `src/server/http/routes/admin.ts`：后台 API 和 Admin guard。
- `src/server/services/app/jobs.ts`：Cron 和 Queue 任务编排。
- `src/server/services/orders/create.ts`：建单。
- `src/server/services/orders/checkout.ts`：收银台、支付选择、查账和确认。
- `src/server/services/orders/repository.ts`：订单查询和 DTO 映射。
- `src/server/services/orders/notifications.ts`：通知任务和投递。
- `src/server/services/orders/review.ts`：人工审核数据和图片处理。
- `src/server/services/app/settings.ts`：系统设置和汇率缓存。
- `src/server/services/app/setup.ts`：初始化流程。
- `src/server/services/telegram/bot.ts`：Telegram Bot 行为。
- `src/server/payments/driver.ts`：支付 driver 编排。
- `src/server/payments/channels.ts`：收款通道保存、公开输出、健康和查账状态。
- `src/server/payments/providers/`：链上、交易所和钱包 provider 实现。
- `src/shared/payments.ts`：支付网络和资产定义。
- `src/shared/types/api.ts`：共享 API 类型。
- `src/server/types/env.ts`：Worker 环境类型。
- `src/app/pages/Pay.vue`：公开收银台页面。
- `src/app/pages/Admin.vue`：后台应用壳。
- `src/app/api/index.ts`：前端 API 模块。
- `wrangler.jsonc`：Cloudflare 运行配置。

---
> Source: [TGDash/HashPay](https://github.com/TGDash/HashPay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-05 -->
