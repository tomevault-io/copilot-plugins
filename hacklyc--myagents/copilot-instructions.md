## myagents

> 基于 Claude Agent SDK 的桌面端通用 Agent 产品。开源项目（Apache-2.0），使用 Conventional Commits，不提交敏感信息。

# MyAgents - Desktop AI Agent

基于 Claude Agent SDK 的桌面端通用 Agent 产品。开源项目（Apache-2.0），使用 Conventional Commits，不提交敏感信息。

## 技术栈

| 层级 | 技术 |
|------|------|
| 桌面框架 | Tauri v2 (Rust) |
| 前端 | React 19 + TypeScript + Vite + TailwindCSS |
| 后端 | Bun + Claude Agent SDK (多实例 Sidecar) |
| 通信 | Rust HTTP/SSE Proxy (reqwest via `local_http` 模块) |
| 运行时 | 双运行时：Bun（Agent Runtime / Sidecar）+ Node.js（MCP Server / 社区工具），均内置于应用包 |

## 项目结构

- `src/renderer/` — React 前端（api/、context/、hooks/、components/、pages/）
- `src/server/` — Bun 后端 Sidecar
- `src/server/plugin-bridge/` — OpenClaw Plugin Bridge（独立 Bun 进程，加载社区 Channel 插件）
- `src/cli/` — 自配置 CLI（`myagents` 命令，同步到 `~/.myagents/bin/`，详见下方说明）
- `src/shared/` — 前后端共享类型
- `src-tauri/` — Tauri Rust 层
- `specs/` — 设计文档（tech_docs/、guides/、prd/、research/）
- `bundled-agents/myagents_helper/` — 内置 MA 小助理（见下方说明）

### 内置 MA 小助理（`bundled-agents/myagents_helper/`）

应用内置了一个 AI 助手（MA 小助理），运行在 `~/.myagents/` 工作区中，职能是产品首席客服 — 帮用户诊断问题、配置工具、管理 Agent。

**核心机制**：小助理通过 `/self-config` Skill 调用内置 `myagents` CLI 工具，**直接执行**用户请求的管理操作（配置 Provider、安装 MCP、管理 Agent Channel、创建定时任务等），而不是输出操作步骤让用户自己做。CLI 通过 Admin API（`/api/admin/*`）与 Rust Management API 通信，能力与 GUI 对等。

**文件结构**：
- `CLAUDE.md` — 小助理的元认知（架构速览、日志格式、错误速查表、诊断工作流）
- `.claude/skills/self-config/SKILL.md` — CLI 操作技能（MCP/Provider/Agent/Cron/Plugin CRUD）
- `.claude/skills/support/SKILL.md` — 用户支持技能（日志分析、Bug Report 生成）

**开发约束**：
- 修改 `bundled-agents/myagents_helper/` 的 CLAUDE.md 或 Skills 后，MUST bump `ADMIN_AGENT_VERSION`（`src-tauri/src/commands.rs`），否则用户端小助理不会更新。
- 修改 `src/cli/myagents.ts` 或 `src/cli/myagents.cmd` 后，MUST bump `CLI_VERSION`（`src-tauri/src/commands.rs`），否则用户端 CLI 不会更新。两个版本门控独立运作。

## 开发命令

```bash
bun install                 # 依赖安装
./start_dev.sh              # 浏览器开发模式 (快速迭代)
npm run tauri:dev           # Tauri 开发模式 (完整桌面体验)
./build_dev.sh              # Debug 构建 (含 DevTools)
./build_macos.sh            # 生产构建
./publish_release.sh        # 发布到 R2
npm run typecheck && npm run lint  # 代码质量检查
```

---

## 核心架构约束

### 第一原则：架构延续性

**每个功能都在已有架构上生长，而不是另起炉灶。** 项目已有成熟的分层设计、通信模式、安全约束和前端规范。新功能 MUST 复用现有模块和模式（如 `local_http`、`process_cmd`、`broadcast()`、`awaitSessionTermination()`），禁止为单点需求发明新的技术方案。

开发前 MUST 做的三件事（这两份文档是项目的全局核心规范，渐进式按需读取）：
1. **读架构文档** @specs/ARCHITECTURE.md — 理解系统分层、模块边界、数据流；从这里再下钻到 `specs/tech_docs/*` 的专题文档
2. **读设计规范** @specs/DESIGN.md — 前端开发 MUST 遵循 Token/组件/页面规范
3. **搜索现有实现** — 先在代码库中搜索类似功能是否已有模式，复用而非重建

如果需求确实需要架构变更（新的通信模式、新的状态管理方式、新的进程类型），MUST 先与用户讨论方案，不得自行引入。对接外部 SDK/插件时，MUST 先读源码确认接口约定（函数签名、config schema、返回值格式），再写适配层。

### Claude Agent SDK 交互规范

项目的核心 AI 运行时是 Claude Agent SDK（`@anthropic-ai/claude-agent-sdk`），所有 Agent 会话、工具调用、子 Agent 派发都通过它驱动。SDK 持续迭代，API 行为、环境变量、消息类型可能随版本变更。

**禁止凭假设编写 SDK 交互代码。** 涉及 SDK 的任何开发（`query()` 参数、`SDKMessage` 类型处理、环境变量设置、Hook 注册、MCP 集成等），MUST 先查阅官方文档确认实际行为：
- **SDK 文档**：https://platform.claude.com/docs/zh-CN/agent-sdk/overview
- **SDK 类型定义**：`node_modules/@anthropic-ai/claude-agent-sdk/sdk.d.ts`（当前版本 0.2.111）
- **SDK 工具类型**：`node_modules/@anthropic-ai/claude-agent-sdk/sdk-tools.d.ts`

典型错误案例：臆测 `seedReadState` 的调用时机导致"先读后改"语义被绕过、臆测环境变量名导致模型别名不生效。这类问题的根因都是没有查文档就动手写代码。

### Tab-scoped 隔离

每个 Chat Tab 拥有独立的 Bun Sidecar 进程（Tab/CronTask/BackgroundCompletion/Agent 四种 Owner 共享 SidecarManager）。MUST 在 Tab 内使用 `useTabState()` 返回的 `apiGet`/`apiPost`，**禁止**在 Tab 内使用全局 `apiPostJson`/`apiGetJson`（会发到 Global Sidecar）。

### Rust 代理层

所有前端 HTTP/SSE 流量 MUST 通过 Rust 代理层（`invoke` → Rust → reqwest → Bun Sidecar），**禁止**从 WebView 直接发起 HTTP 请求。

### local_http 模块（致命陷阱）

所有连接本地 Sidecar（`127.0.0.1`）的 reqwest 客户端 MUST 通过 `crate::local_http::builder()` / `blocking_builder()` / `json_client()` / `sse_client()` 创建。内置 `.no_proxy()` 防止系统代理拦截 localhost。**禁止**裸 `reqwest::Client::builder()` 或 `reqwest::Client::new()` 连接 localhost，否则系统代理（Clash/V2Ray）会导致 502。

### process_cmd 模块（Windows 控制台窗口陷阱）

所有 Rust 层子进程 MUST 通过 `crate::process_cmd::new()` 创建，**禁止**裸 `std::process::Command::new()`。内置 Windows `CREATE_NO_WINDOW` 标志，防止 GUI 应用启动子进程（bun.exe Sidecar / Plugin Bridge / bun init 等）时弹出黑色控制台窗口。遵循与 `local_http` 相同的 "pit of success" 模式。例外：`#[cfg(windows)]` 守卫内的系统工具命令（taskkill/powershell/wmic）已内联处理；`commands.rs` 的 OS opener（open/explorer/xdg-open）和 Unix pgrep 是用户可见的系统命令，无需隐藏；`terminal.rs` 的 PTY 进程由 `portable-pty` 的 `CommandBuilder` + `slave.spawn_command()` 管理，不走 `std::process::Command`。

### proxy_config 子进程代理策略（Bun fetch 陷阱）

所有可能发起 HTTP 请求的 Rust 层子进程（Bun Sidecar、Plugin Bridge、npm install 等）MUST 在 spawn 前调用 `crate::proxy_config::apply_to_subprocess(&mut cmd)`。该函数确保：用户配置代理时注入 `HTTP_PROXY` + `NO_PROXY`；未配置时继承系统网络行为但**始终注入 `NO_PROXY`** 保护 localhost。**禁止**手动 `cmd.env("HTTP_PROXY", ...)` 或 `cmd.env_remove("HTTP_PROXY")`。Bun 的 `fetch()` 会读取 `HTTP_PROXY` 环境变量，没有 `NO_PROXY` 的话，Sidecar 内部的 localhost 通信（admin-api、cron-tool、bridge-tools 等）会被系统代理拦截 → 502。

### 零外部依赖与双运行时

应用内置三个运行时依赖，用户无需自行安装任何东西：

| 依赖 | 用途 | 打包位置 |
|------|------|---------|
| **Bun** | Agent Runtime — 运行 Claude Agent SDK（`executable: 'bun'`），Sidecar 主进程、Plugin Bridge | `src-tauri/binaries/bun-*` |
| **Node.js** | 功能层 — MCP Server 执行（`npx`）、社区 npm 包、AI Bash 环境中的 `node`/`npx`/`npm` | `src-tauri/resources/nodejs/`（v0.1.44+，含 node + npm/npx） |
| **Git** | SDK 依赖 — Claude Code 需要 `git`（代码操作）+ `bash`（工具执行），Windows 无自带 → NSIS 静默安装 Git for Windows | `src-tauri/nsis/Git-Installer.exe`（仅 Windows） |
| **cuse** (v0.1.67+) | 预置 Computer-Use MCP 原生二进制（截图/点击/输入/滚动，仅 macOS/Windows） | `src-tauri/binaries/cuse-*-<triple>[.exe]`，构建时 `scripts/download_cuse.sh`/`.ps1` 从 `hAcKlyc/MyAgents-Cuse` GitHub Release 拉取 latest |

**分层原则**：Bun 跑我们自己的代码（启动快、行为可控），Node.js 跑社区生态代码（MCP Server / npm 包，设计目标是 Node.js，Bun 兼容性存在系统性缺陷）。原生二进制 MCP（cuse 等）作为第三类，用于性能或 OS API 敏感场景，通过 `PRESET_MCP_SERVERS` 的 `command: '__bundled_xxx__'` 哨兵注册、`platforms?:` 字段做平台门控、`build_macos.sh` 的 `src-tauri/binaries/*-apple-darwin` 通配符自动继承应用签名与 TCC 权限。

**PATH 注入**：`buildClaudeSessionEnv()` 构造 SDK 子进程的 PATH，决定 AI Bash 工具能找到哪些命令。优先级：`bundledBunDir` → `systemNodeDirs`（用户安装的 Node.js） → `bundledNodeDir` → `~/.myagents/bin` → 系统路径。Node.js 系统优先（用户维护、npm 更可靠），Bun 内置优先（跑我们自己的代码）。

**运行时发现**：`src/server/utils/runtime.ts` 提供 `getBundledRuntimePath()`（Bun）、`getBundledNodePath()`（Node.js）。

详见：@specs/prd/prd_0.1.44_dual_runtime.md

### 持久 Session 架构

- `messageGenerator()` 使用 `while(true)` 持续 yield，SDK subprocess 全程存活
- 所有中止场景 MUST 使用 `abortPersistentSession()`（设置 abort 标志 + 唤醒 generator Promise 门控 + interrupt subprocess），禁止直接设置 `shouldAbortSession = true`（generator 会永久阻塞）
- 配置变更时 MUST 先设 `resumeSessionId` 再 abort，否则 AI 会"失忆"
- `abortPersistentSession()` 的调用场景：`setMcpServers`、`setAgents`、`resetSession`、`switchToSession`、`enqueueUserMessage` provider change、`rewindSession`

### Pre-warm 机制

- MCP/Agents 同步触发 `schedulePreWarm()`（500ms 防抖），Model 同步不触发
- 持久 Session 中 pre-warm 就是最终 session，用户消息通过 `wakeGenerator()` 注入。任何 `!preWarm` 条件守卫都可能导致逻辑在持久模式下永远不执行
- 新增配置同步端点时，确保 `currentXxx` 变量在 pre-warm 前已设置
- **MCP 配置权威来源分离**：Tab 会话的 MCP 由前端 `/api/mcp/set` 配置（`initializeAgent` 中 MUST NOT self-resolve MCP），IM/Cron 会话的 MCP 由 self-resolve 从磁盘读取。混用会导致 fingerprint 差异 → abort → 30s 重启循环

### Multi-Agent Runtime (v0.1.60)

除 builtin（Claude Agent SDK）外，支持 Claude Code CLI（NDJSON/stdio）和 Codex CLI（JSON-RPC 2.0/stdio）作为外部 Runtime。功能门控：`config.multiAgentRuntime`（默认关闭）。

- **抽象层**：`src/server/runtimes/` — `AgentRuntime` 接口 + `UnifiedEvent` 统一事件 + `external-session.ts` 会话管理
- **门控链路**：Rust `sidecar.rs` 注入 `MYAGENTS_RUNTIME` 环境变量 → Bun `factory.ts` 读取 → `shouldUseExternalRuntime()` 分流。前端 `Chat.tsx` 的 `currentRuntime` 在定义处门控（`multiAgentRuntimeEnabled ? agent.runtime : 'builtin'`），下游派生自动安全
- **Config 变更**：Model/Permission 变更 → `setExternalModel()`/`setExternalPermissionMode()` → 停进程 → 下次消息以新配置 resume
- **跨 Runtime Session 保护**：`initializeAgent` 检测 `meta.runtime !== 'builtin'` → 跳过 SDK resume → 前端弹确认框引导新开会话
- **`schedulePreWarm()` 已内置 external runtime 跳过守卫**（agent-session.ts:1145），外部 runtime 不走 SDK pre-warm
- 新增 config 同步端点时，MUST 检查 `shouldUseExternalRuntime()` 并分流到 `external-session.ts` 对应函数
- 详见 @specs/tech_docs/multi_agent_runtime.md

### 定时任务系统

Rust `CronTaskManager` 统一管理所有定时任务（Chat 定时、独立创建、AI 工具调用、IM Cron、Heartbeat），支持三种调度：固定间隔 / Cron 表达式 / 一次性。Cron Tool（`im-cron` MCP server）已泛化为**所有 Session 可用**（不仅 IM Bot），始终信任。新增 `CronTask` 字段 MUST 带 `#[serde(default)]`。详见 @specs/ARCHITECTURE.md 的「定时任务系统」节。

### Config 持久化（disk-first）

`AppConfig` 同时存在于磁盘（config.json）和 React 状态中，两者可能不同步。`useConfig` 已重构为 `ConfigDataContext` + `ConfigActionsContext` 双 Context 分离。写入配置时 MUST 以磁盘为准（`await loadAppConfig()` 读最新再合并），禁止直接使用 React `config` 状态写盘。

Agent 配置通过 Rust 命令 `cmd_update_agent_config` 写盘，写盘后 MUST 调用 `refreshConfig()` 同步 React 状态。

### 内嵌终端（Embedded Terminal）

Chat 分屏右侧面板的交互式 PTY 终端。Rust `terminal.rs`（`portable-pty`）+ 前端 `TerminalPanel.tsx`（xterm.js），通过 Tauri IPC 通信（不走 SSE/Sidecar）。终端绑定 Tab 生命周期，面板关闭不杀进程。PTY 进程由 `portable-pty` 管理，**不走** `process_cmd`；Proxy 手动复用 `proxy_config` 常量。详见 @specs/ARCHITECTURE.md 的「内嵌终端」节。

### 内嵌浏览器（Embedded Browser）

Chat 分屏右侧面板的 URL 预览器（Tauri Multi-Webview）。Rust `browser.rs` + 前端 `BrowserPanel.tsx`，通过 Tauri IPC 通信。AI Markdown 链接和 HTML 文件优先在此打开。

- **依赖 Tauri `"unstable"` feature**（`Window::add_child()` 多 Webview API），Cargo.toml 已开启
- **安全隔离**：`browser.json` Capability 零权限，Webview 无法访问 Tauri IPC；`on_navigation` 限制 http/https scheme
- **Overlay 协调**：原生 Webview 浮于 React DOM 之上，Overlay 出现时通过 `closeLayer.hasOverlayLayer()` 自动 hide
- **Cookie 持久化**：同 App 所有 Webview 共享，默认持久化磁盘，关闭即销毁（不后台保活）

详见 @specs/ARCHITECTURE.md 的「内嵌浏览器」节。

### 层级关闭系统（Close Layer）

Cmd+W 层级关闭：Overlay → 分屏面板 → Tab。`closeLayer.ts` 模块级注册表，`useCloseLayer(handler, zIndex)` 一行集成。新增 Overlay 组件 MUST 调用 `useCloseLayer`，z-index 与 CSS 保持一致。详见 @specs/ARCHITECTURE.md 的「层级关闭系统」节。

### 全文搜索引擎

Rust `SearchEngine` 单例（Tantivy + jieba），提供 Session 历史与工作区文件全文检索，仅 Tauri 可用。修改搜索相关代码前 MUST 读 @specs/tech_docs/search_architecture.md。

### Plugin Bridge（OpenClaw 插件）

- Bridge 是独立 Bun 进程，MUST 与 Sidecar 保持同等待遇：环境变量注入（`proxy_config`、`NO_PROXY`）、日志宏（`ulog_*` 不是 `log::*`）、config 查询范围（`imBotConfigs` + `agents[].channels[]`）
- Bun 对 Node.js `http` 模块兼容性不完整，使用 axios 的 npm 包可能静默挂起。新接入插件 MUST 验证其 HTTP 调用在 Bun 下正常（不能只验证 import 成功）
- 兼容层验证 MUST 跑完整消息收发链路（不能只验证 `register()` 成功）
- **SDK Shim 全量覆盖**：shim 覆盖 OpenClaw 全部 `plugin-sdk/*` 导出（手写 + 自动生成 stub）。手写模块受 `_handwritten.json` 清单保护
- **Shim 修改 MUST bump 版本**：源码 shim 经 build → app bundle → Bridge 启动时 `install_sdk_shim()` 拷贝到插件 `node_modules/openclaw/`。**版本不变则跳过拷贝**。三处同步 bump：`sdk-shim/package.json` version、`compat-runtime.ts` SHIM_COMPAT_VERSION、`bridge.rs` SHIM_COMPAT_VERSION
- 详细架构：@specs/tech_docs/plugin_bridge_architecture.md

### OpenClaw 插件通用性原则

MyAgents 是 OpenClaw 的**通用 Plugin 适配层**，不是各家 IM 的硬编码集成。开发准则：

- **协议优先**：所有功能 MUST 基于 OpenClaw SDK 协议（`ChannelPlugin` 接口），禁止为单个插件硬编码逻辑。能力检测用 duck-typing（`plugin.gateway?.loginWithQrStart` 存在 → 支持 QR 登录），不用 if/else 分平台。
- **SDK shim 对齐源码**：新增 shim 函数 MUST 先读 OpenClaw 源码确认签名和行为（`/Users/zhihu/Documents/project/openclaw/`），禁止臆造实现。
- **预设 = 最小定制**：`promotedPlugins.ts` 只声明元数据（npmSpec、icon、authType），功能逻辑走通用路径。预设插件与自定义插件的代码路径 MUST 相同。
- **安装输入清洗**：用户可能粘贴 `npx -y @scope/pkg install` 等完整命令，`sanitize_npm_spec()` 统一剥离，安装/查找/manifest 全链路 MUST 用清洗后的值。
- **鉴权方式自适应**：config 填写 vs QR 扫码由插件能力决定（`supportsQrLogin`），向导流程自动切换，不绑死某种鉴权方式。

---

## 补充禁止事项

> 核心架构约束（Rust 代理层、local_http、process_cmd、Tab 隔离、持久 Session、Config disk-first 等）已在上方各节以 MUST/禁止 形式给出，此处不重复。以下为上方未覆盖的补充规则。

| 禁止 | 后果 | 正确做法 |
|------|------|----------|
| 依赖用户系统安装的运行时 | 用户未安装 → 功能不可用 | 使用内置 Bun 或内置 Node.js（`runtime.ts`） |
| 新增 SSE 事件不注册白名单 | 前端静默丢弃该事件 | 在 `SseConnection.ts` 的 `JSON_EVENTS` 注册 |
| Sidecar 用 `__dirname` / `readFileSync` | bun build 硬编码路径，生产环境出错 | 内联常量或 `getScriptDir()` |
| 日志日期用 UTC `toISOString` | 与本地日期文件名不匹配 | 统一用 `localDate()`（`src/shared/logTime.ts`） |
| Rust 日志用 `log::info!` | 不进统一日志 | MUST 用 `ulog_info!` / `ulog_error!` |
| 裸 `which::which()` 查找系统工具 | Finder 启动时 PATH 缺少 homebrew 等路径 | `crate::system_binary::find()` |
| 前端 `@tauri-apps/plugin-fs` 读写工作区文件 | Tauri fs scope 仅覆盖 `~/.myagents/**` | `invoke('cmd_read_workspace_file')` / `invoke('cmd_write_workspace_file')` |
| UI 硬编码颜色（`#fff`、`bg-blue-500`） | 破坏设计系统一致性 | 使用 CSS Token `var(--xxx)`，参考 specs/DESIGN.md |
| 表单用原生 `<select>` | 系统下拉框样式各平台不一致 | 使用 `<CustomSelect>` 组件（`@/components/CustomSelect`） |
| 函数参数用 `undefined`/`null` 表示特定业务动作 | 内部调用方无意触发该动作 | 业务动作用自解释字面量（如 `'subscription'`），`undefined` 只表示"未提供 / 保持现状" |
| 新增手写 shim 不加入 `_handwritten.json` | `generate:sdk-shims` 下次运行覆盖手写文件 | 手写 shim MUST 同步加入 `sdk-shim/plugin-sdk/_handwritten.json` |
| 新增 overlay/可关闭面板不调用 `useCloseLayer` | Cmd+W 跳过该面板直接关 Tab | 在组件内调用 `useCloseLayer(() => { onClose(); return true; }, zIndex)`，zIndex 取组件 CSS z-index 值 |
| Overlay 遮罩层用裸 `<div>` + 手写 `onClick`/`onMouseDown` | 选中文字拖拽到面板外松手会误关闭 | 使用 `<OverlayBackdrop>` 组件（`@/components/OverlayBackdrop`），内置 `onMouseDown` + target guard |

---

## 日志与排查

日志来自三层（React/Bun Sidecar/Rust），汇入统一日志 `~/.myagents/logs/unified-{YYYY-MM-DD}.log`。用户报告问题时 MUST 主动读取日志，不等用户粘贴。

- **IM Bot 问题**：搜 `[feishu]` `[im]` `[telegram]` `[dingtalk]` `[bridge]` `[openclaw]`
- **AI/Agent 异常**：搜 `[agent]` `pre-warm` `timeout`
- **定时任务问题**：搜 `[CronTask]`（初始化/恢复/执行日志已切换到统一日志 `ulog_*`）
- **终端问题**：搜 `[terminal]`（PTY 创建/关闭/Shell 退出/自清理）
- **Rust 层问题**：额外查系统日志 `/Users/{user}/Library/Logs/com.myagents.app/MyAgents.log`

详细日志架构：@specs/tech_docs/unified_logging.md

---

## Git 与工作流

- **提交前 MUST**：`npm run typecheck`，检查当前分支（`git branch --show-current`）
- **分支策略**：`dev/x.x.x` 开发 → 合并到 `main`。MUST NOT 在 main 直接提交
- **合并到 main**：需 typecheck + lint 通过 + 用户明确确认
- **Commit 格式**：Conventional Commits（`feat:` / `fix:` / `refactor:`）
- **发布流程**：先更新 CHANGELOG.md → `npm version` → `./build_macos.sh` → `./publish_release.sh` → push tag

---

## 深度文档

修改相关模块前建议先阅读：

- 整体架构（全局核心规范，必读）：@specs/ARCHITECTURE.md
- 设计系统（全局核心规范，前端必读）：@specs/DESIGN.md
- 自配置 CLI（myagents 命令、Admin API、版本门控）：@specs/tech_docs/cli_architecture.md
- React 稳定性规范（Context/useEffect/memo 等 5 条规则）：@specs/tech_docs/react_stability_rules.md
- IM Bot 集成：@specs/tech_docs/im_integration_architecture.md
- Plugin Bridge（OpenClaw 插件加载、SDK shim、消息流转）：@specs/tech_docs/plugin_bridge_architecture.md
- Session ID 架构：@specs/tech_docs/session_id_architecture.md
- 代理配置：@specs/tech_docs/proxy_config.md
- Windows 平台适配：@specs/tech_docs/windows_platform_guide.md
- Multi-Agent Runtime（CC/Codex 协议、会话管理、门控链路）：@specs/tech_docs/multi_agent_runtime.md

---
> Source: [hAcKlyc/MyAgents](https://github.com/hAcKlyc/MyAgents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
