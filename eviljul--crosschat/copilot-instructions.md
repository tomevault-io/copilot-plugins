## crosschat

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

<!-- CODEGRAPH_START -->
## CodeGraph

This repository is indexed by CodeGraph (`.codegraph/` exists at the repo root). **Reach for it BEFORE grep/find or reading files when you need to understand or locate code:**

- **MCP tool** (when available): `codegraph_explore` answers most code questions in one call — the relevant symbols' verbatim source plus the call paths between them, including dynamic-dispatch hops grep can't follow. Name a file or symbol in the query to read its current line-numbered source. If it's listed but deferred, load it by name via tool search.
- **Shell** (always works): `codegraph explore "<symbol names or question>"` prints the same output.

If there is no `.codegraph/` directory, skip CodeGraph entirely — indexing is the user's decision.
<!-- CODEGRAPH_END -->

## 项目一句话

CrossChat 是基于 **Tauri 2 (Rust) + React 19 + TypeScript** 的跨 LLM 桌面端 AI 助手。**已完成向单一「六边形架构」的重写**：旧的双栈共存（Tauri command 全家桶 / 内嵌 HTTP 服务）**已从编译链移除**，当前只有一条激活路径。**该路径已不再是薄壳**：核心链路已接通 **OpenAI / Anthropic / DeepSeek** 三家适配器的**流式输出（SSE）+ 只读工具 ReAct 循环**，API Key 已迁至系统钥匙串（keyring）。六边形其余部分（MCP / 事件总线 / 写类工具审批）仍是「已搭好但未接线」的脚手架。

> ⚠️ **版本号与文档超前警告**：`Cargo.toml` / `package.json` / `tauri.conf.json` 均为 `0.2.3`，但 `CHANGELOG.md`（标 v0.3.0）、`QUICKSTART.md`、`docs/REFACTOR_*.md` 描述的 **Axum HTTP Server / SSE / 21434 端口 / sqlx / 10-50x 流式**，在当前代码里**并不存在**（`Cargo.toml` 无 `axum`、无 `sqlx`，`lib.rs` 无 `http_server` 模块）。**以代码为准，别信这些文档**。

## 开发命令

环境要求：Node.js 22+、Rust（`dtolnay/rust-toolchain@stable`，未锁版本）、Python 3.11（仅构建期打包 office 依赖用）。

```bash
# 启动开发（前端固定端口 5174 + Tauri 主进程）
npm install
npm run tauri dev

# 生产构建（产物在 src-tauri/target/release/bundle/）
npm run tauri build

# 仅前端
npm run dev
npm run build           # tsc 类型检查 + vite build
npm run preview

# Rust 检查
cd src-tauri
cargo check
cargo build

# 重新生成 Tauri 图标
npm run icon design/icon-1024.png   # 或 npx @tauri-apps/cli icon design/icon-1024.png
```

**没有前端测试框架、没有 lint 脚本**。改动 TS 至少跑 `npm run build`（含 `tsc`）；改动 Rust 跑 `cargo check`。
注意：仓库里 `*_tests.rs`（`agent/`、`mcp/`、`memory/`、`metrics/`、`skills/`）**大多位于已停用的死代码模块**，不参与编译，`cargo test` 跑不到它们。

## 架构现状（最关键认知）

**只有一条激活路径**，现已接通流式与只读工具调用：

```
CanvasView.tsx ─invoke("send_chat_message", Channel<StreamEvent>)─▶ commands/chat_cmd.rs
   │
   ├─ make_model_client ── 按 providerType 路由：
   │     openai-compat → OpenAIClient（SSE 真流式）
   │     anthropic     → AnthropicClient
   │     deepseek      → DeepSeekClient
   │     └─ ReAct 循环（MAX_TOOL_ITERATIONS）：流式生成 → 若请求工具则执行 → 回填 → 再生成
   ├─ LocalToolHost（adapters/tool）── 只读工具白名单：read_file / list_directory
   └─ SqliteThreadStore（rusqlite）── ~/.../.crosschat/threads.db（items 累积落单个 Turn）
```

- **发消息**：`chat_cmd::send_chat_message` 按 `providerType` 选择 ModelClient → **流式**生成（`tauri::ipc::Channel<StreamEvent>` 逐块回推前端）→ 进入 **ReAct 循环**（上限 `MAX_TOOL_ITERATIONS`）：模型若请求工具，就经 `LocalToolHost` 执行**只读工具**并把结果回填 messages，再次流式生成，直到某轮无工具调用为止 → 全过程 items（AssistantText / ToolCall / ToolResult…）累积落库到单个 Turn。
- **当前激活链已包含**：多 Provider 路由（OpenAI / Anthropic / DeepSeek）、真流式（SSE 逐块）、只读工具调用（`read_file` / `list_directory`）、ReAct 多轮循环、`<think>` 标签实时折叠、AI 回复 markdown 渲染。
- **当前激活链仍没有**：MCP、写类工具与审批门、上下文压缩、skill、向量记忆。
- **`lib.rs` 装配 8 个模块**：`core / ports / adapters / application / migration / commands / error / python_env`。
- **`invoke_handler` 注册 22 个 command**：原 15 个（含 `migrate_data`）+ 新增 `set_session_status` `rename_session` `set_session_pinned`（会话归档 / 重命名 / 置顶）、`set_api_key` `get_api_key` `delete_api_key`（系统钥匙串）、`ocr_image`（本地 OCR）。

## 后端地图（`src-tauri/src/`）

### 激活代码（真正编译、真正跑）

```
core/models/       # 纯数据：thread.rs / turn.rs / tool.rs / message.rs（无 I/O）
                   # Turn 用 #[serde(tag="type")] 的 TurnItem 枚举：UserMessage/AssistantText/
                   # AssistantReasoning/ToolCall/ToolResult/Compaction/Approval/Error
ports/             # 5 个 trait：ModelClient / ToolHost / ThreadStore / EventBus / ApprovalGate
                   # ModelClient / ToolHost / ThreadStore 已有激活实现方；EventBus / ApprovalGate 仍无
adapters/store/    # sqlite_store.rs = SqliteThreadStore（rusqlite）
                   # 三张表：threads / turns(data 存 Turn 的 JSON) / todos
adapters/model/    # openai_client / anthropic_client / deepseek_client —— 均已由 chat_cmd 接线激活
adapters/tool/     # local_tool_host.rs = LocalToolHost（只读工具 read_file / list_directory）——已激活
                   # mcp_persistent / sandbox（未被 chat_cmd 调用）
commands/          # mod.rs 声明 chat_cmd / file_ops / keychain_cmd / ocr_cmd / session_cmd
                   #   chat_cmd.rs     → send_chat_message（中枢：流式 + ReAct + 只读工具）、fetch_models
                   #   session_cmd.rs  → create/list/get/save/delete + 归档/重命名/置顶（读写 SqliteThreadStore）
                   #   file_ops.rs     → 目录/文件读写删（已加敏感路径黑名单沙箱，见「安全」）
                   #   keychain_cmd.rs → set/get/delete_api_key（keyring，API Key 存系统钥匙串）
                   #   ocr_cmd.rs      → ocr_image（调嵌入式 Python 跑 resources/ocr.py 做本地 OCR）
python_env.rs      # 嵌入式 Python 运行器（get_python_executable / is_python_available）——被 ocr_cmd 激活
migration.rs       # 旧 ~/.crosschat/sessions/*.json → 新 threads.db
                   # 首次启动后台 spawn，成功写 .migrated 标记，原文件备份到 sessions_backup/
lib.rs             # setup：建 data_dir、初始化 SqliteThreadStore、spawn 迁移；注册 22 command
```

### 未接线脚手架（编译，但激活链没调用）

六边形其余部分**已实现但没接进 command 层**，改这些**不会影响运行行为**：

```
adapters/model/    # anthropic_client / deepseek_client 已接线；openai_client 已激活
adapters/tool/     # mcp_persistent / sandbox（未被 chat_cmd 调用；local_tool_host 已激活）
adapters/event/    # memory_bus（tokio::broadcast，未被调用）——EventBus trait 仍无激活实现方
application/        # agent_loop.rs（另一套 ReAct，未被 command 调用；chat_cmd 自带循环，没走它）
```

> ⚠️ **ReAct 有两套**：真正跑的是 `chat_cmd.rs` **内联**的循环；`application/agent_loop.rs` 是独立实现、**未接线**。改「发消息 / 工具」行为改前者，别动 agent_loop。

### 磁盘死代码（**不在 `lib.rs` 编译链**，别在里面改 bug）

以下目录/文件**仍在磁盘上但没有 `mod` 声明**，grep 会撞到它们，但改了**完全无效**：

```
agent/  providers/  tools/  mcp/  streaming/  skills/  memory/  metrics/  security/
http_server/
commands/ 下：agent_cmd / chat.rs / checkpoint_cmd / mcp_cmd / mcp_health_cmd /
              memory_cmd / metrics_cmd / provider_cmd / python_cmd / skills_cmd / stream_cmd
```

**要改后端行为，先确认目标文件在「激活代码」里；若功能只存在于死代码/脚手架，说明它当前根本没通电，需要先接线（改 `commands/mod.rs` + `lib.rs`）——这属于架构决策，应先与用户确认。**

## 前端地图（`src/`）

```
App.tsx                       # 只渲染 <CanvasView/> + <WelcomeDialog/>（无新旧 UI 切换）
main.tsx                      # 挂载 <ErrorBoundary><App/>；主题切换（dark/light/system）写 localStorage
components/
  canvas/CanvasView.tsx       # 唯一主界面：左侧会话列表 + 右侧消息 + 输入框 + 设置入口
                              # 内含 ThinkingBlock / ToolCallBlock；解析 <think> 标签、合并推理消息
  WelcomeDialog.tsx           # 新用户引导
  ErrorBoundary.tsx           # 顶层错误边界
  settings/                   # SettingsDialog / ProviderTab / McpSection / GeneralTab / FeedbackDialog
lib/
  tauri-bridge.ts             # 有效 invoke：file/session/chat/fetchModels 等
  officeParser.ts             # 纯前端解析 office 预览（xlsx/mammoth/pptxgenjs/docx，不走后端 Python）
  cn.ts
stores/                       # Zustand（persist 到 localStorage）：
                              #   settingsStore(crosschat-settings) — ProviderType 单一来源
                              #   providerStore(crosschat-providers) — PRESET_PROVIDERS 预设来源
                              #   workspaceStore
shared/ui/                    # Button/Input/Select/Textarea/Avatar
styles/globals.css            # Tailwind 4 入口，紫蓝渐变主题，ds-* 语义色
```

> `components/chat/*`（旧 ChatView 全家桶 13 个组件）、`hooks/*`（useChat/useAgent/useCheckpoint/useContextUsage）、`stores/chatStore.ts`、`lib/http-client.ts` **均已删除**。若旧文档或代码引用它们，视为过时。

## 数据存储（`~/.crosschat` 系）

- **统一走 `dirs::data_dir().join(".crosschat")`** → `threads.db`（rusqlite）。
  - macOS: `~/Library/Application Support/.crosschat/`
  - Linux: `~/.local/share/.crosschat/`
  - Windows: `%APPDATA%\.crosschat\`
- **迁移的跨平台坑**：`migration.rs` **读旧数据**用的是 `dirs::home_dir().join(".crosschat/sessions")`（即 `~/.crosschat/sessions`），**写新库**用 `data_dir()`。在 macOS/Linux 上这**不是同一个目录**——旧数据在 `~/.crosschat`，新库在 data_dir。调试迁移时注意这个不对称。
- 改 schema 必须同步 `migration.rs` 与 `SqliteThreadStore::new` 里的建表 SQL，否则老用户 `threads.db` 读不到。

## 安全 / 配置点（P0 已加固）

- `tauri.conf.json`：**CSP 已启用**（`default-src 'self'`；另有放宽 `unsafe-eval` 的 `devCsp` 供开发热更新）、**`withGlobalTauri: false`**、**`assetProtocol.scope: ["$RESOURCE/**"]`**（已从 `["**"]` 收窄）。P0 三项已修（见 `feat: P0 安全加固` 提交）。
- **`file_ops.rs` 已加敏感路径黑名单沙箱**：`is_path_forbidden` + `blacklist_dirs` 拦截 `~/.ssh` `~/.aws` `~/.gnupg` `~/.config/gcloud` `~/.kube` `~/.docker` `~/.netrc` `~/.crosschat` 及 `/etc` `/root` `/var/root`，并对目标与黑名单双向 `canonicalize` 防软链绕过。新增文件类 command 仍须自己接入该校验。
- **API Key 已可存系统钥匙串**：后端 `keychain_cmd.rs`（`keyring` v3，service=`com.tian.crosschat`，account=provider_id）提供 `set/get/delete_api_key`。⚠️ 前端是否已完全弃用 localStorage 明文存储，以 `settingsStore` / `ProviderTab` 当前代码为准。
- `capabilities/default.json`：Tauri 权限清单。**新增 command 若需新权限，Rust 端 `invoke_handler!` + 此文件两处都要加**。当前含 `shell:allow-execute`、`dialog`、`global-shortcut`、`clipboard-manager` 等。

## Python 打包（构建期，与运行时解耦）

`tauri.conf.json` 把 `resources/python` 打进包（`setup_python.py` 下载 embed Python + `install_office_deps.py` 装 openpyxl / python-docx / pptx / PyPDF2 + **`rapidocr_onnxruntime`**）。**`python_env.rs` 已重新激活**：`ocr_cmd::ocr_image` 通过嵌入式 Python 运行 `resources/ocr.py`（RapidOCR / ONNX）做**本地 OCR**——这是运行时**重新消费**内嵌 Python 的唯一路径。注意：office 预览仍走前端 `officeParser.ts` 纯 JS、不碰内嵌 Python；`tools/python_sandbox.rs` 仍是死代码。

CI 缓存 key（改 Python 依赖时 bump）：Windows `python-embed-windows-3.11.9-ocr`、Linux `python-embed-linux-3.11.7-ocr`（加 OCR 依赖时已 bump 过 `-ocr` 后缀）。改 setup 脚本注意 Windows 编码问题（见 `fix: resolve Windows CI encoding issue` 提交）。

## CI

`.github/workflows/build.yml`：push 到 `master` 或 tag `v*` / `Beta*` 触发，构建 Windows (msi/nsis) + Linux (deb/rpm)。**macOS 未配置**（无签名包）。

## 改代码前必读

- **先分清三层**：激活代码 / 未接线脚手架 / 磁盘死代码（见「后端地图」）。改错层 = 白改。
- 改「发消息 / 流式 / 工具调用」逻辑 → `commands/chat_cmd.rs`（内联的流式 + ReAct 循环；**不是**死代码 `commands/chat.rs`、**也不是**脚手架 `application/agent_loop.rs`）。
- 改「只读工具」集合 → `adapters/tool/local_tool_host.rs` + `chat_cmd.rs` 里的 `ALLOWED_TOOLS` 白名单。
- 改「会话增删改查 / 归档置顶」→ `commands/session_cmd.rs` + `adapters/store/sqlite_store.rs`。
- 改「本地 OCR」→ `commands/ocr_cmd.rs`（后端）+ `resources/ocr.py`（RapidOCR）+ `CanvasView.tsx`（选图 / 粘贴 / 拖拽）；依赖在 `install_office_deps.py`，改依赖记得 bump CI 缓存 key。
- 改「API Key 存储」→ 后端 `commands/keychain_cmd.rs`（keyring）+ 前端 provider 设置。
- 改前端界面 → 只有 `components/canvas/CanvasView.tsx` 一个主界面。
- 想恢复 MCP / 写类工具 → 这些代码在脚手架或死代码里，需要**接线 + 架构决策**，先与用户确认，别默默启用。
- 改存储 schema → 同步 `migration.rs` 与建表 SQL。
- 新增 Tauri command → `lib.rs` 的 `invoke_handler!` + `commands/mod.rs` + 必要时 `capabilities/default.json`。
- **不要参照** `CHANGELOG.md` / `QUICKSTART.md` / `docs/REFACTOR_*.md` 里的 HTTP/SSE/端口 21434/sqlx 描述——它们与当前代码不符（`Cargo.toml` 确无 `axum` / `sqlx`）。

---
> Source: [EvilJul/CrossChat](https://github.com/EvilJul/CrossChat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
