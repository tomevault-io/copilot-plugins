## zmq-go-starter

> Full-duplex ZeroMQ communication example using pure Go implementation.

# ZeroMQ Go Starter / ZeroMQ Go 入门

## Overview / 概述

Full-duplex ZeroMQ communication example using pure Go implementation.
纯 Go 实现的 ZeroMQ 全双工通信示例。

## ZeroMQ Libraries / ZeroMQ 库

- **Use `github.com/go-zeromq/zmq4` v0.5.0** - Pure Go, no C dependencies / 纯 Go，无 C 依赖
- **Avoid / 避免** `gopkg.in/zeromq/goczmq.v4` and `github.com/pebbe/zmq4` - Both require system C libraries (czmq.h / zmq.h) / 都需要系统 C 库

## Run / 运行

```bash
go run main.go
```

## Notes / 注意事项

- PAIR socket provides 1-to-1 full-duplex communication / PAIR 套接字提供一对一全双工通信
- Peer2 listens on `tcp://*:5555`, Peer1 dials to connect / Peer2 监听 `tcp://*:5555`，Peer1 拨号连接
- Uses `context.Background()` and `zmq4.NewPair(ctx)` to create sockets / 使用 `context.Background()` 和 `zmq4.NewPair(ctx)` 创建套接字

---
> Source: [qiuzhanghua/zmq-go-starter](https://github.com/qiuzhanghua/zmq-go-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
