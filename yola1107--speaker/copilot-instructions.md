## speaker

> 本项目指南用于指导 Claude Code 协助开发。

# CLAUDE.md

本项目指南用于指导 Claude Code 协助开发。

## 项目简介

**Kratos v2 微服务**，用于游戏服务器平台。使用定制版 Kratos 框架 (`github.com/yola1107/kratos/v2`)，支持多协议实时通信。

## 常用命令

```bash
make init    # 安装工具
make all     # 生成所有代码（api + config + wire）
make build   # 构建
./bin/server -conf ./configs  # 运行
go test ./tmp/xxg2/... -v     # 测试游戏模块
```

## 项目结构

```
cmd/server/        # 入口（main.go, wire.go, wire_gen.go）
api/speaker/v1/   # Proto 定义及生成代码
internal/
  ├── service/    # 服务层（实现 gRPC/HTTP 端点）
  ├── biz/        # 业务逻辑层
  ├── data/       # 数据访问层
  ├── server/     # 服务器配置（http, grpc, tcp, ws, gnet）
  └── conf/       # 配置结构
configs/          # 运行时配置（config.yaml）
tmp/              # 游戏模块（xxg2, hbtr2, mahjong4, sgz, xslm2, jqs, champ）
test/             # 测试客户端
```

## 架构

**整洁架构** + **Google Wire** 依赖注入：

```
cmd/server → internal/server → internal/service → internal/biz → internal/data
```

每层在主文件定义 `ProviderSet`，Wire 生成 `wire_gen.go` 自动组装依赖。

## 传输协议

| 协议 | 端口 | 说明 |
|-----|------|-----|
| HTTP | 8000 | REST API |
| gRPC | 9000 | gRPC 服务 |
| TCP | 3101 | 自定义 TCP 协议 |
| WebSocket | 3102 | 浏览器客户端 |
| Gnet | 3103 | 高性能 TCP |

## 技术栈

- **Go** 1.24.5
- **Kratos** v2（定制版）
- **Google Wire** 依赖注入
- **XORM** MySQL ORM
- **go-redis v9** 缓存
- **RabbitMQ** 消息队列
- **gnet** 高性能网络
- **Protocol Buffers** API 定义

## 项目特定错误处理

使用 Kratos `errors` 包定义项目级别错误，在 `internal/biz/` 层定义错误变量：

```go
var (
    ErrUserNotFound = errors.NotFound(v1.ErrorReason_USER_NOT_FOUND.String(), "user not found")
)
```

> 通用开发规范、Go 编码规范、Git 工作流等见 `~/.claude/CLAUDE.md`

---
> Source: [yola1107/speaker](https://github.com/yola1107/speaker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
