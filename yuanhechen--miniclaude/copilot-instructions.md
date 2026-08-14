## miniclaude

> This file provides guidance when working with code in this repository.

# CLAUDE.md

This file provides guidance when working with code in this repository.

## Project Overview

DeepAnalyze（miniclaude）：CLI AI agent，**TypeScript + ink**（对标 Claude Code 的终端渲染）。连接智谱 BigModel 的 Anthropic 兼容端点（glm-5.x），支持工具调用、流式输出、skill、会话存储、剪贴板图片粘贴、联网搜索/阅读。中文 UI / 中文 prompt。

**为什么 ink**：为同时实现「流式逐字 + 底部状态栏常驻 + 鼠标滚轮翻原生 scrollback 历史」三者兼得 —— 这正是 Claude Code 用 ink 渲染做到的。靠 ink 的 `<Static>`（已完成消息进原生 scrollback）+ 动态区每帧 diff（当前流式 + 状态栏 + 输入）。

## 安装与运行

### 全局安装（用户）
```bash
npm install -g @yuanhechen/miniclaude
miniclaude            # 任意目录启动
```
配置：复制 `config.example.json` → `~/.miniclaude/config.json`，填 `apiKey`（智谱 BigModel key）。

### 开发态
```bash
npm install
npm start             # = tsx src/main.tsx（直接跑源码，免 build）
npm run build         # tsup 打包 → dist/main.mjs
npm install -g .      # 本地全局安装测试
```
`q` 退出，或 Ctrl-C。其它：`npm run probe`（验证端点流式兼容）、`npm run typecheck`（tsc --noEmit）。

硬依赖：`ink` / `react` / `@anthropic-ai/sdk` / `sharp`（图片压缩）/ `ink-text-input` / `ink-spinner` / `gray-matter`（skill frontmatter）/ `turndown`（web_fetch HTML→markdown）/ `tsup`（打包）。运行时开发用 `tsx`，web 工具用 Node 内置 fetch（零依赖）。见 `package.json`。

## 配置（config.json）

开发态读项目根 `config.json`；全局安装态读 `~/.miniclaude/config.json`（`config.ts` 的 `loadConfig` 按此顺序 fallback）。字段见 `config.example.json`：

- `apiBaseUrl`（默认 `https://open.bigmodel.cn/api/anthropic`）
- `apiKey`（智谱 BigModel key，必须）
- `model`（默认 `glm-5.1`）
- `maxTokens` / `contextWindow`（默认 192000）/ `temperature`
- web 工具：`webSearchEngine`（默认 `search_std`）/ `webSearchContentSize`（`medium`）/ `webSearchCount`（10）/ `webResultMaxChars`（30000）/ `webApiKey`（可选，默认复用 `apiKey`）

## Architecture（src/）

### 核心模块
- **config.ts** — 配置常量 + `loadConfig`（项目 config.json → `~/.miniclaude/config.json`）+ `getApiKey()`/`getWebApiKey()` + `MINICLAUDE_HOME` + `SKILL_DIRS` + 符号/主色 `#D97757`。
- **types.ts** — Anthropic messages 类型 + `StreamEvent`（5 事件）+ `CommittedItem`（进 `<Static>` 的视图模型）。
- **store.ts** — `createStore`（CC 风格）+ `useSyncExternalStore`。**核心契约**：`committed` 只增（喂 `<Static>`）；`streamingText`/`phase` 等动态区字段每帧整体替换（ink 行级 diff）。
- **llm.ts** — `callLLMStream` async generator。`@anthropic-ai/sdk` 配 Anthropic 兼容端点（`baseURL` + `authToken` + `signal`）。自己累积 `content_blocks`；yield 5 事件 + timing。
- **tools.ts** — 8 工具（bash/read/write/edit/skill/ask/web_search/web_fetch）+ `compressImage`（sharp）+ `TOOLS_SCHEMAS`。`ask` 异步化（store `askResolver`）。
- **websearch.ts** — `searchWeb`（智谱 `/paas/v4/web_search`，复用 apiKey）+ `readUrl`（纯客户端 fetch + turndown HTML→MD + 重定向护栏，零平台依赖）。
- **executor.ts** — `partitionToolCalls`（连续 safe 合并发批，unsafe 串行）+ `executeBatch` + `buildToolResultBlock`（按 type 分发：bash/image/error/text/web_search/web_fetch）。
- **agent.ts** — `handleSubmit`（命令/图片/对话分发）+ `runTurn`（多轮工具循环，`MAX_TOOL_ROUNDS`，ESC 中断保留 partial）。
- **commands.ts** — 15 斜杠命令 + `parseCommand`/`handleCommand` + welcome logo。
- **skills.ts** — `loadSkills`（扫 `SKILL_DIRS`，gray-matter 解析 frontmatter）+ `getSkillPrompt`（`$ARGUMENTS` 替换）+ `getSkillListing`（注入 system）。
- **session.ts** — `~/.miniclaude/sessions/`；autosave/save/load/list + compressContent（持久化去 base64）。
- **compaction.ts** — `compactMediaMessages`/`compactOldToolResults`。
- **clipboard.ts** — `getClipboardImage`（WSL2 powershell / xclip / wl-paste / macOS osascript / Windows powershell）。

### UI 组件（components/）
- **App.tsx** — `<Static items={committed}>`（已完成进 scrollback）+ 动态区（Spinner/流式/状态栏/Input）+ `useInput` 全局 ESC 中断。
- **Input.tsx** — `ink-text-input` + Tab 命令补全 + Ctrl-V 粘贴剪贴板图片 + ask resolve。
- **MessageList.tsx** — `CommittedItem` 渲染（user `❯` / assistant text / tool `⏺`+`⎿` / system / logo）。
- **StatusBar.tsx** — 底部常驻 context/perf 状态栏。
- **lib/markdown.tsx** — 行内 + 整行 markdown → ink 渲染。
- **lib/format.ts** — fmtSize/fmtDur/gradientHex。

## Key 设计

- **流式 → 状态链路**：SDK `content_block_delta` → `appendText(delta)`（动态区 `streamingText` 累加）→ `done`/`tool_call` 时快照 `commit` 进 `<Static>` + `resetStream()`。
- **`<Static>` 只增**：已 commit 的项永不修改（流式文本 done 时才 commit 定型）。
- **skill progressive disclosure**：`getSkillListing` 注入 system（name+description 摘要），模型按需调 `skill(name,args)` 拉完整 body。
- **web 工具分工**：`web_search` 依赖智谱搜索 API（合理——搜索本质是搜索引擎能力）；`web_fetch` 纯客户端抓取（不绑平台，返回原始 markdown 全文，加工交给模型）。
- **ESM + bundle**：`"type":"module"`，import 带 `.js` 后缀；开发用 `tsx` 直接跑 `.tsx`，发布用 `tsup` bundle 成 `dist/main.mjs`（external 化 sharp 等 native 依赖）。

## Tests / 验证脚本
- `src/scripts/probe.ts` — 验证端点流式兼容（配好 config.json 后 `npm run probe`，看 text 事件流 + done usage/timing）。
- `src/scripts/test-markdown.tsx` — markdown 渲染单测（`npm run test:md`）。
- `src/scripts/test-app.tsx` — App 集成渲染测试（`npm run test:app`）。
- `src/scripts/test-store.ts` — store 单测（`npm run test:store`）。

---
> Source: [yuanhechen/MiniClaude](https://github.com/yuanhechen/MiniClaude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
