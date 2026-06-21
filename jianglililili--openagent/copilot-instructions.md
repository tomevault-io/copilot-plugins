## openagent

> OpenAgent 是一个 Tauri 1.6 桌面应用，前端使用 React/Vite/TypeScript，后端使用 Rust。它在共享聊天界面中承载多个编码智能体，并将非 mock 智能体通过 `agent-bridge/legacy-stdio.cjs` 路由到安装在应用数据目录 `sdk-agents/<agent_id>` 下的 SDK 包。

# AGENTS.md

## 项目概览

OpenAgent 是一个 Tauri 1.6 桌面应用，前端使用 React/Vite/TypeScript，后端使用 Rust。它在共享聊天界面中承载多个编码智能体，并将非 mock 智能体通过 `agent-bridge/legacy-stdio.cjs` 路由到安装在应用数据目录 `sdk-agents/<agent_id>` 下的 SDK 包。

核心区域：

- `src/`：React UI、Zustand 聊天状态、供应商导入、多智能体队列。
- `src-tauri/src/`：Tauri 命令、进程生命周期、AppData 文件访问。
- `agent-bridge/`：JSON-RPC 兼容桥接层和 SDK 适配器。
- `tests/`：前后端行为契约测试。
- `scratch/`：验证脚本和隔离的 E2E 测试夹具。不要把 scratch 夹具当成生产配置。

## 启动流程

1. 先阅读本文件，再查看 `feature_list.json`、`progress.md` 和 `session-handoff.md`。
2. 编辑前检查当前工作区状态。这个项目在某些环境里可能不是 git 仓库。
3. 搜索优先使用 `rg` / `rg --files`。读取包含中文的文件时使用 `Get-Content -Encoding UTF8 -Raw`。
4. 修改前先定位负责该行为的最小项目区域。
5. 区分运行时验证和静态证据。正在运行的应用或 AppData 配置，可能要重建/重启后才会反映源码改动。

## 编码规则

- 在 Windows/PowerShell 下读取中文源码、Markdown 和 JSON 时，使用 `Get-Content -Encoding UTF8 -Raw`。
- 修改包含中文字符串的文件时保持 UTF-8。
- 如果涉及中文的补丁失败，先用 UTF-8 重新读取目标片段，再使用更小上下文打补丁。
- 修改中文后，搜索乱码标记：`鍔|瀹|澶|辫|锛|�`。
- 不要把已经乱码的终端输出复制回源码。

## 范围规则

- 手动修改源码/文档时使用 `apply_patch`。
- 除非任务明确要求修改本地应用状态，否则不要把测试夹具写入真实 `%APPDATA%\com.openagent.client`。
- 浏览器 E2E 应通过 `OPENAGENT_E2E_APPDATA` 或 `scratch/e2e-appdata` 使用隔离 AppData。
- 除非用户在当前轮明确允许，否则不要发起真实 Claude/Codex/OpenCode 模型请求。
- 未经明确批准，不要运行 `git reset --hard`、递归删除、配置清空等破坏性命令。
- 保留用户配置。如果必须修改本地 AppData，先备份并报告备份路径。

## 验证命令

开发过程中先跑最小有用检查；声明完成前再跑更完整的验证门。

主要验证门：

```powershell
npm test
npm run build
Set-Location src-tauri; cargo check; cargo test
```

桥接/运行时验证：

```powershell
node scratch\verify-bridge-runtime.cjs
node scratch\browser-ui-agent-e2e.cjs
```

真实 Codex SDK smoke test，只能在用户批准后运行：

```powershell
node scratch\verify-real-codex-bridge.cjs
```

注意事项：

- 在受管沙箱中，`npm test`、`npm run build`、Playwright/Vite 和部分 bridge 脚本可能因 `spawn EPERM` 失败；应使用已批准/提升的路径重跑，不要为了绕过沙箱而改代码。
- `browser-ui-agent-e2e.cjs` 应打印 `scratch/e2e-appdata` 下的 AppData 路径，而不是真实用户 AppData。
- `verify-real-codex-bridge.cjs` 会使用用户的 Codex 登录态，必须保持显式 opt-in。

## 多智能体不变量

- 智能体 B 应通过聊天历史和已清洗的 handoff summary 收到智能体 A 的最终回答。
- 智能体 B 不需要智能体 A 的原始工具调用轨迹。只共享结果、变更文件、验证、失败和下一步。
- bridge prompt 必须包含 workspace root、transcript、previous handoff summary 和 current instruction。
- `agent-bridge` 适配器必须从传入的 SDK root 加载 SDK 包，而不是从全局依赖加载。
- 除非 provider env 或 `OPENAGENT_CODEX_MODEL` 覆盖，Codex 默认使用 `gpt-5.4`。

## 完成定义

报告完成前：

1. 重新阅读相关需求，并列出什么证据可以证明完成。
2. 检查当前源码/配置证据，不要只依赖之前的聊天记录。
3. 运行覆盖本次改动行为的验证命令。
4. 报告精确命令和结果，包括跳过的检查及原因。
5. 当工作影响项目状态或后续恢复时，更新 `progress.md` 和 `session-handoff.md`。

---
> Source: [jianglililili/openagent](https://github.com/jianglililili/openagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-21 -->
