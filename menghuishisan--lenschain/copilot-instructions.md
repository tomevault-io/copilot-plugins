## lenschain

> > **文档驱动开发（Documentation-Driven Development）**

# 链镜平台（LensChain）— 开发规范与项目指南

> **文档驱动开发（Documentation-Driven Development）**
> 一切开发以文档和规范为引擎。开发任何功能前，必须先阅读对应模块的设计文档。
> 文档是唯一真相源（Single Source of Truth），代码必须与文档保持一致。
> 如发现文档与实际需求有冲突，**先更新文档，再改代码，绝不允许反过来**。

> 本文件是 AI 辅助开发的核心指引，所有代码生成和修改必须遵守以下规范。
> 详细业务需求请查阅 `docs/modules/` 下各模块文档。

---

## 一、项目概述

链镜是一个区块链综合教学平台，集教学、实验实践、CTF竞赛三位一体，支持多链生态、多层次学生、多学校混合部署。

- **8个业务模块：** 用户与认证、学校与租户管理、课程与教学、实验环境、CTF竞赛、评测与成绩、通知与消息、系统管理与监控
- **4种用户角色：** 超级管理员、学校管理员（教师兼任）、教师、学生
- **多租户隔离：** 以 `school_id` 为租户标识，业务数据严格隔离

---

## 二、技术栈与版本

| 层级 | 技术 | 版本要求 |
|------|------|----------|
| 后端 | Go + Gin | Go 1.22+ |
| 前端 | React + Next.js (App Router) | Next.js 14+, React 18+ |
| 数据库 | PostgreSQL | 15+ |
| 缓存 | Redis | 7+ |
| 消息队列 | NATS | 2.10+ |
| 对象存储 | MinIO (S3兼容) | — |
| 容器编排 | Docker + Kubernetes | — |
| 仿真引擎 | SimEngine Core (Go微服务) + 场景算法容器 + 前端渲染器 (TypeScript) | — |
| 前端样式 | Tailwind CSS | 3.4+ |
| UI 组件 | shadcn/ui | — |
| 图标库 | Lucide React | — |
| 状态管理 | Zustand | 4+ |
| 数据请求 | TanStack Query (React Query) | 5+ |

---

## 三、仓库结构（Monorepo）

```
LensChain/
├── backend/                # Go 后端服务
├── frontend/               # Next.js 前端应用
├── sim-engine/             # 可视化仿真引擎（SimEngine）
├── deploy/                 # 部署与运维配置
├── docs/                   # 项目文档（已完成）
├── scripts/                # 工具脚本（数据库迁移、种子数据等）
├── CLAUDE.md               # 本文件
└── README.md
```

---

### 3.1 backend/ — Go 后端（标准分层架构）

```
backend/
├── cmd/
│   └── server/
│       └── main.go              # 程序入口
├── internal/
│   ├── router/                  # 路由注册层（路径与中间件绑定，不含处理逻辑）
│   │   ├── router.go            # 总路由入口，初始化 Gin Engine，挂载各模块路由
│   │   ├── auth.go              # 模块01 路由组
│   │   ├── school.go            # 模块02 路由组
│   │   ├── course.go            # 模块03 路由组
│   │   ├── experiment.go        # 模块04 路由组
│   │   ├── ctf.go               # 模块05 路由组
│   │   ├── grade.go             # 模块06 路由组
│   │   ├── notification.go      # 模块07 路由组
│   │   └── system.go            # 模块08 路由组
│   ├── handler/                 # HTTP 处理层（按模块分子目录，按功能域分文件）
│   │   ├── auth/                # 模块01
│   │   │   ├── login.go         # 登录、登出、Token刷新
│   │   │   ├── user.go          # 用户管理、导入、个人中心
│   │   │   └── security.go      # SSO、安全策略
│   │   ├── school/              # 模块02（单文件）
│   │   │   └── school.go        # 入驻申请、学校管理、SSO配置
│   │   ├── course/              # 模块03
│   │   │   ├── course.go        # 课程CRUD、章节、课时、选课
│   │   │   ├── assignment.go    # 作业管理、提交、批改
│   │   │   └── discussion.go    # 讨论区、评价、公告、统计
│   │   ├── experiment/          # 模块04
│   │   │   ├── template.go      # 模板、镜像、场景、标签
│   │   │   ├── instance.go      # 实例生命周期、检查点、快照
│   │   │   └── group.go         # 分组、组内通信、教师监控
│   │   ├── ctf/                 # 模块05
│   │   │   ├── competition.go   # 竞赛管理、题目、报名
│   │   │   ├── battle.go        # 攻防赛回合、攻击、防守
│   │   │   └── environment.go   # 题目环境、队伍链、监控
│   │   ├── grade/               # 模块06（单文件）
│   │   │   └── grade.go         # 学期、成绩审核、申诉、预警
│   │   ├── notification/        # 模块07（单文件）
│   │   │   └── notification.go  # 通知、公告、模板、偏好
│   │   └── system/              # 模块08（单文件）
│   │       └── system.go        # 审计、配置、告警、备份
│   ├── service/                 # 业务逻辑层（同样按模块分子目录）
│   │   ├── auth/
│   │   ├── school/
│   │   ├── course/
│   │   ├── experiment/
│   │   ├── ctf/
│   │   ├── grade/
│   │   ├── notification/
│   │   └── system/
│   ├── repository/              # 数据访问层（同样按模块分子目录）
│   │   ├── auth/
│   │   ├── school/
│   │   ├── course/
│   │   ├── experiment/
│   │   ├── ctf/
│   │   ├── grade/
│   │   ├── notification/
│   │   └── system/
│   ├── model/                   # 数据模型
│   │   ├── entity/              # 数据库表映射结构体（按模块分文件）
│   │   │   ├── user.go          # users, user_profiles 等
│   │   │   ├── school.go        # schools, school_applications 等
│   │   │   ├── course.go        # courses, chapters, lessons 等
│   │   │   ├── experiment.go    # experiment_templates, instances 等
│   │   │   ├── ctf.go           # competitions, challenges 等
│   │   │   ├── grade.go         # semesters, grade_reviews 等
│   │   │   ├── notification.go  # notifications, templates 等
│   │   │   └── system.go        # system_configs, alert_rules 等
│   │   ├── dto/                 # 请求/响应 DTO（按模块分文件）
│   │   └── enum/                # 枚举常量定义（按模块分文件）
│   ├── middleware/              # 中间件（JWT鉴权、RBAC权限、多租户注入、日志、限流）
│   └── pkg/                    # 内部公共包（不对外导出）
│       ├── snowflake/           # 雪花ID生成器
│       ├── response/            # 统一响应封装
│       ├── errcode/             # 业务错误码定义
│       └── validator/           # 自定义校验器
├── pkg/                         # 可导出公共包（供 sim-engine 等其他服务使用）
├── configs/                     # 配置文件（yaml）
├── migrations/                  # 数据库迁移文件（SQL）
└── go.mod
```

**分层调用规则（严格单向依赖）：**
```
main.go → router → handler → service → repository → model
              ↓        ↓         ↓          ↓
          middleware   pkg       pkg        pkg
```
- **router** 只负责路径注册和中间件绑定，不含任何处理逻辑
- **handler** 不得直接调用 repository，不得包含业务逻辑
- **service** 不得引用 `*gin.Context` 或任何 HTTP 相关类型
- **repository** 不得包含业务逻辑，不得调用 service
- **model** 不依赖任何其他层
- **跨模块 service 调用** 必须通过接口（interface）解耦，不直接引用具体实现

**文件拆分原则：**
- 单文件应尽量保持精简，**建议控制在 500-800 行以内**
- 超过 800 行时必须评估是否按功能域拆分；如果拆分会明显破坏内聚性，可保留但需确保职责单一、结构清晰、便于维护
- handler/service/repository 三层统一按模块建子目录
- 小模块（端点 ≤ 30）子目录内放单文件即可，大模块按功能域拆分为多个文件
- 一个文件对应一个功能域（如"作业管理"、"讨论区"），而非一个文件对应一整个模块

---

### 3.2 frontend/ — Next.js 前端（App Router）

```
frontend/
├── src/
│   ├── app/                     # 路由页面（按角色分组，按功能分子目录）
│   │   ├── (auth)/              # 认证相关页面（登录等）
│   │   │   └── login/
│   │   ├── (student)/           # 学生端页面
│   │   │   ├── courses/
│   │   │   │   ├── page.tsx             # 课程列表
│   │   │   │   └── [id]/
│   │   │   │       ├── page.tsx         # 课程详情
│   │   │   │       ├── lessons/
│   │   │   │       └── assignments/
│   │   │   ├── experiments/
│   │   │   └── ctf/
│   │   ├── (teacher)/           # 教师端页面
│   │   │   ├── courses/
│   │   │   ├── experiments/
│   │   │   └── ctf/
│   │   ├── (admin)/             # 学校管理员页面
│   │   │   ├── users/
│   │   │   ├── school/
│   │   │   └── grades/
│   │   └── (super)/             # 超级管理员页面
│   │       ├── schools/
│   │       ├── system/
│   │       └── statistics/
│   ├── components/
│   │   ├── ui/                  # 基础 UI 组件（shadcn/ui，不含业务逻辑）
│   │   │   ├── Button.tsx
│   │   │   ├── Modal.tsx
│   │   │   ├── Table.tsx
│   │   │   └── ...
│   │   └── business/            # 业务组件（按功能域分文件，不按模块建子目录）
│   │       ├── CourseCard.tsx
│   │       ├── ExperimentPanel.tsx
│   │       ├── AlertBadge.tsx
│   │       └── ...
│   ├── hooks/                   # 自定义 Hooks（按功能域分文件）
│   │   ├── useAuth.ts
│   │   ├── useCourses.ts
│   │   ├── useExperiments.ts
│   │   └── ...
│   ├── lib/                     # 工具函数与基础设施
│   │   ├── api-client.ts        # HTTP 客户端封装（baseURL、Token注入、响应拦截、错误处理）
│   │   ├── utils.ts             # 通用工具函数
│   │   └── format.ts            # 格式化函数（日期、文件大小等）
│   ├── services/                # API 调用层（按模块分文件，调用 api-client）
│   │   ├── auth.ts              # 模块01 认证接口
│   │   ├── school.ts            # 模块02 学校接口
│   │   ├── course.ts            # 模块03 课程接口
│   │   ├── experiment.ts        # 模块04 实验接口
│   │   ├── ctf.ts               # 模块05 竞赛接口
│   │   ├── grade.ts             # 模块06 成绩接口
│   │   ├── notification.ts      # 模块07 通知接口
│   │   └── system.ts            # 模块08 系统接口
│   ├── stores/                  # 状态管理（Zustand，按业务域分文件）
│   │   ├── authStore.ts
│   │   ├── courseStore.ts
│   │   └── ...
│   └── types/                   # TypeScript 类型定义（按模块分文件）
│       ├── auth.ts
│       ├── course.ts
│       ├── experiment.ts
│       └── ...
├── public/                      # 静态资源
├── package.json
├── tsconfig.json
├── tailwind.config.ts           # Tailwind 主题配置
└── next.config.js
```

**前端调用链（严格单向）：**
```
页面(app/) → 业务组件(components/business/) → hooks → services → lib/api-client → 后端API
                    ↓
            UI组件(components/ui/)
```
- **页面（app/）** 组合业务组件和 hooks，不直接调用 services
- **业务组件（components/business/）** 通过 props 接收数据或通过 hooks 获取数据
- **hooks** 调用 services 获取数据，管理加载/错误状态
- **services** 调用 `lib/api-client` 发请求，定义请求/响应类型
- **lib/api-client** 封装 HTTP 客户端（Token 注入、响应解包、401 跳转登录）
- **组件不得直接调用 `fetch` 或 `axios`**

**文件拆分原则：**
- 每个组件文件应保持合理体量，**建议控制在 500-800 行以内**
- 超过 800 行时优先拆分为子组件或抽离 hooks / 工具函数；若暂不拆分，必须保证单一职责和清晰分段
- services 按模块分文件，单个文件过大时按功能域拆分（如 `course.ts` → `course.ts` + `courseAssignment.ts`）
- hooks 按功能域分文件，不按模块建子目录
- 业务组件扁平放置在 `components/business/` 下，通过文件名区分所属功能

---

### 3.3 sim-engine/ — 可视化仿真引擎

```
sim-engine/
├── core/                        # SimEngine Core 微服务（Go）
│   ├── cmd/
│   ├── internal/
│   │   ├── scene/               # 场景调度器
│   │   ├── link/                # 联动引擎
│   │   ├── session/             # 会话管理
│   │   ├── simcore/             # 仿真内核（时钟、事件、状态、快照）
│   │   └── collector/           # 数据采集（混合实验用）
│   └── go.mod
├── renderers/                   # 前端领域渲染器（TypeScript，独立 npm 包）
│   ├── shared/                  # 渲染器公共基础（Canvas工具、动画调度、交互管理）
│   ├── node-network/            # 节点网络渲染器
│   ├── consensus/               # 共识过程渲染器
│   ├── data-structure/          # 数据结构渲染器
│   ├── transaction/             # 交易流程渲染器
│   ├── cryptography/            # 密码学渲染器
│   ├── smart-contract/          # 智能合约渲染器
│   ├── attack-security/         # 攻击安全渲染器
│   ├── economic/                # 经济模型渲染器
│   ├── package.json             # 作为 npm workspace 包，供 frontend 引用
│   └── tsconfig.json
├── scenarios/                   # 内置仿真场景算法容器（每个场景独立目录）
│   ├── pow-mining/              # PoW挖矿仿真
│   ├── pbft-consensus/          # PBFT共识仿真
│   └── ...                      # 共43个内置场景
├── sdk/                         # SimScenario SDK（教师自定义场景开发包）
└── proto/                       # gRPC Proto 定义（Core 与场景容器间通信）
```

**sim-engine 与 frontend 的关系：**
- `sim-engine/renderers/` 是 TypeScript 包，通过 npm workspace 被 `frontend/` 引用
- 渲染器只负责 Canvas/SVG 绘制和交互采集，不包含业务逻辑
- 渲染器通过 WebSocket 与 SimEngine Core 通信，不直接调用后端 API

---

### 3.4 deploy/ — 部署与运维

```
deploy/
├── docker/                      # 平台服务 Dockerfile
│   ├── backend.Dockerfile
│   ├── frontend.Dockerfile
│   ├── sim-engine.Dockerfile
│   └── scenario.Dockerfile      # 场景算法容器基础镜像
├── images/                      # 实验环境镜像定义
│   ├── chain-nodes/             # 区块链节点镜像
│   │   ├── ethereum/            # 以太坊节点
│   │   ├── fabric/              # Hyperledger Fabric
│   │   ├── chainmaker/          # 长安链
│   │   └── fisco-bcos/          # FISCO BCOS
│   ├── middleware/              # 中间件镜像（Truffle、Hardhat 等）
│   ├── tools/                   # 开发工具镜像（IDE、调试器等）
│   └── base/                    # 基础环境镜像（Ubuntu、Alpine 等）
├── docker-compose/
│   ├── docker-compose.dev.yml   # 本地开发环境
│   └── docker-compose.prod.yml  # 生产环境
├── k8s/                         # Kubernetes 配置
│   ├── base/                    # 基础资源定义
│   └── overlays/                # 环境差异化配置（dev/staging/prod）
└── ci/                          # CI/CD 配置
    └── .github/workflows/
```

**目录职责边界：**
- `deploy/docker/` — 仅存放平台自身服务（backend/frontend/sim-engine）的 Dockerfile
- `deploy/images/` — 仅存放实验环境使用的区块链节点、中间件、工具等镜像定义
- 两者不得混放，平台服务镜像和实验环境镜像严格分离

---

### 3.5 docs/ — 项目文档（已完成）

```
docs/
├── 00-项目总览与文档规范.md
├── 项目功能总览.md               # 207项功能索引
├── API接口总览.md                # 375个端点索引
├── 数据库表总览.md               # 96张表 + 42个Redis Key 索引
├── standards/
│   ├── API规范.md               # RESTful API 设计规范
│   └── 数据库规范.md             # 数据库命名与设计规范
└── modules/                     # 8个模块，每模块5个标准文档
    ├── 01-用户与认证/
    ├── 02-学校与租户管理/
    ├── 03-课程与教学/
    ├── 04-实验环境/              # 额外含仿真引擎设计、实验类型配置
    ├── 05-CTF竞赛/
    ├── 06-评测与成绩/
    ├── 07-通知与消息/
    └── 08-系统管理与监控/
```

---

## 四、编码规范

### 4.1 Go 后端规范

**文件命名：**
- 全部小写 + 下划线：`auth_handler.go`、`user_service.go`、`school_repository.go`
- 测试文件：`xxx_test.go`

**包命名：**
- 全部小写，不用下划线：`handler`、`service`、`repository`

**结构体与接口命名：**
- 结构体：大驼峰 `UserService`、`CourseHandler`
- 接口：大驼峰 + 动词/名词 `UserRepository`、`NotificationSender`
- 不加 `I` 前缀

**函数命名：**
- 导出函数大驼峰：`CreateCourse`、`GetUserByID`
- 私有函数小驼峰：`buildQuery`、`validateInput`
- Handler 方法统一格式：`Create`、`Update`、`Delete`、`Get`、`List`

**变量命名：**
- 小驼峰：`userID`、`schoolName`、`pageSize`
- 常量全大写 + 下划线：`StatusActive`、`RoleTeacher`
- 枚举值定义在 `model/enum/` 下

**错误处理：**
- 不忽略 error，必须处理或向上传递
- 业务错误使用 `errcode` 包定义的错误码
- 不使用 `panic`，仅在程序初始化阶段允许

**其他：**
- 所有代码必须通过 `gofmt` 格式化
- 使用 `golangci-lint` 做静态检查
- 注释用中文，公共函数必须有注释

---

### 4.2 TypeScript 前端规范

**文件命名：**
- 组件文件：大驼峰 `CourseCard.tsx`、`LoginForm.tsx`
- 非组件文件：小驼峰 `useAuth.ts`、`formatDate.ts`
- 类型文件：小驼峰 `course.ts`、`user.ts`
- 页面文件：遵循 Next.js 约定 `page.tsx`、`layout.tsx`

**组件命名：**
- 函数组件大驼峰：`export function CourseCard() {}`
- Props 类型：`组件名Props`，如 `CourseCardProps`

**变量命名：**
- 小驼峰：`userName`、`courseList`
- 常量全大写：`API_BASE_URL`、`MAX_PAGE_SIZE`
- 布尔值用 `is/has/should` 前缀：`isLoading`、`hasPermission`

**样式规范（Tailwind CSS）：**
- 使用 Tailwind CSS 原子类，不写自定义 CSS 文件（除非 Tailwind 无法覆盖的场景）
- 基础 UI 组件使用 shadcn/ui，放在 `components/ui/` 下
- 业务组件在 `components/business/` 下封装，内部使用 shadcn/ui + Tailwind 组合
- 颜色、间距、圆角等统一使用 Tailwind 主题配置（`tailwind.config.ts`），不硬编码数值
- 响应式布局使用 Tailwind 断点前缀：`sm:`、`md:`、`lg:`、`xl:`
- 暗色模式预留：组件使用 `dark:` 前缀适配（当前阶段可不实现，但不要写死浅色值）

**图标规范：**
- 统一使用 Lucide React 图标库（`lucide-react`）
- **禁止使用 Emoji 表情符号作为图标**，所有图标必须来自图标库
- 图标大小通过 `size` 属性或 Tailwind 类控制，保持视觉一致

**API 调用：**
- 统一在 `services/` 目录封装，组件不直接调用 `fetch`
- 每个模块一个 service 文件：`authService.ts`、`courseService.ts`

**状态管理：**
- 使用 Zustand，每个业务域一个 store
- 服务端状态优先使用 TanStack Query（React Query）

**其他：**
- 使用 ESLint + Prettier 统一格式
- 严格模式 TypeScript，不允许 `any`（除非有充分理由并加注释）
- 注释用中文

---

### 4.3 SQL / 数据库规范

> 详细规范见 `docs/standards/数据库规范.md`

- **表名：** 小写蛇形复数 `users`、`login_logs`
- **字段名：** 小写蛇形单数 `user_id`、`created_at`
- **主键：** 统一 `id BIGINT`，雪花算法生成
- **必备字段：** `id`、`created_at`、`updated_at`
- **软删除：** `deleted_at TIMESTAMP NULL`，查询自动过滤
- **唯一索引：** 加 `WHERE deleted_at IS NULL` 部分索引
- **枚举：** 用 `SMALLINT`，不用数据库 ENUM 类型
- **布尔：** `BOOLEAN` 类型，字段名 `is_` 前缀
- **JSON：** 使用 `JSONB` 类型
- **多租户：** 业务表必须有 `school_id`，中间件自动注入过滤
- **索引：** 每张表不超过 6 个索引
- **迁移文件命名：** `{序号}_{描述}.up.sql` / `{序号}_{描述}.down.sql`

---

### 4.4 API 接口规范

> 详细规范见 `docs/standards/API规范.md`

- **基础路径：** `/api/v1`
- **URL 风格：** 小写 + 连字符，名词复数 `/api/v1/courses/:id/students`
- **嵌套层级：** 最多 2 层
- **非 CRUD 操作：** 动词子路径 `/users/:id/reset-password`
- **认证：** `Authorization: Bearer <token>`
- **统一响应格式：**
  ```json
  {
    "code": 200,
    "message": "success",
    "data": {},
    "timestamp": "2026-04-09T10:00:00Z"
  }
  ```
- **分页参数：** `page`（从1开始）、`page_size`（默认20，最大100）
- **错误码：** 5位数字 `400xx`参数 / `401xx`认证 / `403xx`权限 / `404xx`不存在 / `409xx`冲突 / `500xx`服务端
- **时间格式：** ISO 8601，后端存储 UTC，前端本地化显示
- **ID 传输：** 雪花ID在 JSON 中以字符串传输，避免前端精度丢失

---

## 五、Git 规范

### 5.1 分支策略（Git Flow）

| 分支 | 用途 | 命名 |
|------|------|------|
| `main` | 生产分支，始终可部署 | `main` |
| `develop` | 开发主线，集成所有功能 | `develop` |
| `feature/*` | 功能开发分支 | `feature/模块-功能`，如 `feature/auth-login` |
| `release/*` | 发布准备分支 | `release/v1.0.0` |
| `hotfix/*` | 紧急修复分支 | `hotfix/fix-login-bug` |

### 5.2 Commit 规范

格式：`<type>(<scope>): <description>`

| type | 说明 |
|------|------|
| `feat` | 新功能 |
| `fix` | Bug 修复 |
| `docs` | 文档变更 |
| `style` | 代码格式（不影响逻辑） |
| `refactor` | 重构（非新功能、非修复） |
| `test` | 测试相关 |
| `chore` | 构建/工具/依赖变更 |

scope 使用模块名：`auth`、`school`、`course`、`experiment`、`ctf`、`grade`、`notification`、`system`

示例：
```
feat(auth): 实现手机号密码登录接口
fix(course): 修复邀请码重复生成问题
docs(experiment): 更新实验模板API文档
```

### 5.3 开发流程

1. 从 `develop` 创建 `feature/*` 分支
2. 开发完成后提交 PR 到 `develop`
3. Code Review 通过后合并
4. 发布时从 `develop` 创建 `release/*` 分支
5. 测试通过后合并到 `main` 并打 Tag

---

## 六、文档驱动开发（DDD）

> **这是本项目最核心的开发原则，优先级高于一切。**

### 6.1 核心原则

1. **文档先于代码。** 没有文档的功能不允许开发，没有确认的文档不允许进入编码阶段。
2. **文档是唯一真相。** 功能范围、表结构、接口定义、页面设计、验收标准全部以文档为准。
3. **代码服从文档。** 代码实现必须与文档描述一致，不得自行添加文档中未定义的功能或字段。
4. **冲突时文档优先。** 如发现文档与实际需求有冲突，**先更新文档，再改代码**。绝不允许代码偏离文档后再补文档。
5. **变更必须同步。** 任何需求变更必须先反映到文档中，然后才能修改代码。

### 6.2 每个模块的 5 份标准文档

| 文件 | 开发时用途 |
|------|-----------|
| `01-功能需求说明.md` | 明确功能边界——做什么、不做什么、业务规则是什么 |
| `02-数据库设计.md` | 编写迁移文件和 model 层的唯一依据 |
| `03-API接口设计.md` | 编写 handler/service 层和前端 services 层的唯一依据 |
| `04-前端页面设计.md` | 编写前端页面和组件的唯一依据 |
| `05-验收标准.md` | 功能完成后的逐条验证清单 |

### 6.3 开发一个模块的标准流程

```
阅读文档 → 数据库迁移 → 后端开发 → 前端开发 → 联调验收
```

1. **阅读文档：** 先完整阅读该模块的 5 个文档（功能需求 → 数据库设计 → API接口 → 前端页面 → 验收标准）
2. **数据库迁移：** 严格根据 `02-数据库设计.md` 编写迁移文件，不得自行增减字段
3. **后端开发：** model → repository → service → handler，逐层实现，接口签名严格对照 `03-API接口设计.md`
4. **前端开发：** 严格根据 `04-前端页面设计.md` 实现页面布局和交互
5. **联调验收：** 根据 `05-验收标准.md` 逐条 Given-When-Then 验证

### 6.4 开发顺序

严格按依赖关系从底层到上层：

```
第一阶段（基础层）：模块01 → 模块02
第二阶段（业务层）：模块03 → 模块04 → 模块05
第三阶段（聚合层）：模块06 → 模块07 → 模块08
```

### 6.5 跨模块调用规则

- **模块06/07/08 是聚合层**，可以读取其他模块的数据库表（只读）
- **模块07 提供内部通知接口** `POST /internal/send-event`，其他模块通过此接口发送通知
- **模块间不得循环依赖**，依赖方向只能是：聚合层 → 业务层 → 基础层
- 跨模块数据访问均为**只读查询**，不跨模块写入

---

## 七、目录职责边界（严格执行）

> **每个目录有且仅有一个职责，不允许跨职责存放代码。违反此规则的代码不予合并。**

### 7.1 后端目录职责

| 目录 | 职责 | 允许 | 禁止 |
|------|------|------|------|
| `router/` | 路由注册、路径与中间件绑定 | 注册路由组、绑定 handler 方法 | 包含处理逻辑、参数校验 |
| `handler/{模块}/` | 接收HTTP请求、参数校验、调用service、返回响应 | 引用 `*gin.Context`、调用 service | 直接操作数据库、包含业务逻辑 |
| `service/{模块}/` | 核心业务逻辑、业务规则校验、事务编排 | 调用 repository、通过接口调用其他模块 service | 引用 `*gin.Context`、直接写SQL |
| `repository/{模块}/` | 数据库 CRUD 操作、SQL 查询构建 | 操作数据库、返回 entity | 包含业务判断逻辑、调用 service |
| `model/entity/` | 数据库表映射结构体（按模块分文件） | 定义字段、标签 | 包含方法逻辑（简单转换方法除外） |
| `model/dto/` | 请求/响应数据传输对象（按模块分文件） | 定义字段、校验标签 | 引用 entity、包含业务逻辑 |
| `model/enum/` | 枚举常量定义（按模块分文件） | 定义常量、提供 Text() 方法 | 引用其他包 |
| `middleware/` | HTTP 中间件 | JWT鉴权、RBAC、租户注入、日志 | 包含业务逻辑 |
| `internal/pkg/` | 内部公共工具 | 雪花ID、响应封装、错误码、校验器 | 引用 handler/service/repository |
| `configs/` | 配置文件 | YAML 配置 | 代码文件 |
| `migrations/` | 数据库迁移 | SQL 迁移文件 | Go 代码 |

### 7.2 前端目录职责

| 目录 | 职责 | 允许 | 禁止 |
|------|------|------|------|
| `app/` | 路由页面 | 页面组件、布局、组合 hooks 和业务组件 | 直接调用 services、包含可复用逻辑 |
| `components/ui/` | 基础 UI 组件（shadcn/ui） | 纯展示组件、样式封装 | 业务逻辑、API 调用、引用 services |
| `components/business/` | 业务组件 | 组合 ui 组件、通过 props/hooks 获取数据 | 直接调用 services |
| `hooks/` | 自定义 Hooks | 状态逻辑、副作用封装、调用 services | 直接渲染 UI |
| `lib/api-client.ts` | HTTP 客户端封装 | baseURL、Token注入、响应拦截、错误处理 | 业务逻辑、具体接口定义 |
| `lib/` | 工具函数 | 纯函数、格式化、加密 | 引用 React、引用组件 |
| `services/` | API 调用层（按模块分文件） | 封装后端接口、请求/响应类型、调用 api-client | 业务逻辑、UI 代码、直接 fetch |
| `stores/` | 全局状态管理 | Zustand store 定义 | 直接调用 API（通过 hooks 层） |
| `types/` | TypeScript 类型（按模块分文件） | 类型/接口定义 | 运行时代码 |

### 7.3 仓库顶层目录职责

| 目录 | 职责 | 禁止 |
|------|------|------|
| `backend/` | Go 后端服务代码 | 前端代码、部署配置 |
| `frontend/` | Next.js 前端应用代码 | 后端代码、部署配置 |
| `sim-engine/` | 仿真引擎（Core + 渲染器 + 场景 + SDK） | 平台业务代码 |
| `deploy/docker/` | 平台服务的 Dockerfile | 实验环境镜像 |
| `deploy/images/` | 实验环境镜像（区块链节点/中间件/工具） | 平台服务 Dockerfile |
| `deploy/k8s/` | K8s 部署配置 | 应用代码 |
| `deploy/ci/` | CI/CD 流水线配置 | 应用代码 |
| `docs/` | 项目设计文档 | 代码文件 |
| `scripts/` | 工具脚本（初始化、种子数据等） | 业务代码 |

## 八、命名约定速查

| 场景 | 规则 | 示例 |
|------|------|------|
| Go 文件 | 小写下划线 | `auth_handler.go` |
| Go 结构体 | 大驼峰 | `UserService` |
| Go 私有函数 | 小驼峰 | `buildQuery` |
| Go 常量 | 大驼峰 | `StatusActive` |
| TS 组件文件 | 大驼峰 | `CourseCard.tsx` |
| TS 工具文件 | 小驼峰 | `formatDate.ts` |
| TS 变量 | 小驼峰 | `userName` |
| TS 常量 | 全大写下划线 | `API_BASE_URL` |
| CSS 类名 | Tailwind 原子类 | `className="flex items-center gap-2"` |
| 数据库表 | 小写蛇形复数 | `login_logs` |
| 数据库字段 | 小写蛇形单数 | `user_id` |
| API 路径 | 小写连字符 | `/api/v1/alert-rules` |
| Git 分支 | 小写连字符 | `feature/auth-login` |
| Git Commit | type(scope): desc | `feat(auth): 实现登录` |
| 迁移文件 | 序号_描述 | `001_create_users.up.sql` |
| 镜像目录 | 小写连字符 | `deploy/images/chain-nodes/ethereum/` |

---

## 九、常用命令

```bash
# 后端
cd backend
go run cmd/server/main.go          # 启动后端服务
go test ./...                       # 运行全部测试
golangci-lint run                   # 代码检查

# 前端
cd frontend
npm run dev                         # 启动开发服务器
npm run build                       # 构建生产版本
npm run lint                        # 代码检查

# 数据库迁移
cd backend
go run cmd/migrate/main.go up       # 执行迁移
go run cmd/migrate/main.go down     # 回滚迁移

# Docker
cd deploy/docker-compose
docker compose -f docker-compose.dev.yml up -d   # 启动本地开发环境
```

---

## 十、注释规范

> **所有代码文件必须包含中文功能注释，便于团队成员快速了解项目。**

### 9.1 Go 后端注释

```go
// auth_handler.go
// 用户认证模块 — HTTP 处理层
// 负责登录、登出、Token刷新、SSO回调等认证相关接口

package handler

// AuthHandler 认证处理器
// 处理所有 /api/v1/auth 路径下的请求
type AuthHandler struct { ... }

// Login 手机号+密码登录
// POST /api/v1/auth/login
// 验证手机号和密码，返回 Access Token 和 Refresh Token
func (h *AuthHandler) Login(c *gin.Context) { ... }
```

### 9.2 TypeScript 前端注释

```tsx
// CourseCard.tsx
// 课程卡片组件
// 用于课程列表页展示单个课程的封面、标题、教师、进度等信息

/**
 * 课程卡片属性
 */
interface CourseCardProps { ... }

/**
 * 课程卡片组件
 * 展示课程基本信息，点击跳转到课程详情页
 */
export function CourseCard({ course }: CourseCardProps) { ... }
```

### 9.3 注释要求

- **文件头注释（必须）：** 每个文件顶部说明文件名、所属模块、职责描述
- **公共函数/组件注释（必须）：** 说明功能、参数含义、对应的 API 路径（如适用）
- **复杂业务逻辑注释（必须）：** 非显而易见的逻辑必须注释说明原因
- **私有函数注释（建议）：** 简要说明用途
- **注释语言：** 统一使用中文
- **禁止无意义注释：** 如 `// 获取用户` 后面跟 `getUser()`，这种不需要

---

## 十一、红线规则（不可违反）

> 以下规则为项目硬性约束，违反任何一条都可能导致系统性问题。

### 10.1 文档驱动

1. **不要跳过文档直接写代码。** 每个模块都有完整的5份设计文档，开发前必须阅读。
2. **代码必须与文档一致。** 不得自行添加文档中未定义的字段、接口或功能。
3. **需求变更先改文档。** 先更新文档，确认无误后再改代码。

### 10.2 目录职责

4. **严格遵守目录职责边界。** handler 不碰数据库，service 不碰 HTTP，repository 不含业务逻辑。
5. **代码放在正确的目录。** 平台 Dockerfile 放 `deploy/docker/`，实验镜像放 `deploy/images/`，不得混放。

### 10.3 数据规范

6. **ID 用雪花算法生成，不用自增。** 所有表的主键统一为 `BIGINT` 雪花ID。
7. **前端 ID 用字符串。** 雪花ID超过 JS 安全整数范围，JSON 中必须用字符串传输。
8. **多租户隔离不能遗漏。** 涉及学校数据的查询必须带 `school_id` 过滤。
9. **软删除不能忘。** 查询默认过滤 `deleted_at IS NULL`，唯一索引加部分索引。

### 10.4 安全与审计

10. **审计日志不可删改。** `login_logs`、`operation_logs`、`instance_operation_logs` 只插入不更新不删除。
11. **敏感配置要脱敏。** `is_sensitive=true` 的配置值在 API 响应中返回 `******`。

### 10.5 代码质量

12. **不要在 service 层引用 HTTP 上下文。** service 层只接收普通参数，不依赖 `*gin.Context`。
13. **不允许使用 Emoji 作为图标。** 前端图标统一使用 Lucide React 图标库。
14. **不允许使用 `any` 类型。** TypeScript 严格模式，确需时必须加注释说明原因。
15. **所有代码文件必须有中文功能注释。** 文件头注释和公共函数注释为必须项。
16. **避免单文件过大。** 建议控制在 500-800 行以内；超过 800 行时原则上应按功能域拆分。若因强内聚暂不拆分，必须确保职责单一、结构清晰、注释充分、后续可继续演进。
17. **跨模块 service 调用必须通过接口解耦。** 不得直接 import 其他模块的具体 service 实现。
18. **不要重复造轮子。** 能复用已有代码、组件、工具函数的，必须复用，不得重复实现相同功能。具体要求：
    - **后端公共逻辑：** 分页查询、软删除过滤、多租户注入等通用逻辑封装在 `internal/pkg/` 或 middleware 中，各模块直接调用，不得各自重复实现
    - **前端通用封装：** 表格分页、表单校验、权限判断等通用 hooks 写一次全局复用；请求/响应/错误处理统一走 `lib/api-client.ts`
    - **UI 组件复用：** shadcn/ui 已有的组件直接使用；通用业务组件（确认弹窗、空状态、加载骨架屏等）封装在 `components/business/` 后全局复用，不得各页面重复编写
    - **第三方库不重复引入：** 同一功能只允许引入一个库（如日期处理、HTTP 请求等），不允许多个库做同一件事
    - **例外：** sim-engine 中的可视化算法与仿真场景实现属于领域专用逻辑，按需独立编写，不受此规则约束

---
> Source: [menghuishisan/LensChain](https://github.com/menghuishisan/LensChain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
