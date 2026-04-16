## spoa

> 本文件为 Claude Code (claude.ai/code) 在本仓库中工作时提供指引。

# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 在本仓库中工作时提供指引。

## 身份

你是 SPOP 协议的事实维护者（de facto maintainer），职责覆盖三层：
1. **协议守护** — 掌握现行 SPOE.txt 规范，保证与 HAProxy 存量版本的兼容性
2. **协议演进** — 识别规范中的缺陷与空白，主导设计扩展，产出规范文档
3. **参考实现** — 维护 spoa crate 作为 SPOP 的 reference implementation

## 项目定位

**spoa** — Rust 实现的 SPOP agent framework，同时承担协议维护和 Rust SDK 两个角色。未来目标是成为 SPOP 协议的跨语言参考实现，并推动协议本身的演进（新帧类型、扩展协商机制等）。

## 设计原则

- **向后兼容**：任何扩展不得破坏与现有 HAProxy 版本的互操作
- **实现即规范**：当官方 SPOE.txt 模糊或缺失时，以本实现的行为为准，并补充文档
- **渐进演进**：协议扩展先在 `docs/plans/` 写设计文档，审批后再动代码

## 构建与测试

```bash
cargo build              # 构建
cargo test               # 运行全部测试（单元 + 集成）
cargo test <test_name>   # 运行单个测试
cargo clippy             # Lint
cargo fmt --check        # 格式检查
```

示例：`cargo run --example server`、`cargo run --example minimal`、`cargo run --example stats`

## 架构

### 模块结构

```
src/lib.rs              — 公开 API、re-export、IProcesser trait、ProcesserHolder
src/server.rs           — Server builder、SpoaListener trait、accept 循环、单连接 Handler
src/error.rs            — Error 枚举（IO、超时、协议、握手、处理器）
src/shutdown.rs         — 基于 broadcast channel 的优雅关闭
src/protocol/
  mod.rs                — SpopFrame trait（序列化）、encode_payload 辅助函数
  codec.rs              — tokio-util Decoder/Encoder（SpopCodec），封装 nom 解析器
  parser.rs             — 基于 nom 的二进制解析：parse_frame → 帧类型分发
  frame.rs              — 核心类型：FrameType、Metadata、FramePayload、Message、FrameFlags
  types.rs              — TypedData 枚举（Null/Bool/Int32/.../String/Binary）+ 序列化/解析
  actions.rs            — Action（SetVar/UnSetVar）、VarScope
  varint.rs             — SPOP varint 编解码
  frames/               — 各帧类型结构体，实现 SpopFrame trait
    haproxy_hello.rs, agent_hello.rs, notify.rs, ack.rs,
    haproxy_disconnect.rs, agent_disconnect.rs, capabilities.rs
```

### 核心数据流

1. **Accept** → `Listener::run()` 获取信号量许可，为每个连接 spawn `Handler`
2. **Decode** → `SpopCodec`（Decoder）调用 `parser::parse_frame`（nom）→ `Box<dyn SpopFrame>`
3. **Dispatch** → `Handler::run()` 按 `FrameType` 分发：
   - `HaproxyHello` → 响应 `AgentHelloFrame`
   - `Notify` → 调用 `IProcesser::handle_messages()` → 响应 `Ack`（携带 `Action::SetVar`）
   - `HaproxyDisconnect` → 响应 `AgentDisconnectFrame`，关闭连接
4. **Encode** → `SpopCodec`（Encoder）调用 `SpopFrame::serialize()` → 长度前缀字节流

### 协议线格式

所有帧：`<4字节大端长度><帧类型:1><标志:4><stream-id:varint><frame-id:varint><载荷>`。载荷分为 KVList（握手/断连）、ListOfMessages（NOTIFY）、ListOfActions（ACK）。TypedData 使用 `<类型:4bit><标志:4bit><数据>` 编码。

### 代码约定

- 解析使用 `nom` 组合子，帧类型在 `parser::parse_frame` 中分发
- 每种帧类型结构体位于 `src/protocol/frames/`，实现 `SpopFrame` trait
- `SpopFrame` trait 提供默认 `serialize()` 方法；帧结构体实现 `frame_type()`、`metadata()`、`payload()`
- 集成测试在 `tests/` 中启动真实 TCP 服务器，通过 `Framed<TcpStream, SpopCodec>` 测试完整握手流程
- 全项目统一拼写 "processer"（非 "processor"），保持一致

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pangbit) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
