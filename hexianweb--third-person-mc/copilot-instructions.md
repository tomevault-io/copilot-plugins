## git-linear-automation

> 当用户需要执行 git 操作时，自动应用本规则；如涉及 Linear issue（如 HX-{number}），则同时应用 Linear MCP 规则。

# 自动化 git 操作与 Linear issue 相关操作标准流程

## 适用范围
- 本规则适用于所有涉及 git 版本管理与 Linear issue 协同的开发流程。
- 适用于团队成员与 AI 助手自动化操作。

## 1. git 操作
- 有 staged 文件时，优先将其作为一次 commit。
- 仅有 unstaged 文件时，需分批 git add，确保每次提交内容与 Linear issue 关联合理，粒度清晰。
- commit message 必须通过 Linear MCP 获取 issue 详细内容，按 commitizen 规范撰写，结尾加上 Linear issue 及其 GitHub issue 编号。
- 自动执行 git add、git commit。

## 2. Linear issue 操作
- 创建 issue 时自动分配 hexianweb，状态 📝 To Do，优先级中等，label 智能分配。
- issue 描述自动引用并提取 docs/PRD.md、docs/TD.md 相关内容，丰富 issue 信息，多用 emoji。
- 支持自动添加评论、子任务、变更状态、调整优先级等。
- 支持 issue 与 GitHub PR/commit 自动关联。

## 3. 操作指令
- 只需告知要提交的文件、关联的 Linear issue（HX-编号）、是否自动生成/更新 issue、评论、子任务等，AI 助手自动完成剩余操作。

---
> Source: [hexianWeb/Third-Person-MC](https://github.com/hexianWeb/Third-Person-MC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
