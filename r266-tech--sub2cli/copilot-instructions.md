## sub2cli

> 供任何 AI agent (Claude Code / Codex / Cursor 等) 接手本项目时遵循。

# AGENTS.md — sub2cli 项目 agent 协作约定

供任何 AI agent (Claude Code / Codex / Cursor 等) 接手本项目时遵循。

## 🎨 前端 / UI 设计 — 必须走 gemini-3.5-flash

任何涉及视觉方向、布局探索、组件设计、样式重塑的工作, **不要由 agent (Claude/Codex) 凭审美直接出方案**。先调 gemini-3.5-flash via OpenRouter, 让模型出 N 个独立方向, V 选定后再由 agent 落地实现。

- **模型**: `google/gemini-3.5-flash`
- **接入**: OpenRouter API, env `OPENROUTER_API_KEY`
- **调用模式**: 并发 N 个独立 call (temperature ~0.95), **不预先 prescribe 设计方向** — 让 gemini 自主选审美
- **参考脚本**: `/tmp/gen_sub2cli_mockups.py` (并行生成 5 版 mockup + iframe browser index.html)
- **产物位置**: `~/Desktop/sub2cli-mockups/` (V 本机, 浏览器开 `index.html` 横向比对)

### Why

设计审美应由模型多样性 + V 的判断决定, 不应由实施 agent 的训练偏好决定。Agent 负责工程落地, 不负责审美拍板。

### 已选定方向 (2026-05-21)

V 从 5 版 mockup 中选定 **v2**: dark cyberpunk industrial, cyan accent `#00f0ff` + monospace 主导 + glow 状态指示灯 + 终端式 `[SYS]/[INFO]/[READY]` log tag。所有后续前端工作向 v2 的视觉语言对齐。

参考: `~/Desktop/sub2cli-mockups/v2.html` (V 本机, gemini-3.5-flash 原始产出)。

---

## 其他约定

- **CLI 主体不能因 desktop GUI 倒退** — `desktop/` 是 GUI 包装层; 修后端时若涉及 `sub2cli` / `sub2cli-inject` 也要保 CLI 链路。
- **测试**: 改前端先在 `desktop/dist/sub2cli.app/Contents/Resources/ui/` 同步 + 重启 .app 看真数据 (不只看浏览器 mockup)。
- **不签名分发**: V 决定现阶段不掏 Apple Developer 钱; release 全部 unsigned dmg/zip, README 必须配 Gatekeeper bypass 说明。
- **commit 风格**: 跟 git log 现有风格 — `feat(desktop):` / `fix(desktop):` / 中文 commit body 可选。

---
> Source: [r266-tech/sub2cli](https://github.com/r266-tech/sub2cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
