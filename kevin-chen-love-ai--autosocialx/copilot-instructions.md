## autosocialx

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **架构文档**: 核心模块设计、接口定义、目录结构、数据表 DDL 统一维护在 [`ARCHITECTURE.md`](./ARCHITECTURE.md)。
> **维护规范**: 每次架构模块新增、接口变更、分层调整时，**必须同步更新 `ARCHITECTURE.md` 的对应章节及版本变更记录表**。

---

## 常用命令

所有命令在 `apps/desktop/` 下执行（或从根目录用 `pnpm -F desktop <script>`）：

```bash
# 开发（Vite dev server + Electron 同时启动）
pnpm dev

# 生产构建（tsc + vite build + electron-builder）
pnpm build

# Lint
pnpm lint

# TypeScript 类型检查（两端分开）
npx tsc -p tsconfig.app.json --noEmit    # Renderer (src/)
npx tsc -p tsconfig.node.json --noEmit   # Electron main (electron/)

# 仅编译 Electron 主进程
pnpm build:electron
```

---

## 架构与分层

详细模块设计见 `ARCHITECTURE.md`。以下是理解代码库必须知道的跨文件架构规律。

### 双 tsconfig 隔离

| 文件 | 覆盖范围 | module | 用途 |
|------|---------|--------|------|
| `tsconfig.app.json` | `src/` | ESNext bundler | React Renderer，noEmit，严格 lint |
| `tsconfig.node.json` | `electron/` | ESNext Node | Electron 主进程，输出到 `dist-electron/` |

`tsconfig.app.json` 额外开启 `noUnusedLocals` / `noUnusedParameters` / `erasableSyntaxOnly`，Renderer 代码必须满足。

### IPC 通信模式

Renderer **永远不能**直接调 Node API，全部通过 `window.electronAPI.*`：

```
Renderer (src/)
  └─ window.electronAPI.db.query(sql)       # 由 contextBridge 暴露
        │
        │ ipcRenderer.invoke('db:query')
        ▼
Main (electron/)
  └─ ipcMain.handle('db:query', handler)    # 实际执行 SQLite
```

`ElectronAPI` 接口定义在 `electron/preload.ts`，同时也声明了 `Window['electronAPI']` 全局类型，Renderer 可直接使用。

### 设计系统

UI 颜色 token 集中在 `src/components/Icon.tsx` 的 `gv` 对象（`bg / glass / card / ac / cc / pk` 等）。页面组件使用 **inline styles** 而非 Tailwind class，原因是大量颜色为动态计算值。Tailwind 在此项目中主要提供 reset 和 CSS 变量支持，`src/index.css` 中定义了对应的 CSS 变量（`--color-bg` 等）。

### Mock 数据与未来 SQLite 替换

当前所有页面数据来自 `src/data/mock.ts`。P1 阶段将通过 React Query + `window.electronAPI.db.query()` 替换为真实 SQLite 查询，替换点是各 Page 组件顶层的数据引用。

---

## 工程化约束

- **类型安全**: `strict: true`，禁用 `any`，用 `unknown` + 类型守卫替代。
- **状态**: UI 状态 → Zustand；异步请求生命周期 → React Query；业务数据持久化 → SQLite (Main Process)。
- **凭证**: AI Key / 平台 Token 只存 macOS Keychain / Windows Credential Store，Renderer 和 Worker 均不可见明文，只能持有 `CredentialId`（`cred_${string}`）。
- **Worker 隔离**: Playwright 实例跑在独立 `child_process.fork` 子进程，Main 通过 JSON-RPC 2.0 通信，防止内存泄漏污染主进程。

## 可插拔模块约束

以下四类业务**禁止硬编码**，必须通过各自接口实现：

1. **Platform Adapter** (`IPlatformAdapter`) — 每个社交平台独立实现，位于 `packages/plugins/src/adapters/`
2. **LLM Gateway** (`ILLMGateway`) — 统一大模型出口，位于 `packages/ai/src/gateway.ts`，支持 Claude / GPT / Ollama / DeepSeek 热切换
3. **Persona Hub** — 人设以 YAML 文件管理，运行时动态注入 System Prompt，位于 `electron/agents/persona.ts`
4. **Skills Registry** (`ISkillPlugin`) — Tool Call 实现，位于 `packages/plugins/src/skills/`，实现接口后自动注册到 Tool Use Manager

---
> Source: [kevin-chen-love-ai/autosocialX](https://github.com/kevin-chen-love-ai/autosocialX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
