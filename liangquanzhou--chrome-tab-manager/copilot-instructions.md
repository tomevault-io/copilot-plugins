## chrome-tab-manager

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CTM Project — AI Coding Rules

**CTM = terminal-first browser workspace manager**

单二进制 Go 程序，通过 Chrome Extension + Native Messaging + Unix Socket 实现终端对浏览器的完整控制、长期资源管理、跨设备同步。

产品四层模型（见 `doc/01_PRODUCT.md`）：
1. Browser Runtime Layer — 控制浏览器运行态（tabs/groups/windows/targets）
2. Library Layer — 维护长期知识库（sessions/collections/bookmarks mirror/tags/workspaces）
3. Sync Layer — 云同步（Google Sync 管书签，iCloud 管 CTM library）
4. Interaction Layer — CLI + TUI + 命令面板

本文件是 AI agent 编码时的硬性约束。每次动手写代码前必须读。

## 构建与测试命令

```bash
# 构建
go build ./...

# 静态检查
go vet ./...

# 单元测试（必须带 -race）
go test -race ./...

# 单个包测试
go test -race ./internal/protocol/
go test -race ./internal/config/
go test -race ./internal/client/

# Fuzz 测试
go test -fuzz=FuzzNDJSON ./internal/protocol/ -fuzztime=30s
go test -fuzz=FuzzNMFrame ./internal/nmshim/ -fuzztime=30s

# 运行
go run . version
go run . daemon --foreground
go run . tui
```

## 架构概览

```
Chrome Extension (JS, 不变)
    |
    | Chrome Native Messaging (stdin/stdout, 4-byte length prefix)
    |
ctm nm-shim (Go)
    |
    | NDJSON over Unix socket
    |
ctm daemon (Go, 长驻进程, Hub actor 模式)
    |
    +--- ctm tui (Bubble Tea, 持久连接)
    +--- ctm CLI (cobra, 一次性连接)
```

所有组件编译为同一个二进制 `ctm`，cobra 子命令区分角色。

### 模块依赖图

```
cmd/ → internal/tui, internal/client, internal/daemon, internal/config
internal/tui → internal/protocol, internal/config (禁止 import daemon)
internal/client → internal/protocol, internal/config
internal/daemon → internal/protocol, internal/config
internal/nmshim → internal/protocol, internal/config
internal/protocol → nothing (叶节点)
internal/config → nothing (叶节点)
```

### 技术栈

| 用途 | 库 |
|------|-----|
| CLI | `cobra` |
| TUI | `bubbletea` + `lipgloss` + `bubbles` |
| 剪贴板 | `atotto/clipboard` |
| JSON | 标准库（禁止第三方 JSON 库） |

### 当前 Phase

以 `doc/18_CURRENT_PHASE.md` 为准。Phase 定义见 `doc/11_PLAN.md`。

## 必读文档（按优先级）

**所有文档在 `doc/` 目录下**。编码前必须先读对应文档，不可凭记忆猜测。

导航索引：**`doc/00_START_HERE.md`**

产品方向：
1. **`doc/01_PRODUCT.md`** — 产品定义、四层模型、Source of Truth、原则、决策日志
2. **`doc/02_DOMAIN.md`** — 16 个领域对象、属性、生命周期、关系
3. **`doc/03_CAPABILITIES.md`** — 7 个能力域、8 个 build 阶段、feature 列表
4. **`doc/04_INTERACTION.md`** — 导航、command surfaces、用户旅程

实现方案：
1. **`doc/12_CONTRACTS.md`** — 每个 action 的 request/response 精确 JSON 格式。不可猜测。
2. **`doc/09_DESIGN.md`** — 模块职责、接口定义、依赖方向。不可违反依赖图。
3. **`doc/10_TUI.md`** — 键绑定、交互不变量、状态机。不可在 view 代码中硬编码按键。
4. **`doc/13_ACCEPTANCE.md`** — 当前 Phase 的完成标准。不可跳过验证。
5. **`doc/14_LESSONS.md`** — 已知陷阱，每条附检查项。

流程守则：
1. **`doc/15_AGENT_RULES.md`** — Agent 操作守则，防止偷偷缩小产品范围
2. **`doc/16_CHANGE_POLICY.md`** — 需求变更流程：先改文档再改代码

## 编码红线（违反任何一条 = 必须修复后才能继续）

### 架构红线
- **禁止循环依赖**：`protocol` 和 `config` 是叶节点，不 import 任何 internal 包
- **禁止 tui 直接 import daemon**：tui 通过 client 接口与 daemon 通信
- **禁止在非 Hub goroutine 中访问 Hub 状态**：所有可变状态在 Hub.Run 中访问
- **禁止跳过原子写入**：session/collection 文件必须 tmp → fsync → rename

### 协议红线
- **list 返回 summary，get 返回 full**：`sessions.list` 返回 `tabCount`，不返回 `tabs[]`
- **event 必须含 _target**：每个 event payload 必须包含 `_target.targetId`
- **response ID 必须匹配 request ID**
- **不可猜测 response shape**：必须参考 `doc/12_CONTRACTS.md`

### TUI 红线
- **键绑定唯一来源**：所有键绑定定义在 `keymap.go`，help/statusbar 从这里生成
- **Esc 不可修改状态**：只能取消/返回/清除
- **持久删除必须 D-D 确认**
- **cursor isolation**：数据更新只影响对应 view 的 cursor/selection
- **三通道反馈**：Toast（自动消失）、Error（Esc 清除）、ConfirmHint（非确认键清除）

### 代码质量红线
- **不 panic**（除非真正不可恢复）
- **不吞 error**（每个 error 要么处理要么传播）
- **session/collection name 必须 validate**：`^[a-zA-Z0-9_-]+$`，最长 128 字符
- **Writer 必须线程安全**（多 goroutine 写同一 conn）
- **Reader 不需要线程安全**（每 conn 单读 goroutine）

## 编码流程（每个功能必须按此顺序）

```
1. 读 doc/12_CONTRACTS.md → 确认 action 的 request/response 格式
2. 读 doc/09_DESIGN.md → 确认该功能属于哪个 module，接口是什么
3. 读 doc/10_TUI.md → 如涉及 TUI，确认键绑定和交互
4. 写代码
5. 写测试
6. 对照 doc/13_ACCEPTANCE.md → 逐项验证当前 Phase 的测试
7. 对照 doc/14_LESSONS.md → 检查相关检查项
```

## 不确定时的行为

- **不确定 response shape** → 读 `doc/12_CONTRACTS.md`，不要猜
- **不确定键绑定** → 读 `doc/10_TUI.md`，不要猜
- **不确定模块职责** → 读 `doc/09_DESIGN.md`，不要猜
- **不确定是否需要做** → 读 `doc/11_PLAN.md` 的 Phase 定义，当前 Phase 不需要的功能不做
- **遇到设计决策** → 停下来问用户，不要自己决定

## 自检清单（每次提交前）

- [ ] `go build ./...` 通过
- [ ] `go vet ./...` 无警告
- [ ] `go test -race ./...` 通过
- [ ] 新增 action 的 response 格式与 `doc/12_CONTRACTS.md` 一致
- [ ] 新增键绑定在 keymap.go 中注册
- [ ] 没有引入循环依赖
- [ ] 没有在 Hub.Run 外部访问 Hub 状态
- [ ] session/collection 写入使用 atomicWriteJSON

## Phase 管控

当前正在执行的 Phase 以 `doc/11_PLAN.md` 为准。参见 `doc/03_CAPABILITIES.md` 了解领域建设顺序。
- **不要提前实现后续 Phase 的功能**
- **但当前 Phase 的设计必须兼容 build order 定义的领域骨架**（持久对象必须有 UUID + updatedAt，路由必须可扩展，ViewType 必须用 iota）
- **不要添加 `doc/11_PLAN.md` 中未提及的功能**
- 如果发现当前 Phase 的需求不明确，停下来问用户

## 外部模型调用规范

调用 Codex 或 Gemini 时，**始终使用当前可用的最强模型、最高性能配置**：

| 工具 | 模型 | 配置 |
|------|------|------|
| Codex CLI | `gpt-5.4`（或更新的最强模型） | `-s danger-full-access -c model_reasoning_effort="xhigh"` |
| Codex MCP | `gpt-5.4` | sandbox: `read-only` |
| Gemini CLI | `gemini-3.1-pro-preview`（或更新的最强模型） | — |
| Gemini MCP | `gemini-3.1-pro-preview` | — |

不要使用默认模型（gpt-5.3-codex / gemini-2.5-flash），除非用户明确指定。

## 技术约定

- JSON 编解码用标准库，不引入第三方 JSON 库
- 错误处理用 `fmt.Errorf("xxx: %w", err)` 包装，保留错误链
- context 贯穿所有 I/O 操作
- 日志用 `log.Printf`，格式 `[daemon HH:MM:SS] message`
- 测试文件与被测文件同目录，命名 `xxx_test.go`

---
> Source: [liangquanzhou/chrome-tab-manager](https://github.com/liangquanzhou/chrome-tab-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
