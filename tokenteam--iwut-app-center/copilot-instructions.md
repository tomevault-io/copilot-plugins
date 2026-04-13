## iwut-app-center

> iWUT App Center 是一个基于 **Kratos v2** 框架的微服务，提供应用管理、版本发布、灰度分发和测试邀请等功能。存储层使用 **MongoDB + Redis**，API 通过 **Protobuf** 定义，同时暴露 HTTP 和 gRPC 双栈。

# AGENTS.md — iwut-app-center

## 项目概览

iWUT App Center 是一个基于 **Kratos v2** 框架的微服务，提供应用管理、版本发布、灰度分发和测试邀请等功能。存储层使用 **MongoDB + Redis**，API 通过 **Protobuf** 定义，同时暴露 HTTP 和 gRPC 双栈。

## 架构

采用 Kratos 推荐的分层架构（类 Clean Architecture），依赖方向：`service → biz →(接口)← data`

```
cmd/iwut-app-center/     入口、Wire 注入、日志/Tracer 初始化
internal/
  server/                HTTP/gRPC 服务器装配、中间件注册、自定义 ErrorEncoder
  service/               实现 proto 生成的 Server 接口，做 JWT 鉴权 + proto/biz 类型转换
  biz/                   领域模型、Usecase 业务逻辑、Repo 接口定义
  data/                  Repo 实现（MongoDB/Redis）
  middleware/            JWT 信息提取中间件
  conf/                  内部配置 proto（conf.proto → conf.pb.go）
  util/                  横切工具：JWT、灰度计算、规则解析、外部服务调用、请求追踪等
api/                     Git 子模块，含 proto 定义与生成代码
configs/                 config.yaml（支持 ${ENV_VAR:default} 占位）
```

## 关键技术栈

- **Go 1.24**，框架 **Kratos v2**
- **Protobuf + gRPC**：API 定义在 `api/app_center/v1/` 下的 `.proto` 文件
- **Google Wire**：依赖注入，每层暴露 `ProviderSet`
- **MongoDB**（`go.mongodb.org/mongo-driver/v2`）：主存储
- **Redis**（`github.com/go-redis/redis/v8`）：缓存、邀请链接存储
- **OpenTelemetry**：链路追踪，导出至 OTLP gRPC
- **Zap**：结构化 JSON 日志

## 编码约定

### 错误处理

- 使用 `github.com/go-kratos/kratos/v2/errors` 构造业务错误（`NotFound`、`BadRequest`、`Forbidden` 等）
- Reason 字符串**必须**使用 `api/gen/go/app_center/v1/error_reason` 中的 proto 枚举常量，例如：
  ```go
  errors.BadRequest(string(v1.ErrorReason_INVALID_APP_NAME), "message")
  ```
- HTTP 层通过自定义 `CreatedErrorEncoder` 始终返回 HTTP 200 + JSON 业务错误体（含 `traceId`）

### 日志

- 使用 `log.NewHelper(log.WithContext(ctx, uc.log.Logger()))` 获取带链路上下文的 logger
- 生产环境自动过滤 Debug 级别，对 `args` 字段脱敏（`MaskArgsString`）
- 日志格式为 JSON，包含 `trace_id`、`span_id`、`service_name` 等标准字段

### 分层职责

- **biz 层**：定义 `Repo` 接口（如 `AppRepo`、`AppVersionRepo`、`TesterRepo`），包含领域模型（`Application`、`ApplicationVersionInfo` 等）和所有业务逻辑
- **data 层**：实现 `Repo` 接口，负责 MongoDB/Redis 操作；每个操作设置独立的 `context.WithTimeout`（通常 5-10 秒）
- **service 层**：实现 proto 生成的 `Unimplemented*Server`；负责 JWT 鉴权（通过 `util.JwtUtil`）、proto 与 biz 类型转换；使用 `util.GetProcesses` 统一处理 traceId 注入
- **不要**在 data 层放业务逻辑，不要在 service 层做复杂判断

### 命名与风格

- 领域模型字段使用 `bson:"snake_case"` tag
- 状态常量定义为包级 `const`，使用全大写字符串（如 `DEVELOPING`、`STABLE`）
- Wire `ProviderSet` 在每个包中以 `var ProviderSet = wire.NewSet(...)` 声明
- 构造函数统一命名 `New*`（如 `NewAppUsecase`、`NewAppRepo`）
- Usecase 方法签名：`func (uc *XxxUsecase) MethodName(ctx context.Context, ...) (result, error)`

### JWT 与认证

- 中间件从 `X-Auth-Jwt-Type` 头判断类型（`official`/`oauth`/`service`），从对应的 `X-Auth-*-Claim` 头解析 claims 并注入 context
- service 层通过 `jwtUtil.GetBaseAuthClaims(ctx)` 或 `jwtUtil.GetServiceClaims(ctx)` 获取鉴权信息
- 路径前缀 `/auth_center.v1.auth.Auth/` 和 `/auth_center.v1.oauth2.OAuth2/getToken` 跳过 JWT 校验

### 配置

- 配置加载顺序：环境变量（前缀 `AppCenter_`） > `configs/config.yaml`
- 配置结构由 `internal/conf/conf.proto` 定义，生成 `Bootstrap` → `Server` / `Data` / `Service`

## 构建与部署

- `make api`：从 proto 生成 Go 代码 + OpenAPI spec
- `make config`：生成内部配置 proto
- `make wire`：生成依赖注入代码
- `make build`：编译到 `bin/`
- Docker：`prod.Dockerfile`（生产）/ `dev.Dockerfile`（带 dlv 调试）
- CI：GitHub Actions → CNB 同步 → 腾讯云部署

## 新增功能指南

1. 在 `api/app_center/v1/` 的 proto 中定义接口，运行 `make api`
2. 在 `biz/` 中定义领域模型、`Repo` 接口方法和 Usecase 业务逻辑
3. 在 `data/` 中实现新增的 `Repo` 接口方法
4. 在 `service/` 中实现 proto 生成的 Server 方法，做类型转换和鉴权
5. 如有新的依赖，在对应包的 `ProviderSet` 中注册，运行 `make wire`
6. 如有新的错误码，在 `error_reason.proto` 中添加枚举值

## 注意事项

- `api/` 是 Git 子模块（`iwut-api-proto`），proto 修改需在子模块仓库操作
- GORM/MySQL 初始化代码已注释，当前**不使用** MySQL
- 测试覆盖较少，仅 `internal/util/grey_calc_test.go` 有单测
- MongoDB 集合名：`application`（应用）、`application_version`（版本）
- Redis key 统一使用 `data.GetRedisKey(...)` 构建，带前缀

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TokenTeam) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
