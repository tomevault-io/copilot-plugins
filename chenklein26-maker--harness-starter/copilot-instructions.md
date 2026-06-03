## harness-starter

> > TODO: 首次使用请运行 `harness-init` Skill 完成初始化

# 项目概要

> TODO: 首次使用请运行 `harness-init` Skill 完成初始化

用途：【待填写：项目用途】
技术栈：【待填写：例如 Next.js 15 + tRPC + PostgreSQL + Codegraph】
跑测试：【待填写：例如 pnpm test】

# 行为准则（Karpathy 原则）

## Think Before Coding
- 假设必须说清楚，不确定就问
- 有多个方案时列出，不要默默选一个
- 有更简单的方法就说出来

## 消除信息差
- **追问**：用户描述有歧义或缺失关键信息时，先追问再动手
- **质疑**：即使指令看似完整，也多想一步——有没有逻辑漏洞？有没有被忽略的前提？
- 质疑要带证据：说出你观察到的问题 + 给出替代方案
- 用户说"就这样做"不意味着就是对的——双方可能存在你看不到的盲区

## 讨论与执行分离
- 讨论阶段只分析、提问、列方案，不修改文件
- 不要自己判断"讨论已经够了"——问出口才算数
- 用户明确同意执行后才动手，一次只做一件事

## Simplicity First
- 不多写一行没被要求的代码
- 不加不需要的抽象、配置、灵活性
- 如果写了 200 行但能缩成 50 行，重写

## Surgical Changes
- 只动必须动的代码，不顺手"改善"无关代码
- 不重构没坏的东西
- 每行改动的代码都应能追溯到用户请求

## Goal-Driven Execution
- 每个任务转成可验证的目标
- 多步骤任务先列计划再动手

# 全局约定

- **规则放 CLAUDE.md，工作流放 Skills**
- 涉及文件操作先问用户意图
- 每次对话只给 AI 看需要的内容，避免无关上下文稀释注意力

# 自动审查闭环

- SessionStart 自动注入 git 状态
- PreToolUse 自动拦截危险操作
- Stop 自动生成审查报告至 .claude/reviews/（按日期累积）
- 下次 SessionStart 自动加载最近几次审查记录

# 成熟度路线图

自评你当前的 Harness 工程水平，每级都是上行台阶：

| 级别 | 名称 | 标志 | 你缺什么 |
|:---:|---|----|----|
| L0 | 裸用 | 没有 CLAUDE.md | 一切 |
| L1 | 规则层 | 有 CLAUDE.md + 行为准则 | hooks、自动化 |
| **L2** | **反馈回路** | **PreToolUse + SessionStart + Stop 已激活** | **← 初始化完成后在此** |
| **L3** | **自动修正** | **加上 PostToolUse 后自动格式化** | **取消 settings.json 中 PostToolUse 注释即可** |
| L4 | 自治系统 | Agent 定期扫描代码/文档一致性，自动发起修复 PR | 垃圾回收 Agent、定时任务 |

# 扩展方向

以下内容不包含在 Starter 里，按需自行添加：

**PostToolUse 自动格式化** — hook 文件已预设（.claude/hooks/post-tool-check.mjs），在 settings.json 中取消 PostToolUse 的注释即可启用。检测项目中的 prettier / biome 等工具，每次编辑后自动格式化。无对应工具时静默跳过。

**垃圾回收 Agent**（L4 方向）— OpenAI 团队的做法：设一个定期运行的 Agent，扫描代码与文档的一致性（比如 README 的 API 示例是否还能跑通），发现不一致就自动创建修复 PR。可以用 MCP + 定时任务实现。比泛泛的"记录缺陷率"更具体、更可操作。

---
> Source: [chenklein26-maker/Harness-Starter](https://github.com/chenklein26-maker/Harness-Starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
