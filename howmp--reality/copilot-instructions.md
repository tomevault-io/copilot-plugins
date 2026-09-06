## reality

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

`grs`（Golang Reverse SOCKS5）是基于 REALITY 协议的反向 SOCKS5 内网穿透工具。服务端部署在公网，客户端部署在内网，用户端在外部通过 SOCKS5 代理访问内网资源。所有通信伪装成对真实 HTTPS 站点的 TLS 1.2/1.3 连接，消除网络特征和防止主动探测。

模块路径: `github.com/howmp/reality`，Go 1.24。

## 构建与测试

```bash
# 快速构建当前平台
go build -trimpath -ldflags "-s -w" -o dist/grss ./cmd/grss
go build -trimpath -ldflags "-s -w" -o dist/grsc ./cmd/grsc
go build -trimpath -ldflags "-s -w" -o dist/grsu ./cmd/grsu

# 构建所有平台二进制（需要 go-bindata）
go install github.com/go-bindata/go-bindata/...@latest
./build.sh

# 运行单个测试
go test -v -run TestRealityHandshake_Gitee -count=1 -timeout 30s .

# 并发运行全部测试（推荐, ~7s 完成）
go test -count=1 -timeout 60s -parallel 16 .

# 静态检查
go vet ./...
```

**构建注意事项**: `build.sh` 先交叉编译 `grsc` 和 `grsu` 到各平台，用 `go-bindata` 嵌入 `cmd/grss/files.go`，再编译 `grss`。服务端二进制内嵌所有平台的客户端/用户端，`grss gen` 从中提取并注入配置。

## 文件职责

### `server.go`（410 行）— 服务端核心

- **`handshake()`** — 8 步清晰流水线（~60 行）：连接目标 → 建立 `flightObserver` → 解密 ClientHello → `observeTLS()` → 提取 seq → 读取 overlay → 发送 ed25519 签名 → 返回 `warpConn`
- **`flightObserver`** — 封装双向 `io.TeeReader` + `bufio.Reader`，提供 `readTarget()`/`readClient()`/`buffer()`/`keep()`/`greedyTarget()` 方法
- **`handshakeResult`** — 解密后的 REALITY 会话状态（aead + plaintext）
- **`readClientHello()`** — `*Listener` 方法，从 ClientHello 提取 ECDH 公钥、派生会话密钥、解密验证 REALITY 前缀，返回 `*handshakeResult`
- **`serverFSM`** + **`clientFSM`** — 两张 `map[handshakeState]stateHandler` 流转表，每条一行 `state(matchRule{...}, next)`
- **`consumeServerHello()`** — 唯一的版本判定点：读首 record，处理 HRR（TLS 1.3 retry），根据 `handshakeMsgs` 类型返回 next 状态
- **`observeTLS()`** — 状态机遍历（~14 行）：`for state != stateDone { fsm[state](fo) }`

### `utils.go`（403 行）— 通用工具

- **`tlsRecord`** + **`handshakeMsg`** — TLS record 结构体（含自动解析的 `handshakeMsgs`）
- **`readTlsRecord()`** — 读取 record 并自动解析 Handshake record 中的握手消息类型
- **`warpConn`** — AES-GCM 加密封装 `net.Conn`，递增 seq 做 nonce 防重放
- **`handshakeState`** 枚举 — 17 个状态（1-99 服务端，100+ 客户端）
- **`matchRule`** + **`match()`** — 握手 record 匹配规则
- **`state()`** — 通用状态 handler 工厂（~20 行），替代了 10 个专用工厂函数
- **`generateNonce()`** — HKDF + 过期时间窗口
- **`dup()`**、**`isHelloRetryRequest()`** — 工具函数

### `client.go` — 客户端

- `ClientConfig`、`NewClient()`（utls 发起 TLS 握手，将临时公钥和加密数据写入 Client Hello，验证服务端 ed25519 签名后返回 `warpConn`）
- `Marshal()`/`UnmarshalClientConfig()`（配置 zlib 压缩后嵌入二进制 1024 字节占位符）

### `server_test.go` — 36 个真实目标集成测试

覆盖国内主流电商/社交/技术站点，TLS 1.2 和 TLS 1.3 各半。

### 命令行入口 (`cmd/`)

- **`cmd/grss/`**（服务端）：`gen`（生成配置+客户端/用户端二进制）、`serv`（监听，根据 `overlayData` 分发 grsc/grsu 连接）、`sessionManager`（128 槽位 yamux session）
- **`cmd/grsc/`**（客户端）：`reality.NewClient()` 连接服务端，建立 `yamux.Client()` session，接受 stream 交本地 SOCKS5 server
- **`cmd/grsu/`**（用户端）：本地监听 SOCKS5，通过 `reality.NewClient()` 连服务端，`OpenStream()` 发往服务端

## 通信协议栈

```
应用数据
  └── yamux 多路复用 (stream 复用)
       └── warpConn (AES-GCM 加密 + TLS record 封装)
            └── TLS 1.2/1.3 连接伪装 (utls 指纹)
                 └── TCP
```

## 握手流程架构

### 私有握手

1. 客户端用临时 x25519 公钥替换 Client Hello 的 `Random`，用加密的 `REALITY`+随机数据替换 `Session ID`
2. 服务端 ECDH 后解密验证前缀，成功则用 ed25519 签名原文明文返回
3. 所有数据通过 `warpConn` 加密为 TLS Application Data record 传输
4. 握手失败或非授权连接 → 服务端透明转发到伪装目标站点

### 服务端状态机（两张表 + 一个循环）

`consumeServerHello` 读首条 ServerHello，根据 `handshakeMsgs` 类型返回 next 状态：

```
HRR → TLS 1.3 retry → 获取真正 SH
SH 合并（msgs[1]=Certificate） → stateClientCert → clientFSM → stateSrvTicket → serverFSM
SH 单条 + next=CCS/AppData → stateSrvCCS13 → serverFSM → clientFSM（TLS 1.3）
SH 单条 + next=Certificate → stateSrvSKX → serverFSM → clientFSM → serverFSM（TLS 1.2）
```

`observeTLS` 是简单遍历：
```go
state := stateServerHello
for state != stateDone {
    state = (serverFSM|clientFSM)[state](fo)
}
```

每个状态通过 `state(matchRule{htype/rtype, optional, fromClient, greedy}, next)` 生成 handler，统一处理：读取 record → 匹配 → 可选 buffer/跳过 → 贪心消费 AppData → 返回 next。

### `matchRule` 工厂

```
matchRule{htype: typeXxx}               → 按握手类型匹配（必须）
matchRule{htype: typeXxx, optional:true} → 按握手类型匹配（可选，不匹配则 buffer）
matchRule{rtype: typeXxx}                → 按 record 类型匹配（必须）
matchRule{rtype: typeXxx, greedy:true}   → 匹配后贪心消费同类型 record
```

## 与上游 XTLS/REALITY 的差异

- 使用**两组**预共享密钥：x25519（ECDH）+ ed25519（验签），而非一组
- 支持 **TLS 1.2 和 1.3** 伪装目标（推荐 AEAD 密码套件）
- 不修改 TLS 库，通过 `io.TeeReader` 双向转发监听握手状态

## 关键外部依赖

| 依赖 | 用途 |
|------|------|
| `github.com/refraction-networking/utls` | TLS 指纹伪装 |
| `github.com/hashicorp/yamux` | 单 TCP 连接多路复用 |
| `github.com/armon/go-socks5` | SOCKS5 代理服务 |
| `github.com/jessevdk/go-flags` | 命令行参数解析 |
| `github.com/sirupsen/logrus` | 结构化日志（`-d` 开启 Debug） |

---
> Source: [howmp/reality](https://github.com/howmp/reality) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
