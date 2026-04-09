## echoshell-extension

> 一个podcast transcript，以chrome浏览器插件的形式，能利用几种功能实现播客文字稿的转译：1. 通过monitor屏幕，把视频中的字幕转成文字稿 2. 通过音频转译。用户可以选择转译的方式。如果需要配置什么模型或者服务的API，也显示在web或者工具页面中让用户可以配置自己的服务key

# What we are building

一个podcast transcript，以chrome浏览器插件的形式，能利用几种功能实现播客文字稿的转译：1. 通过monitor屏幕，把视频中的字幕转成文字稿 2. 通过音频转译。用户可以选择转译的方式。如果需要配置什么模型或者服务的API，也显示在web或者工具页面中让用户可以配置自己的服务key

你需要完成产品的界面优化，功能测试
**注意**：需要设计多种测试用例，并在实现后保证工具完全通过测试。

git 仓库：git@github.com:Ericonaldo/echoshell-extension.git
echoshell-extension: A BYOK-based Chrome Extension for high-precision podcast transcription. Capture real-time audio (ASR) and on-screen subtitles (OCR) simultaneously. Supports OpenAI Whisper, Deepgram, and custom endpoints.

already installed in chrome://extensions/?id=mbifkmmjeijejofaljclklepkahimldc

---

# Public Server

You can access the public server using:

```bash
ssh newserver
```

This server is publicly accessible and will host the page.

---

# Your Job

Continue working until **all features are confirmed working and all tests pass**.

Use your **bash and Chrome MCP** to verify that all functions work.

Keep working until **all functions are confirmed working**.

---

# EchoShell (Chrome Extension) 产品需求文档 (PRD)

**EchoShell** 是一款专为播客（Podcast）和在线视频设计的浏览器插件。它通过 **音频抓取 (ASR)** 和 **视觉字幕识别 (OCR)** 双模驱动，并采用 **BYOK (Bring Your Own Key)** 模式，为用户提供高隐私、低成本的实时转译解决方案。

---

## 1. 项目概览
* **项目名称**: EchoShell
* **形态**: Chrome Extension (Manifest V3)
* **核心模式**: BYOK (用户自备 OpenAI/Deepgram/Anthropic 等 API Key)
* **目标**: 实现网页音视频的“所听即所得”与“所见即所得”。

---

## 2. 核心功能模块

### 2.1 双重捕获引擎 (Dual-Capture Engine)
1.  **音频转译模式 (Audio-to-Text)**
    * **原理**: 调用 `chrome.offscreen` API 捕获当前标签页频流（Tab Capture）。
    * **处理**: 将音频流进行 PCM 编码处理，并按设定的时间间隔（如 10s）或静音检测（VAD）进行切片。
    * **分发**: 发送至用户配置的 ASR API（如 OpenAI Whisper）。
2.  **视觉 OCR 模式 (Screen-to-Text)**
    * **原理**: 使用 `getDisplayMedia` 获取视频轨道帧。
    * **逻辑**: 针对 YouTube/Bilibili 等带硬字幕的视频，实时提取画面底部区域进行 OCR 识别。
    * **优化**: 采用帧差异算法，仅当画面发生显著变化时才触发 API 调用，节省 Token。

### 2.2 BYOK 配置中心 (Settings Panel)
用户可自主配置服务商，数据严格存储于 `chrome.storage.local`：
* **ASR Provider**: OpenAI, Deepgram, Groq, Whisper.cpp (Local)。
* **LLM Provider**: 用于文本校阅（GPT-4o-mini, Claude 3.5 Sonnet）。
* **Custom Endpoint**: 支持 OpenAI 兼容格式的自定义中转地址。
* **API Key**: 加密存储，仅在请求时调用。

### 2.3 实时交互 UI (User Interface)
* **Side Panel (侧边栏)**: 实时流式展示转录文本，支持点击时间戳跳转视频进度。
* **Floating Subtitles (悬浮窗)**: 在视频播放器上方叠加一层经过 LLM 优化后的“精修字幕”。
* **History Manager**: 本地存储转译记录，支持一键导出为 `.txt`, `.md`, `.srt` 格式。

---

## 3. 技术架构 (Technical Architecture)

### 3.1 核心组件
* **Background Service Worker**: 处理插件生命周期与消息转发。
* **Offscreen Document**: 突破 V3 限制，用于处理 `AudioContext` 和 `MediaRecorder` 等媒体流操作。
* **Content Scripts**: 负责注入 DOM，抓取特定网站（如 YouTube）的元数据。
* **Side Panel API**: 提供原生侧边栏交互体验。

### 3.2 数据流向
1. `Tab Audio/Video` -> `Offscreen Document` (处理流)
2. `Offscreen` -> `ASR/OCR API` (携带用户私钥)
3. `API Response` -> `Side Panel` (渲染文本)
4. `Text Blocks` -> `LLM` (可选：润色/总结)

---

## 4. 权限需求 (Manifest Permissions)
```json
{
  "permissions": [
    "tabCapture",
    "storage",
    "sidePanel",
    "offscreen",
    "activeTab",
    "scripting"
  ],
  "host_permissions": [
    "[https://api.openai.com/](https://api.openai.com/)*",
    "[https://api.deepgram.com/](https://api.deepgram.com/)*",
    "<all_urls>"
  ]
}

---

# Submit Code

All code should be committed on a **task branch**:

```bash
git commit
```

---

# Merge + Test

```bash
git fetch origin && git merge origin/main
npm test
```

---

# Auto Commit and Merge to Main

After each feature is completed and all tests pass, you must **commit and then merge to `main`**.

Before each commit, update related documentation if new features are added.

Workflow:

1. Sync main branch

```bash
git fetch origin main
```

2. Rebase your task branch

```bash
git rebase origin/main
```

3. If rebase fails, follow the **Conflict Resolution** section below.

4. If rebase succeeds:

```bash
git merge main task-xxx
git push origin main
```

5. Continue with the next task.

6. If **any step fails**, return to **Step 5** in the workflow.

---

# Mark Task Completion

Update `dev-tasks.json` **before cleanup** to prevent losing task status if the process is killed.

---

# Cleanup

After task completion:

- Remove the worktree:

```bash
git worktree remove
```

- Delete the local branch
- Delete the remote task branch
- Restart the development server

---

# Knowledge Capture (Optional)

Record lessons learned in `PROGRESS.md`.

This is optional because task status is already recorded in `dev-tasks.json`, so even if the process is killed, the task state is preserved.

---

# Multi-Instance Parallel Development (Git Worktree)

## Architecture Overview

Multiple Claude Code instances can run **in parallel**, with each instance working in an **independent `git worktree`**.

---

## Parallel Development Workflow

```
┌──────────────────────────────────────────────┐
│              Parallel Development Workflow   │
└──────────────────────────────────────────────┘

   ┌────────────────────┐   ┌────────────────────┐   ┌────────────────────┐
   │      Worker 1      │   │      Worker 2      │   │      Worker 3      │
   │     port:5200      │   │     port:5201      │   │     port:5202      │
   │      worktree      │   │      worktree      │   │      worktree      │
   └────────────────────┘   └────────────────────┘   └────────────────────┘
            │                         │                         │
        ┌────────┐               ┌────────┐               ┌────────┐
        │ data/  │               │ data/  │               │ data/  │
        └────────┘               └────────┘               └────────┘

                     (isolated experimental data)
```

---

# ⚠️ Symlink Is Forbidden

Do **not create symbolic links** for:

```
PROGRESS.md
```

Always edit the main repository file directly using:

```bash
git -C
```

---

# Conflict Resolution

## Handling Rebase Failures

1. If the error is `"unstaged changes"`:

Commit or stash the current modifications first.

```bash
git commit
```

or

```bash
git stash
```

---

2. If there are **merge conflicts**:

Check conflicting files:

```bash
git status
```

Open the conflicting files and understand the changes from both sides.

Manually resolve the conflicts.

Stage the resolved files:

```bash
git add <resolved-files>
```

Continue the rebase:

```bash
git rebase --continue
```

Repeat until the rebase completes.

---

# Handling Test Failures

1. Run tests:

```bash
npm test
```

2. If tests fail:

- Analyze the error messages
- Fix the bugs in the code

3. Run tests again until **all tests pass**.

4. Commit the fix:

```bash
git commit -m "fix: ..."
```

---

# Never Give Up

If a **rebase or test fails**, you **must resolve the issue before continuing**.

Do **not mark the task as failed**.

---

# Knowledge Recording

Whenever you encounter a problem or complete an important change, record it in:

```
PROGRESS.md
```

Include:

- What problem occurred
- How it was solved
- How to avoid it in the future
- **The corresponding Git commit ID**

---

# Important Rule

**Do not make the same mistake twice.**

And remember:

> **After every new feature, update the README and documentation.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ericonaldo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Ericonaldo)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
