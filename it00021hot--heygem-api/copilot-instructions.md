## gf-rule

> title: 项目开发规范 (Project Rules)

---
title: 项目开发规范 (Project Rules)
---

### 1. 总则

*   **目标**: 本规范旨在统一项目开发标准，提高代码质量、可维护性和团队协作效率。
*   **范围**: 本规范适用于 `heygem-api` 项目的所有开发人员和所有代码。
*   **原则**: 遵循 Go 语言最佳实践、GoFrame 框架设计哲学以及本规范定义的具体细则。

### 2. 技术栈规范

*   **核心框架**:
    *   强制使用 `github.com/gogf/gf/v2` (GoFrame v2) 作为项目的基础 Web 及工具框架。
    *   业务逻辑、路由、配置、日志、数据库操作等应优先利用 GoFrame 提供的组件和能力。
*   **数据库**:
    *   使用 `github.com/gogf/gf/contrib/drivers/mysql/v2` 连接 MySQL 数据库。
    *   数据库操作强制使用 GoFrame 提供的 ORM (`gdb`) 功能，禁止拼接 SQL 字符串（除非 ORM 无法满足的复杂场景，需评审）。
*   **认证与授权**:
    *   使用 `github.com/golang-jwt/jwt/v5` 进行 JWT (JSON Web Tokens) 的生成与校验，实现用户认证和接口授权。
    *   业务层获取认证后的用户ID通过 `contexts.Get(ctx)` 获取
*   **支付集成**:
    *   支付相关功能必须使用 `github.com/go-pay/gopay` 库进行对接。
    *   配置信息（如商户号、密钥等）需通过 GoFrame 的配置管理进行加载，严禁硬编码。
*   **微信生态**:
    *   微信公众号、小程序、企业微信等相关功能开发，统一使用 `github.com/silenceper/wechat/v2` SDK。
    *   同样，AppID、Secret 等敏感配置信息通过配置文件管理。
*   **依赖管理**:
    *   强制使用 Go Modules (`go mod`) 进行依赖管理。
    *   定期执行 `go mod tidy` 清理不必要的依赖。
    *   添加新依赖需经过讨论和评估。

### 3. 项目结构规范

推荐遵循 GoFrame 推荐或社区约定的目录结构，例如：

```plaintext
heygem-api/
├── api/         # API 接口定义 (Proto Buffers 或 Go struct)
│   └── {module}/ # 按业务模块划分，例如 client, admin
│       └── v1/   # API 版本
│       └── v1/   # API 版本
├── internal/    # 内部业务逻辑 (项目核心代码)
│   ├── consts/  # 常量定义
│   ├── controller/ # 控制器层 (HTTP/gRPC 请求处理，按模块划分，由  gf gen ctrl -m -s api/{module} -d internal/controller/{module} 生成)
│   │   └── {module}/ # 例如 controller/client
│   ├── dao/     # 数据访问对象 (数据库底层操作封装)
│   ├── logic/   # 业务逻辑层 (处理复杂业务，按模块划分)
│   │   └── {module}/ # 例如 logic/user
│   ├── model/   # 数据模型 (数据库表结构实体、输入输出结构体)
│   │   ├── do/    # 数据库 ORM 对象 (由工具生成或手写)
│   │   └── entity/ # 数据库实体对象 (由工具生成或手写)
│   │   └── input/ # 数据传输对象 (API 输入输出)
│   ├── packed/  # 资源打包 (将静态资源等打包进二进制文件) - 可选
│   ├── service/ # 服务层 (业务逻辑接口定义，按模块划分，由 gf gen service 生成)
│   │   └── {module}/ # 例如 service/user
│   └── utility/ # 工具类
├── manifest/    # 配置与部署相关文件
│   ├── config/  # 配置文件 (config.yaml/toml)
│   ├── deploy/  # 部署脚本 (Dockerfile, k8s yaml 等)
│   └── sql/     # SQL 初始化脚本
├── hack/        # 项目维护脚本 (代码生成、编译脚本等)
├── main.go      # 程序入口
├── go.mod       # Go Modules 文件
└── go.sum       # Go Modules 校验和文件

```

*   **`api/{module}/v1`**: 定义对外暴露的接口契约，按业务模块（如 `client`, `admin`）组织，再按版本划分。
*   **`internal`**: 存放项目核心的私有代码。
    *   **`controller/{module}`**: 处理 HTTP/gRPC 请求，参数校验，调用 `service`。按业务模块组织，使用 `gf gen ctrl` 命令生成。保持轻量。
    *   **`service/{module}`**: 业务逻辑层接口定义。使用 `gf gen service` 命令生成，供 `controller` 和 `logic` 调用。
    *   **`logic/{module}`**: 业务逻辑的具体实现层，实现 `service` 定义的接口，组合 `dao` 或其他 `logic`。处理复杂业务逻辑。按业务模块组织。
    *   **`dao`**: 数据访问层，直接与数据库交互，封装底层 SQL 操作。与 `model/do` 或 `model/entity` 关联。
    *   **`model`**: 数据结构定义。`do` 和 `entity` 用于数据库映射，`input` 用于接口数据传输。
    *   **`consts    *   **`utility`**: 通用工具类。
*   **`manifest`**: 存放配置、部署、数据库脚本等非 Go 代码文件。
*   **`hack`**: 存放辅助开发和维护的脚本（如代码生成脚本）。
*   **`main.go`**: 程序启动入口，负责初始化和启动 GoFrame 应用。

### 4. 编码规范

*   **代码风格**:
    *   强制使用 `gofmt` 或 `goimports` 格式化代码。
    *   遵循 Go 官方推荐的编码风格 (@Effective Go)。
    *   命名清晰、表意明确，遵循 Go 的驼峰命名法（公有成员首字母大写，私有成员首字母小写）。
*   **错误处理**:
    *   必须显式处理 `error`，禁止忽略。
    *   优先使用 GoFrame 提供的错误处理机制（如 `gerror`）来包装和传递错误信息，方便追踪。
    *   API 返回错误时，应包含明确的错误码和错误信息。
*   **日志记录**:
    *   强制使用 GoFrame 的日志组件 (`g.Log()` 或其封装) 进行日志记录。
    *   根据场景选择合适的日志级别（Debug, Info, Notice, Warning, Error, Critical）。
    *   关键操作、错误信息、重要状态变更必须记录日志。
    *   禁止在日志中记录密码、密钥等敏感信息，除非经过脱敏处理。
*   **配置管理**:
    *   所有配置项（数据库连接、第三方服务密钥、端口号等）必须通过 GoFrame 的配置组件 (`gcfg`) 从配置文件（如 `manifest/config/config.yaml`）加载。
    *   严禁在代码中硬编码配置信息。
*   **并发安全**:
    *   处理并发场景时，必须注意数据竞争问题，合理使用 `sync` 包中的锁、`channel` 或 GoFrame 提供的并发安全工具。
*   **Context 使用**:
    *   强制在请求处理链、跨服务调用、数据库操作、耗时任务中使用 `context.Context` 进行传递，用于控制超时和传递元数据。
*   **注释**:
    *   公共函数、方法、类型、重要逻辑段必须添加清晰的注释。
    *   遵循 GoDoc 规范编写注释。
*   **测试**:
    *   鼓励编写单元测试和集成测试，确保代码质量。
    *   测试文件应与源文件放在同一目录下，以 `_test.go` 结尾。
    *   优先使用 GoFrame 提供的测试组件和工具。

### 5. 版本控制与协作

*   **版本控制**: 使用 Git 进行版本控制。
*   **分支模型**: 推荐使用 Gitflow 或类似的分支模型（如 `main`/`master`, `develop`, `feature/xxx`, `release/xxx`, `hotfix/xxx`）。
*   **提交信息**: Commit Message 需清晰、简洁地描述本次提交的内容，建议遵循一定的格式（如 Conventional Commits）。
*   **代码评审**: 关键代码、新功能代码必须经过 Code Review 后方可合入主开发分支 (`develop` 或 `main`)。

### 6. 附则

*   本规范由项目组共同制定和维护，如有修订需经过讨论通过。
*   所有项目成员应自觉遵守本规范。

---
> Source: [it00021hot/heygem-api](https://github.com/it00021hot/heygem-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
