## hotelmanagement

> 全栈酒店管理系统，当前主线业务按 `房态 / 客单 / 现付账 / 报表 / 设置` 五大方向收敛。

# AGENTS.md — 酒店管理系统 (Hotel PMS) 编码规范

## 项目概况

全栈酒店管理系统，当前主线业务按 `房态 / 客单 / 现付账 / 报表 / 设置` 五大方向收敛。

- 房态：实时房态、远期房态总览、房间管理，以及后续的批量操作、营业日提示、续住/换房等接待增强
- 客单：订单全生命周期、预订修改、挂账单、联房/团队单、No Show 等
- 现付账：独立于房帐的即时消费收银，不与当前房帐财务客单混用
- 报表：前台交班、付款明细、消费明细、营收与经营分析报表
- 设置：酒店基础信息、班次、零售商品、用户账号、钟点房配置

说明：
- 当前仓库中的 `Members.jsx`、`More.jsx` 仅为占位页，不再作为独立产品方向扩写
- 前端继续沿用现有霓虹赛博视觉风格，不做风格重置

```text
HotelManagement/
├── pms-backend/        # Python FastAPI 后端
├── pms-frontend/       # React + Vite 前端
├── reference/          # 逆向需求与参考资料
├── jiagou.md           # 架构设计文档
└── AGENTS.md           # 本文件
```

## 技术栈（锁定版本，不要随意升级）

| 层 | 技术 | 备注 |
|---|---|---|
| 后端框架 | FastAPI 0.115+ | ASGI，async 优先 |
| ORM | SQLAlchemy 2.0+ | 声明式模型 |
| 数据校验 | Pydantic v2 | BaseModel 做 schema |
| 数据库 | SQLite (开发) → PostgreSQL (生产) | 通过 DATABASE_URL 切换 |
| 前端框架 | React 19 | 函数组件 + Hooks |
| 构建工具 | Vite 7 | ES modules |
| UI 组件库 | Material-UI (MUI) 7 | 唯一 UI 库，不引入其他 |
| 状态管理 | Zustand 5 | 轻量 store |
| 路由 | React Router DOM 7 | 嵌套路由 |
| HTTP 客户端 | Axios | 统一走 api.js |
| CSS 方案 | Emotion (MUI sx prop) | 不使用 CSS Modules / Tailwind |

## 启动命令

```bash
# 后端
cd pms-backend && pip install -r requirements.txt && uvicorn main:app --reload

# 前端
cd pms-frontend && npm install && npm run dev
```

---

## 一、通用规范

### 语言
- 前端：JavaScript (JSX)，不使用 TypeScript
- 后端：Python 3.12+
- UI 文案、注释、变量命名说明均使用中文（变量名本身用英文）
- API 错误消息使用中文（如 `"房间不存在"`）

### 命名约定
| 场景 | 风格 | 示例 |
|---|---|---|
| Python 函数/变量 | snake_case | `check_in`, `room_id` |
| Python 类 | PascalCase | `CheckInRequest` |
| SQLAlchemy 列名 | camelCase | `roomNumber`, `checkInTime` |
| JS 变量/函数 | camelCase | `loadRooms`, `guestName` |
| React 组件 | PascalCase | `RoomStatus`, `MainLayout` |
| 文件名 (前端组件) | PascalCase.jsx | `RoomStatus.jsx` |
| 文件名 (前端工具) | camelCase.js | `api.js`, `store.js` |
| 文件名 (后端) | snake_case.py | `main.py`, `models.py` |
| API 路径 | kebab-case 或小写 | `/api/rooms`, `/api/night-audit` |
| 数据库表名 | 复数小写 | `rooms`, `guests`, `orders` |

### 状态枚举（字符串常量，全大写）
- 房间状态：当前已使用 `VC / VD / OC / OD / OOO / RES`，后续新增锁房语义时需单独设计，不与维修房混用
- 订单状态：`ACTIVE` / `COMPLETED` / `CANCELLED`
- 支付状态：`PENDING` / `PAID` / `REFUNDED`

### 文档更新原则
- `AGENTS.md` 中的“模块开发指南”只写尚未完成或尚未业务闭环的未来蓝图
- 已有真实前后端实现的能力，不要重复写成后续任务
- 新需求的来源以 `reference/missionmap.md` 为准，`reference/devnote.md` 用于约束去重策略与风格延续

---

## 二、后端规范 (pms-backend/)

### 当前基础目录
```text
pms-backend/
├── main.py
├── database.py
├── models.py
├── schemas/
│   ├── room.py
│   ├── order.py
│   └── finance.py
├── routers/
│   ├── rooms.py
│   ├── orders.py
│   └── finance.py
├── services/
│   ├── room_service.py
│   ├── reservation_service.py
│   ├── order_service.py
│   └── finance_service.py
├── tests/
└── requirements.txt
```

### 未来扩展目录目标
在保持现有结构不破坏的前提下，未来新增模块按以下方向扩展：

```text
routers/
├── auth.py
├── reports.py
├── settings.py
├── cashier.py
├── night_audit.py
├── channels.py
└── hardware.py

schemas/
├── auth.py
├── report.py
├── setting.py
├── cashier.py
├── night_audit.py
├── channel.py
└── hardware.py

services/
├── auth_service.py
├── report_service.py
├── setting_service.py
├── cashier_service.py
├── night_audit_service.py
├── channel_service.py
└── hardware_service.py
```

### API 设计
- 所有端点以 `/api/` 开头
- RESTful 风格：GET 查询、POST 创建、PUT/PATCH 更新、DELETE 删除
- 路由函数使用 `Depends(get_db)` 注入数据库 Session
- 请求体用 Pydantic BaseModel 校验，不要在路由函数里手动校验字段
- 响应统一返回 dict 或 Pydantic model，不要直接返回 ORM 对象
- 错误使用 `HTTPException`，status_code 遵循 HTTP 语义，detail 用中文
- 后续新增模块推荐前缀：
  - `/api/auth`
  - `/api/night-audit`
  - `/api/cashier`
  - `/api/reports`
  - `/api/settings`
  - `/api/channels`
  - `/api/hardware`

### 数据库 & 模型
- 模型定义在 `models.py`，一个文件管理所有表
- 外键关系必须定义 `relationship` 双向绑定
- 主键统一用自增 Integer `id`
- 时间字段用 `DateTime`，默认值用 `datetime.now`（注意不是 `datetime.now()`）
- 字符串枚举直接用 String 列 + 注释说明可选值，不用 Python Enum
- 新增表时在 `models.py` 底部追加，保持导入顺序一致
- 已有局部字段不代表业务已闭环，新增能力仍需补足专属模型，不能只靠备注字段硬撑

### 业务逻辑
- 简单 CRUD 可以直接写在路由里
- 涉及多表操作、事务、外部调用的逻辑抽到 `services/` 层
- 数据库事务：一个请求一个 Session，service 层不要自己创建 Session
- 不要在路由层 catch 通用异常来返回 200，让 FastAPI 的异常处理机制工作
- 夜审、报表、渠道同步、硬件对接、现付账、认证鉴权逻辑必须进 service 层
- 在增加和修改功能时，前端与后端要对齐，也就是说永远不要出现前端仅供演示的情况而没有实际做后端。如果需要演示，请在做出相应后端后进行实际业务级别的演示。
- 在完成TASKS.md中的任务时，可以在根目录中查看前置任务的完成与测试情况，所有完成的任务都有一个对应的T0?Test.md文件，?替换成第几个任务，里面详细写了任务完成情况以及测试结果

---

## 三、前端规范 (pms-frontend/)

### 当前目录结构
```text
pms-frontend/src/
├── main.jsx
├── App.jsx
├── api.js
├── store.js
├── theme.js
├── layouts/
│   └── MainLayout.jsx
├── pages/
│   ├── RoomStatus.jsx
│   ├── FutureRooms.jsx
│   ├── Orders.jsx
│   ├── Payments.jsx
│   ├── Reports.jsx
│   ├── RoomManage.jsx
│   ├── Members.jsx      # 当前占位，不再扩写主线需求
│   └── More.jsx         # 当前占位，不再扩写主线需求
├── components/
│   ├── ReservationOrderDialog.jsx
│   └── WaveBackground.jsx
└── assets/
```

### 未来页面扩展方向
后续新增正式页面时，优先补齐以下产品主线页面：

```text
pages/
├── Login.jsx
├── Cashier.jsx
├── Reports.jsx      # 从占位页升级为正式报表中心
└── Settings.jsx
```

### 组件编写
- 只用函数组件 + Hooks，不用 class 组件
- 组件用 `export default function ComponentName()` 导出
- 一个文件一个组件，文件名 = 组件名
- 页面组件放 `pages/`，可复用组件放 `components/`
- props 不需要 PropTypes（项目不用 TypeScript 也不用 PropTypes）
- 组件内部状态用 `useState`，跨组件共享用 Zustand store
- 副作用用 `useEffect`，依赖数组必须写完整

### 样式规范
- 统一使用 MUI 的 `sx` prop，不写外部 CSS 文件
- 颜色、间距、圆角等从 `theme.js` 取值，不要硬编码新的设计 token
- 保持现有霓虹赛博视觉语言，后续新页面只做同风格扩展，不做整站换肤
- 设计系统色板（严格遵守，不要引入新颜色）：

| 用途 | 色值 |
|---|---|
| 主色 (霓虹蓝) | `#00d4ff` |
| 辅色 (霓虹青) | `#00ffcc` |
| 背景 | `#000000` |
| 纸面 | `#0a0a0a` |
| 主文字 | `#e0e0e0` |
| 次文字 | `#666666` |
| 空净房 | `#00ff88` |
| 在住房 | `#ff3366` |
| 脏房 | `#888888` |
| 维修房 | `#ffaa00` |

- 霓虹发光效果用 `box-shadow` 和 `text-shadow`，保持与现有卡片风格一致
- 全局圆角 14px（`theme.shape.borderRadius`）
- 过渡动画统一 `transition: 'all 0.2s ease'` 或 `0.3s ease`

### 状态管理 (Zustand)
- 全局 store 在 `store.js`，模块增多后可拆分为 `stores/roomStore.js` 等
- store 里只放跨页面共享的数据（rooms、user 信息、营业日、权限、财务选中对象等）
- 页面内部临时状态（弹窗开关、表单值）用 `useState`
- 异步操作（API 调用）写在 store 的 action 里，组件里调用 action

### API 调用
- 所有请求通过 `api.js` 的 axios 实例发出
- 每个端点导出一个函数：`export const fetchRooms = () => api.get('/api/rooms')`
- 组件不要直接 `import axios`，必须走 `api.js`
- `baseURL` 当前为 `http://localhost:8000`，后续通过环境变量配置

### 路由
- 路由表集中在 `App.jsx`
- 所有页面嵌套在 `MainLayout` 下
- 导航主线最终收敛为：`房态 / 客单 / 现付账 / 报表 / 设置`
- 当前 `Members`、`More` 仅作占位和过渡，不再继续扩写主业务功能
- 新增页面：1) 创建 `pages/Xxx.jsx` 2) 在 `App.jsx` 加 Route 3) 在 `MainLayout.jsx` 的导航配置中加入口

---

## 四、未来模块开发蓝图

以下内容仅记录目前尚未完成或尚未形成业务闭环的功能。已经有真实实现的实时房态基础链路、预订单创建/取消/入住基础、客单基础视图、财务客单基础、房态总览、房间管理等能力，不再重复列为后续任务。

### 4.1 登录认证与营业日
- 新增登录/登出体系，按酒店专属 URL 或酒店标识隔离实例
- 新增用户、角色、权限模型，至少覆盖：
  - `administrator`
  - `酒店总经理`
  - `酒店财务`
  - `前台接待`
- 新增营业日（business date）与夜审机制，业务统计以营业日为准，不以系统日期直接替代
- 当营业日落后于系统日期时，房态首页需阻断关键业务并提示先做夜审
- 营业日推进后需联动房费生成、房态流转、No Show 失效和固化报表口径

### 4.2 房态与前台接待增强
- 新增房态批量操作：批量制净、制脏、置维修、锁房、批量预订、批量入住
- 新增独立“锁房”语义，不与“维修房”混用
- 新增房态标签体系：
  - 渠道筛选
  - 客源类型筛选
  - 今日预抵仅统计“已排房”预订单
  - `欠`
  - `无押`
- 新增房态个性化设置：
  - 房间排列方式
  - 卡片大小
  - 字体大小
  - 标签显隐
  - 房态颜色配置
- 新增续住能力：散客续住、OTA 续住、连单续住，支持续住房价策略与收款联动
- 新增换房/升房能力：平级换房、补差升房、免费升房，并记录换房原因与差价处理
- 新增在住详单快捷操作：
  - 改房价
  - 修改入住类型
  - 补录额外消费
  - 退款/补收
  - 退房加收规则
- 门锁发卡流程要与入住、续住、换房联动，不能只留一个按钮占位

### 4.3 客单深化
- 新增挂账退房生命周期：挂账退房、已挂账列表、后续补结账
- 新增联房/团队单能力：联房主单、团队预订、团队在住
- 新增预订单修改能力：修改抵离时间、客人信息、房型、间数、价格、外部单号、升级房型
- 当前仅占位的以下菜单必须在后续接入真实数据和真实操作：
  - `posted`
  - `master_linked`
  - `group_reserved`
  - `group_in_house`
- OTA 直连订单修改后不同步的问题，要在客单和预订修改流程中给予人工同步提示

### 4.4 现付账与账务纠错
- 新增独立“现付账”模块，用于非住店即时消费、零售、赔偿、洗衣等场景，支持独立开单、即时收款、即时退款、流水查询
- 现付账不得与当前房帐财务客单混为一个页面逻辑
- 新增账务纠错三层工具，并明确使用边界：
  - `冲账`
  - `调账`
  - `入账`
- 扩展 AR 账能力：不仅是付款方式枚举，还要支持平台账户、挂账单状态、后续对账
- 新增退款方式区分规则，收款方式与退款方式分开建模，避免同名收退款混用
- 新增结账分支：
  - 提前退房
  - 应离未离
  - 半天加收
  - 全天加收
  - 手工加收
- 智能收银、扫码枪、预授权、退款回流等必须有真实接口闭环，不允许只做演示按钮

### 4.5 报表体系
- 将当前占位的 `Reports` 升级为真实报表中心
- 新增前台交班报表
- 新增付款明细报表
- 新增消费明细报表
- 新增管理层报表：
  - `CW02`
  - `CW10`
  - `经理01`
  - `经营01`
- 明确“固化报表”依赖夜审完成后生成，不允许把当天未夜审数据当作正式口径

### 4.6 设置中心
- 新增设置模块，包含：
  - 酒店基础信息
  - 班次配置，且时间段不可重叠
  - 零售商品/赔偿物品及分类管理，保留未来库存开关扩展位
  - 用户账号管理：新增、修改、禁用、重置密码、首次登录改密
  - 钟点房配置：按时长建模并绑定可售房型
- 零售商品、赔偿物品若未来接库存，模型设计需预留库存扩展位，不要重做一套结构

### 4.7 OTA 与硬件对接
- 新增渠道对接模型与流程：渠道、渠道订单、OTA 外部单号、搬单、直连同步、修改单人工同步提示
- 新增智能收银集成蓝图：扫码枪、预授权、退款回流、设备状态
- 新增门锁发卡/退卡接口蓝图
- 新增身份证阅读器对接蓝图，支持读取证件填充入住信息
- 渠道、硬件、夜审、报表相关逻辑必须统一落到独立 `services/`，前端通过真实接口联动，不允许只做演示按钮

---

## 五、未来接口与模型方向

以下是后续新增能力应在 `AGENTS.md` 中明确的模型与接口方向。它们是未来扩展蓝图，不代表当前代码库已经全部落地。

### 认证域
- `User`
- `Role`
- `AuthSession`

### 营业日域
- `BusinessDate`
- `NightAuditLog`

### 接待变更域
- `RoomLock`
- `StayExtension`
- `RoomChangeRecord`
- `StayPriceAdjustment`

### 账务域
- `CashierOrder`
- `AccountReceivableAccount`
- `FinanceReversal`
- `FinanceAdjustment`

### 设置域
- `Shift`
- `RetailCategory`
- `RetailItem`
- `HourlyRoomRule`

### 渠道域
- `Channel`
- `ChannelOrder`
- `ChannelSyncLog`

### 硬件域
- `DoorCardTask`
- `IdentityReadRecord`
- `PaymentDeviceLog`

### 接口约束
- API 前缀仍统一为 `/api/`
- 新增模块遵循 REST 风格，不改变现有技术栈与中文错误消息要求
- 报表、夜审、渠道同步、硬件通信的状态查询接口要有明确的可观察字段，避免前端靠轮询猜状态

---

## 六、代码质量

### 不要做的事
- 不要引入新的 UI 库（antd、chakra 等），MUI 覆盖所有需求
- 不要引入 CSS 框架（Tailwind、Bootstrap），用 sx prop
- 不要把业务逻辑写在组件的 JSX 里，抽成函数
- 不要在前端硬编码后端 URL（除了 `api.js` 的 `baseURL`）
- 不要用 `any` 类型的变通写法绕过校验
- 不要在 Python 里用 `print` 调试，用 `logging`
- 不要直接修改 `theme.js` 的色板，除非设计系统变更
- 不要创建空的占位文件或无意义的 `index.js` 导出
- 不要把“已有真实能力”再次包装成未来任务，文档更新时必须先核对当前实现
- 不要把现付账、房帐、AR 账、挂账单混成一个模糊概念，模型和页面边界必须清楚

### 提交前检查
- 后端：`uvicorn main:app` 能正常启动
- 前端：`npm run build` 无报错
- 新增 API 端点必须在 `api.js` 同步添加对应函数
- 新增页面必须在 `App.jsx` 注册路由 + `MainLayout.jsx` 添加导航
- 数据库模型变更后确认 `Base.metadata.create_all` 能正确建表

### 未来模块验收清单
- 夜审验收：营业日推进、住净转住脏、No Show 自动失效、固化报表生成口径一致
- 登录权限验收：不同角色只能访问授权菜单与接口；登出后会话失效
- 续住/换房验收：房价、账务、房态、房卡流程联动正确，历史记录可追溯
- 现付账验收：非住店消费可独立开单、收款、退款、查询，不污染房帐
- 挂账/AR 验收：挂账退房后房间释放，客单进入已挂账列表，可后续补结账
- 报表验收：交班、付款明细、消费明细、`CW02`、`CW10` 与原始流水汇总一致
- 设置验收：班次不允许重叠；账号角色、钟点房规则、零售商品配置可生效到业务流程
- OTA/硬件验收：未对接时有清晰降级路径；已对接时订单/发卡/读证/收银能走真实接口闭环

---

## 七、工具
- 目前已经安装 playwright MCP 和 chrome devtools MCP，可进行随时调用，例如前端设计
- 目前已经安装 context7 MCP，可随时查看大量产品文档
- 你可以自由写需要的 skills

---
> Source: [woshieeee123321/HotelManagement](https://github.com/woshieeee123321/HotelManagement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-01 -->
