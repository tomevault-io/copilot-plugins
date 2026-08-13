## iflow-desktop

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

iFlow Desktop — 基于 Tauri 2.0 + Vue 3 + Rust + Naive UI 的 AI 编程助手桌面应用。当前版本 1.8.1，使用智谱 AI (GLM 系列模型) 作为后端。内嵌了一个完整的 Phaser 3 水墨风格无尽地牢 RPG 游戏模块。

## 常用命令

```bash
npm run tauri dev          # 启动开发环境（Vite 前端 + Tauri 后端热更新）
npm run tauri build        # 构建生产版本，产物在 src-tauri/target/release/bundle/
npm run test               # 运行单元测试 (vitest, 配置文件 vite.config.test.ts)
npm run test:watch         # 监听模式运行测试
npm run lint               # ESLint 代码检查（自动修复）
npm run format             # Prettier 代码格式化
npm run build              # 仅构建前端（vue-tsc 类型检查 + vite build）
```

运行单个测试文件：`npx vitest run src/__tests__/chatStore.test.ts --config vite.config.test.ts`

## 架构

### 分层结构与数据流

```
Vue Components (UI) → Pinia Stores (状态) → Utils/API (工具) → Tauri Commands (Rust后端)
```

**核心规则：不允许跨层调用。**
- 组件不能直接调用 `invoke()`，必须通过 `utils/api/` 封装层
- 组件不能直接操作 localStorage，必须通过 Store
- Store 中不写复杂业务逻辑，提取到 utils 或 composables

### UI 规范

- **Naive UI 默认主题**：不使用自定义主题覆盖，直接使用 Naive UI 默认风格
- **零自定义 CSS**：布局全部用 `NLayout`/`NSpace`/`NGrid`/`NCard`，不用 flex/position 手写
- **组件优先**：能用 Naive UI 组件实现的绝不手写 HTML+CSS
- **例外**：MessageList 的 markdown 代码块样式（`:deep(.code-block)` 等）是唯一保留自定义 CSS 的地方

### 路径别名 (vite.config.ts)

`@` → `src/`, `@components` → `src/components/`, `@stores` → `src/stores/`, `@utils` → `src/utils/`, `@types` → `src/types/`

### 前端核心模块 (src/)

- **stores/**: Pinia 状态管理 — `chatStore.ts` (对话/消息/流式响应), `fileStore.ts` (文件浏览), `quickToolsStore.ts` (快捷工具/代码片段/笔记), `gameStore.ts` (游戏进度/装备/背包/商店)
- **components/**: Vue 组件，全部使用 `<script setup lang="ts">` Composition API
- **utils/api/**: Tauri 命令统一封装层 — `zhipu.ts` (智谱AI), `conversation.ts` (对话持久化), `totp.ts` (TOTP密钥), `game.ts` (游戏存档), `tauri.ts` (环境检测)
- **utils/message.ts**: Naive UI 消息提示封装（showSuccess/showError/showWarning/confirmDelete），禁止使用 alert()
- **utils/tokenUtils.ts**: Token 估算与上下文压缩
- **utils/errorHandler.ts**: 全局错误处理
- **constants/**: 应用常量 — `APP_CONSTANTS`, `STORAGE_KEYS`, `DEFAULTS`, `DEFAULT_ROLES`, `DEFAULT_MODEL_LIST`
- **types/**: 全局 TypeScript 类型定义 — Message, Conversation, AppConfig, CustomRole 等
- **composables/**: 可复用的组合式函数 — useChatHandler, useApiKey, useConversationExport, useQuickTools, useKeyboardShortcuts, useMarkdown
- **game/**: Phaser 3 游戏模块（约 40 个 TS 文件）— entities/ (Player, Monster, Boss), systems/ (Combat, Skill, Collision), scenes/ (EndlessDungeonScene), data/ (boss/monster 定义), mechanics/ (战斗/装备/商店/背包/成就规则)

### 组件懒加载

`SettingsPanel`、`StatsPanel`、`QuickToolsPanel`、`BossGame` 通过 `defineAsyncComponent` 延迟加载，骨架屏作为 fallback。

### Rust 后端 (src-tauri/src/)

- **commands/zhipu.rs**: AI 调用/流式响应，使用 Tauri `manage()` + `State` 管理客户端状态 (`ZhipuState { client: Mutex<Option<ZhipuAiClient>> }`)
- **commands/conversation.rs**: 对话持久化，使用强类型 `Conversation`/`ConversationMessage` 结构体
- **commands/totp.rs**: TOTP 密钥管理
- **commands/game.rs**: 游戏存档 load/save/delete
- **zhipu_ai.rs**: 智谱 AI HTTP 客户端 (`https://open.bigmodel.cn/api/paas/v4`)，SSE 流式解析，全局模型缓存 (`once_cell`)
- **config.rs**: 应用配置读写，统一路径逻辑（支持 `IFLOW_CONFIG_DIR` 环境变量）
- **logging.rs**: 基于 tracing 的日志系统（按日滚动文件，console warn+），自动清理旧日志
- **lib.rs**: 应用入口，注册 24 个命令和 `ZhipuState`，加载 opener/dialog/fs 插件
- **main.rs**: 仅初始化日志并调用 `run()`

全局状态使用 `tauri::Manager::manage()` 注册、`tauri::State` 获取，禁止使用 `static` 全局变量。
所有 Tauri 命令返回 `Result<T, String>`，使用 `tracing::info!/error!` 记录日志，禁止 `unwrap()`。

### Tauri 插件

`tauri-plugin-opener` (URL打开), `tauri-plugin-dialog` (系统对话框), `tauri-plugin-fs` (文件系统访问)

### 流式响应模式

Rust 后端解析智谱 AI 的 SSE 流，通过 Tauri `Emitter` 向前端发送三种事件：
- `ai-chunk`: 逐 token 的流式文本片段
- `ai-complete`: 完整响应结束信号
- `ai-error`: 错误信息

前端 `useChatHandler` composable 监听这些事件并更新 Pinia store。

## 开发规范

### 前端

- TypeScript 严格模式，避免 `any`（ESLint 配置为 warn）
- 组件使用 `storeToRefs` 解构 store 保持响应式
- 消息提示统一使用 `utils/message.ts`，禁止 `alert()`
- Naive UI 组件按需引入，优先使用 Naive UI 而非自定义元素
- 组件命名 PascalCase，文件名与组件名一致
- 布局使用 Naive UI 的 `NSpace`/`NGrid`/`NLayout`，禁止手写 flex/position CSS

### Rust

- Tauri 命令必须 `#[tauri::command]` 属性 + `Result<T, String>` 返回类型
- 全局状态用 `tauri::Manager::manage()` 注册、`tauri::State` 获取，禁止 `static` 全局变量
- 错误处理用 `Result` 和 `?` 操作符，禁止 `unwrap()`
- 使用 `tracing` 宏记录日志，不用 `println!`
- 数据结构使用强类型 `struct`，禁止 `serde_json::Value`

### 提交规范

Conventional Commits: `feat(chat): 描述`, `fix(api): 描述`, `refactor(ui): 描述`

### 环境要求

Node.js >= 18, Rust >= 1.70, Windows 需要 Visual Studio Build Tools (MSVC)

### 测试配置

- 测试框架：vitest + happy-dom 环境，globals: true（无需导入 describe/it/expect）
- 测试文件位于 `src/__tests__/*.test.ts`，排除 `.worktrees/`
- 45 个测试文件，覆盖游戏战斗/技能/装备/进度、chatStore、tokenUtils 等

### 构建配置要点

- 生产构建使用 esbuild 压缩（比 terser 快 20-40 倍），移除 console.log/console.info
- 代码分割：vue-vendor, naive-ui, markdown, tauri, otpauth, phaser 分 chunk
- 开发服务器固定端口 1420（Tauri 要求），HMR 在 1421
- Gzip 压缩阈值 10KB（需设置 `VITE_ENABLE_PRECOMPRESSED_ASSETS=true`）
- Rust release: LTO + strip + abort-on-panic + codegen-units=1

### CI/CD

GitHub Actions (`.github/workflows/build.yml`)：推送 `v*` 标签或手动触发，构建 Windows (MSI + NSIS) 和 macOS (x86_64 + aarch64 DMG) 版本。

---
> Source: [XY-JX/iflow-desktop](https://github.com/XY-JX/iflow-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
