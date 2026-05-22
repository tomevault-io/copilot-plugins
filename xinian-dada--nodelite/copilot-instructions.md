## nodelite

> 本文档为 AI 辅助开发工具（Claude Code、Cursor、GitHub Copilot 等）提供项目上下文和开发约定。

# NodeLite - AI 开发指南

本文档为 AI 辅助开发工具（Claude Code、Cursor、GitHub Copilot 等）提供项目上下文和开发约定。

## 项目概述

NodeLite 是一个轻量级的 Rust 监控系统，采用 Server-Agent 架构。

**核心特性：**
- WebSocket 实时通信
- Token 认证 + 可选 TOTP 2FA
- SQLite 历史数据存储
- 低资源占用（服务端 4-10MB，Agent 800KB）

**性能指标：**
- 200 节点并发：18,677 指标/秒
- p95 延迟：< 5ms
- 内存占用：服务端 < 15MB，Agent < 2MB

## 架构

### 工作区结构

```
NodeLite/
├── nodelite-proto/    # 协议定义（共享类型）
├── nodelite-agent/    # Agent 端（数据采集）
└── nodelite-server/   # Server 端（数据接收和展示）
```

### 关键模块

**nodelite-server:**
- `main.rs` (1,674 行) - 服务启动和编排
- `registry.rs` (1,439 行) - 节点注册和 token 管理
- `ws.rs` (565 行) - WebSocket 连接处理
- `handlers.rs` (616 行) - HTTP 路由处理
- `history.rs` (805 行) - SQLite 历史数据
- `auth.rs` (408 行) - 认证和 2FA
- `sanitize.rs` (279 行) - 输入验证
- `admission.rs` (408 行) - 限流和准入控制
- `state.rs` (643 行) - 共享状态管理

**nodelite-agent:**
- `main.rs` - Agent 主循环
- `collector.rs` - 系统指标采集

### 数据流

```
Agent → WebSocket → Server → Registry (内存)
                           → History (SQLite)
                           → Web UI (实时显示)
```

## 代码约定

### 错误处理

**强制要求：**
- ❌ 生产代码禁止使用 `.unwrap()`
- ❌ 生产代码禁止使用 `.expect()`（除非有充分理由并注释说明）
- ✅ 使用 `?` 操作符传播错误
- ✅ 使用 `anyhow::Context` 添加错误上下文
- ✅ 测试代码可以使用 `.expect("clear error message")`

**示例：**
```rust
// ❌ 错误
let config = load_config().unwrap();

// ✅ 正确
let config = load_config()
    .context("Failed to load server config")?;
```

### 安全要求

**强制要求：**
1. **输入验证**：所有外部输入必须通过 `sanitize.rs` 验证
2. **常量时间比较**：密码/token 比较使用 `subtle` crate
3. **参数化查询**：所有 SQL 查询必须使用参数化
4. **文件权限**：敏感文件/目录强制 0600/0700
5. **CSPRNG**：使用 `getrandom` 生成 token

**禁止：**
- 硬编码密码/token
- 字符串拼接 SQL
- 不安全的随机数生成器
- 在日志中输出敏感信息

**示例：**
```rust
// ✅ 正确：常量时间比较
use subtle::ConstantTimeEq;
if token_bytes.ct_eq(expected_bytes).into() {
    // 验证通过
}

// ✅ 正确：参数化查询
conn.execute(
    "INSERT INTO nodes (id, name) VALUES (?1, ?2)",
    params![node_id, node_name],
)?;

// ❌ 错误：字符串拼接 SQL
let sql = format!("INSERT INTO nodes VALUES ('{}')", node_id);
```

### 并发模式

**推荐模式：**
- 使用 `Arc<RwLock<T>>` 共享状态
- 使用 `tokio::spawn` 处理独立任务
- 使用 RAII 管理资源（如 `WsConnectionPermit`）
- 使用 `AtomicU64` 实现无锁计数器

**示例：**
```rust
pub struct AppState {
    config: Arc<ServerConfig>,
    registry: Arc<RwLock<Registry>>,
    next_session_id: Arc<AtomicU64>,
}
```

### 测试要求

**强制要求：**
- 新功能必须有单元测试
- Bug 修复必须有回归测试
- 测试覆盖率目标：75%+
- 关键模块（auth、registry、ws）目标：85%+

**测试组织：**
```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_happy_path() {
        // 正常流程
    }
    
    #[test]
    fn test_error_handling() {
        // 错误处理
    }
}
```

## 依赖管理

### 添加依赖原则

1. **最小化依赖**：优先使用标准库
2. **安全优先**：选择维护活跃、安全记录良好的 crate
3. **版本固定**：在 `Cargo.toml` 中固定版本
4. **审查许可证**：确保兼容 MIT/Apache-2.0

### 核心依赖

- `tokio` - 异步运行时
- `axum` - Web 框架
- `rusqlite` - SQLite 绑定
- `serde`/`serde_json` - 序列化
- `tracing` - 日志
- `rustls` - TLS（不使用 OpenSSL）
- `subtle` - 常量时间比较
- `getrandom` - CSPRNG

## 性能要求

### 基准

- 200 节点并发：18,677 指标/秒
- p95 延迟：< 5ms
- 内存占用：服务端 < 15MB，Agent < 2MB

### 优化原则

1. **避免不必要的克隆**：使用引用或 `Arc`
2. **批量操作**：SQLite 写入使用事务
3. **限流保护**：防止资源耗尽
4. **连接池**：复用 SQLite 连接

## 文件大小限制

**目标：**
- 单个文件 < 500 行（理想）
- 单个文件 < 800 行（最大）
- main.rs < 200 行

**当前状态：**
- ⚠️ `main.rs` (1,674 行) - 需要拆分
- ⚠️ `registry.rs` (1,439 行) - 需要拆分

## 提交规范

### Commit Message 格式

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Type:**
- `feat` - 新功能
- `fix` - Bug 修复
- `docs` - 文档
- `refactor` - 重构
- `test` - 测试
- `chore` - 构建/工具
- `perf` - 性能优化
- `security` - 安全修复

**Scope:**
- `server` - 服务端
- `agent` - Agent 端
- `proto` - 协议定义
- `auth` - 认证模块
- `ws` - WebSocket
- `ui` - Web UI

**示例：**
```
feat(auth): add TOTP 2FA support

Implement time-based one-time password authentication:
- Add TOTP secret generation
- Add QR code generation for authenticator apps
- Add verification endpoint
- Add session-based 2FA state tracking

Closes #42
```

## 常见任务

### 添加新的 HTTP 端点

1. 在 `handlers.rs` 中添加处理函数
2. 在 `main.rs` 的路由中注册
3. 添加认证中间件（如需要）
4. 添加单元测试
5. 更新 API 文档

### 添加新的 WebSocket 消息类型

1. 在 `nodelite-proto/src/message.rs` 中定义类型
2. 在 `ws.rs` 中添加处理逻辑
3. 在 Agent 中添加发送逻辑
4. 添加集成测试
5. 更新协议文档

### 添加新的系统指标

1. 在 `nodelite-agent/src/collector.rs` 中添加采集逻辑
2. 在 `nodelite-proto/src/snapshot.rs` 中添加字段
3. 在 `sanitize.rs` 中添加验证规则
4. 在 Web UI 中添加显示
5. 添加单元测试

## 调试技巧

### 启用详细日志

```bash
RUST_LOG=debug cargo run
RUST_LOG=nodelite_server=trace cargo run
```

### 运行单个测试

```bash
cargo test test_name -- --nocapture
```

### 性能分析

```bash
cargo build --release
perf record ./target/release/nodelite-server
perf report
```

## 参考资料

- [README.md](README.md) - 用户文档
- [SECURITY.md](SECURITY.md) - 安全模型
- [CONTRIBUTING.md](CONTRIBUTING.md) - 贡献指南
- [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)

---
> Source: [XiNian-dada/NodeLite](https://github.com/XiNian-dada/NodeLite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
