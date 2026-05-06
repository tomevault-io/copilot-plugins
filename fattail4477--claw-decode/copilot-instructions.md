## claw-decode

> **Claw Decode = Claude Code 的 X 光片。**

# CLAUDE.md — Project Context

## 项目定位

**Claw Decode = Claude Code 的 X 光片。**

一个网页体验，让任何人（不只是开发者）在 30 秒内看完 Claude Code 内部最惊人的发现。

一句话："Anthropic 不想让你看到的，都在这一页里。"

**目标：可传播的体验，不是文档。** 视觉冲击力 > 文字深度。

## 项目信息

- **Repo:** https://github.com/fattail4477/claw-decode
- **Author:** Avery Chai (@chatoliciuh)
- **GitHub Pages:** 需要做，是最高优先级

## 泄露源码位置

`~/Desktop/claw-decode-sources/hangsman/src/`

关键文件：
- `src/constants/prompts.ts` — 主系统提示词 (914行)
- `src/tools/*/prompt.ts` — 各工具提示词
- `src/buddy/sprites.ts` — ASCII 宠物动画帧（18种生物）
- `src/buddy/types.ts` — 宠物稀有度系统
- `src/buddy/prompt.ts` — 宠物 prompt
- `src/tasks/DreamTask/DreamTask.ts` — 做梦模式
- `src/services/autoDream/consolidationPrompt.ts` — 记忆整合提示词
- `src/utils/undercover.ts` — 卧底模式
- `src/constants/cyberRiskInstruction.ts` — 安全指令

## 已完成

- ✅ README.md — "10 Biggest Secrets" 格式
- ✅ 完整系统提示词重建 (prompts/FULL_SYSTEM_PROMPT.md)
- ✅ 速查表 (CHEATSHEET.md)
- ✅ 7篇隐藏功能分析 (hidden-features/)
- ✅ 40个工具定义 (tool-definitions/)
- ✅ 架构图 (architecture/)
- ✅ 实操指南 (guides/steal-these-patterns.md)
- ✅ Logo + GitHub Sponsors

## 最高优先级：GitHub Pages 网站

在 `/docs` 文件夹做一个单页网站：

**设计要求：**
- 暗黑主题，视觉冲击力强
- 打开 3 秒内要 "哇"
- ASCII 宠物 sprite 要能动画播放（从 sprites.ts 提取）
- 10 大秘密做成卡片，带动画
- 内部版 vs 外部版对比区域
- 43 个工具做成可搜索列表
- 好的 Open Graph meta tags（分享时好看）
- 用原生 HTML/CSS/JS，不要框架
- 适配手机

**内容来源：**
- README.md 的 10 Biggest Secrets
- CHEATSHEET.md 的工具列表
- hidden-features/ 的详细分析
- prompts/FULL_SYSTEM_PROMPT.md 的 prompt 内容
- 源码 sprites.ts 的 ASCII art

## 其他待做

- [ ] usable/ — 可直接用在 Claude Code 里的资产（CLAUDE.md 模板、agent 配置等）
- [ ] comparisons/ — vs Cursor/Copilot/Cline 架构对比
- [ ] 更多工具的完整 prompt 原文

## 规则

- 不要直接复制源码文件到 repo
- 小片段引用（<10行）用于分析是 OK 的（fair use）
- 每次改完都要 commit + push：
  ```
  git add -A && git commit -m "描述" && git push
  ```

---
> Source: [fattail4477/claw-decode](https://github.com/fattail4477/claw-decode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
