## go-boot-opentelemetry

> go-boot-opentelemetry 是一个基于 [github.com/xudefa/go-boot](https://github.com/xudefa/go-boot) 的 OpenTelemetry 分布式追踪集成模块。本模块将 OpenTelemetry SDK 无缝集成到 go-boot 的 IoC 容器和自动配置体系中，遵循 go-boot 项目的开发规范。

# go-boot-opentelemetry 项目开发规范文档

go-boot-opentelemetry 是一个基于 [github.com/xudefa/go-boot](https://github.com/xudefa/go-boot) 的 OpenTelemetry 分布式追踪集成模块。本模块将 OpenTelemetry SDK 无缝集成到 go-boot 的 IoC 容器和自动配置体系中，遵循 go-boot 项目的开发规范。

## 1. 项目定位

### 1.1 与 go-boot 的关系

- **基础框架**：go-boot 提供核心 IoC 容器、AOP、自动配置、生命周期管理等基础设施
- **集成模块**：go-boot-opentelemetry 是 go-boot 的追踪层集成，将 OpenTelemetry 作为分布式追踪实现
- **规范继承**：完全遵循 go-boot 的开发规范、命名约定、代码风格

### 1.2 核心职责

- 将 OpenTelemetry TracerProvider 和 Tracer 注册为 go-boot 容器中的 Bean
- 提供基于 go-boot 自动配置的 OpenTelemetry 启动器
- 实现 `tracing.Tracer` 和 `tracing.Span` 接口的 OpenTelemetry 适配器
- 支持 OTLP gRPC、OTLP HTTP、Stdout 等多种导出器
- 支持 W3C TraceContext 和 Baggage 上下文传播

## 2. 项目架构

### 2.1 整体架构

```
┌───────────────────────────────────────────────────────────────────────┐
│                    go-boot ApplicationContext                         │
│  ┌───────────┐ ┌──────────────┐ ┌───────────┐ ┌───────────┐           │
│  │ Container │ │  Environment │ │ Lifecycle │ │ EventBus  │           │
│  └───────────┘ └──────────────┘ └───────────┘ └───────────┘           │
│                       ┌─────────────────────┐                         │
│                       │ AutoConfig Registry │                         │
│                       └─────────────────────┘                         │
└───────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
                    ┌───────────────────────────────┐
                    │ go-boot-opentelemetry Starter │
                    │  ┌─────────────────────────┐  │
                    │  │ TracerProvider Bean     │  │
                    │  │ Tracer Bean             │  │
                    │  │ Span Wrapper            │  │
                    │  │ Exporter (gRPC/HTTP)    │  │
                    │  │ Propagator              │  │
                    │  └─────────────────────────┘  │
                    └───────────────────────────────┘
```

- **基础依赖**：依赖 go-boot 核心框架（`github.com/xudefa/go-boot`）
- **追踪系统**：集成 OpenTelemetry（`go.opentelemetry.io/otel`）
- **职责边界**：仅负责分布式追踪集成，不包含其他业务逻辑
- **示例代码**：统一放在 `examples/` 目录，演示 OpenTelemetry 集成用法

### 2.2 go-boot-opentelemetry 核心包结构

| 包 | 说明 | 接口定义 |
|---|------|----------|
| `autoconfig.go` | OpenTelemetry 自动配置 | `AutoConfiguration` |
| `tracer_adapter.go` | Tracer/Span 适配器 | `TracerWrapper`, `SpanWrapper` |
| `propagator.go` | 上下文传播器 | `Propagator` |
| `bridge/` | 追踪桥接实现 | `TracingBridge` |

### 2.3 go-boot 核心包参考

go-boot-opentelemetry 依赖 go-boot 的以下核心包：

| 包 | 说明 | 接口定义 |
|---|------|----------|
| `core/` | IoC 容器（依赖注入核心） | `core.Container` |
| `aop/` | AOP 框架（5 种通知类型 + 多种切点匹配） | `aop.Advice`, `aop.PointCut`, `aop.Weaver` |
| `boot/` | 应用启动器、自动配置注册、横幅、失败分析 | `boot.AutoConfiguration`, `boot.Starter` |
| `context/` | 应用上下文（聚合容器、环境、生命周期、事件） | `context.ApplicationContext` |
| `environment/` | 环境配置管理（分层 PropertySource + Profile） | `environment.Environment` |
| `condition/` | 条件判断（OnProperty / OnBean / OnClass 等） | `condition.Condition` |
| `event/` | 事件驱动支持（发布/订阅） | `event.ApplicationEvent`, `event.EventPublisher` |
| `life/` | 生命周期阶段管理（7 个阶段） | `life.Lifecycle`, `life.Phase` |
| `data/` | 数据访问抽象（Repository[T] / Transactor） | `data.Repository[T]`, `data.Transactor` |
| `cache/` | 缓存抽象（Cache 接口） | `cache.Cache` |
| `config/` | 配置管理（Config 接口 + Loader 链 + Validator） | `config.Config`, `config.Loader` |
| `log/` | 日志抽象（Logger 接口 + slog 默认实现） | `log.Logger` |
| `net/` | HTTP 服务器/客户端抽象接口 | `net.Server`, `net.HttpClient` |
| `health/` | 健康指标（Indicator + Aggregator） | `health.Indicator`, `health.HealthAggregator` |
| `metrics/` | 指标收集（Counter + Gauge + Registry） | `metrics.Counter`, `metrics.Gauge`, `metrics.MeterRegistry` |
| `tracing/` | 分布式追踪抽象 + LocalTracer 实现 | `tracing.Tracer`, `tracing.Span` |
| `actuator/` | 运维端点（健康、指标、环境信息） | `actuator.Endpoint` |
| `schedule/` | 定时任务调度（Cron 解析、最小堆调度器、@Scheduled 注解） | `schedule.Task`, `schedule.Scheduler` |
| `center/` | 注册中心抽象（Registry 接口 + Selector 接口 + 内置选择器） | `center.Registry`, `center.Selector` |
| `refresh/` | 配置刷新（RefreshScope 动态刷新） | `refresh.RefreshScopeManager` |

### 2.4 接口抽象原则

go-boot-opentelemetry 遵循 go-boot 的接口抽象原则，所有集成层通过核心框架中的接口抽象定义，实现运行时互换：

- `net.Server` / `net.HttpClient` — HTTP 服务器/客户端
- `data.Repository[T]` / `data.Transactor` — 数据访问/事务
- `cache.Cache` — 缓存抽象
- `config.Config` — 配置管理
- `log.Logger` — 日志抽象
- `health.Indicator` — 健康指标
- `metrics.MeterRegistry` — 指标注册表
- `tracing.Tracer` / `tracing.Span` — 分布式追踪
- `center.Registry` / `center.Selector` — 注册发现/负载均衡
- `core.Container` — IoC 容器
- `boot.AutoConfiguration` — 自动配置
- `boot.Starter` — 启动器生命周期

## 3. 开发规范

### 3.1 命名约定

- **包名**：小写、多个单词中间用"-"连接，除开main包，其他包名和最里层目录名保持一致。例如 `user-service`
- **导出标识符**：大写驼峰（`UserID`）
- **非导出标识符**：小写驼峰（`userID`）
- **常量**：使用驼峰，而非全大写加下划线（`MaxConnections` 而不是 `MAX_CONNECTIONS`）
- **测试函数**：`TestFunctionName_Condition_ExpectedBehavior`
- **错误变量**：以 `Err` 前缀（`ErrNotFound`）
- **接口**：通常以 `er` 后缀（`Reader`, `Writer`）或功能描述（`Logger`, `Cache`）

### 3.2 导入规范

- 使用标准库分组 → 本地包，每组之间用空白行分隔
- 禁止相对导入（如 `../foo`），使用模块路径完整导入
- 核心框架仅使用 Go 标准库

```go
import (
    "context"
    "fmt"
    "sync"

    "github.com/xudefa/go-boot/core"
    "github.com/xudefa/go-boot/log"
)
```

### 3.3 函数式选项模式

整个框架优先使用函数式选项模式，而非建造者模式或配置结构体：

```go
// 良好 — Bean 构建器选项
container.Register("service",
    core.Bean(&Service{}),
    core.Singleton(),
    core.DependsOn("db"),
    core.Init(func(s *Service) error { return s.Start() }),
    core.Condition(func(c core.Container) bool { return c.Has("db") }),
)
```

### 3.4 注释与文档规范

#### 3.4.1 代码注释
- 使用中文注释，保持国际化友好
- 接口、结构体需要 doc 注释，接口注释需要使用示例
- 代码实现细节较复杂的，处理步骤>=3的，都需要注释说明执行逻辑和流程
- 导出类型和函数必须有文档注释
- 注释内容应说明"为什么这样做"而不是"做了什么"

#### 3.4.2 文档注释格式
```go
// CalculateDiscount 计算应用分级折扣后的最终价格。
// 折扣根据订单数量逐步应用：每个等级解锁额外的百分比减免。
// 如果数量无效或基础价格在应用折扣后会导致负值，则返回错误。
//
// 参数:
//   - basePrice: 任何折扣前的原始价格（必须为非负数）
//   - quantity: 订单的数量（必须为正数）
//   - tiers: 按最小数量阈值排序的折扣等级切片
//
// 返回最终折扣价格，四舍五入到小数点后两位。
// 如果 basePrice 为负数，返回 ErrInvalidPrice。
// 如果 quantity 为零或负数，返回 ErrInvalidQuantity。
//
// 示例:
//
//	tiers := []DiscountTier{
//	    {MinQuantity: 10, PercentOff: 5},
//	    {MinQuantity: 50, PercentOff: 15},
//	    {MinQuantity: 100, PercentOff: 25},
//	}
//	finalPrice, err := CalculateDiscount(100.00, 75, tiers)
//	if err != nil {
//	    log.Fatalf("折扣计算失败: %v", err)
//	}
//	log.Printf("订购了 75 件单价 $100 的商品: 最终价格 = $%.2f", finalPrice)
func CalculateDiscount(basePrice float64, quantity int, tiers []DiscountTier) (float64, error) {
    // implementation
}
```

### 3.5 IoC 容器规范

- 使用 `core.New()` 创建容器，`core.EnableFieldTag(true)` 启用字段注入
- Bean 注册使用 `container.Register("id", core.Bean(value))`
- 字段注入使用 `inject:"beanId"` 结构体标签
- 方法注入使用 `container.Invoke(func(s *Service) { ... })` 自动解析参数
- 工厂 Bean 使用 `core.Factory(func(c core.Container) (any, error), reflect.TypeOf((*Target)(nil)).Elem())`
- 类型安全的泛型注册使用 `core.BeanOf[T](value)` 和 `core.FactoryOf[T](fn)`
- 条件注册使用 `core.Condition(func(c core.Container) bool)`

### 3.6 AOP 规范

- 通知类型：`aop.Before`、`aop.After`、`aop.Around`、`aop.AfterReturning`、`aop.AfterThrowing`
- 切点匹配器：`aop.MatchByName`、`aop.MatchByPrefix`、`aop.MatchByRegex`、`aop.MatchByAnnotation`、`aop.MatchByInterface`、`aop.MatchAll`、`aop.MatchClass`、`aop.MatchMethod`
- 多个通知通过 `aop.WithOrder(n)` 或者 `AspectMeta.Order` 排序，值越小优先级越高
- Around 通知必须调用 `proceed` 使调用链继续
- 通过 `aop.NewAdvisor(pointcut, advice)` 组装切面
- 通过 `aop.NewWeaver()` + `weaver.AddAspects()` + `weaver.Weave(target)` 织入

### 3.7 组件扫描与注解

- 支持使用 Go 注释标签实现类似 Spring 的组件扫描：

```go
// @Service("userService")
type UserService struct {
    DB *Database `inject:"database"`
}

// @Repository
// @Component("myBean")
// @Configuration
```

- 使用 `container.Scan("./path/to/package")` 自动扫描注册

### 3.8 错误处理

- 不忽略任何返回错误
- 使用 `fmt.Errorf` 或 `errors.New`，必要时用 `%w` 包装
- 自定义错误类型时实现 `Error()` 方法
- 框架层错误使用 sentinel errors（如 `cache.ErrNotFound`、`core.ErrDuplicateBean`）
- 错误信息应清晰描述问题和可能的解决方案

### 3.9 泛型使用

- 优先使用 Go 泛型实现类型安全 API：`Repository[T]`、`BeanOf[T]`、`FactoryOf[T]`
- 泛型工具函数：`core.ZeroOf[T]()`、`core.TypeOfGeneric[T]()`、`core.ValueOfGeneric[T]()`、`core.Clone[T](v)`
- 避免过度使用泛型，清晰优先于抽象

### 3.10 代码风格规范

#### 3.10.1 总体原则
- **清晰优于巧妙**：代码应该易于理解和维护
- **简单优于复杂**：优先选择简单直接的实现方式
- **可读性第一**：代码首先是给人阅读的，其次才是给机器执行的
- **零外部依赖**：核心框架不引入外部依赖，仅使用Go标准库

#### 3.10.2 行长度与换行
- 无严格行长度限制，但超过 ~120 字符时应考虑换行
- 函数调用超过 4 个参数时，每个参数独占一行
- 复杂条件表达式应在语义边界处换行

#### 3.10.3 变量声明
- 非零值使用短变量声明 `:=`
- 零值初始化使用 `var`
- 切片和映射必须初始化，不允许为 nil
- 复合字面值必须使用字段名

#### 3.10.4 控制流
- 优先处理错误和边界条件（早期返回）
- 消除不必要的 `else`
- 复杂条件提取为命名布尔变量
- 使用 `switch` 替代多层 `if-else` 链

#### 3.10.5 函数设计
- 函数应简短专注，单一职责
- 参数不超过 4 个，超过时使用选项结构体
- `context.Context` 总是第一个参数
- 使用 `range` 迭代优于索引循环

#### 3.10.6 字符串处理
- 简单转换使用 `strconv`（性能更好）
- 复杂格式化使用 `fmt.Sprintf`
- 错误消息中使用 `%q` 显示字符串边界
- 循环中拼接使用 `strings.Builder`

### 3.11 代码组织规范

#### 3.11.1 文件内组织
- 相关声明分组：类型、构造函数、方法一起
- 顺序：包文档、导入、常量、类型、构造函数、方法、辅助函数
- 每个主要类型单独一个文件（当有大量方法时）

#### 3.11.2 包组织
- 包注释应使用完整句子描述包的功能
- 相关功能应放在同一个包中
- 避免过大包，适时拆分

### 3.12 测试规范

#### 3.12.1 测试结构
- 使用表格驱动测试
- 测试函数名应描述测试场景和期望结果
- 为边界条件和错误路径编写测试

```go
func TestCalculateDiscount(t *testing.T) {
    tests := []struct {
        name        string
        basePrice   float64
        quantity    int
        tiers       []DiscountTier
        expected    float64
        expectError bool
    }{
        {
            name:      "normal calculation",
            basePrice: 100.0,
            quantity:  10,
            expected:  95.0,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result, err := CalculateDiscount(tt.basePrice, tt.quantity, tt.tiers)

            if tt.expectError {
                assert.Error(t, err)
                return
            }

            assert.NoError(t, err)
            assert.Equal(t, tt.expected, result)
        })
    }
}
```

#### 3.12.2 测试覆盖率
- 重要功能必须有单元测试覆盖
- 边界条件和错误路径应有对应测试
- 核心框架测试不依赖外部服务
- 定期检查测试覆盖率，保持较高水平

### 3.13 类型与函数规范

- **接受接口，返回具体类型**
- **结构体字段顺序**：先 `sync.Mutex` 等互斥体，再其他字段
- **函数长度**：尽量不超过 50 行，文件不超过 500 行
- **参数传递**：使用 `context.Context` 作为第一个参数传递请求上下文
- **返回值**：对于有错误返回的函数，错误应作为最后一个返回值
- **方法接收器**：值接收器用于小型、不可变的结构；指针接收器用于需要修改或大型结构

### 3.14 测试规范补充

- **表驱动测试**（table-driven tests）优先
- **测试函数命名**：`TestFunctionName_Condition_ExpectedBehavior`
- **并行测试**：使用 `t.Parallel()` 进行并行测试
- **测试隔离**：核心框架测试不依赖外部服务
- **覆盖率**：关键逻辑应达到 80% 以上覆盖率
- **基准测试**：对性能敏感的函数编写基准测试

## 4. 代码质量与工具

### 4.1 构建命令

- 构建所有包：`make build` 或 `go build ./...`

### 4.2 测试命令

- 运行所有测试：`make test` 或 `go test ./...`
- 运行单个测试：`go test -run <TestName> ./path/to/package`
- 带覆盖率：`make test-cover` 或 `go test -cover ./...`
- 数据竞争检测：`make test-race` 或 `go test -race ./...`

### 4.3 Lint 与格式化

- 格式化代码：`make fmt` 或 `go fmt ./...`
- 静态检查：`make lint` 或 `golangci-lint run`

## 5. 应用启动与配置

### 5.1 应用启动模式

- 推荐入口：`boot.NewApplication(opts...)` 创建应用实例
- 应用上下文 `DefaultApplicationContext` 聚合 Container、Environment、Lifecycle、EventBus
- `boot.RegisterAutoConfig()` 注册自动配置，通过 init() 在各模块中调用
- `boot.RegisterStarter()` 注册启动器，支持依赖拓扑排序

### 5.2 自动配置机制

核心模块在 `init()` 中注册自动配置：

```go
func init() {
    boot.RegisterAutoConfig(
        &TracingAutoConfiguration{},
        condition.OnProperty("tracing.enabled", "true"),
    )
}
```

支持条件控制（OnProperty / OnBean / OnMissingBean / OnClass / OnProfile）和排序（WithOrder / WithDependsOn）。

### 5.3 配置管理

- 使用 `environment` 包管理配置，支持多层级配置源
- 配置优先级：命令行参数 > 环境变量 > 配置文件 > 默认值
- 支持 Profile 机制，通过 `--profile=dev` 或环境变量激活

## 6. 最佳实践

### 6.1 性能优化

- 避免不必要的内存分配
- 合理使用缓存
- 适当使用并发和并行
- 使用连接池管理资源
- 避免 goroutine 泄漏

### 6.2 安全考虑

- 输入验证和清理
- 防止 SQL 注入
- 适当的身份验证和授权
- 敏感信息加密存储

### 6.3 可维护性

- 保持函数短小精悍
- 遵循单一职责原则
- 适当的抽象层次
- 清晰的错误处理
- 完善的测试覆盖

### 6.4 可测试性

- 依赖注入便于 Mock
- 接口抽象便于替换
- 避免全局状态
- 明确的输入输出

## 7. 项目贡献

### 7.1 提交规范

- 提交信息应遵循 conventional commits 规范
- 格式：`<type>(<scope>): <subject>`
- 类型包括：feat, fix, docs, style, refactor, test, chore

### 7.2 分支管理

- 主分支为 `master`
- 功能开发使用 `feature/` 前缀
- 修复使用 `fix/` 前缀
- 发布使用 `release/` 前缀

### 7.3 代码审查

- 代码应符合本规范
- 必须包含相应测试
- 文档应及时更新
- 遵循项目的安全和性能标准

---
> Source: [xudefa/go-boot-opentelemetry](https://github.com/xudefa/go-boot-opentelemetry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-30 -->
