## structforge

> StructForge 是一个全功能的AI工作流编排平台，支持多种AI大模型（开源、自建、公共API）和各种工作节点，提供可视化的流程设计界面。类似n8n，但专注于AI工作流场景。


# StructForge 项目开发规则

## 1. 项目目标与定位

### 1.1 项目目标

StructForge 是一个全功能的AI工作流编排平台，支持多种AI大模型（开源、自建、公共API）和各种工作节点，提供可视化的流程设计界面。类似n8n，但专注于AI工作流场景。

### 1.2 核心特性

- 多AI模型支持：支持OpenAI、Gemini、Ollama等各类AI模型
- 丰富的工作节点：触发、AI、数据处理、集成、控制、工具等节点
- 可视化编辑器：拖拽式工作流设计，直观易用
- 高性能执行：支持同步/异步执行，适合各种场景
- 安全可靠：完善的认证授权、数据加密、多租户隔离
- 易于扩展：插件化架构，支持自定义节点和模型

## 2. 技术架构

### 2.1 前端技术栈

- **框架**: Vue 3.x (Composition API)
- **语言**: TypeScript
- **构建工具**: Vite
- **状态管理**: Pinia
- **UI组件库**: Element Plus
- **工作流编辑器**: Vue Flow
- **HTTP客户端**: Axios
- **WebSocket**: Socket.io-client 或原生 WebSocket

### 2.2 后端技术栈

- **框架**: Go + Kratos微服务框架
- **协议**: gRPC + HTTP Gateway
- **数据库**: PostgreSQL (主数据库) + Redis (缓存/队列)
- **消息队列**: RabbitMQ (可选，用于异步任务)
- **服务发现**: Consul / etcd (Kratos内置支持)
- **配置中心**: Kratos配置系统
- **日志**: 统一使用 `backend/common/log` 日志系统
- **链路追踪**: OpenTelemetry (可选)

### 2.3 基础设施

- **容器化**: Docker + Docker Compose
- **CI/CD**: GitHub Actions / GitLab CI
- **监控**: Prometheus + Grafana (可选)
- **平台支持**: Windows、Linux

### 2.4 文档系统

- **API文档**: OpenAPI 3.0 + Swagger UI（集成到API Gateway）
- **节点说明**: 数据库存储 + API提供（集成到节点服务）
- **工作流说明**: Markdown文件 + API提供（集成到工作流服务）
- **独立文档站**: VitePress（可选，用于对外文档）

## 3. 项目结构

### 3.1 根目录结构

```text
StructForge/
├── frontend/                 # 前端项目
├── backend/                  # 后端项目（Go + Kratos）
│   ├── api/                  # API定义（Protobuf）
│   ├── apps/                 # 微服务应用
│   │   ├── gateway/          # API Gateway服务
│   │   ├── user/             # 用户服务
│   │   ├── workflow/         # 工作流服务
│   │   ├── ai/               # AI模型服务
│   │   ├── node/             # 节点服务
│   │   ├── tool/             # 工具服务
│   │   ├── scheduler/        # 调度服务
│   │   └── log/              # 日志服务
│   ├── common/               # 公共代码
│   ├── configs/              # 配置文件
│   ├── deploy/               # 部署相关
│   ├── script/               # 开发脚本
│   ├── third_party/          # 第三方依赖
│   ├── go.mod                # Go模块定义
│   └── Makefile              # Make命令
├── docs/                     # 项目文档
├── docker/                   # Docker相关配置
└── docker-compose.yml        # 本地开发环境配置
```

### 3.2 前端项目结构

```text
frontend/
├── src/
│   ├── api/                  # API接口定义
│   ├── assets/               # 资源文件
│   ├── components/           # 公共组件
│   │   ├── common/           # 通用组件
│   │   ├── workflow/         # 工作流相关组件
│   │   └── layout/           # 布局组件
│   ├── composables/          # Composition API组合函数
│   ├── router/               # 路由配置
│   ├── stores/               # Pinia状态管理
│   ├── utils/                # 工具函数
│   └── views/                # 页面视图
```

### 3.3 后端服务结构（Kratos规范）

```text
apps/service-name/
├── cmd/
│   └── service-name/          # 启动入口
│       ├── main.go
│       └── wire.go
└── internal/                 # 内部实现
    ├── biz/                  # 业务逻辑层（核心）
    │   ├── service-name.go   # 核心业务逻辑
    │   └── ...               # 其他业务逻辑文件
    ├── data/                 # 数据访问层
    │   ├── service-name.go   # 数据访问实现
    │   └── db.go             # 数据库连接
    ├── service/              # 服务层（gRPC实现）
    │   └── service-name.go   # gRPC服务实现
    ├── server/               # 服务器配置
    │   ├── grpc.go
    │   └── http.go
    └── conf/                 # 配置
        └── config.yaml       # 服务配置

# API定义在 backend/api/service-name/v1/ 目录
backend/api/
└── service-name/
    └── v1/
        └── service-name.proto
```

### 3.5 后端特殊目录说明

- **backend/apps/ai/internal/adapters/**: AI模型适配器实现目录
  - `openai/`: OpenAI适配器
  - `gemini/`: Gemini适配器
  - `ollama/`: Ollama适配器
  - `custom/`: 自定义模型适配器

- **backend/apps/node/internal/nodes/**: 节点实现目录
  - `trigger/`: 触发节点（webhook.go, timer.go, manual.go）
  - `ai/`: AI节点（text_generation.go, image_generation.go）
  - `data/`: 数据处理节点（transform.go, filter.go）
  - `integration/`: 集成节点（http.go, database.go）
  - `control/`: 控制节点（condition.go, loop.go）
  - `tool/`: 工具节点（script.go）

- **backend/common/**: 跨服务公共代码
  - `middleware/`: 中间件（auth, cors, logger, ratelimit）
  - `data/`: 数据访问公共代码（database, redis）
  - `utils/`: 工具函数（crypto, response, validator）
  - `log/`: **统一日志系统**（所有后端服务必须使用此日志系统，禁止使用其他日志库）

- **backend/apps/gateway/internal/handler/**: API Gateway HTTP处理器
  - HTTP请求处理
  - 路由转发
  - 认证授权

- **backend/api/**: 所有服务的Protobuf定义
  - `user/v1/user.proto`: 用户服务API定义
  - `workflow/v1/workflow.proto`: 工作流服务API定义
  - `ai/v1/ai.proto`: AI模型服务API定义
  - `node/v1/node.proto`: 节点服务API定义
  - `gateway/v1/gateway.proto`: Gateway服务API定义
  - `common/v1/common.proto`: 公共定义

### 3.4 微服务划分

- **apps/gateway**: API Gateway，统一HTTP入口，路由、认证、限流
- **apps/user**: 用户注册、登录、授权、权限管理、组织管理
- **apps/workflow**: 工作流定义、版本控制、执行引擎、状态管理
- **apps/ai**: AI模型统一接入、配置管理、调用封装、限流监控
- **apps/node**: 各种工作流节点实现、节点类型管理、参数验证
- **apps/tool**: 外部工具集成、工具配置管理、工具调用封装
- **apps/scheduler**: 定时任务调度、工作流定时执行、任务队列管理
- **apps/log**: 工作流执行日志收集、系统日志聚合、日志查询分析

## 4. 编程命名规范

### 4.1 服务命名

- **服务目录名**: 小写字母，无连字符
  - 示例: `user`, `workflow`, `ai`, `gateway`
- **服务标识**: 在代码中使用时可以用kebab-case或snake_case
  - 示例: `user-service`, `workflow-service`, `ai-service`

### 4.2 Go语言命名规范

- **包名**: 小写字母，无下划线，简短有意义
  - 示例: `user`, `workflow`, `ai`
- **文件名**: snake_case
  - 示例: `user.go`, `workflow_execution.go`, `ai_adapter.go`
- **结构体名**: PascalCase
  - 示例: `User`, `WorkflowExecution`, `AIModelAdapter`
- **函数/方法名**:
  - 公开函数/方法: PascalCase（首字母大写）
    - 示例: `GetUser()`, `ExecuteWorkflow()`, `CreateWorkflow()`
  - 私有函数/方法: camelCase（首字母小写）
    - 示例: `validateConfig()`, `parseRequest()`, `handleError()`
- **变量名**: camelCase（首字母小写）
  - 示例: `userID`, `workflowName`, `isActive`
- **常量**: PascalCase 或 UPPER_SNAKE_CASE
  - 示例: `MaxRetries`, `DEFAULT_TIMEOUT`, `MAX_EXECUTION_TIME`
- **接口名**: PascalCase，通常以 `er` 结尾或描述行为
  - 示例: `NodeExecutor`, `ModelAdapter`, `WorkflowRunner`
- **错误变量**: 以 `Err` 开头
  - 示例: `ErrUserNotFound`, `ErrInvalidWorkflow`

### 4.3 TypeScript/JavaScript命名规范

- **文件名**: camelCase
  - 示例: `userApi.ts`, `workflowEditor.vue`, `useAuth.ts`
- **类名**: PascalCase
  - 示例: `WorkflowEditor`, `NodeExecutor`, `AIModelClient`
- **函数/方法名**: camelCase
  - 示例: `getUser()`, `executeWorkflow()`, `validateNode()`
- **变量名**: camelCase
  - 示例: `userInfo`, `workflowList`, `isLoading`
- **常量**: UPPER_SNAKE_CASE
  - 示例: `MAX_RETRIES`, `DEFAULT_TIMEOUT`, `API_BASE_URL`
- **接口/类型名**: PascalCase
  - 示例: `User`, `Workflow`, `NodeConfig`
- **组件名**: PascalCase
  - 示例: `WorkflowEditor.vue`, `NodePalette.vue`

### 4.4 数据库命名规范

- **表名**: snake_case，复数形式
  - 示例: `users`, `workflows`, `workflow_executions`
- **字段名**: snake_case
  - 示例: `user_id`, `workflow_name`, `created_at`
- **索引名**: `idx_表名_字段名`
  - 示例: `idx_users_email`, `idx_workflows_user_id`

## 5. 架构原则与设计模式

### 5.1 核心设计理念

- **可扩展性**: 插件化架构，节点、AI模型、工具都采用插件化设计
- **高可用性**: 微服务架构，服务独立部署，故障隔离
- **易用性**: 可视化编辑，拖拽式工作流设计，降低使用门槛

### 5.2 设计模式

- **适配器模式**: AI模型通过适配器统一接口，新增模型只需实现适配器
- **策略模式**: 不同执行策略可插拔替换
- **工厂模式**: 节点和模型创建使用工厂模式
- **观察者模式**: 工作流执行状态通过WebSocket推送

### 5.3 分层架构（Kratos规范）

- **biz层（业务逻辑层）**: 核心业务逻辑，不依赖具体数据源
- **data层（数据访问层）**: 数据库操作、外部API调用
- **service层（服务层）**: gRPC服务实现，调用biz层

### 5.4 依赖关系规则

- **框架不应反向导入**: 框架层不应导入具体的业务实现
- **业务模块导入框架**: 业务模块应导入并使用框架提供的接口
- **服务间通信**: 通过gRPC进行服务间通信，避免直接数据库访问
- **公共代码**: 跨服务共享代码放在 `pkg/` 目录

## 6. 代码组织规范

### 6.1 Go代码组织

- 遵循Kratos框架的目录结构
- 业务逻辑放在 `biz` 层
- 数据访问放在 `data` 层
- 服务接口放在 `service` 层
- 公共代码放在 `pkg/` 目录

### 6.2 TypeScript代码组织

- 使用Composition API，避免Options API
- 公共组件放在 `components/common/`
- 业务组件放在对应的功能目录
- 组合函数（Composables）放在 `composables/`
- API接口统一放在 `api/` 目录
- 工具函数放在 `utils/` 目录

### 6.3 文件组织原则

- 一个文件一个主要功能/类
- 相关功能放在同一目录
- 避免过长的文件（建议不超过500行）
- 使用合理的目录层级（建议不超过4层）

## 7. 工作流执行引擎规范

### 7.1 执行模式

- **同步执行**: 适用于简单、快速的工作流（< 30秒），直接返回结果
- **异步执行**: 适用于复杂、耗时的任务，返回任务ID，通过WebSocket推送进度

### 7.2 节点执行流程

1. 参数验证（必填参数检查、参数类型验证、参数值范围检查）
2. 前置处理（输入数据转换、上下文准备、依赖检查）
3. 节点执行（调用节点执行器、超时控制、错误捕获）
4. 后置处理（结果验证、数据转换、日志记录）
5. 结果传递（序列化输出、传递给下游节点、更新执行上下文）

### 7.3 节点类型分类

- **触发节点**: Webhook, Timer, Manual, Event
- **AI节点**: TextGeneration, ImageGeneration, CodeGeneration, DataAnalysis
- **数据处理节点**: Transform, Filter, Aggregate, Sort
- **集成节点**: HTTP, Database, File, Email
- **控制节点**: Condition, Loop, Parallel, Delay
- **工具节点**: CodeExecutor, Script, ExternalTool

## 8. AI模型接入规范

### 8.1 统一接口设计

所有AI模型必须实现统一的接口：

```go
type AIModel interface {
    // 模型调用
    Call(ctx context.Context, request *ModelRequest) (*ModelResponse, error)
    // 获取模型配置
    GetConfig() *ModelConfig
    // 健康检查
    HealthCheck(ctx context.Context) error
}

// 模型请求结构
type ModelRequest struct {
    Model     string                 // 模型标识
    Prompt    string                 // 提示词
    Params    map[string]interface{} // 模型参数
    Context   map[string]interface{} // 上下文
}

// 模型响应结构
type ModelResponse struct {
    Content   string                 // 响应内容
    Metadata  map[string]interface{} // 元数据
    Usage     *UsageInfo             // 使用统计
    Error     error                  // 错误信息
}
```

### 8.2 适配器实现

- 每个AI模型通过适配器实现统一接口
- 适配器放在 `backend/apps/ai/internal/adapters/` 目录
- 新模型只需实现适配器接口即可接入

### 8.3 模型配置管理

- 模型配置存储在数据库
- 支持环境变量配置API密钥
- 支持模型调用限流和监控

### 8.4 适配器实现规范

- 每个适配器必须实现 `ModelAdapter` 接口
- 适配器放在 `backend/apps/ai/internal/adapters/{model-name}/` 目录
- 适配器文件命名: `adapter.go`
- 适配器配置通过环境变量或配置文件管理

## 9. 数据库设计规范

### 9.1 数据存储策略

- **工作流定义**: 使用PostgreSQL的JSONB类型存储nodes和edges
- **元数据**: 存储在关系表中（id, name, version等）
- **执行日志**: 实时日志存Redis，历史日志存PostgreSQL

### 9.2 核心数据表

- **用户相关**:
  - `users`: 用户表
  - `roles`: 角色表
  - `permissions`: 权限表
  - `user_roles`: 用户角色关联表
  - `role_permissions`: 角色权限关联表
  - `organizations`: 组织表
  - `user_organizations`: 用户组织关联表

- **工作流相关**:
  - `workflows`: 工作流定义表（JSONB存储nodes和edges）
  - `workflow_versions`: 工作流版本表
  - `workflow_executions`: 工作流执行记录表
  - `workflow_nodes`: 工作流节点定义表（可选，用于复杂查询）
  - `workflow_edges`: 工作流边定义表（可选）
  - `execution_logs`: 执行日志表

- **AI模型相关**:
  - `ai_models`: AI模型配置表
  - `model_configs`: 模型配置详情表
  - `model_usage_logs`: 模型使用日志表

- **节点相关**:
  - `node_types`: 节点类型表
  - `node_templates`: 节点模板表
  - `node_documentations`: 节点文档表（JSONB存储详细说明）

### 9.3 数据库迁移规范

- 迁移文件放在 `backend/deploy/migrations/` 或 `backend/script/migrations/` 目录
- 文件命名: `{序号}_{描述}.sql`
- 示例: `001_init_users.sql`, `002_init_workflows.sql`
- 使用事务确保迁移的原子性

## 10. 安全规范

### 10.1 认证授权

- 使用JWT Token进行无状态认证
- 实现Refresh Token刷新机制
- 基于RBAC的权限控制
- 服务间调用使用API Key认证

### 10.2 数据安全

- 敏感信息加密存储（API密钥等）
- 使用HTTPS/WSS进行传输加密
- 输入验证防止注入攻击
- 输出转义防止XSS攻击

### 10.3 资源隔离

- 支持多租户数据隔离
- 实现资源限制防止滥用
- API限流保护

## 11. 开发规范

### 11.1 代码质量

- **Go**: 遵循Go官方规范，使用gofmt格式化
- **TypeScript**: 使用ESLint + Prettier
- **提交规范**: 遵循Conventional Commits

### 11.2 测试要求

- 核心业务逻辑必须有单元测试
- 关键接口必须有集成测试
- 测试覆盖率要求：核心模块 > 80%

### 11.3 文档要求

- 公共API必须有注释
- 复杂业务逻辑必须有注释说明
- 新增功能需要更新相关文档

### 11.4 日志使用规范

**强制要求**：整个项目的 backend 后端系统必须统一使用 `backend/common/log` 日志系统。

#### 11.4.1 日志系统要求

- **统一日志库**: 所有后端服务（gateway、user、workflow、ai、node、tool、scheduler、log等）必须使用 `backend/common/log` 日志系统
- **禁止使用其他日志库**: 禁止使用 `fmt.Println`、`log.Println`、`logrus`、`zap` 等标准库或第三方日志库直接输出日志
- **禁止直接输出**: 禁止使用 `fmt.Printf`、`os.Stdout`、`os.Stderr` 等直接输出到控制台或文件

#### 11.4.2 日志使用方式

**正确示例**：
```go
import (
    "context"
    "StructForge/backend/common/log"
)

// 初始化日志系统（在main.go中）
func main() {
    config := log.DefaultConfig()
    config.ServiceName = "gateway"
    config.Level = log.InfoLevel
    
    logger, err := log.NewLogger(config)
    if err != nil {
        panic(err)
    }
    defer logger.Sync()
    
    log.SetGlobalLogger(logger)
}

// 使用日志
func handleRequest(ctx context.Context) {
    log.Info(ctx, "处理请求",
        log.String("method", "GET"),
        log.String("path", "/api/users"),
    )
    
    if err != nil {
        log.Error(ctx, "处理请求失败",
            log.ErrorField(err),
            log.String("path", "/api/users"),
        )
    }
}
```

**错误示例**（禁止）：
```go
// ❌ 禁止：使用fmt直接输出
fmt.Println("处理请求")
fmt.Printf("错误: %v\n", err)

// ❌ 禁止：使用标准库log
import "log"
log.Println("处理请求")
log.Printf("错误: %v\n", err)

// ❌ 禁止：使用其他日志库
import "github.com/sirupsen/logrus"
logrus.Info("处理请求")

// ❌ 禁止：直接输出到stderr
os.Stderr.WriteString("错误信息\n")
```

#### 11.4.3 日志级别使用

- **Debug**: 详细的调试信息，开发环境使用
- **Info**: 一般信息，记录正常业务流程
- **Warn**: 警告信息，需要注意但不影响运行
- **Error**: 错误信息，记录错误但不中断程序
- **Fatal**: 致命错误，记录后程序会退出

#### 11.4.4 日志字段规范

- 使用结构化字段，避免字符串拼接
- 使用类型安全的字段函数：`String`、`Int`、`Bool`、`ErrorField` 等
- 敏感信息使用脱敏功能（自动脱敏 password、token 等）
- 使用 Context 传递追踪信息（trace_id、span_id 等）

#### 11.4.5 特殊情况处理

- **启动阶段**: 在日志系统初始化之前，可以使用 `fmt.Fprintf(os.Stderr, ...)` 输出关键错误信息
- **日志系统自身**: `backend/common/log` 内部的错误处理可以使用 `fmt.Fprintf(os.Stderr, ...)` 避免循环依赖
- **测试代码**: 测试代码可以使用标准库 `log` 或 `fmt`，但生产代码必须使用统一日志系统

## 12. 部署规范

### 12.1 开发环境

- 使用Docker Compose进行本地开发
- 所有服务运行在localhost
- 简化配置便于快速启动

### 12.2 生产环境

- 支持Kubernetes部署（可选）
- 服务高可用配置
- 负载均衡配置
- 自动扩缩容配置

## 13. 扩展性规范

### 13.1 节点扩展

- 支持插件化节点系统
- 提供自定义节点开发SDK
- 节点实现放在 `node-service/internal/nodes/` 目录

### 13.2 模型扩展

- 模型适配器插件化
- 通过配置添加新模型
- 提供SDK便于开发新适配器

### 13.3 工具扩展

- 工具插件系统
- 支持第三方工具集成

## 14. 注意事项

### 14.1 禁止事项

- 禁止框架层反向导入业务实现
- 禁止服务间直接访问数据库（必须通过gRPC）
- 禁止在业务逻辑层直接操作数据库（必须通过data层）
- 禁止硬编码配置（使用配置文件或环境变量）
- **禁止使用非统一日志系统**: 禁止在 backend 后端代码中使用 `fmt.Println`、`log.Println`、`logrus`、`zap` 等非统一日志库，必须使用 `backend/common/log` 日志系统
- **禁止直接输出**: 禁止使用 `fmt.Printf`、`os.Stdout.Write`、`os.Stderr.Write` 等直接输出（日志系统内部和启动阶段的关键错误除外）

### 14.2 推荐做法

- 使用接口抽象，便于测试和扩展
- 错误处理要完善，避免panic
- 日志记录要详细，便于排查问题
- 代码要有注释，特别是复杂逻辑

## 15. 开发流程

### 15.1 本地开发环境

1. **环境准备**
   - Docker Desktop（用于PostgreSQL和Redis）
   - Node.js 18+（前端开发）
   - Go 1.21+（后端开发）
   - PostgreSQL客户端（可选，用于数据库管理）

2. **启动步骤**

   ```bash
   # 1. 启动基础设施
   docker-compose up -d postgres redis
   
   # 2. 运行数据库迁移
   make migrate-up
   
   # 3. 生成proto代码和wire代码
   cd backend
   make proto
   make wire
   
   # 4. 启动后端服务
   # 方式1: 使用Makefile启动所有服务
   make run
   
   # 方式2: 单独启动某个服务
   cd apps/gateway && go run cmd/gateway/main.go
   cd apps/user && go run cmd/user/main.go
   
   # 5. 启动前端
   cd ../../frontend && npm install && npm run dev
   ```

3. **开发调试**
   - 前端: Vite HMR自动热重载
   - 后端: 使用 `air` 工具实现热重载
   - API文档: 访问 `http://localhost:8000/api/docs`

### 15.2 代码提交

- **提交前检查**:
  - 运行测试：`make test`
  - 格式化代码：`make fmt`
  - 代码检查：`make lint`
  
- **提交信息规范**: 遵循Conventional Commits
  - `feat`: 新功能
  - `fix`: 修复bug
  - `docs`: 文档更新
  - `style`: 代码格式调整
  - `refactor`: 重构
  - `test`: 测试相关
  - `chore`: 构建/工具相关
  
  示例: `feat(workflow): 添加工作流执行引擎`

### 15.3 代码审查

- 所有代码必须经过审查才能合并到主分支
- 审查重点：
  - 代码质量（可读性、可维护性）
  - 架构合理性（是否符合Kratos规范）
  - 测试覆盖（核心逻辑是否有测试）
  - 安全性（是否有安全隐患）
  - 性能（是否有性能问题）

### 15.4 Git工作流

- **主分支**: `main`（生产环境）
- **开发分支**: `develop`（开发环境）
- **功能分支**: `feature/{功能名}`（新功能开发）
- **修复分支**: `fix/{问题描述}`（bug修复）
- **发布分支**: `release/{版本号}`（版本发布）

## 16. 文档系统规范

### 16.1 API文档

- **位置**: `backend/apps/gateway/internal/handler/docs/` 或 `backend/api/gateway/docs/`
- **格式**: OpenAPI 3.0 (YAML/JSON)
- **访问**: `/api/docs` (Swagger UI)
- **更新**: 每次API变更必须同步更新文档

### 16.2 节点文档

- **存储**: PostgreSQL数据库（`node_documentations`表）
- **API**: 节点服务提供 (`/api/v1/nodes/docs`)
- **内容**: 节点说明、参数说明、使用示例、最佳实践
- **更新**: 新增或修改节点时必须更新文档

### 16.3 工作流文档

- **存储**: Markdown文件 (`backend/apps/workflow/docs/workflows/`)
- **API**: 工作流服务提供 (`/api/v1/workflows/docs`)
- **内容**: 快速开始、核心概念、设计指南、示例、故障排查
- **更新**: 重要功能变更时更新相关文档

### 16.4 代码注释规范

- **Go代码**: 使用标准注释格式

  ```go
  // GetUser 根据用户ID获取用户信息
  // ctx: 上下文
  // userID: 用户ID
  // 返回: 用户信息和错误
  func GetUser(ctx context.Context, userID string) (*User, error) {
      // ...
  }
  ```

- **TypeScript代码**: 使用JSDoc格式

  ```typescript
  /**
   * 获取用户信息
   * @param userId - 用户ID
   * @returns 用户信息
   */
  async function getUser(userId: string): Promise<User> {
      // ...
  }
  ```

---

**最后更新**: 2024年  
**版本**: v1.0

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Samuel521199) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
