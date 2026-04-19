## aleph

> 以下为最高优先级约束，所有开发决策必须遵守。违反红线的代码不得合入。

# CLAUDE.md

## 🛑 架构红线 (Architectural Redlines)

以下为最高优先级约束，所有开发决策必须遵守。违反红线的代码不得合入。

### R1. 大脑与四肢绝对分离 (Brain-Limb Separation)

- **禁令**: 严禁在 `src` 中直接调用特定平台系统 API (AppKit, Vision, CoreGraphics, windows-rs)
- **原则**: 核心层只定义"能力契约 (Trait)"，物理实现由 Desktop Bridge (Tauri-Rust) 通过 IPC 提供

### R2. UI 逻辑唯一源 (Single Source of UI Truth)

- **禁令**: 严禁在 Tauri 中实现具有业务逻辑的复杂设置页面、表单或列表
- **原则**: 所有复杂业务 UI 在 Leptos (WASM) 中实现。原生外壳仅负责窗口容器、原生动画和菜单栏

### R3. 核心轻量化 (Core Minimalism)

- **禁令**: 严禁为单一非核心功能在 core 中引入沉重的第三方库
- **原则**: 优先实现为 Skill (Python/Bash) 或 MCP Server。内核只调度，不搬砖
- **备注**: 代码层面的奥卡姆剃刀原则和 Rust 大文件拆分规范与此不冲突

### R4. Interface 层禁止业务逻辑 (I/O-Only Interfaces)

- **禁令**: 禁止在 App/Bot/CLI 中处理数据持久化、记忆检索或任务规划逻辑
- **原则**: Interface 层是"纯 I/O"— 输入转为 JSON-RPC 发给 Server，响应渲染给用户

### R5. 菜单栏优先，按需展窗 (Menu Bar First)

- **默认形态**: macOS 无 Dock 图标，菜单栏常驻，Halo 浮窗为主要快捷交互入口
- **允许窗口**: 复杂场景（设置、长对话、调试面板）应使用正常窗口，不要为"隐形"牺牲可用性
- **原则**: 轻量入口 + 按需展开，而非"绝对无窗口"

### R6. AI 主动到达 (AI Comes to You)

- **原则**: 减少用户切换上下文的成本，AI 尽量在用户当前工作环境中提供帮助
- **实现**: Halo 浮窗、通知、内联建议等
- **边界**: 不打扰用户 (不抢焦点、不弹模态对话框)，但不要因此拒绝提供必要的 UI

### R7. 一核多端 (One Core, Many Shells)

- **原则**: Rust Core 是唯一大脑，UI 通过 Leptos/WASM 统一，原生壳只负责窗口容器和系统集成
- **备注**: 这已在 R1 和 R2 中体现，此条作为产品层面的重申

### R8. LLM 主权原则 (LLM Sovereignty)

- **禁令**: 严禁用确定性代码替代 LLM 擅长的推理判断（意图识别、任务评估、路由决策、内容分类等）
- **原则**: 极简系统 + 强大 prompt = 释放 LLM 全部推理能力。把复杂留给模型，把简单留给系统
- **判断标准**: 对每个模块问 — "这是在赋能 LLM（给它做不到的能力），还是在越俎代庖（替它做推理）？"
- **赋能层（保留）**: Gateway 多端触达、Memory 持久记忆、Daemon 事件感知、Soul 人格、Provider 多厂商、Tool 执行能力、MCP 外部服务、Extension 插件生态、上下文压缩、安全硬过滤
- **越俎代庖（禁止）**: Intent Detection 规则引擎、POE 目标验证管线、多层 Tool Filter、Context Aggregation 多层合并、Dispatcher 意图分析

### R9. 工具即一切 (Everything is a Tool)

- **原则**: Aleph 自身的所有可配置操作都应暴露为工具，让 LLM 通过自然语言对话完成配置
- **实现**: Agent 管理（创建/切换/删除）、Provider 配置、Channel 配置、Skill/MCP 安装卸载、Daemon 订阅规则 — 全部是 Tool
- **核心循环**: `用户自然语言 → LLM 理解意图 → LLM 选择工具 → 工具执行 → 结果返回 LLM → LLM 回复用户`
- **效果**: 对话即管理面板。用户无需学习配置文件或 API，自然语言驱动一切

### R10. 智慧在 Prompt 中 (Intelligence Lives in the Prompt)

- **原则**: 被移除的中间件的"智慧"不是丢弃，而是迁移到 system prompt 模板中
- **实现**: 主循环的 LLM 一次调用自然覆盖所有判断（意图理解 + 工具选择 + 安全评估 + 完成度判断）
- **效果**: 零额外 LLM 调用，零中间件税，模型推理能力完整释放

---

## 🧬 设计原则 (Design Principles)

以下原则指导 Aleph 每一行代码的编写决策，是架构红线之下的工程纪律。

### P1. 低耦合 (Low Coupling)

- **模块间通过 Trait 通信，不依赖具体实现** — 模块只知道对方的契约，不知道对方的内部结构
- **禁止跨层直接调用** — Core 不直接调用 UI，UI 不直接操作数据库，Interface 不处理业务逻辑
- **依赖方向单向流动** — `Interface → Core → Domain`，禁止反向依赖
- **事件驱动解耦** — 模块间优先通过事件/消息传递状态变化，而非直接方法调用

### P2. 高内聚 (High Cohesion)

- **单一职责** — 每个模块/struct/函数只做一件事，做好一件事
- **相关逻辑物理聚合** — 紧密相关的类型、函数、trait 放在同一模块目录下，不要分散到不同子系统
- **命名即文档** — 模块名、函数名、类型名应准确反映其唯一职责，无需注释解释"它是干什么的"
- **大文件及时拆分** — 单文件超过 500 行应考虑按职责拆分为子模块 (参见 [CODE_ORGANIZATION.md](docs/reference/CODE_ORGANIZATION.md))

### P3. 可扩展性 (Extensibility)

- **开放-封闭原则 (OCP)** — 对扩展开放，对修改封闭。新增功能通过实现 trait / 注册插件完成，不修改已有核心逻辑
- **策略模式优于条件分支** — 用 trait object / enum dispatch 替代 `if-else` 链或 `match` 的无限膨胀
- **插件化优先** — 非核心功能优先实现为 Skill / MCP Server / WASM 插件，而非硬编码进 Core
- **Schema 驱动** — 接口使用 JSON Schema (schemars) 自描述，新增字段不破坏旧客户端

### P4. 依赖倒置 (Dependency Inversion)

- **高层模块不依赖低层模块，两者都依赖抽象** — Core 定义 trait，具体实现在 crate 边界之外
- **实践**: `DesktopCapability` trait 在 core 中定义，native 实现在 `desktop/shared/`；`MemoryStore` trait 在 core 中定义，LanceDB 实现在同层但可替换
- **构造时注入** — 通过 `AppContext` / Builder 模式在启动时组装依赖，运行时不 `new` 具体类型

### P5. 最小知识原则 (Least Knowledge / Law of Demeter)

- **只与直接协作者通信** — `a.b().c().d()` 链式调用是设计缺陷的信号
- **封装内部结构** — 不暴露 struct 内部字段的引用链，提供有意义的方法代替
- **接口最小化** — pub API 只暴露调用者真正需要的，`pub(crate)` 优于 `pub`

### P6. 简洁性 (Simplicity — KISS & YAGNI)

- **奥卡姆剃刀** — 如无必要，勿增实体。不为假想的未来需求预留抽象
- **三次法则** — 代码重复不超过两处时不要提前抽象，第三次出现再提取
- **删除优于注释** — 废弃代码直接删除，不要注释掉保留。Git 是时光机
- **扁平优于嵌套** — 优先使用 early return / `?` 操作符，减少缩进层级

### P7. 防御性设计 (Defensive Design)

- **系统边界校验** — 在用户输入、外部 API 响应、IPC 消息的入口处严格校验，内部传递信任已校验的数据
- **优雅降级** — 外部依赖 (LLM/网络/文件系统) 失败时提供 fallback，不 panic
- **锁安全** — `.lock().unwrap_or_else(|e| e.into_inner())`，永远处理 poison
- **UTF-8 安全** — 字符串切片使用 `char_indices()` / `.get(..n)`，不用 `&s[..n]`

### P8. LLM 优先 (LLM-First)

- **语义理解交给 LLM** — 所有需要将自然语言转换为结构化意图的任务（意图识别、参数提取、命令路由），优先使用 LLM 语义理解，而非正则表达式或关键词匹配
- **禁止脆弱的模式匹配** — 不要用 regex 解析用户自然语言输入。正则只适用于格式固定的机器生成文本（如 JSON、URL、日志格式）
- **LLM 可做则 LLM 做** — 如果一项任务 LLM 能完成（分类、提取、推理、生成），就交给 LLM，而非硬编码规则
- **结构化输出** — LLM 返回 JSON 结构化结果，代码层只负责解析和执行，不做语义判断

---

## 🔧 开发指南

### 构建命令

| Command | Description |
|---------|-------------|
| `cargo run --bin aleph-server` | Start server (debug) |
| `cargo check -p alephcore` | Quick compile check |
| `cargo test -p alephcore --lib` | Run core tests |
| `just dev` | Dev server (rebuilds WASM first) |
| `just build` | Release build (WASM + server) |
| `just test-all` | All tests (core + desktop + proptest) |
| `just clippy` | Lint |
| `just release YYYY.MM.DD` | **发版**: 更新 VERSION + 提交推送 + 触发 GitHub workflow（需先写 changelog） |

### 版本管理

- **CalVer (日历版本)** — 格式 `YYYY.MM.DD`（如 `2026.03.29`），每天最多发布一个版本
- **VERSION 文件是唯一版本源** — `build.rs` 读取 VERSION → 注入 `ALEPH_VERSION` 环境变量 → 所有代码通过 `env!("ALEPH_VERSION")` 使用
- **禁止** 在代码中硬编码版本号，使用 `env!("ALEPH_VERSION")` 代替 `env!("CARGO_PKG_VERSION")`
- Panel System Info、Gateway 版本、MCP/ACP 协议版本、CLI --version 全部从 VERSION 文件读取
- GitHub workflow 也读取 VERSION 文件作为 release tag

### 发版流程 (Release Process)

**由 AI (Claude) 驱动的两步流程：**

1. **AI 写版本日志** — 读取**上一个 release 版本到 HEAD 之间**的 git log（通过 `git log <上次release commit>..HEAD`），总结 10-20 条有价值的内容，分为 Added（新增功能）和 Fixed（修复）两个分类，写入 CHANGELOG.md
2. **运行 `just release YYYY.MM.DD`** — 自动完成：版本号更新 + 提交推送 + 触发四平台构建

`just release` 会校验 CHANGELOG.md 中是否有对应版本的条目，没有则拒绝发布。GitHub Release 页面自动从 CHANGELOG.md 提取版本日志。

### Feature Flags

所有生产功能始终编译，无需 feature flags。仅保留测试用 features：`loom` (并发测试)、`test-helpers` (集成测试工具)。

### 提交规范

English commit messages. Format: `<scope>: <description>` — Example: `gateway: add WebSocket server foundation`

### 分支策略

**单分支开发模式**：所有开发工作直接在 main 分支进行。

### 语言规范

- Reply in Chinese
- Code comments in English
- Documentation in both

### Git Worktree 注意事项

`EnterWorktree` 会在每次 Bash 命令后强制重置 CWD 到 worktree 目录，即使 `cd` 切回主仓库也无效。因此在同一会话内执行 `git worktree remove` 会导致 Shell 永久损坏。**正确做法**：在 `EnterWorktree` 会话内只合并不删除，用新会话清理 worktree；或不用 `EnterWorktree`，手动用绝对路径管理。

### 进程管理 (Process Management) ⚠️ CRITICAL

**重启 Aleph 前必须杀掉所有旧进程**。多个 aleph 进程同时运行会竞争写入 `.shared_token` 文件，导致 HMAC 验证失败和 **vault 数据永久丢失**（所有 API key、OAuth token、embedding key 等）。

```bash
# 重启前：杀掉所有 aleph 进程（debug + release）
pkill -f "target/release/aleph-server" 2>/dev/null
pkill -f "target/debug/aleph-server" 2>/dev/null
sleep 2
# 确认全部清除
ps aux | grep "[a]leph-server" | grep -v zsh | grep -v cp | grep -v tail

# 然后再启动
target/release/aleph-server start
```

**绝对禁止**：
- 在有旧 aleph 进程运行时启动新进程
- 同时运行多个使用同一 `~/.aleph/data/` 的 aleph 实例
- 用 `kill -9` 强制杀进程后不等待就立即重启（等 2 秒让文件锁释放）

---

## 📚 文档索引

| 文档 | 链接 |
|------|------|
| ARCHITECTURE.md | [docs/reference/ARCHITECTURE.md](docs/reference/ARCHITECTURE.md) |
| AGENT_SYSTEM.md | [docs/reference/AGENT_SYSTEM.md](docs/reference/AGENT_SYSTEM.md) |
| MULTI_AGENT_SYSTEM.md | [docs/reference/MULTI_AGENT_SYSTEM.md](docs/reference/MULTI_AGENT_SYSTEM.md) |
| GATEWAY.md | [docs/reference/GATEWAY.md](docs/reference/GATEWAY.md) |
| TOOL_SYSTEM.md | [docs/reference/TOOL_SYSTEM.md](docs/reference/TOOL_SYSTEM.md) |
| MEMORY_SYSTEM.md | [docs/reference/MEMORY_SYSTEM.md](docs/reference/MEMORY_SYSTEM.md) |
| EXTENSION_SYSTEM.md | [docs/reference/EXTENSION_SYSTEM.md](docs/reference/EXTENSION_SYSTEM.md) |
| PLUGIN_SYSTEM.md | [docs/reference/PLUGIN_SYSTEM.md](docs/reference/PLUGIN_SYSTEM.md) |
| SECURITY.md | [docs/reference/SECURITY.md](docs/reference/SECURITY.md) |
| DESIGN_PATTERNS.md | [docs/reference/DESIGN_PATTERNS.md](docs/reference/DESIGN_PATTERNS.md) |
| CODE_ORGANIZATION.md | [docs/reference/CODE_ORGANIZATION.md](docs/reference/CODE_ORGANIZATION.md) |
| DOMAIN_MODELING.md | [docs/reference/DOMAIN_MODELING.md](docs/reference/DOMAIN_MODELING.md) |
| AGENT_DESIGN_PHILOSOPHY.md | [docs/reference/AGENT_DESIGN_PHILOSOPHY.md](docs/reference/AGENT_DESIGN_PHILOSOPHY.md) |
| SERVER_DEVELOPMENT.md | [docs/reference/SERVER_DEVELOPMENT.md](docs/reference/SERVER_DEVELOPMENT.md) |

---

## 🏢 官方仓库 (Official Repositories)

| 仓库 | 路径 | 说明 |
|------|------|------|
| Aleph (主项目) | `/Users/zouguojun/Workspace/Aleph` | Rust Core + 多端架构 |
| Aleph-docs | `/Users/zouguojun/Workspace/Aleph-docs` | 官方文档 |
| Aleph-homepage | `/Users/zouguojun/Workspace/Aleph-homepage` | 官方首页 |
| Aleph-mcp | `/Users/zouguojun/Workspace/Aleph-mcp` | 官方 MCP 项目 |
| Aleph-plugins | `/Users/zouguojun/Workspace/Aleph-plugins` | 官方插件 |
| Aleph-skills | `/Users/zouguojun/Workspace/Aleph-skills` | 官方技能 |

---

## 📝 Session Context

- **项目**: 自托管个人 AI 助手，Rust Core + 多端架构
- **核心循环**: Think → Act（极简两步循环，LLM 主权原则）
- **语言**: 使用中文对话

### Memory Prompt

When token is low to 10%, summarize this session to generate a "memory prompt" for next session inheritance.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rootazero) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
