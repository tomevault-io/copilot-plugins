## winpower-g2-exporter

> <!-- OPENSPEC:START -->

<!-- OPENSPEC:START -->
# OpenSpec Instructions

These instructions are for AI assistants working in this project.

Always open `@/openspec/AGENTS.md` when the request:
- Mentions planning or proposals (words like proposal, spec, change, plan)
- Introduces new capabilities, breaking changes, architecture shifts, or big performance/security work
- Sounds ambiguous and you need the authoritative spec before coding

Use `@/openspec/AGENTS.md` to learn:
- How to create and apply change proposals
- Spec format and conventions
- Project structure and guidelines

Keep this managed block so 'openspec update' can refresh the instructions.

<!-- OPENSPEC:END -->

# AGENTS.md

面向本项目的 AI 助手工作指引（对齐最新设计/实现/协议文档）。

## 项目上下文（最新）
- 目标：采集 WinPower G2 设备数据、计算能耗，并以 Prometheus 指标导出。
- 技术栈：Go 1.25+、Gin、zap、HTTP、Prometheus，测试驱动开发（TDD）。
- 服务端点：`/metrics`（只读）、`/health`；`/debug/pprof` 可选开启用于诊断。
- 项目地址：https://github.com/lay-g/winpower-g2-exporter.git

## 关键约束与行为
- 调度器统一周期 5s；与 Prometheus 抓取间隔无关，`/metrics` 不触发采集或计算。
- 仅累计电能持久化到设备级文件；其他指标不存历史。
- 支持自签证书：通过 `--skip-ssl-verify`（或环境变量）跳过 SSL 验证；生产建议由反向代理终止 TLS。
- 不要随意创建任何文档，除非明确要求的时候才需要

## 开发命令（Makefile）
```bash
make build              # 构建当前平台
make build-linux        # 构建 Linux AMD64
make build-all          # 构建所有支持平台
make clean              # 清理构建产物

make fmt                # 格式化代码
make lint               # 静态分析
make deps               # 安装依赖
make update-deps        # 更新依赖

make test               # 单元测试
make test-coverage      # 测试覆盖率
make test-integration   # 集成测试
make test-all           # 运行所有测试

make docker-build       # 构建镜像
make docker-run         # 运行容器
```

## 测试策略（TDD）
- 测试命令：`make test`、`make test-coverage`、`make test-integration`、`make test-all`。
- 组织约定：测试文件与实现同包、同目录，命名 `*_test.go`。
- Mock 建议：为 WinPower 客户端、Energy 模块提供 Mock 隔离外部副作用。
- 观察性：按需开启 `/debug/pprof`；关键路径统一结构化日志。

## 项目结构
```
winpower-g2-exporter/
├── cmd/                         # 应用程序入口点
│   └── winpower-g2-exporter/
│       └── main.go               # 主程序入口
├── internal/                    # 内部实现包
│   ├── cmd/                     # 命令行工具实现
│   ├── pkgs/                    # 内部公共包
│   │   └── log/                 # 日志模块
│   ├── config/                  # 配置管理模块
│   ├── storage/                 # 存储模块
│   ├── winpower/                # WinPower模块
│   ├── collector/               # Collector模块
│   ├── energy/                  # 电能计算模块
│   ├── metrics/                 # 指标模块
│   ├── server/                  # HTTP服务模块
│   └── scheduler/               # 调度器模块
├── docs/                        # 项目文档
│   ├── design/                  # 设计文档
│   └── examples/                # 使用示例
├── tests/                       # 测试文件
│   ├── integration/             # 集成测试
│   └── mocks/                   # Mock对象
├── deployments/                 # 部署配置
├── Dockerfile                   # Docker构建文件
├── Makefile                     # 构建和开发脚本
├── go.mod                       # Go模块定义
└── README.md                    # 项目说明文档
```

## 代码规范
- 遵循 Go 官方代码规范
- 使用 `gofmt` 格式化代码
- 使用 `golangci-lint` 进行静态检查
- 编写单元测试，保持 80% 以上的测试覆盖率
- 遵循测试驱动开发(TDD)原则

## 参考文档
- 设计：`docs/design/architecture.md`、`docs/design/metrics.md`、`docs/design/cmd.md`、`docs/design/server.md`
- 设计：`docs/design/config.md`、`docs/design/storage.md`、`docs/design/winpower.md`、`docs/design/collector.md`
- 设计：`docs/design/energy.md`、`docs/design/scheduler.md`、`docs/design/logging.md`

以上内容与 README、设计/实现/协议文档保持一致，作为日常协作与实现的最新依据。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lay-g) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
