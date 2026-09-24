## netsgo

> 服务端核心。承载 REST API、管理认证、Client 会话、控制/数据通道、隧道生命周期、SSE 事件流、P2P 协调。

# internal/server 模块指南

## 模块边界

服务端核心。承载 REST API、管理认证、Client 会话、控制/数据通道、隧道生命周期、SSE 事件流、P2P 协调。
单端口架构：同一监听器服务 Web 面板、API、SSE、`/ws/control`、`/ws/data`。
持久化使用 SQLite（`internal/storage/sqlite.go`），迁移 SQL 在 `migrations/` 目录。

## 包内分层

| 职责域 | 关键文件 | 说明 |
|---|---|---|
| 初始化/生命周期 | `server.go`, `init.go`, `server_bootstrap.go`, `server_lifecycle.go` | Server 结构体、启动、关闭 |
| 认证/鉴权 | `auth_middleware.go`, `auth_service.go`, `admin_security_*.go`, `admin_webauthn.go` | API token、WebAuthn、速率限制 |
| 会话管理 | `session.go`, `session_manager.go`, `client_conn.go` | Client 在线状态、控制+数据通道绑定 |
| 控制通道 | `control_auth.go`, `control_loop.go` | WebSocket 控制面、心跳、指令下发 |
| 数据通道 | `data.go`, `proxy.go`, `http_tunnel*.go`, `udp_proxy.go`, `socks5_proxy.go` | yamux 复用、HTTP/TCP/UDP/SOCKS5 代理 |
| 隧道管理 | `tunnel_*.go`, `unified_tunnel_*.go` | 隧道 CRUD、运行态、注册表、恢复 |
| 存储/迁移 | `store.go`, `storage_schema.go`, `migrations_embed.go`, `migrations/` | SQLite schema、迁移嵌入与执行 |
| API 层 | `admin_api.go`, `tunnel_api.go`, `activity_api.go`, `traffic_api.go`, `version_api.go` | REST 端点 |
| SSE/事件 | `events.go`, `activity_*.go` | 实时事件推送、活动记录 |
| P2P | `p2p_*.go`, `stun.go` | P2P 协调、STUN、投影 |

## 高风险路径

- `auth_middleware.go`：所有 API 请求的鉴权入口；改动影响全局安全。
- `session_manager.go` + `control_loop.go`：在线状态语义核心；不要引入伪在线。
- `storage_schema.go` + `migrations/`：迁移必须顺序递增，不可修改已发布的迁移文件。
- `data.go` + `proxy.go`：数据面核心路径；改动需考虑直连、nginx、caddy 三类路径。
- `unified_tunnel_runtime.go`：运行态语义来源必须是 `TunnelSpec` endpoint 字段。

## 局部验证

```bash
go test ./internal/server/...          # 包级测试
go test -race ./internal/server/...    # 竞态检测
```

涉及协议变更时需同步跑 `go test ./pkg/protocol/...` 和 `go test ./internal/client/...`。

---
> Source: [zsio/netsgo](https://github.com/zsio/netsgo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
