## cscan

> **cscan** — 企业级分布式网络资产扫描平台

# CLAUDE.md
## 一、项目架构

### 1.1 项目概述

**cscan** — 企业级分布式网络资产扫描平台

### 1.2 技术栈

| 层级 | 技术 | 版本 |
|------|------|------|
| 后端框架 | Go + go-zero 微服务框架 | Go 1.25+ / go-zero v1.7.3 |
| 前端框架 | Vue 3 + Vite + Element Plus | Vue 3.4 / Vite 5 / Element Plus 2.4 |
| 数据库 | MongoDB + Redis | MongoDB 6 / Redis 7 |
| RPC 通信 | gRPC + Protobuf | grpc v1.76 |
| 扫描引擎 | ProjectDiscovery + Nmap/Masscan | nuclei v3.6 / httpx v1.7 / naabu v2.3 / subfinder v2.11 |
| 截图引擎 | Chromedp (Chrome 无头浏览器) | chromedp v0.14 |
| 任务调度 | robfig/cron + Redis Sorted Set | cron/v3 |
| 认证 | JWT (golang-jwt/v4) | jwt/v4 v4.5 |
| 前端状态管理 | Pinia | v2.1 |
| 国际化 | vue-i18n | v11 |
| 图表 | ECharts | v5.4 |
| CSS 预处理 | SCSS (modern-compiler API) | sass v1.97 |
| 测试 (Go) | testify + gopter (属性测试) | testify v1.11 / gopter v0.2 |
| 测试 (前端) | Vitest + happy-dom + fast-check | vitest v4 |

### 1.3 系统架构图

```
[Browser] → [Vue 3 Frontend (web/) :3000]
                  │
            [Vite Proxy /api → :8888]
                  │
           [HTTP API (api/) - go-zero REST :8888]
                  │
      ┌───────────┼────────────┐
      │           │            │
[gRPC RPC     [MongoDB]    [Redis]
 (rpc/) :9000]    │            │
                  │      ┌─────┴──────────┐
                  │  [Sorted Set     [Pub/Sub
                  │   任务队列]     cscan:cron:execute]
                  │      │               │
              [Scheduler (scheduler/)]───┘
                  │
         [Worker nodes (worker/)]  ← 通过 Install Key 认证，WebSocket 长连接
                  │
         [Scanner modules (scanner/)]
                  │
    ┌─────┬──────┼──────┬────────┬──────┐
  Naabu  Nmap  Httpx  Subfinder Nuclei Chromedp
```

### 1.4 核心架构模式

- **多租户隔离**: 所有数据按 `workspace_id` 过滤；MongoDB 集合命名 `{workspaceId}_{entity}`（如 `default_asset`、`ws1_tasks`）
- **任务流**: MainTask → 按目标数量分批 (batchSize=50) → SubTasks → 推入 Redis Sorted Set 队列 → Worker 拉取执行
- **孤儿恢复**: 后台协程每 5 分钟检查 STARTED 状态超过 30 分钟未更新的任务，重置为 PENDING 并重新入队
- **定时调度**: Redis Pub/Sub 频道 `cscan:cron:execute` 触发定时扫描，基于 `robfig/cron/v3`
- **Worker 通信**: Install Key 认证注册 → WebSocket (`/api/v1/worker/ws`) 保持长连接 → 心跳上报 → REST 接口回传结果

---

## 二、项目模块划分

### 2.1 文件与文件夹布局

```
cscan/
├── api/                          # HTTP API 服务（主入口）
│   ├── cscan.go                  # API 服务入口点
│   ├── etc/cscan.yaml            # API 配置（端口8888, JWT, MongoDB, Redis, RPC）
│   └── internal/
│       ├── config/config.go      # 配置结构体定义
│       ├── handler/              # 路由处理器（按资源域分子包）
│       │   ├── routes.go         # 统一路由注册（4 层认证级别）
│       │   ├── asset/            # 资产管理 Handler
│       │   ├── task/             # 任务管理 Handler
│       │   ├── vul/              # 漏洞管理 Handler
│       │   ├── worker/           # Worker 管理 Handler
│       │   ├── fingerprint/      # 指纹管理 Handler
│       │   ├── poc/              # POC 管理 Handler
│       │   ├── onlineapi/        # 在线 API 搜索 Handler
│       │   ├── user/             # 用户管理 Handler
│       │   ├── workspace/        # 工作空间 Handler
│       │   ├── organization/     # 组织管理 Handler
│       │   ├── blacklist/        # 黑名单 Handler
│       │   ├── dirscan/          # 目录扫描 Handler
│       │   ├── subdomain/        # 子域名字典 Handler
│       │   ├── subfinder/        # Subfinder 配置 Handler
│       │   ├── notify/           # 通知配置 Handler
│       │   ├── report/           # 报告 Handler
│       │   └── ai/               # AI POC 生成 Handler
│       ├── logic/                # 业务逻辑层（平铺，{动作}{实体}logic.go）
│       ├── middleware/           # 中间件（JWT/WorkerAuth/ConsoleAuth）
│       ├── svc/                  # 服务上下文（DI 容器 + 服务实现）
│       │   ├── servicecontext.go # ServiceContext 核心结构体
│       │   ├── scanresult_service.go
│       │   ├── history_service.go
│       │   └── sync/             # 同步服务（模板/指纹/POC 同步）
│       └── types/                # 请求/响应类型定义
├── rpc/                          # gRPC 内部服务
│   └── task/
│       ├── task.go               # RPC 服务入口（端口 9000）
│       ├── task.proto            # Protobuf 定义
│       ├── pb/                   # 生成的 pb 代码
│       ├── taskservice/          # 服务实现
│       ├── client/               # 客户端封装
│       └── etc/task.yaml         # RPC 配置
├── model/                        # MongoDB 数据模型（30+ 模型文件）
│   ├── asset.go                  # 资产模型（按 workspace 分集合）
│   ├── task.go                   # 主任务/子任务模型
│   ├── vul.go                    # 漏洞模型
│   ├── user.go                   # 用户模型（全局集合）
│   ├── workspace.go              # 工作空间模型
│   ├── fingerprint.go            # 指纹模型
│   ├── scantemplate.go           # 扫描模板
│   ├── indexes.go                # MongoDB 索引定义
│   ├── base.go                   # 基础类型
│   └── errors.go                 # 模型层错误定义
├── pkg/                          # 共享工具包
│   ├── xerr/                     # 业务错误码体系
│   │   ├── errcode.go            # 错误码常量 + 消息映射
│   │   ├── errors.go             # CodeError 结构体 + 工厂函数
│   │   └── scan_errors.go        # 扫描相关错误
│   ├── response/response.go      # 统一 HTTP 响应封装
│   ├── cache/                    # 缓存工具
│   ├── circuitbreaker/           # 熔断器
│   ├── httpclient/               # HTTP 客户端封装
│   ├── notify/                   # 多渠道通知发送
│   ├── retry/                    # 重试机制
│   ├── risk/                     # 风险等级计算
│   └── utils/                    # 通用工具函数
├── scheduler/                    # 任务调度器（Redis 队列 + Cron）
├── scanner/                      # 扫描模块（Naabu/Nmap/Httpx/Subfinder/Nuclei/Dnsx）
├── worker/                       # 分布式 Worker 进程
├── cmd/
│   └── worker/main.go            # Worker 命令行入口
├── onlineapi/                    # 在线资产搜索（FOFA/Hunter/Quake）
├── screenshot/                   # Web 截图服务（Chromedp）
├── tools/                        # 辅助工具脚本
├── poc/                          # POC 定义文件
├── docker/                       # Docker 配置
│   ├── Dockerfile.api            # API 镜像
│   ├── Dockerfile.rpc            # RPC 镜像
│   ├── Dockerfile.worker         # Worker 镜像
│   ├── mongo-init.js             # MongoDB 初始化脚本
│   └── entrypoint.sh             # 容器启动脚本
├── web/                          # Vue.js 前端
│   ├── src/
│   │   ├── main.js               # 应用入口
│   │   ├── App.vue               # 根组件
│   │   ├── router/index.js       # 路由配置（History 模式 + 懒加载）
│   │   ├── stores/               # Pinia 状态管理
│   │   │   ├── user.js           # 用户信息、token、登录/登出
│   │   │   ├── workspace.js      # 当前工作空间 ID、列表
│   │   │   ├── theme.js          # 主题模式（light/dark/system）
│   │   │   ├── locale.js         # 语言设置
│   │   │   └── onlineSearch.js   # 在线搜索状态
│   │   ├── api/                  # HTTP 请求封装
│   │   │   ├── request.js        # axios 实例（baseURL=/api/v1, 自动注入 Token + WorkspaceId）
│   │   │   └── {domain}.js       # 按业务域分文件（asset/task/worker/poc 等）
│   │   ├── views/                # 页面视图（PascalCase.vue）
│   │   ├── components/           # 可复用组件
│   │   │   └── asset/            # 资产相关子组件
│   │   ├── layouts/MainLayout.vue # 主框架布局
│   │   ├── i18n/                 # 国际化（zh-CN / en-US）
│   │   ├── utils/                # 工具函数
│   │   └── styles/index.css      # 全局样式
│   ├── vite.config.js            # Vite 配置（代理、分包、SCSS）
│   └── package.json
├── docker-compose.yaml           # 生产环境全栈部署
├── docker-compose.dev.yaml       # 本地开发依赖（仅 MongoDB + Redis）
├── docker-compose-worker.yaml    # 独立 Worker 部署
├── .github/workflows/            # CI/CD（Docker 镜像构建推送）
├── go.mod / go.sum               # Go 模块定义
├── VERSION                       # 版本号文件
├── cscan.sh / cscan.bat          # 一键启动脚本
└── README.md / README_EN.md      # 项目文档
```

### 2.2 业务模块清单

| 模块 | 后端 Handler | 前端视图 | 说明 |
|------|-------------|---------|------|
| 资产管理 | `handler/asset/` | `AssetManagement.vue` + `components/asset/` | 端口/站点/域名/IP/截图/历史/标签 |
| 任务管理 | `handler/task/` | `Task.vue`, `TaskCreate.vue`, `CronTask.vue` | 创建/分批/暂停/恢复/停止/重试/定时 |
| 漏洞管理 | `handler/vul/` | `VulnerabilityManagement.vue` | 漏洞列表/详情/统计 |
| Worker 管理 | `handler/worker/` | `Worker.vue`, `WorkerLogs.vue`, `WorkerConsole.vue` | 注册/心跳/WebSocket/日志流/控制台 |
| 指纹管理 | `handler/fingerprint/` | `Fingerprint.vue` | 指纹 CRUD/分类/同步/主动指纹 |
| POC 管理 | `handler/poc/` | `Poc.vue` | 自定义 POC/Nuclei 模板/AI 生成 |
| 在线搜索 | `handler/onlineapi/` | `OnlineSearch.vue` | FOFA/Hunter/Quake API 聚合 |
| 目录扫描 | `handler/dirscan/` | `DirectoryManagement.vue` | 字典管理/扫描结果 |
| 用户管理 | `handler/user/` | `Settings.vue (tab=user)` | CRUD/登录/密码重置 |
| 工作空间 | `handler/workspace/` | `Settings.vue (tab=workspace)` | 工作空间 CRUD |
| 组织管理 | `handler/organization/` | `Settings.vue (tab=organization)` | 组织 CRUD/状态切换 |
| 黑名单 | `handler/blacklist/` | `Blacklist.vue` | 黑名单规则配置 |
| 通知 | `handler/notify/` | Settings 页面内 | 通知配置/主题/高危过滤器 |
| 报告 | `handler/report/` | `Report.vue` | 报告详情/导出 |
| 扫描模板 | `handler/task/` | `ScanTemplate.vue` | 扫描配置模板管理 |

---

## 三、代码风格与规范

### 3.1 Go 命名约定

| 元素 | 规范 | 示例 |
|------|------|------|
| 文件名 | lowercase_underscore | `scanresult_service.go`, `asset_history.go` |
| 包名 | 小写单词 | `model`, `svc`, `handler`, `xerr` |
| 结构体/类型 | PascalCase | `AssetModel`, `ScanResultService`, `CodeError` |
| 导出函数 | PascalCase | `GetAsset()`, `NewAssetModel()` |
| 未导出函数 | camelCase | `parseResult()`, `unauthorized()` |
| 常量 | PascalCase | `ServerError`, `UserNotFound` |
| Context Key | 具名类型 `ContextKey` | `UserIdKey`, `WorkspaceIdKey` |
| Handler 函数 | `{Entity}{Action}Handler` | `AssetListHandler`, `WorkerHeartbeatHandler` |
| Logic 文件 | `{动作}{实体}logic.go` | `loginlogic.go`, `tasklogic.go` |

### 3.2 Vue 前端命名约定

| 元素 | 规范 | 示例 |
|------|------|------|
| 视图页面 | PascalCase.vue | `AssetManagement.vue`, `TaskCreate.vue` |
| 通用组件 | PascalCase.vue | `ScanWorkflow.vue`, `LanguageSwitcher.vue` |
| 内容视图组件 | PascalCase + `View` 后缀 | `SiteView.vue`, `VulView.vue` |
| 标签页组件 | PascalCase + `Tab` 后缀 | `AssetInventoryTab.vue` |
| 布局组件 | PascalCase + `Layout` 后缀 | `MainLayout.vue` |
| API 文件 | camelCase.js | `asset.js`, `crontask.js` |
| Store 文件 | camelCase.js | `workspace.js`, `onlineSearch.js` |
| 工具函数文件 | camelCase.js | `screenshot.js`, `performance.js` |

### 3.3 Import 规则

**Go — 三组空行分隔**：
```go
import (
    // 1. 标准库
    "context"
    "time"

    // 2. 内部包
    "cscan/model"
    "cscan/pkg/xerr"
    "cscan/api/internal/middleware"

    // 3. 第三方包
    "go.mongodb.org/mongo-driver/bson"
    "github.com/zeromicro/go-zero/core/logx"
)
```

**Vue — 使用 `@/` 别名**：
```js
import request from '@/api/request'
import { useUserStore } from '@/stores/user'
import { useWorkspaceStore } from '@/stores/workspace'
```

### 3.4 依赖注入

**ServiceContext 作为根 DI 容器**（`api/internal/svc/servicecontext.go`）：

```go
// 工厂函数创建，所有依赖通过构造函数注入
svcCtx := svc.NewServiceContext(config)

// 多租户工厂方法 — 按 workspaceId 动态获取模型
assetModel := svcCtx.GetAssetModel(workspaceId)    // 集合: {workspaceId}_asset
taskModel := svcCtx.GetMainTaskModel(workspaceId)  // 集合: {workspaceId}_tasks
vulModel := svcCtx.GetVulModel(workspaceId)        // 集合: {workspaceId}_vul

// 全局模型 — 不分 workspace
userModel := svcCtx.UserModel
workspaceModel := svcCtx.WorkspaceModel
```

**服务构造器模式**：
```go
type ScanResultService struct {
    db *mongo.Database
}

func NewScanResultService(db *mongo.Database) *ScanResultService {
    return &ScanResultService{db: db}
}
```

**MongoDB 模型构造器模式**：
```go
// 多租户模型
func NewAssetModel(db *mongo.Database, workspaceId string) *AssetModel {
    coll := db.Collection(workspaceId + "_asset")
    return &AssetModel{coll: coll}
}

// 全局模型
func NewUserModel(db *mongo.Database) *UserModel {
    return &UserModel{db: db}
}
```

### 3.5 日志规范

- **必须**使用 go-zero 的 `logx`（`logx.Infof / logx.Errorf / logx.Info`）
- **禁止**在业务逻辑中使用 `fmt.Println`
- 日志前缀标签约定：`[模块名]` 格式，如 `[OrphanedTaskRecovery]`
- 配置中 Log Level 通过 `api/etc/cscan.yaml` 的 `Log.Level` 控制

### 3.6 异常处理

**错误码体系**（`pkg/xerr/errcode.go`）：

| 范围 | 类型 | 示例 |
|------|------|------|
| 0 | 成功 | `OK = 0` |
| 400-500 | HTTP 标准码 | `ParamError=400`, `Unauthorized=401`, `Forbidden=403`, `NotFound=404`, `ServerError=500` |
| 10001-10099 | 用户错误 | `UserNotFound=10001`, `UserPasswordError=10002`, `UserDisabled=10003` |
| 10101-10199 | 任务错误 | `TaskNotFound=10101`, `TaskStatusError=10103` |
| 10201-10299 | 工作空间错误 | `WorkspaceNotFound=10201` |
| 10301-10399 | 资产错误 | `AssetNotFound=10301` |
| 10401-10699 | 其他业务错误 | `VulNotFound=10401`, `FingerprintNotFound=10501`, `PocNotFound=10601` |

**错误使用方式**：
```go
// 创建业务错误
xerr.NewCodeError(xerr.UserNotFound)          // 使用预定义消息
xerr.NewCodeErrorMsg(xerr.ParamError, "自定义消息") // 自定义消息
xerr.NewParamError("字段不能为空")               // 参数错误快捷函数
xerr.NewServerError("")                        // 服务器错误快捷函数
xerr.NewNotFoundError("")                      // 资源不存在快捷函数
```

**统一响应封装**（`pkg/response/response.go`）：
```go
// 统一响应结构: { "code": 0, "msg": "success", "data": {...} }
response.Success(w, data)                     // 成功
response.Error(w, err)                        // 自动识别 CodeError 或普通 error
response.ErrorWithCode(w, xerr.NotFound, "")  // 指定错误码
response.ParamError(w, "参数校验失败")          // 参数错误
```

### 3.7 参数校验

- 前端请求拦截器自动注入 `Authorization: Bearer <token>` 和 `X-Workspace-Id` Header
- 后端通过 `middleware.GetWorkspaceId(ctx)` 从 Context 获取 workspaceId
- 后端通过 `middleware.GetUserId(ctx)` / `GetUsername(ctx)` / `GetRole(ctx)` 获取用户信息
- 管理员权限检查：`middleware.RequireAdmin(next)` 中间件，校验 `role == "admin"`

### 3.8 Struct Tag 规范

所有 MongoDB 模型**必须同时包含 `bson` 和 `json` 标签**：
```go
type Asset struct {
    Id         primitive.ObjectID `bson:"_id,omitempty" json:"id"`
    Host       string             `bson:"host" json:"host"`
    Port       int                `bson:"port" json:"port"`
    Labels     []string           `bson:"labels,omitempty" json:"labels"`
    CreateTime time.Time          `bson:"create_time" json:"createTime"`
    UpdateTime time.Time          `bson:"update_time" json:"updateTime"`
    RiskScore  float64            `bson:"risk_score,omitempty" json:"riskScore,omitempty"`
}
```

注意 bson 使用 `snake_case`，json 使用 `camelCase`。

### 3.9 API 路由规范

- 所有端点前缀：`/api/v1/*`
- HTTP 方法：除健康检查 (`GET /health`) 和 Worker WebSocket (`GET /api/v1/worker/ws`) 外，**所有业务接口均为 POST**
- 四层认证级别：
  1. **无认证**：`/health`、`/api/v1/login`、Worker 下载/验证/WebSocket
  2. **Worker Key 认证**（`WorkerAuthMiddleware`）：Worker 任务上报、心跳、配置拉取
  3. **JWT 认证**（`AuthMiddleware`）：所有用户操作路由（60+ 个端点）
  4. **JWT + Console 认证**（`ConsoleAuthMiddleware`）：Worker 控制台文件/终端/审计

### 3.10 Redis 使用规范

| 用途 | Key 模式 | 数据结构 |
|------|---------|---------|
| 任务队列 | `cscan:task:queue` | Sorted Set（score = 时间戳，优先级调度） |
| 任务状态 | `cscan:task:status:{taskId}` | String |
| 任务信息 | `cscan:task:info:{taskId}` | String (JSON) |
| 处理中集合 | `cscan:task:processing` | Set |
| 定时触发 | `cscan:cron:execute` | Pub/Sub Channel |
| Worker Install Key | 自定义 Key | String |
| Worker 心跳 | 自定义 Key | String |

### 3.11 前端其他规范

- **组件风格**：始终使用 `<script setup>` Composition API
- **UI 组件**：Element Plus 全量引入，图标全局注册（可直接在模板中使用 `<Edit />`）
- **样式**：SCSS，使用 Element Plus CSS 变量实现暗色模式
- **国际化**：模板中使用 `$t('key')`，语言文件位于 `web/src/i18n/locales/`（zh-CN / en-US）
- **状态管理**：Pinia stores 位于 `web/src/stores/`
- **路由懒加载**：所有组件通过 `lazyLoad()` 包装，chunk 加载失败自动刷新
- **路由 meta 字段**：`requiresAuth`（默认 true）、`title`（中文标题）、`icon`（Element Plus 图标名）、`hidden`（隐藏菜单）
- **纯 JavaScript 项目**：无 TypeScript，所有文件为 `.js` / `.vue`

---

## 四、测试与质量

### 4.1 Go 单元测试

测试文件位于 `api/internal/svc/` 和 `api/internal/handler/` 目录。

**表格驱动测试模式**：
```go
testCases := []struct {
    name     string
    input    string
    expected int
}{
    {"empty", "", 0},
    {"valid", "test", 4},
}
for _, tc := range testCases {
    t.Run(tc.name, func(t *testing.T) { /* 断言 */ })
}
```

**属性测试模式（gopter）**：
```go
parameters := gopter.DefaultTestParameters()
parameters.MinSuccessfulTests = 100
properties := gopter.NewProperties(parameters)

properties.Property("属性描述", prop.ForAll(
    func(workspaceId string, port int) bool {
        if workspaceId == "" || port <= 0 || port > 65535 {
            return true  // guard clause 跳过无效输入
        }
        return someInvariant(workspaceId, port)
    },
    gen.AlphaString().SuchThat(func(s string) bool { return len(s) > 0 }),
    gen.IntRange(1, 65535),
))
properties.TestingRun(t)
```

已实现的属性测试覆盖：资产-结果关联、目录扫描计数、分页正确性、排序默认值、合并时字段保留（labels/memo/color_tag）、跨视图一致性等。

### 4.2 Go 集成测试

- `handler/worker/security_test.go` — 使用 `httptest` + `miniredis` 的中间件安全测试
- `handler/asset/scanresult_integration_test.go` — 扫描结果集成测试
- `handler/api_compatibility_test.go` — API 兼容性测试

### 4.3 前端测试

框架已配置（Vitest + happy-dom + fast-check），配置在 `web/vite.config.js` 中：
```js
test: {
    globals: true,
    environment: 'happy-dom',
    setupFiles: ['./src/tests/setup.js'],
    coverage: { provider: 'v8', reporter: ['text', 'json', 'html'] }
}
```

---

## 五、项目构建、测试与运行

### 5.1 环境与配置

**本地开发依赖启动**：
```bash
docker-compose -f docker-compose.dev.yaml up -d   # 启动 MongoDB(:27017) + Redis(:6379)
```

**后端服务启动（按顺序）**：
```bash
go run rpc/task/task.go -f rpc/task/etc/task.yaml     # 1. 启动 gRPC 服务 (:9000)
go run api/cscan.go -f api/etc/cscan.yaml              # 2. 启动 HTTP API (:8888)
go run cmd/worker/main.go -k <install_key> -s http://localhost:8888  # 3. 启动 Worker
```

**前端启动**：
```bash
cd web && npm install && npm run dev    # 开发服务器 (:3000)，代理 /api → :8888
```

**关键配置文件**：
| 文件 | 说明 |
|------|------|
| `api/etc/cscan.yaml` | API 配置：端口 8888，超时 300s，MaxBytes 100MB，JWT 24h |
| `rpc/task/etc/task.yaml` | RPC 配置：端口 9000 |
| `docker/cscan-api.yaml` | 容器内 API 配置 |
| `docker/task.yaml` | 容器内 RPC 配置 |
| `docker/mongo-init.js` | MongoDB 初始化脚本 |

**MongoDB 连接池**：MaxPoolSize=100, MinPoolSize=10, ConnectTimeout=10s

**Redis 连接池**：PoolSize=100, MinIdleConns=10, MaxRetries=3

### 5.2 构建命令

```bash
# Go 后端
go build -o cscan ./api/cscan.go       # 构建 API 服务
go build -o worker ./worker/            # 构建 Worker
go mod download && go mod tidy          # 依赖管理

# Vue 前端
cd web
npm install                             # 安装依赖
npm run build                           # 生产构建（terser 压缩，移除 console）
npm run dev                             # 开发服务器
```

### 5.3 测试命令

```bash
# Go 测试
go test ./...                                        # 运行所有测试
go test -v ./api/internal/svc/ -run TestFunctionName # 指定测试函数
go test -v -run TestProperty1 ./api/internal/svc/    # 属性测试
go test -cover ./...                                  # 覆盖率
go test -race ./...                                   # 竞态检测

# Vue 前端测试
cd web
npm run test                                          # vitest
npx vitest run src/tests/MyComponent.test.js          # 单个测试文件
npm run test:coverage                                 # 覆盖率
```

### 5.4 生产部署

```bash
# 全栈部署
docker-compose up -d

# 独立 Worker 部署
docker-compose -f docker-compose-worker.yaml up -d

# 一键启动脚本
./cscan.sh        # Linux/macOS
.\cscan.bat       # Windows
```

访问 `https://ip:3443`，默认账号 `admin / 123456`

---

## 六、Git 工作流程

### 6.1 分支策略

- 主分支：`main`
- CI/CD 触发：push 到 `main` 或 `master`（忽略 `*.md`、`LICENSE`、`.gitignore` 变更）

### 6.2 CI/CD

`.github/workflows/build-images.yml` — 4 个并行 Job：

| Job | 镜像 | Dockerfile |
|-----|------|-----------|
| `build-api` | `cscan-api` | `docker/Dockerfile.api` |
| `build-rpc` | `cscan-rpc` | `docker/Dockerfile.rpc` |
| `build-web` | `cscan-web` | `web/Dockerfile` |
| `build-worker` | `cscan-worker` | `docker/Dockerfile.worker` |

所有镜像推送至阿里云容器镜像服务（`registry.cn-hangzhou.aliyuncs.com/txf7`），平台 `linux/amd64`，使用 GitHub Actions cache。

### 6.3 .gitignore 要点

```
web/node_modules/    # 前端依赖
web/dist/            # 前端构建产物
*.exe                # 编译产物
.idea/ .vscode/      # IDE 配置
output/              # 输出目录
screenshot/          # 截图缓存
.env                 # 环境变量
```

---

## 七、文档目录

### 7.1 文档存储规范

| 文件 | 位置 | 说明 |
|------|------|------|
| `README.md` | 根目录 | 中文项目说明、功能特性、快速开始 |
| `README_EN.md` | 根目录 | 英文项目说明 |
| `CLAUDE.md` | 根目录 | AI 编码助手指导文件（已 .gitignore） |
| `VERSION` | 根目录 | 当前版本号（V2.19） |
| `LICENSE` | 根目录 | MIT 许可证 |

项目无独立 `docs/` 目录，无 lint 配置文件（ESLint/Prettier/golangci-lint），无 Makefile。

---

## 八、关键规则

1. **工作空间隔离**: 所有数据查询**必须**按 `workspace_id` 过滤，通过 `svcCtx.GetXxxModel(workspaceId)` 获取对应集合
2. **保留用户数据**: 更新资产时**必须**保留 `labels`、`memo`、`color_tag`、布尔标志（`isNew`/`isUpdated`）、风险字段、任务追踪字段
3. **API 稳定性**: 禁止修改已有端点路径或 HTTP 方法
4. **错误码**: 使用 `pkg/xerr/errcode.go` 中的错误码常量
5. **日志**: 使用 go-zero `logx` — 禁止业务逻辑中使用 `fmt.Println`
6. **旧数据兼容**: 优雅处理缺少新字段的旧数据（fallback 逻辑，如 `Version==0` 自动赋值 `1`，`ScanTime` 零值回退 `CreateTime`）
7. **类型安全**: 禁止使用 `as any`、`@ts-ignore` 等类型抑制
8. **MongoDB**: 始终使用 `primitive.ObjectID` 作为 `_id`；必须包含 `create_time` 和 `update_time` 字段
9. **Struct Tag**: 所有模型必须同时声明 `bson` + `json` 标签（bson 用 snake_case，json 用 camelCase）
10. **响应格式**: 统一使用 `pkg/response` 封装，返回 `{ "code": 0, "msg": "success", "data": {...} }` 结构

---
> Source: [tangxiaofeng7/cscan](https://github.com/tangxiaofeng7/cscan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
