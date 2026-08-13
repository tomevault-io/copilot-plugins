## pulsar-lite

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 文档导航

根据你的需求查看相应文档：

- **项目介绍**: `README.md` - 快速开始、功能列表、使用示例
- **项目概览**: `docs/PROJECT_OVERVIEW.md` - 架构设计、技术栈、项目进度
- **贡献指南**: `docs/CONTRIBUTING.md` - 环境设置、开发流程、代码规范
- **协议细节**: `docs/PULSAR_BINARY_PROTOCOL.md` - Pulsar 协议实现细节

## 快速参考

### 核心概念

Pulsar Lite 是嵌入式轻量级消息队列，实现标准 Pulsar 二进制协议。关键特性：
- **双模式**: 嵌入式（本地文件）或远程（pulsar://URI）
- **协议兼容**: 使用官方 Pulsar 客户端，无需修改代码
- **进程管理**: Python SDK 自动管理服务器生命周期

### 常用命令

```bash
# 构建
make build              # 构建 Rust broker

# 测试
make test              # 所有测试
make test-rust         # Rust 单元测试
make test-python       # Python 集成测试

# 开发
rust/pulsar-lite.sh start                          # 启动服务器
tail -f /tmp/pulsar-lite.log                       # 查看日志

# Perf 压测
python3 tests/perf/run_non_persistent_stress.py        # Stress 压测（不限速找 ceiling）
python3 tests/perf/run_non_persistent_e2e_matrix.py    # Coverage 基线（速率控制）

# 代码质量
make fmt              # 格式化代码
make lint             # 代码检查
```

### 项目结构速览

```
pulsar-lite/
├── rust/src/
│   ├── main.rs              # 入口
│   ├── broker/              # 服务核心
│   │   ├── service.rs      # 连接管理
│   │   └── handler.rs      # 命令处理
│   ├── protocol/codec.rs   # 帧编解码
│   └── storage/mod.rs      # 存储
│
├── python/src/pulsar_lite/
│   ├── client.py            # 主客户端
│   ├── process_manager.py   # 进程管理
│   └── binary_finder.py     # 二进制定位
│
└── tests/
    └── test_binary_protocol.py  # 协议测试
```

### 关键技术点

**协议实现**:
- 使用官方 `PulsarApi.proto`
- 帧格式: `[4B 总大小][4B 命令大小][Protobuf 命令][可选元数据+载荷]`
- 已实现: Connect, Lookup, Producer, Send, CloseProducer, Ping/Pong
- 待实现: Subscribe, Flow, Message, Ack

**进程管理**:
- Python SDK 使用单例 ProcessManager
- 引用计数支持多客户端共享
- 自动端口分配（6650+）
- 日志输出到 `/tmp/pulsar-lite.log`

**双模式切换**:
```python
# 嵌入式 - 本地文件自动启动服务器
client = PulsarClient("./my.db")

# 远程 - 连接独立服务器
client = PulsarClient("pulsar://localhost:6650")
# 或使用官方客户端
client = pulsar.Client("pulsar://localhost:6650")
```

### 开发注意事项

**添加新协议命令**:
1. 在 `rust/src/broker/handler.rs` 添加处理函数
2. 在 `rust/src/protocol/codec.rs` 的 `ServerCommand` 枚举添加变体
3. 在 `rust/src/protocol/codec.rs` 实现编码逻辑
4. 在 `tests/test_binary_protocol.py` 添加测试

**修改 Python SDK**:
- 客户端逻辑: `python/src/pulsar_lite/client.py`
- 进程管理: `python/src/pulsar_lite/process_manager.py`
- 所有方法都代理到 `pulsar.Client`

**存储相关**:
- 当前: 内存 HashMap（`rust/src/storage/mod.rs`）
- 消息 ID: `(ledger_id, entry_id)` 格式
- 计划: 集成 RocksDB

### 测试策略

- **单元测试**: `cargo test` (Rust)
- **集成测试**: `pytest tests/` (Python + 官方客户端)
- **手动测试**: 启动 broker + 运行 Python 脚本

### 常见问题

**Q: protoc 未找到**
```bash
brew install protobuf  # macOS
```

**Q: 测试连接被拒绝**
```bash
# 使用脚本启动服务器
rust/pulsar-lite.sh start
sleep 2
```

**Q: 清理构建**
```bash
make clean
```

### 代码风格

- **Rust**: `cargo fmt` + `cargo clippy`（零警告）
- **Python**: Black (100字符) + Ruff (E,F,I,N,W)
- **提交**: 约定式提交（feat/fix/docs/test/refactor）

## 更多信息

详细架构设计 → `docs/PROJECT_OVERVIEW.md`
开发流程和规范 → `docs/CONTRIBUTING.md`
协议细节 → `docs/PULSAR_BINARY_PROTOCOL.md`

---
> Source: [ascentstream/pulsar-lite](https://github.com/ascentstream/pulsar-lite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
