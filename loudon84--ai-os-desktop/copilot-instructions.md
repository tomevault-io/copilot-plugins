## ai-os-desktop

> > 面向 Cursor / AI Agent 的项目速查。详细设计见 `docs/INDEX.md`、`docs/ARCHITECTURE.md`、`docs/MODULES.md`。

# Hermes Desktop — Agent 编码指南

> 面向 Cursor / AI Agent 的项目速查。详细设计见 `docs/INDEX.md`、`docs/ARCHITECTURE.md`、`docs/MODULES.md`。

## 项目是什么

**hermes-desktop** 是基于 Electron + React + TypeScript 的 AI 助手桌面应用：安装/配置 Hermes Agent、管理 Gateway、聊天、多 Profile 运行时（Portal Desktop）。

| 项 | 值 |
|---|---|
| 版本 | 0.3.6（… + **v9.0.0 Serve-First Runtime Phase 0/1/2/3** + **v8.2.0 Persistent Chat Workspace + Session Catalog** + **v8.1.1 Durable Runtime Closure** + **v8.1.0 Durable Chat Runtime** + **v8.0.5 Chat Interaction Loop** + **v8.0.4 Chat Turn Lifecycle** + **v8.0.3 Chat Workspace Layout** + **v8.0.2 Chat Full Integration** + **v8.0.1 Chat 迁移闭环** + **v7.6 Hermes Agent MCP Host Mode** + **v7.5.1 Runtime Skill Fixed Route** + **v7.4.2 Chat-first Work Controls** + …） |
| appId | `com.smc.smc-ai-copilot`（productName: **SMC-Copilot**；主程序 **desktop.exe**） |
| 后端 | Hermes Python Gateway，`http://127.0.0.1:8642`（default Profile） |

## 架构：三层进程（必须遵守）

```
Renderer (React)  →  window.hermesAPI / smcShell / desktopAuth / desktopUserConfig / profileRuntime / aiosBrowser / shellView
       ↓ ipcRenderer.invoke
Preload (contextBridge)  →  src/preload/index.ts + auth-api / shell-api / user-config-api
       ↓ IPC
Main (Node.js)  →  src/main/index.ts + 各 *-ipc.ts / *.ts 模块
       ↓ HTTP / child_process
Portal Auth Backend (:8000)  +  Hermes Python Gateway (:8642)
```

**硬性规则：**

- 渲染进程**禁止** `import` Node 模块；只通过 `window.*` API 访问主进程
- **禁止**猜测新增 IPC channel；新通道必须：Main 注册 handler → Preload 封装 → `index.d.ts` 类型 → `docs/API_CONTRACTS.md`
- **禁止**绕过 Preload 使用 `ipcRenderer`（除 Preload 自身）
- 改 Gateway 启停/进程管理前，先读 `src/main/hermes.ts` 与 `profile-runtime-manager.ts`
- 不要改 `electron-builder.yml` 的 appId / publish、i18n 语言代码与回退链
- **禁止**修改含 `#command by loudon` 的注释行与整块注释（软约束，见 `.cursor/rules/008-loudon-command-comments.mdc`）

## 目录地图

| 路径 | 职责 | 常见改动 |
|---|---|---|
| `src/main/` | 主进程：IPC、Gateway、配置、SQLite、Enterprise Install | 新 IPC、后端逻辑 |
| `src/main/copilot-serve/` | **V1.3** Serve 进程探测 / deploy / logs；**v9.0** production 禁止 spawn/stop | Runtime 进程策略 |
| `src/main/copilot-runtime-client/` | **v9.0** Main-only Serve SDK：pairing / keytar Device Token / handshake / proxy-fetch / SSE scaffold | Serve-First 连接层 |
| `src/main/runtime-adapters/` | **v9.0** Phase 2 Instance/Config/MCP/Diagnostics + **Phase 3 ServeChatRuntimeAdapter**（Session/Task/Files 仍 stub） | cutover |
| `src/main/workspace-chat/` | **team_v1.8** Workspaces Chat：resolve / 模型 / 附件 / SSE 代理到 `copilot-serve` | Chat 面板 IPC |
| `src/shared/workspace-chat/` | `workspace-chat-contract.ts` — Renderer/Main/Preload 共享类型 | 改 chat 契约时 |
| `src/main/browser/` | Web Operator（BrowserView、安全、审计、Tool Server） | 浏览器自动化 |
| `src/main/enterprise/` | 企业安装、本地 zip/git 源、**agent-deps-installer**、**pip-mirror-config** | 安装/预检/Doctor/依赖 |
| `src/main/runtime/` | **V5.3** `runtime-paths.ts` + **`portal-root-resolver.ts`** — hermes/serve/portal 统一路径契约 + `buildCopilotRuntimeEnv` + `resolveEffectivePortalMonorepoRoot` | 改安装目录结构 / 运行时路径 / Portal 根解析 |
| `src/main/window/` | V1.4.1 窗口 IPC（`registerWindowIpc`） | 标题栏按钮 |
| `src/main/shell/` | **main-window-controller.ts**（默认 1280×800）、ShellView、菜单、`portal-view-coordinator` | 主窗口尺寸、壳层 IPC |
| `src/main/auth/` | **V3.3** Auth Client、Token Vault、Endpoint Config、Header 注入 | Portal 登录 + Bearer 注入 |
| `src/main/mcp-skill-gateway-runtime/` | **V6.4** nodeskclaw MCP Skill Gateway 本地 Proxy + Hermes `config.yaml` 注册 | MCP Gateway 远程桥接 |
| `src/main/genehub/` | **V6.5** nodeskclaw GeneHub 连接、设备/Profile 注册、Bundle 安装执行器 | 企业 Skill 同步 |
| `src/main/user-config/` | **V3.3** 本地 bootstrap apply / diff / applier | 登录后桌面配置落盘 |
| `src/main/startup/` | **V3.3.1** `startup-decision.ts`、`startup-ipc.ts` | 启动门控（auth + bootstrap 前置） |
| `src/preload/` | contextBridge：`hermesAPI`、`smcShell`、`desktopAuth`、`desktopUserConfig`、`aiosBrowser`、`profileRuntime`、`profileEntry`、`shellView` | API 桥接（影响面大） |
| `src/renderer/src/screens/MainPage/` | **V2.0** 主界面壳：`MainPage`、`MainTopBar`、`MainViewTabs` 等 | 顶栏 / 工作区 Tabs |
| `src/renderer/src/components/overlay/` | **V5.7.8** `OverlayProvider` / `DialogLayer` / `DrawerLayer` / `NativeShellLayerGate` | 全局弹层 + native layer gate |
| `src/renderer/src/` | React UI：`screens/`、`components/`、`modules/auth/` | **主要 UI 开发区** |
| `src/shared/shell/` | **`main-page-constants.ts`**、**`browser-partitions.ts`**、shell-view-contract、view-contract | 布局尺寸、Session 分区常量、ShellView 契约 |
| `src/shared/workspace/` | **workspace-contract**、**workspace-secondary-nav** | Workspace 模块元数据、二级 panel 导航 |
| `src/renderer/src/workspace/` | **workspace-registry**、workspace-tabs、resolve-workspace | 顶栏 Tab 与 `WorkspaceRenderer` 路由 |
| `resources/profiles/` | Profile 模板 + SOUL.md | Profile 种子配置 |
| `resources/skills/` | 内置技能包 | 新技能 |
| `tests/` | Vitest | 单测/集成测 |
| `docs/` | 架构、契约、Renderer 专项文档（见「文档体系」） | 随功能更新；见 `.cursor/rules/007-sync-project-docs.mdc` |
| `prd/` | 版本 PRD（V2.x MainPage、V5.x Hermes/WebOperator 等） | 对应里程碑任务 |

## Preload 暴露的全局 API

| 全局对象 | 文件 | 用途 |
|---|---|---|
| `window.hermesAPI` | `src/preload/index.ts` | 安装、配置、聊天、会话、模型、技能等；**含 `windowControls`、`getInstallerPrecheck`、`.mcp`（V6.1 MCP Registry）** |
| `window.aiosBrowser` | `src/preload/browser-api.ts` | Web Operator（legacy 13 方法 + **V5.7** frame/snapshot/结构化动作 API + 事件） |
| `window.profileRuntime` | `src/preload/profile-runtime-api.ts` | 多 Profile 运行时（20 方法，含日志/自动重启） |
| `window.profileRole` | `src/preload/profile-role-api.ts` | **V4.0** 专家角色预设安装、角色库同步、role spec 查询/重编译 |
| `window.profileEntry` | `src/preload/profile-entry-api.ts` | Profile 页面入口与布局（5 方法） |
| `window.mainPageState` | `src/preload/main-page-state-api.ts` | MainPage 壳状态 V2（workspaceOrder / externalTabs / sidebarMode / workspaceSecondaryState） |
| `window.smcShell` | `src/preload/shell-api.ts` | **启动门控** `resolveStartupDecision()`；窗口控制 / openExternal |
| `window.desktopAuth` | `src/preload/auth-api.ts` | V3.3 Portal Auth 登录（**不向 Renderer 暴露 token**） |
| `window.desktopUserConfig` | `src/preload/user-config-api.ts` | V3.3 本地 bootstrap apply / diff / bootstrap-state |
| `window.copilotServe` | `src/preload/copilot-serve-api.ts` | **V1.3** 本地 `copilot-serve` 生命周期（get-connection **不含 token** / start / stop / logs）；**不含**任务业务 API |
| `window.copilotRuntime` | `src/preload/copilot-runtime-api.ts` | **v9.0** Serve-First：connection/pairing/repair/diagnostics/**proxyFetch**/Instance 启停与日志；**不向 Renderer 暴露 Device Token** |
| `window.workspaceChat` | `src/preload/workspace-chat-api.ts` | **team_v1.8** Workspaces Chat（resolve / 模型 / 附件 / send SSE）；见 `docs/API_CONTRACTS.md` § Workspace Chat |
| `window.chatRuntime` | `src/preload/chat-runtime-api.ts` | **v8.1.1** `start`/`abort`/`command`/`getState`/`getSnapshot`/`replayEvents`/`recover`/`exportDiagnostics`/`saveDiagnostics` + `queue.*` + ordered event；Continuation `completion`；profile store；`submit` 兼容（PR7 再删） |
| `window.chatWorkspace` | `src/preload/chat-workspace-api.ts` | **v8.2** Chat Tab/Draft 持久化：`getSnapshot`/`open`/`openSession`/`patchRun`/`closeRun`/`setActive`/`reorder`/`migrateV1` + `onChanged`；Main `chat-workspace.db` |
| `window.sessionCatalog` | `src/preload/session-catalog-api.ts` | **v8.2** Profile-aware Session 目录：`list`/`rename`/`archive`/`delete`/`pin` + `onChanged`；直读 profile `state.db`（非 `sessions.json`） |
| `window.chatFiles` | `src/preload/chat-files-api.ts` | **v8.0.5** Chat Files + File Platform + `onChanged`（`chat-files:changed` 驱动 Session Files Badge；不扩展 `hermesAPI.files`） |
| `window.webOperatorTaskSession` | `src/preload/web-operator-task-session-api.ts` | **V5.7.5** WebOperator Page→Hermes 任务会话（`task_session` SQLite） |
| `window.aiosRuntime` | `src/preload/aios-api.ts` | Portal Runtime 启停/Doctor/日志；**V5.3.4** `getPortalInfo()` 展示 monorepo 安装路径 |
| `window.mcpSkillGatewayRuntime` | `src/preload/mcp-skill-gateway-runtime-api.ts` | **V6.4** MCP Skill Gateway Runtime（Proxy 启停、Hermes 注册、远程 MCP 健康检查）；**V6.6.1** `readStructuredLogs` / 运营诊断；**V7.0** Hermes Client bootstrap/agents/tools/readiness/task result/artifact（`hermes-client:*` IPC）；**不向 Renderer 暴露 token** |
| `window.genehubRuntime` | `src/preload/genehub-runtime-api.ts` | **V6.5** GeneHub 连接探测、授权 Skill 列表、安装任务执行与日志；**不向 Renderer 暴露 token** |
| `window.hermesExperts` | `src/preload/hermes-experts-api.ts` | **Expert MCP v6.1** 直连 `/api/v1/expert/mcp`：Workbench / Experts 广场等；**v7.6 Chat 业务禁止** `callCatalogSkill` |
| `window.hermesMcpConfig` | `src/preload/hermes-mcp-config-api.ts` | **v7.6** Hermes Agent `mcp_servers` 配置（读写 `config.yaml` + `.env` token 掩码）；**不向 Renderer 暴露 token** |
| `window.nodeskclawRuntimeSkillAPI` | `src/preload/nodeskclaw-runtime-skill-api.ts` | **v7.5.1** legacy Runtime Skill 路由；**v7.6 Chat 业务禁止** |
| `window.work` | `src/preload/work-api.ts` | **v7.4.1 Work 任务 Hotfix** `task.start` / `resume` / `list` / `getBySession`；首条消息走 `hermesDefaultChat`；元数据 `work-tasks.json`；legacy `send`/`onEvent` 保留 |

类型定义：`src/preload/index.d.ts`。契约类型：`src/shared/profile-runtime/`、`src/shared/enterprise/`、**`src/shared/mcp/`（V6.1）**、**`src/shared/mcp-skill-gateway-runtime/`（V6.4）**、**`src/shared/genehub/`（V6.5）**、**`src/shared/hermes-mcp-config/`（v7.6）**、**`src/shared/nodeskclaw/`（v7.5.1）**、**`src/shared/work/`（v1.4）**、**`src/shared/chat-runtime/` + `src/shared/chat-files/`（v8.0）**。

## 应用路由与 UI 结构

**生命周期路由**（`src/renderer/src/App.tsx` + `hooks/useStartupGate.ts`）：

```
splash → login → welcome → installing → setup → main (Layout)
         ↑
    auth + bootstrap 门控（V3.3.1：全连接模式统一）
```

- `useStartupGate` 调用 `window.smcShell.resolveStartupDecision()` → IPC `startup:resolve-decision` → `startup-decision.ts`
- 登录成功 + 本地 bootstrap apply 完成后 `recheck()` 重新解析路由（可能进入 `main` / `setup` / `welcome`，取决于 Hermes 安装与模型配置）

**主界面壳层（V2.0）** — `Layout.tsx` 编排状态与 `WorkspaceOutlet`；`MainPage` 自持壳层 UI：

```
Layout → MainPage（数据 props：secondaryPanel、update、outlet、StatusBar…）
MainPage
  ├─ MainTopBar        ← Profile、Gateway、Tabs、WindowControls；侧栏按钮仅 web-operator/office
  ├─ DesktopSidebar    ← 条件渲染：有全局二级导航时（非 portal/workspaces）
  ├─ WorkspaceOutlet ← 各 Screen
  ├─ StatusBar
  └─ ModalLayer / DrawerLayer
```

PRD：`prd/v2.0_mainpage.md` / `prd/v2.1_mainpage.md`。Legacy：`DesktopShell`、`PageHeader`（非主链路）。

**V2.1**：Sidebar 三态、动态 Profile Tabs、ShellView 全 IPC、`WebContentsHost` web-operator layer。

**V2.2**：`ShellBrowserViewAdapter` 单轨（`browser.open` → SVM）；`browser.opened` 自动切 Web Operator tab；MainTopBar「+」创建 `external-browser:*` tab；Tab DnD（`@dnd-kit`）；顶栏 Reload/Close。

**V2.3**：ShellView metadata 事件链（title/loading/error/crash → MainViewTabs）；导航 IPC（reload/stop/back/forward/recover）；`main-page-state.json` 持久化（`~/.hermes/desktop/`）；`KeepAliveView` 保活 React 管理页；`layout-calc-parser` 移除 `new Function`。

**V3.0**：View 收敛为 `portal` / `workspaces` / `web-operator` / `office` / `external-browser:*`；**V1.3** 增加 `task-workbench`（本地任务三栏，直调 copilot-serve）。Hermes 运维迁入 **TopBar Settings Drawer**（`modules/hermes-runtime/`）；初版 `LoginGate` + mock Auth/Bootstrap（**V3.3 起改为 splash→login 前置 + 真实 Portal HTTP Auth**）；`aios-home` → `http://127.0.0.1:3000`（无 `/zh`）。

**V3.2**：**Workspace Module Host** — … `token-header-injector` 对 `persist:aios-home` 注入 Bearer（V3.3 重命名自 `persist:aios-desktop`）。

**V3.3**：Endpoint Config + Login 合一；Main Token Vault（keytar → safeStorage → 内存）；`persist:aios-home` origin 白名单 Bearer 注入；启动门控 `login` 在 install 之前。

**V3.3.1（Auth Hotfix + 本地 Bootstrap）**：
- **启动门控**：local / remote / ssh 统一 auth + bootstrap；`bootstrap-pending` → LoginScreen 自动 bootstrap
- **Portal Auth HTTP**：`POST {backendUrl}{authPrefix}/login` 请求体 `{ email, password }`（**非** `username`）；refresh 请求体 `{ refresh_token }`；响应兼容 snake_case（`access_token` 等）
- **默认 Endpoint**：backend `http://127.0.0.1:8000`、authPrefix `/api/v1/auth`、aiosHomeUrl `http://127.0.0.1:3000`（登录的是 Portal Auth，不是 Hermes Gateway）
- **Bootstrap 默认本地合成**：登录后 **不请求** `GET /api/v1/desktop/bootstrap`；Main 根据 session + endpoint 合成 `local-v1` 并 apply；Backend 实现远程接口后设 `HERMES_USE_REMOTE_USER_CONFIG=true`
- **Preload/Main 接线**：`window.smcShell` + `setupStartupIPC()`（`startup:resolve-decision`）
- **portal 显示时机**：`portal-view-coordinator` create/reload 后 **deactivate**，仅当主界面 `WebContentsHost.setBounds("portal")` 时才显示嵌入页
- **Bootstrap apply**：同步 `auth-endpoint-config` + `refreshPortalView()`（prepare，非全屏覆盖）
- **Session 三分区（V3.3 当前值）**：`browser-partitions.ts` — `portal` layer → `persist:aios-home`（token）；`web-operator` → `persist:web-operator`；每个 `external-browser:{uuid}` → `persist:external-browser-{uuid}`。
- **Token 端口**：`token-inject-url.ts` 从 `getAiOsEnvConfig().frontendPort/backendPort` 解析，仅 `127.0.0.1|localhost`；不注入 Gateway `8642` 与 web-operator/external 分区。
- **PRD #5 收敛**：`RuntimeGuard` 打开设置 → `openSettingsDrawer("runtime")`；**禁止**独立 `HermesRuntimeSettingsDrawer`；全局 Agent / copilot-serve / 全局 Profile 在 `SettingsDrawer` → `server/ServerPanel`；外观/网络/备份在 `general/GeneralPanel`；顶栏 `ServersEntry`（非 `MainProfileSwitch`）→ `openSettingsDrawer("server")`。
- **架构**：`WorkspaceRenderer` 按 `module.kind` 分发；`MainViewTabs` 固定 Tab 由 registry `!draggable` 判定（仅 `external-browser:*` 可拖）。
- **i18n**：`navigation.*` 在 **en / zh-CN** 两套 locale 补齐二级侧栏 key（`chat`、`sessions`、`openSettings` 等）。

**主界面**（`WorkspaceOutlet` → `WorkspaceRenderer`）视图：

| 视图 | 路径 | 屏幕文件 |
|---|---|---|
| **Portal** | 顶栏 Tab `portal` | `screens/Portal/Index.tsx`（`PortalScreen` + `WebContentsHost` layer `portal`） |
| **Workspaces** | 顶栏 Tab `workspaces` | `screens/Workspaces/` — **三栏 Shell**（`WorkspacesShell`：顶栏 `WorkspaceStatusCards` + 左 `WorkspacesSidebar` + 中 `registry/workspace-pages` + 右 `WorkspaceRightPanel` / Inspector 四 tab） |
| **Local Hermes** | 顶栏 Tab `local-hermes` | `screens/Hermes/` — **V5.6** default profile 三栏（`HermesShell` + `hermesDefaultApi` → `window.hermesAPI`；与 Workspaces/copilot-serve 隔离） |
| Web Operator | 顶栏 / 侧栏二级 | `screens/WebOperator/WebOperatorScreen.tsx`；侧栏 **Hermes Task** 用 `components/hermes/WebOperatorHermesChatPanel`（`hermesDefaultChat`，全局默认模型，`draft_weboperator`） |
| Office | 顶栏 Tab `office` | `screens/Office/Office.tsx`（KeepAlive） |
| External | 顶栏动态 Tab | `external-browser:{uuid}` → `WebViewWorkspace` |
| Hermes Runtime | TopBar Settings Drawer | `screens/SettingsDrawer/` — `server/`（Agent + Copilot Serve + 全局 Profile）、`general/`（外观/网络）、`HermesRuntimePanel`、`multi-profiles/` |

## 核心数据流（改功能时先定位）

### Auth + Bootstrap + 启动门控（V3.3 / V3.3.1）

```
useStartupGate (splash)
  → smcShell.resolveStartupDecision()
  → startup-decision.ts（auth → bootstrap → Hermes 安装/配置）

LoginScreen
  → desktopAuth.saveEndpointConfig + login({ email, password })
  → auth-client POST {backendUrl}{authPrefix}/login
  → token-store（Main only，keytar → safeStorage → 内存）
  → desktopUserConfig.bootstrap()
  → user-config-client 合成 local-v1（默认不 HTTP）
  → user-config-applier apply + refreshPortalView（deactivate）
  → recheck → 可能进入 main / setup / welcome
```

环境变量：`HERMES_USE_MOCK_AUTH`、`HERMES_USE_MOCK_USER_CONFIG`、`HERMES_USE_REMOTE_USER_CONFIG`（见 `docs/API_CONTRACTS.md`）。

### 聊天

```
Chat.tsx
  → hermesAPI.sendMessage / onChatChunk / onChatDone
  → main/hermes.ts: sendMessage()
      → 远程模式: sendMessageViaApi (HTTPS)
      → 本地: GET /health → SSE POST /v1/chat/completions 或 CLI fallback
  → main/sse-parser.ts: 解析 SSE（含 hermes.tool.progress）
```

**v5.6.4（Hermes Chat 多模型 hotfix）：** Models 页维护 `models.json` + `config.yaml` `custom_providers`（`hermes-config-yaml.ts`）；**Set Default** 走 `hermes-chat:set-model-config`；Chat 下拉为 **session 级**（`session-models.json` + `hermes-chat:get/set-session-model`）；普通发送**不**写 `config.yaml`、**不** restart Gateway；Chat **无** Save as Default。
**v5.6.2（Local Hermes WebChat Surface）：** `screens/Hermes/pages/Chat/*` 通过 `window.hermesDefaultChat`；事件复用全局 `chat-*`。
**v8.0 / v8.0.1 / v8.0.2 / v8.0.3 / v8.0.4 / v8.0.5 / v8.1.0 / v8.1.1 / v8.2.0（Copilot Chat Module）：** 默认入口常驻 `HermesPersistentChatWorkspace` → `MultiRunChatShell`（`VITE_CHAT_ENGINE=legacy` 回退旧 Surface）→ `AiosCopilotChatHost` → `modules/chat` `ChatSurface`；**v8.2** `ChatWorkspaceProvider` 提升至 `HermesScreen`，Main `chat-workspace.db` 权威持久化（localStorage v1 仅迁移）；Sessions 页走 `window.sessionCatalog` 直读 profile `state.db` + `openSession` 联动；Runtime 走 `window.chatRuntime`；Files 走 `window.chatFiles`。

### 单 Gateway（legacy default）

- 启动：`hermes.ts` → `startGateway()` spawn `hermes gateway`
- 健康：15s 轮询 `/health`
- 配置变更（API Key/模型/平台）可触发 `restartGateway()`

### Multi Profile Runtime（V1.1+）

7 个 Profile，端口 8642–8648：

| Profile | 端口 |
|---|---|
| default | 8642 |
| writer / coding / research / recruiters / finance / agenter | 8643–8648 |

状态机：`not_deployed → starting → running → stopping → stopped`，异常 → `failed`。

关键模块链：

```
profile-runtime-ipc.ts
  → profile-runtime-manager.ts（生命周期）
  → hermes-local-adapter.ts（spawn/health/message）
  → gateway-supervisor.ts（健康监管 + V1.2 自动重启）
  → gateway-log-collector.ts（V1.2 日志）
  → runtime-reconciler.ts（V1.2 App 重启恢复）
  → profile-runtime-db.ts（SQLite 控制面）
```

控制面 DB：`~/.hermes/desktop/profile-runtime.db`（9 表）。配置：`~/.hermes/desktop/profile-runtime.yaml`。

能力插件：`delegation-capability`、`skill-sync-capability`、`session-share-capability`。

### Web Operator（V2.2 单轨 ShellView）

```
BrowserToolbar / Hermes tool
  → aiosBrowser.open → BrowserController → ShellBrowserViewAdapter
      → ShellViewManager layer "web-operator"
      → mainWindow.send("browser.opened") → Layout → web-operator tab
  → WebContentsHost.setBounds("web-operator")

MainTopBar (+) external tab
  → useExternalBrowserTabs.openExternalTab
      → shellView.create("external-browser:uuid", { partition: externalBrowserPartition(id) })
      → WebContentsHost(layerId) + navigate tab

browser.* back/forward/reload/getState/...
  → 同一 ShellBrowserViewAdapter WebContents（web-operator）
```

### CRM Desktop Bridge（V5.7.1 + V5.7.6 Host Bridge + V5.7.10 CRM-Lite Demo）

CRM 业务页面运行于 WebOperator WebContentsView 中，通过专用 preload `src/preload/crm-bridge-preload.ts` 建立**受控双向通道**：

- CRM → Desktop：CRM JSSDK 在用户主动点击后提交 `CrmBridgeEvent`（preload 先校验用户手势；Main 再校验 origin / event type / payload size / requestId 去重），并可触发 Renderer 聚焦侧栏与刷新 snapshot。**V5.7.10**：`crm.product.context.submit` + `page.app: crm-lite` + `localhost:5178`（crm-lite demo）。
- Desktop → CRM（商品）：`desktop.crm.product.fillForm` / `desktop.crm.product.create`（`CrmEventPanel` 测试按钮 → `aiosBrowser.sendCrmCommand`）。
- CRM → Desktop（ready）：**V5.7.6** `CopilotDesktopCRM.emitReady` / `crm.page.ready` 专用通道，不校验用户手势；Main `crm-handoff-orchestrator` 匹配 pending handoff 并自动 `pushJson` + `runAction`。
- Desktop → CRM：Renderer 或 Hermes BrowserToolBridge 通过 IPC 发送 `CrmDesktopCommand`（含 expectAck），Main 转发到 WebOperator WebContents，由 preload 转为 `window.postMessage` 交由 CRM JSSDK 消费；JSSDK ack 经 `crm-bridge:command-result` 回传。

入口模块：Main `src/main/crm-bridge/`（含 handoff/command-result store）；Shared DTO `src/shared/crm-bridge/`；页面 SDK `resources/crm-bridge/hermes-crm-bridge-sdk.js`；配置 `resources/crm-bridge/crm-bridge.config.json`。

### Enterprise Install（V1.2.1）

```
Renderer Install 屏
  → enterprise:* IPC（13 个，见 enterprise-installer.ts）
  → 20 步流水线：预检 → Bundle → Agent → Venv → Hermes Home → Profile Bootstrap → Marker
```

安装目录（Windows）：`%LOCALAPPDATA%\Programs\SMC-Copilot\`（默认，无空格）或注册表 `HKCU\Software\SMC\copilot` → `InstallLocation`。**V5.4** 主程序为 `desktop.exe`；兼容读取 legacy `HKCU\Software\SMC\Copilot` 等键。**V5.3** 标准 runtime 布局：

```text
$INSTDIR/desktop.exe
$INSTDIR/runtime/hermes/{src,venv,logs}
$INSTDIR/runtime/serve/{src,venv,.env,logs}
$INSTDIR/runtime/portal/{src,node_modules,.env.local,logs}
$INSTDIR/bin/{desktop,smc-copilot,hermes-desktop,hermes,serve,portal}.cmd
```

旧目录名（`hermes-agent` / `copilot-serve` / `ai-os-full`）仅用于**读取 fallback**与**一次性 schema v4 磁盘迁移**（`004-v53-runtime-layout.ts`）；canonical 布局为上式。

**Portal monorepo 部署（V5.3.4，`build/scripts/deploy-copilot-serve.ps1`）**：企业安装流水线不 clone Portal；运维脚本可 `-SkipServe` 仅部署 Portal：

```powershell
# 默认同时部署 copilot-serve + Portal monorepo
.\build\scripts\deploy-copilot-serve.ps1

# 仅 Portal：git clone ai-os-full → runtime/portal/src → pnpm install
.\build\scripts\deploy-copilot-serve.ps1 -SkipServe
```

脚本写入用户级环境变量与 `runtime/desktop-runtime.json`：

| 变量 / 字段 | 值 |
|---|---|
| `COPILOT_PORTAL_ROOT` | `$INSTDIR\runtime\portal\src`（monorepo 根） |
| `COPILOT_PORTAL_RUNTIME_ROOT` | `$INSTDIR\runtime\portal` |
| `desktop-runtime.json` → `portalSourceRoot` / `portalRuntimeRoot` | 同上 |

Main 解析优先级（`portal-root-resolver.ts` → `aios-paths.ts`）：`COPILOT_PORTAL_ROOT` → `desktop-runtime.json` → 文件系统候选（含 legacy `ai-os-full`）。`buildCopilotRuntimeEnv()` **保留**已有 env，不再无条件覆盖为空目录。

用户数据：`%USERPROFILE%\.hermes\`。

安全：仅 `127.0.0.1`、不写 HKLM/系统 PATH、Token 不落盘、Bundle 必须 SHA-256。

### 用户源安装 + PyPI 镜像（V1.4.1）

欢迎页 / Install 向导选择 **本地 zip** 或 **Git clone** 时：

```
AgentSourceSelect / install-wizard
  → PipMirrorFields（预设：清华/阿里/腾讯/官方/自定义）
  → hermesAPI.startInstallWithSource({ sourceType, localZipPath?, pipIndexUrl, trustedHost, pipMirrorPreset })
  → installer.runInstallWithSource
      → hermes-agent-source-installer（解压 zip / git clone）
      → agent-deps-installer.installHermesAgentDependencies
          → resolvePipMirrorConfig（UI → desktop-runtime.json → deployment.json → 环境变量 → 清华默认）
          → uv --no-config 优先 requirements.txt；有 wheels 则离线；失败回退 pip
  → desktop-runtime.json 持久化 pipMirror + agentSource
```

**环境变量（可选，覆盖 UI）**：`HERMES_PIP_INDEX_URL`、`HERMES_PIP_TRUSTED_HOST`（或 `PIP_INDEX_URL` / `PIP_TRUSTED_HOST`）。

**离线 wheel**：`$INSTDIR/runtime/wheels/` 或 `runtime/hermes/src/wheels/`（`--offline` / `--find-links`）。

### 窗口控制（V1.4.1 + V2.0）

- Preload：`window.hermesAPI.windowControls` → IPC `window:minimize|maximize-or-restore|close|is-maximized`
- Main：`src/main/window/window-ipc.ts` → `registerWindowIpc()`（`setupIPC()` 内一次）
- Main 窗口尺寸：`src/main/shell/main-window-controller.ts` ← `shared/shell/main-page-constants.ts`（默认 **1280×800**，最小 **900×600**；有 `window-state` 持久化时优先历史值）
- Renderer：**主界面** `WindowControls` 挂于 `MainTopBar`（`screens/MainPage/MainTopBar.tsx`）；splash/welcome/install/setup 仍用各屏自有标题栏；macOS 主界面由 `MainTopBar` 拖拽（`App.tsx` 在 `screen === "main"` 时不渲染全局 `drag-region`）

## 配置文件位置

| 文件 | 用途 |
|---|---|
| `~/.hermes/desktop.json` | 连接模式 local/remote |
| `~/.hermes/.env` | API Keys |
| `~/.hermes/config.yaml` | Hermes Agent 配置 |
| `~/.hermes/state.db` | 会话历史（better-sqlite3 只读） |
| `~/.hermes/profiles/<name>/` | 各 Profile 目录 |
| `~/.hermes/desktop/profile-runtime.db` | 运行时控制面 |
| `~/.hermes/desktop/web-operator/` | Web Operator 配置与审计日志 |
| `$INSTDIR/runtime/desktop-runtime.json` | 安装目录、agent 源、**pipMirror**（V1.4.1） |
| `$INSTDIR/runtime/deployment.json` | 企业部署配置；默认 `runtime.pipIndexUrl` 清华源 |
| `$INSTDIR/runtime/installer-precheck.json` | NSIS 预检结果（`enterprise:get-installer-precheck`） |

## 新增/修改功能 checklist

### 新增 IPC

1. `src/main/<module>.ts` 或 `*-ipc.ts`：`ipcMain.handle('channel-name', ...)`
2. `src/main/index.ts` 确保注册（若独立文件则 `setup*IPC()`）
3. `src/preload/index.ts`（或专用 api 文件）封装为 Promise
4. `src/preload/index.d.ts` 补充类型
5. 更新 `docs/API_CONTRACTS.md`
6. 必要时加 `tests/ipc-handlers.test.ts` / `preload-api-surface.test.ts`

### 新增 UI 页面

1. `src/renderer/src/screens/<Name>/` 组件
2. `Layout.tsx` 导航项 + 路由
3. `src/shared/i18n/locales/*/` en 与 zh-CN 的对应模块（共 20 个模块名，见 `MODULES.md`）
4. 通过 Preload API 调主进程，不在 Renderer 写文件 IO

### 新增 Profile Runtime 能力

1. 类型放 `src/shared/profile-runtime/profile-runtime-contract.ts`
2. 错误码放 `profile-runtime-errors.ts`
3. Main 实现 + `profile-runtime-ipc.ts` 注册
4. `profile-runtime-api.ts` + `index.d.ts` 暴露

### 功能/计划完成后：同步文档

代码计划全部阶段完成或功能实现收尾时，**必须**执行 rule [`.cursor/rules/007-sync-project-docs.mdc`](.cursor/rules/007-sync-project-docs.mdc) 与 skill [`.agents/skills/sync-project-docs/SKILL.md`](.agents/skills/sync-project-docs/SKILL.md)：

1. `git --no-pager diff --stat` / `--name-only` 收集变更
2. 按 skill 内 [`doc-map.md`](.agents/skills/sync-project-docs/doc-map.md) 分类（IPC / Preload / 路由 / Screen / 版本）
3. **增量**更新相关文档（见 rule 完整清单）；Renderer 变更时同步 `docs/renderer/` 对应页
4. 自检：路径存在、IPC 与源码一致、`AGENTS.md` 与 `docs/INDEX.md` 版本行对齐
5. 输出「文档同步摘要」后再宣告任务完成

**跳过**：仅 typo/注释/格式化；用户明确说「不要改文档」。

## 技术栈与命令

| 类别 | 技术 |
|---|---|
| 桌面 | Electron 39 + electron-vite 5 |
| UI | React 19 + TailwindCSS 4 + Lucide |
| 语言 | TypeScript 5.9 |
| DB | better-sqlite3 |
| i18n | i18next（en / zh-CN，源语言 en） |
| 测试 | Vitest + Testing Library |

```bash
npm run dev          # 开发
npm run build        # typecheck + 构建
npm run typecheck    # 双 tsconfig 检查
npm test             # Vitest
npm run lint         # ESLint
```

## 主进程模块速查

| 模块 | 文件 | 一句话 |
|---|---|---|
| 入口 | `index.ts` | BrowserWindow + 全部 IPC 注册 |
| Gateway/聊天 | `hermes.ts` | 消息双路由 + Gateway 生命周期 |
| 安装 | `installer.ts` | `runInstallWithSource`、Doctor；委托 **agent-deps-installer** |
| 依赖安装 | `enterprise/agent-deps-installer.ts` | uv/pip、镜像、wheelhouse、requirements 优先 |
| PyPI 镜像 | `enterprise/pip-mirror-config.ts` + `shared/enterprise/pip-mirror-presets.ts` | 解析/预设 |
| 窗口 IPC | `window/window-ipc.ts` | `registerWindowIpc` |
| NSIS 预检读 | `enterprise/installer-precheck-reader.ts` | `installer-precheck.json` |
| 配置 | `config.ts` | desktop.json / .env / config.yaml |
| 会话 | `sessions.ts`, `session-cache.ts` | state.db 查询 + 本地缓存 |
| 模型/档案 | `models.ts`, `profiles.ts` | models.json + profile 目录 |
| 记忆/人格 | `memory.ts`, `soul.ts` | MEMORY.md / SOUL.md |
| 技能/工具/定时 | `skills.ts`, `tools.ts`, `cronjobs.ts` | |
| Claw3D | `claw3d.ts` | Office 可视化 |
| SSE | `sse-parser.ts` | 流式解析 |
| Profile Runtime | `profile-runtime-*.ts`, `gateway-*.ts` | 多实例运行时 |
| Enterprise | `enterprise/*.ts` | 企业安装流水线 |
| Web Operator | `browser/*.ts` | 受控浏览器 + Tool Server |
| Token 注入 | `auth/token-inject-url.ts`, `auth/token-header-injector.ts`, `auth/token-store.ts` | Portal Home Bearer；keytar 优先 |
| Auth / Bootstrap | `auth/auth-client.ts`, `auth/auth-ipc.ts`, `user-config/user-config-client.ts`, `startup/startup-decision.ts` | Portal 登录 + 本地 bootstrap + 启动门控 |
| Session 分区 | `shared/shell/browser-partitions.ts`, `shell/views/view-registry.ts` | 三分区策略常量与注册 |
| Portal Runtime | `aios/aios-*.ts`, `runtime/portal-root-resolver.ts` | Portal frontend/backend 启停、Doctor、monorepo 路径解析 |

## 文档体系

> 完整索引见 [`docs/INDEX.md`](docs/INDEX.md)。功能收尾后按 [007-sync-project-docs](.cursor/rules/007-sync-project-docs.mdc) 增量同步。

### 核心文档（Main / Preload / 契约）

| 路径 | 作用 | 何时读 |
|---|---|---|
| [`docs/INDEX.md`](docs/INDEX.md) | 项目定位、版本特性、核心目录、Renderer 入口 | **每次**或首次 |
| [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | 三层进程、Gateway、主界面布局演进 | 改架构 / 进程边界 |
| [`docs/MODULES.md`](docs/MODULES.md) | Main / Preload / Renderer 模块职责 | 改模块边界（**007 rule 不自动同步**，需用户要求或边界变更时手动更新） |
| [`docs/API_CONTRACTS.md`](docs/API_CONTRACTS.md) | IPC 完整契约（单一事实源） | 新/改 IPC channel |
| [`docs/READING_GUIDE.md`](docs/READING_GUIDE.md) | 分阶段阅读顺序 + 功能域阅读链 | onboarding / 定位入口 |
| [`docs/DO-NOT-TOUCH.md`](docs/DO-NOT-TOUCH.md) | 禁止擅自修改的核心文件 | 动 preload / hermes / i18n 前 |

### Renderer 专项（`docs/renderer/`）

| 路径 | 作用 |
|---|---|
| [`docs/renderer/INDEX.md`](docs/renderer/INDEX.md) | Renderer 总入口、Workspace 启用状态、分发逻辑 |
| [`APP_STARTUP.md`](docs/renderer/APP_STARTUP.md) | splash → login → main 启动门控 |
| [`MAIN_LAYOUT.md`](docs/renderer/MAIN_LAYOUT.md) | Layout / MainPage / MainTopBar / WorkspaceOutlet |
| [`WORKSPACE_ROUTING.md`](docs/renderer/WORKSPACE_ROUTING.md) | workspace-registry / WorkspaceRenderer |
| [`STATE_AND_CONTEXT.md`](docs/renderer/STATE_AND_CONTEXT.md) | mainPageState、Context、KeepAlive |
| [`PRELOAD_API_USAGE.md`](docs/renderer/PRELOAD_API_USAGE.md) | Renderer 侧 `window.*` 使用边界 |
| [`COMPONENTS.md`](docs/renderer/COMPONENTS.md) / [`HOOKS.md`](docs/renderer/HOOKS.md) / [`STYLES.md`](docs/renderer/STYLES.md) | 组件族 / Hooks / 样式策略 |
| [`screens/INDEX.md`](docs/renderer/screens/INDEX.md) | Screen active/retained 分类 + 各 Screen 详情页 |
| [`screens/web-operator/`](docs/renderer/screens/web-operator/) | WebOperator 子域（Page Context、Hermes Task、CRM Bridge 等） |
| [`components/INDEX.md`](docs/renderer/components/INDEX.md) | layout / shell / hermes / workspace / install 组件族 |
| [`workspace/INDEX.md`](docs/renderer/workspace/INDEX.md) | registry、secondary-nav、WorkspaceRenderer |

### Work 专家工作台 Spec Pack（v1.3）

| 路径 | 作用 | 何时读 |
|---|---|---|
| [`docs/specs/v1.3-workbuddy-product-line/00-overview.md`](docs/specs/v1.3-workbuddy-product-line/00-overview.md) | v1.3 Spec Pack 入口、硬约束、阅读顺序 | **改 `screens/Hermes` 前** |
| [`03-layout-boundary.md`](docs/specs/v1.3-workbuddy-product-line/03-layout-boundary.md) | MainPage vs Hermes 三栏、`.hermes-page` 模板 | 改 Layout / 页面壳层 |
| [`13-ai-coding-structure.md`](docs/specs/v1.3-workbuddy-product-line/13-ai-coding-structure.md) | UI 输出质量 Checklist | **写页面组件前** |
| [`16-cursor-execution-prompt.md`](docs/specs/v1.3-workbuddy-product-line/16-cursor-execution-prompt.md) | Agent 任务前缀模板 | 派发 Hermes 任务时 |
| PRD 源 | [`prd_work/v1.3_project-layout-prompt.md`](prd_work/v1.3_project-layout-prompt.md) | 产品全貌 |

Cursor rules：`.cursor/rules/workbuddy-product-line.mdc`、`.cursor/rules/hermes-renderer-structure.mdc`（globs: `screens/Hermes/**`）

### 辅助与历史

| 路径 | 作用 |
|---|---|
| [`docs/code-assets/`](docs/code-assets/) | API / 组件 / 模式索引（辅助检索） |
| [`docs/memory-bank/`](docs/memory-bank/) | 架构决策、踩坑、项目摘要 |
| [`docs/specs/ui-spec-template.md`](docs/specs/ui-spec-template.md) | UI spec 模板 |
| [`docs/superpowers/`](docs/superpowers/) | 发布/平台专项计划与 design spec |
| [`prd/`](prd/) | 产品需求（与版本特性索引对齐） |

## 深入阅读顺序

1. `docs/INDEX.md` — 项目全景与版本特性
2. `docs/ARCHITECTURE.md` — 进程模型与 Gateway
3. `docs/MODULES.md` — 模块边界
4. `docs/API_CONTRACTS.md` — IPC 完整契约
5. `docs/READING_GUIDE.md` — 按功能阅读路径
6. `docs/renderer/INDEX.md` — 改 UI / Screen 时
7. `src/preload/index.ts` + `src/main/index.ts` — 通信全貌

按功能跳转：

| 任务 | 起点文件 |
|---|---|
| 改 Gateway/聊天 IPC | `hermes.ts` → `sse-parser.ts`；UI 在 `modules/hermes-runtime/` |
| 改安装 | `screens/Install/AgentSourceSelect` → `installer.ts` → `agent-deps-installer.ts` |
| 改 PyPI 镜像 | `PipMirrorFields.tsx` → `pip-mirror-presets.ts` → `pip-mirror-config.ts` |
| 改主界面壳层 / 顶栏 | `screens/MainPage/*` → `Layout.tsx`；常量 `shared/shell/main-page-constants.ts` |
| 改窗口按钮 | `MainTopBar.tsx` / `WindowControls.tsx` → `window-ipc.ts` → preload `windowControls` |
| 改多 Profile | `profile-runtime-manager.ts`；UI 在 `modules/hermes-runtime/` 或 `profileRuntime` API |
| 改 Web Operator 视口 / adapter | `shell-browser-view-adapter.ts` + `shell-view-manager.ts` + `WebContentsHost` |
| 改 Web Operator 地址栏 / tool | `BrowserToolbar` → `browser-controller.ts` → `ShellBrowserViewAdapter` |
| 改 external-browser 多 tab | `useExternalBrowserTabs.ts` + `MainViewTabs` + `WorkspaceOutlet` + `shellView.create` |
| 改 Tab DnD / 顶栏操作 | `MainViewTabs.tsx` + `workspace-registry.ts` + `tab-order.ts` + `Layout.tsx` |
| 改 Workspace 路由 | `workspace-registry.ts` → `WorkspaceRenderer.tsx` → `WorkspaceOutlet.tsx` |
| 改 Workspaces 三栏布局 | `screens/Workspaces/panels/WorkspacesShell.tsx` → `WorkspaceStatusCards` / `WorkspacesSidebar` / `WorkspaceRightPanel` / `registry/workspace-pages.tsx` |
| 改 Local Hermes（default profile） | `screens/Hermes/` → **先读** `docs/specs/v1.3-workbuddy-product-line/00-overview.md` → `hermesDefaultApi.ts` → `window.hermesAPI`（禁止引入 `workspaceChat` / `profileRuntime`） |
| 改 Local Hermes Models 库 | `screens/Hermes/pages/Models/HermesDefaultModelsSurface.tsx` → `hermesDefaultApi.models.*` / `providers.setEnv`（对齐 Workspaces `Models.tsx`） |
| 改 Session 分区 / Token | `browser-partitions.ts` → `useExternalBrowserTabs.ts` / `view-registry.ts`；`token-inject-url.ts` |
| 改 Auth / 登录 / Bootstrap | `LoginScreen.tsx` → `desktopAuth` / `desktopUserConfig` → `auth-client.ts` / `user-config-client.ts` / `startup-decision.ts` |
| 改启动门控 / splash→login | `useStartupGate.ts` → `shell-api.ts` → `startup-ipc.ts` → `startup-decision.ts` |
| 改 Settings / Runtime 入口 | `SettingsDrawer.tsx` + `server/ServerPanel.tsx` + `general/GeneralPanel.tsx` + `Layout.openSettingsDrawer`；Workspaces 侧栏已无 `settings` 页 |
| 改 Serve-First Runtime / Pairing / Chat transport（v9） | `copilot-runtime-client/*` → `window.copilotRuntime`；Chat cutover：`ServeChatRuntimeAdapter` + `chat-runtime-ipc`；UI `CopilotRuntimeStatusSection` + Instances；契约 `src/shared/copilot-runtime/`；OpenAPI `npm run generate:serve-client` |
| 改安装 / runtime 路径（V5.3 / V5.4） | `electron-builder.yml` + `build/installer.nsh` → `install-location-resolver.ts` → `runtime/runtime-paths.ts` → `shim-manager.ts` |
| 改 Portal 部署 / Settings Portal Runtime | `build/scripts/deploy-copilot-serve.ps1` → `PortalRuntimeSection.tsx` → `aios:get-portal-info` → `getAiOsPortalInfo()` |
| 改 i18n | `src/shared/i18n/locales/<locale>/navigation.ts`（二级侧栏 + openSettings） |
| 改全局 Dialog/Drawer / WebContentsView 遮挡 | `components/overlay/*` → `Layout.tsx` `legacyDrawerBlocking` → `WebContentsHost` `effectiveEnabled` |
| 改 WebOperator Hermes→Host 表单写回 | `components/hermes/panel/host-form-fill/*` → `HostBridgeCommandContext` → `window.aiosBrowser.sendHostCommand` |

## 版本特性索引

| 版本 | 能力 | 关键路径 |
|---|---|---|
| V1.1 | Multi Profile Gateway、Portal/Experts 工作台 | `profile-runtime-*`, `Workspaces`, `ProfileWorkspace` |
| V1.2 | 崩溃检测、自动重启、端口冲突、启动超时、日志、状态恢复 | `gateway-supervisor`, `gateway-log-collector`, `runtime-reconciler`, `LogViewer` |
| V1.2.1 | Enterprise 一键部署、Preflight、Bundle、Doctor | `src/main/enterprise/`, `src/shared/enterprise/` |
| V1.4 | Desktop Shell、NSIS 预检、WindowControls、RuntimeSetup 预检卡片 | `components/layout/`, `build/nsis/`, `installer-precheck-reader` |
| V1.4.1 | 窗口 IPC 加固、PyPI 镜像 UI、`agent-deps-installer`、`--no-config` uv | `pip-mirror-*`, `agent-deps-installer`, `window-ipc` |
| V1.9 | ShellView IPC、`WebContentsHost`、启动顺序与菜单接管 | `shell-view-*`, `WebContentsHost`, `shell-menu` |
| **V2.0** | **MainPage 主界面壳层**、顶栏 Tabs/Profile/Runtime、默认窗口 1280×800 | `screens/MainPage/`, `main-page-constants.ts`, `Layout.tsx` |
| **V2.1** | Sidebar 三态、动态 Profile Tabs、ShellView 全 IPC、WebOperator→`WebContentsHost` | `main-page-tabs.ts`, `shell-view-contract.ts`, `WebOperatorScreen.tsx` |
| **V2.2** | ShellBrowserViewAdapter、browser.opened、external-browser tabs、Tab DnD | `shell-browser-view-adapter.ts`, `useExternalBrowserTabs.ts`, `MainViewTabs.tsx`, `browser-contract.ts` |
| **V2.3** | metadata 事件、导航 IPC、Tab 持久化、KeepAlive、layout-calc-parser | `shell-view-event-forwarder.ts`, `main-page-state-store.ts`, `KeepAliveView.tsx`, `useShellViewMetadata.ts` |
| **V3.2** | Workspace registry、SettingsDrawer 统一、Sidebar 二级、state V2、token 注入 | `workspace-registry.ts`, `SettingsDrawer/`, `workspace-secondary-nav.ts`, `main-page-state-migrate.ts`, `token-header-injector.ts` |
| **V3.3** | Endpoint Config + Main Token Vault + origin 白名单注入 + login 启动门控 + schema v2 | `auth-endpoint-config-store.ts`, `token-store.ts`, `token-header-injector.ts`, `browser-partitions.ts`, `LoginScreen`, `startup-decision.ts` |
| **V3.3.1** | 本地 bootstrap 默认、email 登录、smcShell 启动 IPC、portal 延迟显示 | `user-config-client.ts`, `auth-client.ts`, `shell-api.ts`, `startup-ipc.ts`, `portal-view-coordinator.ts` |
| **V4.0** | 专家 Profile 预设（9601–9641）、`agency-agents-zh` 角色编译、`profile_role_specs` DB v3、Gateway `HERMES_HOME` 隔离、Settings Profiles 管理 UI | `profile-roles/*`, `profile-role-ipc.ts`, `hermes-local-adapter.ts`, `SettingsDrawer/multi-profiles/*`, `hermes-expert-profiles.v1.yaml` |
| **V3.2.1** | 三分区、per-tab external partition、token 端口可配置、Runtime 入口收敛、kind 分发、registry Tab 规则 | `browser-partitions.ts`, `token-inject-url.ts`, `view-registry.ts`, `RuntimeGuard.tsx`, `MainViewTabs.tsx` |
| **V3.6.2** | **team_v1.6.2** Workspaces 三栏 hotfix：Status Cards、左右折叠、静态 page registry、修复 lazy 页 icons | `WorkspacesShell`, `WorkspaceStatusCards`, `WorkspaceRightPanel`, `registry/workspace-pages.tsx` |
| **V3.6.3** | SettingsDrawer `server`/`general` 面板；Workspaces 移除 settings 页；顶栏 `ServersEntry` | `SettingsDrawer/server/`, `SettingsDrawer/general/`, `MainPage/ServersEntry.tsx` |
| **V5.3** | 安装 runtime 目录重命名：`hermes` / `serve` / `portal`；`src`+`venv` 分层；统一 `runtime-paths.ts`；`bin/*.cmd` shim | `runtime/runtime-paths.ts`, `enterprise/windows/*`, `shim-manager.ts`, `copilot-serve-paths.ts`, `aios-paths.ts` |
| **V5.3.4** | Portal 部署脚本 clone `ai-os-full` → `runtime/portal/src`；`COPILOT_PORTAL_ROOT` 优先级解析；`aios:get-portal-info` + Settings Portal Runtime 路径展示 | `deploy-copilot-serve.ps1`, `portal-root-resolver.ts`, `aios-ipc.ts`, `PortalRuntimeSection.tsx` |
| **V5.4** | 安装身份统一：`SMC-Copilot` 目录、`desktop.exe`、`HKCU\Software\SMC\copilot`；NSIS 不复用带空格 legacy 默认目录；安装包 `SMC-Copilot-*-setup.exe` | `electron-builder.yml`, `build/installer.nsh`, `install-location-resolver.ts`, `shim-manager.ts` |
| **V5.4.1** | Review hotfix：migration schema **5** 刷新 `desktop-runtime.json` 身份字段；`readLegacyInstallLocations` 不把 primary 注册表当 legacy；`AppUserModelId` 对齐 `appId` | `005-v541-install-identity.ts`, `migration-runner.ts`, `install-location-resolver.ts`, `index.ts` |
| **V5.6** | **Local Hermes** 顶层 Tab + 三栏操作模块（Chat/Sessions/Skills/Tools/Memory/Providers/Models）；固定 `default` profile + `hermesAPI` IPC | `screens/Hermes/`, `workspace-registry.ts` `local-hermes`, `WorkspaceRenderer.tsx` |
| **V5.6.1** | Local Hermes hotfix：`syncGatewayModelSection` + Gateway restart；Chat 新对话/tool progress；Models/Sessions/Skills/Providers 功能补齐；`workspaces.hermes.*` i18n | `prd/v5.6.1_hermes-default-hotfix.md`, `src/main/hermes.ts`, `src/main/config.ts`, `screens/Hermes/` |
| **V5.6.2** | Local Hermes WebChat Surface：`hermes-chat:*` IPC + `HermesDefaultWebChatSurface`（模型/附件/流式，无 `workspaceChat`） | `prd/v5.6.2_hermes-webchat-surface.md`, `src/main/hermes-default-chat/`, `screens/Hermes/pages/Chat/` |
| **V5.6.3** | Local Hermes Models 页复刻 Workspaces：卡片网格 + 弹窗增删改 + 模型发现；`hermesDefaultApi.models`（无 Set active） | `prd/v5.6.3_hermes-models-page.md`, `screens/Hermes/pages/Models/HermesDefaultModelsSurface.tsx` |
| **V5.6.4** | Hermes 多模型：`custom_providers` YAML 同步、session 级 Chat 模型、移除 Save as Default、发送不写 config | `prd/v5.6.4_hermes-chat.md`, `src/main/hermes-config/`, `hermes-session-model-store.ts` |
| **V5.7** | **WebContentsView 浏览器核心**：Frame Tree、DOM Snapshot、iframe 元素定位/点击/输入、结构化动作日志；`browser:*` IPC；`PageStructurePanel` | `prd/v5.7_webcontentsview.md`, `src/main/browser/browser-v57-core.ts`, `screens/WebOperator/`, `docs/API_CONTRACTS.md` § Web Operator V5.7 |
| **V5.7.5** | WebOperator `[分析内容]` → Hermes 任务流：`webOperatorTaskSession` IPC + `task_session` SQLite；`HermesTaskStartDialog`；iframe `pageUrl` 派生 | `prd/v5.7.5_hermes_integration.md`, `web-operator-task-session-*`, `HermesTaskPanel`, `components/hermes/` |
| **V5.7.6** | **CRM Host Bridge**：handoff store + `crm.page.ready` 自动交付 + command ack；Hermes 工具 `crm.*`；JSSDK `hermes-crm-bridge-sdk.js`；`CrmEventPanel` 调试区 | `prd/v5.7.6_crm_host_bridge.md`, `crm-handoff-*`, `crm-command-result-store.ts`, `browser-tool-bridge.ts`, `CrmEventPanel.tsx` |
| **V5.7.8** | **MainLayout 全局 Overlay**：`OverlayProvider` / `DialogLayer` / `DrawerLayer` / `NativeShellLayerGate`；阻塞型弹层打开时 hide WebContentsView，关闭后无刷新恢复 | `prd/v5.7.8_main_layout.md`, `components/overlay/*`, `Layout.tsx`, `WebContentsHost.tsx` |
| **V5.7.10** | **CRM-Lite Bridge Demo**：`crm.product.context.submit` + `desktop.crm.product.fillForm/create`；`page.app` 支持 `crm-lite`；origin `localhost:5178`；`CrmEventPanel` 商品上下文与测试按钮 | `prd/v5.7.10_bridge_demo.md`, `src/shared/crm-bridge/`, `resources/crm-bridge/crm-bridge.config.json`, `CrmEventPanel.tsx` |
| **V6.1** | **Hermes MCP Skill Gateway**：独立 `mcp` 左导航、`HermesMCPPage`、MCP registry DB、runtime proxy `:18781`、`mcp-skill-bridge`、`window.hermesAPI.mcp` | `prd/v6.1_mcp-skill-gateway.md`, `src/main/mcp/*`, `src/shared/mcp/*`, `screens/Hermes/pages/MCP/*`, `resources/skills/system/mcp-skill-bridge/` |
| **V6.3** | **WebOperator HermesTaskStartDialog 组件化**：`HermesPanelSkill` / `HermesPanelSession`；HostBridge `requiredSkillName` 校验；session 续写 | `prd/v6.3_bridge-to-hermes.md`, `components/hermes/panel/HermesPanelSkill.tsx`, `HermesPanelSession.tsx`, `screens/WebOperator/HermesTaskStartDialog.tsx` |
| **V6.3.1** | **WebOperator 任务 Chat 持久化 hotfix**：`currentTask` 提升 Context + `sessionStorage`；`getLastActive` IPC；mount hydrate；弹 Dialog 不清任务 | `prd/v6.3.1_hermes-task-persist-hotfix.md`, `context/WebOperatorPageContext.tsx`, `lib/web-operator-current-task-cache.ts`, `web-operator-task-session-store.ts` |
| **V6.3.3** | **WebOperator Task Session 绑定键调整**：`source + requestId` 替代 `pageUrl` 唯一键；schema v2 + v1 迁移；Main 派生 `taskId`；HostBridge `web-host-bridge` | `prd/v6.3.3_task-to-session-request.md`, `web-operator-task-session-*`, `shared/web-operator/build-task-id.ts`, `HermesTaskPanel.tsx`, `HostBridgePanel.tsx` |
| **V6.3.4** | **WebOperator Hermes→Host 表单写回**：Hermes 输出 `host_form_fill` artifact；Panel「写回当前表单」按钮；`HostBridgeCommandContext` 共享 `runCommand`；`desktop.host.form.fill` | `prd/v6.3.4_weboperator-hermes-host-form-fill.md`, `host-bridge/HostBridgeCommandContext.tsx`, `components/hermes/panel/host-form-fill/*`, `WebOperatorHermesPanelMessageList.tsx` |
| **V6.7.1** | **GeneHub MCP Registration Hardening**：bundle-preview 不 claim、ignore 同步服务端、profile-mapping.json、serverProfileId sync、签名校验、scripts provenance、MCP Gateway 卡片增强 | `genehub-profile-mapping.ts`, `genehub-client.ts`, `mcp-registration-service.ts`, `skill-install-worker.ts`, `script-provenance.ts`, `skill-package-validator.ts`, `GeneHubMcpRegistrationPanel.tsx`, `McpGatewayGeneHubRegistrationCard.tsx` |
| **v8.0** | **Copilot Chat Module 迁移**：runId Chat Runtime IPC、`modules/chat` Surface + AI-OS adapters/slots、`window.chatFiles`、File Platform shared 契约、`VITE_CHAT_ENGINE` 切换 | `prd_work/v8.0_upgrade-chat-module.md`, `src/main/chat-runtime/`, `src/shared/chat-runtime/`, `src/renderer/src/modules/chat/`, `scripts/chat-migration/` |
| **v8.0.1** | **Chat 迁移闭环**：Controller Session/History、Abort Promise、Work 参数、SSE 事件、Prompt Hint、`chat-runtime:command`、持久化 File index、ChatRunRegistry、`typecheck:chat` | `prd_work/v8.0.1_migrate.md`, `modules/chat/controller/`, `chat-files-session-store.ts`, `workspace/chatRunRegistry.ts` |
| **v8.0.2** | **Chat Full Integration**：完整 MessageList/Composer/ModelPicker、File Platform `chat-files/platform`（`files:*`）、PromptNavigator、多 Chat 后台并行、删除 `source/**` 与 `_upstream/**` | `prd_work/v8.0.2_chat-full-integrated.md`, `modules/chat/components/**`, `src/main/chat-files/platform/`, `MultiRunChatShell.tsx` |
| **v8.0.3** | **Chat Workspace Layout**：`ChatRunRecord` per-run 隔离、单一 `ChatRunHeader`、Content Rail 统一宽度、Composer Context Chip、Run 状态回传 Tab、`chat-workspace-state.v1` 持久化 | `prd_work/v8.0.3_chat-workspace-layout.md`, `modules/chat/workspace/ChatRunRecord.ts`, `chatWorkspaceReducer.ts`, `ChatRunHeader.tsx`, `AiosCopilotChatHost.tsx` |
| **v9.0.0** | **Serve-First Runtime Phase 0/1/2/3**：OpenAPI client + pairing/keytar + production 禁 spawn；**Phase 2** Instance/Config/MCP/Diagnostics + Gateway/YAML fail-closed；**Phase 3** `window.chatRuntime` → Serve `/api/v1/chat-runs*` + SSE（hand-authored 契约，legacy-direct 可回退） | copilot-runtime-client/, runtime-adapters/, prd_work/v9.0_serve-runtime-migration.md |
| **v8.2.0** | **Persistent Chat Workspace + Session Catalog**：Chat 常驻挂载、`chat-workspace.db`、Draft/Session 区分、Sessions 直读 `state.db`、`openSession` 去重、强制 Electron E2E | `prd_work/v8.2_persistent-chat-session.md`, `src/main/chat-workspace/`, `src/main/session-catalog/`, `HermesPersistentChatWorkspace.tsx`, `tests/e2e/chat/persistent-workspace.spec.ts` |
| **v8.1.1** | **Durable Runtime Closure**：Continuation completion 互斥、profile store+事务 sequence、Snapshot/RecoveryBridge、Queue IPC/UI、turnId Retry、Diagnostics Save、Playwright E2E（PR7 Deprecated 顺延） | `prd_work/v8.1.1_runtime-infrastructure.md`, `hermes-interaction-continuation-adapter.ts`, `chat-runtime-store-router.ts`, `chat-event-replay-service.ts`, `chat-queue-service.ts`, `chat-diagnostics-service.ts`, `tests/e2e/chat/` |
| **v8.1.0** | **Durable Chat Runtime**：事件驱动 `start`、state.db 持久化、Transport/State 分离、Interaction 流式续跑、Turn Ledger/Retry/Queue、Recovery+Diagnostics | `prd_work/v8.1_chat-interaction-contract.md`, `chat-runtime-store.ts`, `chat-event-sequencer.ts`, `hermes-interaction-continuation-adapter.ts`, `chatTurnLedger.ts`, `useChatRuntimeRecovery.ts` |
| **v8.0.5** | **Chat Interaction Loop**：Command+`turnId` 契约、Hermes Interaction Bridge、Turn Snapshot Queue/Retry、Session Files live Badge | `prd_work/v8.0.5_chat-interaction-loop.md`, `hermes-chat-command-adapter.ts`, `chat-interaction-registry.ts`, `chatTurnSnapshot.ts`, `useSessionFilesSummary.ts`, `ApprovalCard.tsx` |
| **v8.0.4** | **Chat Turn Lifecycle**：`initialSessionId`/`BIND_SESSION` 拆分、`submitComposer` 清 Draft、`turnId` 终态保护、`ChatFloatingRail` | `prd_work/v8.0.4_chat-turn-lifecycle.md`, `useChatController.ts`, `chatReducer.ts`, `chat-runtime-ipc.ts`, `ChatFloatingRail.tsx` |
| **v7.6** | **Hermes Agent MCP Host Mode**：Chat 统一 `hermesDefaultChat` + `buildExpertPromptHint`；`hermes-mcp:*` 写 `config.yaml` `mcp_servers`；禁止 Chat 直连 nodeskclaw / `callCatalogSkill` | `src/main/hermes-mcp-config/`, `src/shared/hermes-mcp-config/`, `pages/Chat/utils/buildExpertPromptHint.ts`, `HermesAgentMcpServersPanel.tsx`, `prd_work/v7.6_agent-host-mode.md` |
| **v7.5.1** | **Runtime Skill Fixed Route Hotfix**：Chat Expert+Skill 改 `nodeskclaw:*` IPC + `POST /api/v1/hermes/mcp/skill-gateway`（`tools/call` 仅 `{prompt, context}`）；禁止 Chat 走 `hermes-experts:call-catalog-skill` | `src/main/nodeskclaw/`, `src/shared/nodeskclaw/`, `runtimeSkillApi.ts`, `useRuntimeSkillSend.ts`, `useNodeskclawTaskStream.ts`, `prd_work/v7.5.1_hotfix.md` |
| **v7.4.2** | **Chat-first Work Controls**：Chat 恢复默认入口；`ComposerBar.workControlsSlot` + `WorkComposerControls` / `WorkChatContextBar`；Send 双路径（Hermes SSE vs Runtime Skill）；`tasks` 导航隐藏 | `pages/Chat/components/work/`, `api/runtimeSkillApi.ts`, `hooks/useWorkChatContext.ts`, `hooks/useRuntimeSkillSend.ts`, `prd_work/v7.4.2_hotfix.md` |
| **v7.4.1** | **Work 任务 Hotfix**（导航已由 v7.4.2 回退）：Hermes session 绑定、`HermesDefaultWebChatSurface` 任务窗口、`work-tasks.json`、`work:task-start/list/resume`、Popover 启动器 | `pages/Tasks/`, `workTaskApi.ts`, `work-task-store.ts`, `prd_work/v7.4.1_hotfix.md` |
| **v1.4** | **Work 任务窗口**（主路径已由 v7.4.1 取代）：`tasks` 导航、`pages/Tasks/`、stream blocks 代码保留供 RightDock | `screens/Hermes/pages/Tasks/`, `src/main/work/`, `prd_work/v1.4_work-agent-event-stream.md` |
| **V7.2.1** | **Expert MCP v6.1 Hotfix**：`ExpertMcpClient` class、`callCatalogSkill` 统一调用、`expert-route-guard` 强拦截、`expert-mcp-mappers`、严格 catalog kind 过滤、DB schema v3、`ExpertCatalogCallDrawer`、IPC `list-catalog-skills` / `call-catalog-skill` | `expert-mcp-client.ts`, `expert-route-guard.ts`, `expert-mcp-mappers.ts`, `expert-runtime.ts`, `ExpertCatalogCallDrawer.tsx`, `docs/API_CONTRACTS.md` § Hermes Experts |
| **V7.2** | **Remote Experts Pivot** + **Expert MCP Gateway v6**：直连 `/api/v1/expert/mcp` 同步 `tools/call`（无 HermesTask）；root `tools/list` 目录；`expert-mcp-client.ts`；Workbench Expert Gateway 健康；skill 选择 Call Drawer；本地 Runs/Artifacts | `expert-mcp-client.ts`, `expert-remote-catalog.ts`, `pages/Workbench|Artifacts|Experts`, `docs/specs/v7.2-nodeskclaw-remote-experts/` |
| **V7.1.1** | **Hermes Experts E2E + Desktop Sync**：`expert-profile-manager` Gateway 路由、`expert-run-bridge` Chat↔Run、InstallPlan policy/MCP/Skills、`expert-preflight`、Tools/MCP Inspector、nodeskclaw register/heartbeat/report | `expert-profile-manager.ts`, `expert-run-bridge.ts`, `expert-desktop-client.ts`, `hermes.ts` `getApiUrl(profile)` |
| **V7.1** | **Hermes Experts Workspace**：专家广场/团队/运行三页 + InstallPlan 物化 + summon + leader dispatch + Chat profile-aware + Inspector Timeline/Trust | `src/main/hermes-experts/*`, `src/shared/hermes-experts/*`, `screens/Hermes/pages/Experts|ExpertTeams|ExpertRuns`, `prd_work/v1.0_clone-workbuddy-expert.md` |
| **V7.0** | **Hermes MCP Client Integration**：nodeskclaw v4.3 client bootstrap/agents/tools/readiness、task events token、task result/artifact、structuredContent→recent tasks、MCP Gateway Client Contract UI | `hermes-client-api.ts`, `src/shared/hermes-client/`, `McpGatewayClientContractCard.tsx`, `McpGatewayAgentAliasPanel.tsx`, `McpGatewayTaskResultPanel.tsx`, `prd/v7.0_desktop-hermes-mcp-client-integration.md` |
| **V6.7** | **MCP Write Tools Approval**：Proxy 上下文 header、profile-scoped URL、服务端 grant 状态展示、撤销感知；Desktop 不本地审批 | `mcp-skill-gateway-proxy.ts`, `mcp-profile-url.ts`, `mcp-approval-errors.ts`, `McpGatewayServerAuthorizationPanel.tsx`, `mcp-gateway-operations-contract.ts` |
| **V6.6.2** | **GeneHub MCP Registration**：MCP install job 待确认 UI、pending cache、Bundle 预览、provenance `genehub.json`、MCP Gateway 联动卡片 | `pending-jobs-cache.ts`, `mcp-registration-service.ts`, `GeneHubMcpRegistrationPanel.tsx`, `McpGatewayGeneHubRegistrationCard.tsx` |
| **V6.6.1** | **MCP Gateway Operations**：运营契约 `MCP_OP_*`、10 步诊断、`readStructuredLogs` IPC、工具 category/permission/riskLevel、JSON invoke + 256KB 截断、五 Panel UI + Hermes 重启横幅 | `mcp-gateway-operations-contract.ts`, `mcp-gateway-diagnostics.ts`, `mcp-tools-cache.ts`, `mcp-gateway-invoke-test.ts`, `McpGateway*Panel.tsx`, `HermesMcpGatewayPage.tsx` |
| **V6.6** | **MCP Skill Gateway E2E**：一键诊断、`tools/list` 预览、Invoke Test（只读工具）；`runDiagnostics` / `listRemoteTools` / `invokeRemoteTool` IPC；工具缓存 60s | `prd/v6.6_mcp-skill-gateway-e2e.md`, `mcp-gateway-diagnostics.ts`, `mcp-gateway-invoke-test.ts`, `mcp-tools-cache.ts`, `HermesMcpGatewayPage.tsx` |
| **V6.5.1 Hotfix** | **GeneHub Skill Center 连接修复**：`contextBridge` 暴露 `genehubRuntime`；`team_v3.4.1` API 对齐（`desktop_device_id`、`job_type`、`mapSkill`/`mapBundle`）；删除 `claimed` 状态回传；Connection Card 可读错误 | `prd/v6.5.1_hotfix_genehub-skill-center-connection.md`, `genehub-session.ts`, `genehub-client.ts`, `useGeneHubRuntime.ts` |
| **V6.5** | **GeneHub Hermes Skill Sync**：`system/info.genehub` 连接发现、device/profile 注册、Bundle 安装/更新/卸载、状态回传；`window.genehubRuntime`；Local Hermes 左导航 `skillCenter` | `prd/v6.5_genehub-hermes-skill-sync.md`, `src/main/genehub/*`, `src/shared/genehub/*`, `screens/Hermes/pages/GeneHub/*` |
| **V6.4** | **MCP Skill Gateway Desktop Runtime**：本地 Proxy `:48742` → nodeskclaw `/api/v1/hermes/mcp`；Bearer 仅 Proxy 注入；Hermes `mcp_servers.mcp_skill_gateway` 注册；`window.mcpSkillGatewayRuntime` | `prd/v6.4_mcp-skill-gateway-desktop.md`, `src/main/mcp-skill-gateway-runtime/*`, `src/shared/mcp-skill-gateway-runtime/*`, `screens/Hermes/pages/McpGateway/*` |
| **V6.4.1 Hotfix** | **MCP Gateway 联调**：`system/info` descriptor、Proxy auto-initialize、`/admin/config` + 增强 health/debug、`mcp:sync-tools` 结构化错误、tools 缓存、McpGateway 状态卡片 | `prd/v6.4.1_hotfix_mcp-backend-desktop.md`, `mcp-backend-descriptor.ts`, `mcp-skill-gateway-proxy.ts`, `mcp-tool-sync-service.ts`, `HermesMcpGatewayPage.tsx` |
| **V6.4.1** | **MCP Backend Desktop**：nodeskclaw `account-login` + `/auth/me` member 校验；`AuthEndpointConfig.backendUrl` 为唯一远程 backend 源（测试默认 `http://192.168.0.118:4510`）；MCP Proxy / Test remote MCP / 页面一致性检查统一派生 | `prd/v6.4.1_mcp-backend-desktop.md`, `src/main/auth/nodeskclaw-auth-response.ts`, `src/main/auth/auth-client.ts`, `LoginForm.tsx`, `HermesMcpGatewayPage.tsx` |
| **V6.0** | **HostBridge JSSDK 标准化** + **WebOperator 多页签**：`host.bridge.submit` / `host.page.ready` / `desktop.host.form.fill`；`bridge-config.json`（userData）；callbackUrl 新 tab + handoff 回填；`HostBridgePanel` | `prd/v6.0_hostBridge-JSSDK.md`, `src/main/crm-bridge/host-*`, `src/main/browser/web-operator-tabs.ts`, `HostBridgePanel.tsx`, `WebOperatorTabs.tsx` |
| **V5.7.11** | **Hermes CLI Hide**：Windows default Gateway 启动隐藏 CMD 窗口；`spawnHermesGatewayProcess` 优先 `pythonw.exe`，fallback `python.exe` + `windowsHide`（非 detached） | `prd/v5.7.11_hermes-cli-hide.md`, `src/main/hermes.ts` |
| **V3.0** | View 收敛、初版 LoginGate + mock Auth/Bootstrap（V3.3 取代） | `modules/auth/`, `main/auth/`, `main/user-config/`, `auth-api.ts`, `user-config-api.ts` |

---

编码时：**先确定改哪一层（Renderer / Preload / Main）→ 查是否已有 IPC → 复用现有模块与类型 → 同步文档与测试。**

%% lat:begin %%
# Before starting work

- Run `lat search` to find sections relevant to your task. Read them to understand the design intent before writing code.
- Run `lat expand` on user prompts to expand any `[[refs]]` — this resolves section names to file locations and provides context.

# Post-task checklist (REQUIRED — do not skip)

After EVERY task, before responding to the user:

- [ ] Update `lat.md/` if you added or changed any functionality, architecture, tests, or behavior
- [ ] Run `lat check` — all wiki links and code refs must pass
- [ ] Do not skip these steps. Do not consider your task done until both are complete.

---

# What is lat.md?

This project uses [lat.md](https://www.npmjs.com/package/lat.md) to maintain a structured knowledge graph of its architecture, design decisions, and test specs in the `lat.md/` directory. It is a set of cross-linked markdown files that describe **what** this project does and **why** — the domain concepts, key design decisions, business logic, and test specifications. Use it to ground your work in the actual architecture rather than guessing.

# Commands

```bash
lat locate "Section Name"      # find a section by name (exact, fuzzy)
lat refs "file#Section"        # find what references a section
lat search "natural language"  # semantic search across all sections
lat expand "user prompt text"  # expand [[refs]] to resolved locations
lat check                      # validate all links and code refs
```

Run `lat --help` when in doubt about available commands or options.

If `lat search` fails because no API key is configured, explain to the user that semantic search requires a key provided via `LAT_LLM_KEY` (direct value), `LAT_LLM_KEY_FILE` (path to key file), or `LAT_LLM_KEY_HELPER` (command that prints the key). Supported key prefixes: `sk-...` (OpenAI) or `vck_...` (Vercel). If the user doesn't want to set it up, use `lat locate` for direct lookups instead.

# Syntax primer

- **Section ids**: `lat.md/path/to/file#Heading#SubHeading` — full form uses project-root-relative path (e.g. `lat.md/tests/search#RAG Replay Tests`). Short form uses bare file name when unique (e.g. `search#RAG Replay Tests`, `cli#search#Indexing`).
- **Wiki links**: `[[target]]` or `[[target|alias]]` — cross-references between sections. Can also reference source code: `[[src/foo.ts#myFunction]]`.
- **Source code links**: Wiki links in `lat.md/` files can reference functions, classes, constants, and methods in TypeScript/JavaScript/Python/Rust/Go/C files. Use the full path: `[[src/config.ts#getConfigDir]]`, `[[src/server.ts#App#listen]]` (class method), `[[lib/utils.py#parse_args]]`, `[[src/lib.rs#Greeter#greet]]` (Rust impl method), `[[src/app.go#Greeter#Greet]]` (Go method), `[[src/app.h#Greeter]]` (C struct). `lat check` validates these exist.
- **Code refs**: `// @lat: [[section-id]]` (JS/TS/Rust/Go/C) or `# @lat: [[section-id]]` (Python) — ties source code to concepts

# Test specs

Key tests can be described as sections in `lat.md/` files (e.g. `tests.md`). Add frontmatter to require that every leaf section is referenced by a `// @lat:` or `# @lat:` comment in test code:

```markdown
---
lat:
  require-code-mention: true
---
# Tests

Authentication and authorization test specifications.

## User login

Verify credential validation and error handling for the login endpoint.

### Rejects expired tokens
Tokens past their expiry timestamp are rejected with 401, even if otherwise valid.

### Handles missing password
Login request without a password field returns 400 with a descriptive error.
```

Every section MUST have a description — at least one sentence explaining what the test verifies and why. Empty sections with just a heading are not acceptable. (This is a specific case of the general leading paragraph rule below.)

Each test in code should reference its spec with exactly one comment placed next to the relevant test — not at the top of the file:

```python
# @lat: [[tests#User login#Rejects expired tokens]]
def test_rejects_expired_tokens():
    ...

# @lat: [[tests#User login#Handles missing password]]
def test_handles_missing_password():
    ...
```

Do not duplicate refs. One `@lat:` comment per spec section, placed at the test that covers it. `lat check` will flag any spec section not covered by a code reference, and any code reference pointing to a nonexistent section.

# Section structure

Every section in `lat.md/` **must** have a leading paragraph — at least one sentence immediately after the heading, before any child headings or other block content. The first paragraph must be ≤250 characters (excluding `[[wiki link]]` content). This paragraph serves as the section's overview and is used in search results, command output, and RAG context — keeping it concise guarantees the section's essence is always captured.

```markdown
# Good Section

Brief overview of what this section documents and why it matters.

More detail can go in subsequent paragraphs, code blocks, or lists.

## Child heading

Details about this child topic.
```

```markdown
# Bad Section

## Child heading

Details about this child topic.
```

The second example is invalid because `Bad Section` has no leading paragraph. `lat check` validates this rule and reports errors for missing or overly long leading paragraphs.
%% lat:end %%

---
> Source: [loudon84/ai-os-desktop](https://github.com/loudon84/ai-os-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
