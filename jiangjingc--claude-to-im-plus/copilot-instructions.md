## claude-to-im-plus

> When replying to user-reported issues, always include a **self-help prompt** at the end of the reply. Guide users to use their AI coding assistant (Codex / Codex) to diagnose and fix the problem themselves. Example:

# AGENTS.md — Project Guidelines for Codex-to-im-skill

## Replying to GitHub Issues

When replying to user-reported issues, always include a **self-help prompt** at the end of the reply. Guide users to use their AI coding assistant (Codex / Codex) to diagnose and fix the problem themselves. Example:

> **自助排查提示：** 你可以直接在 Codex（或 Codex）中发送以下提示，让 AI 帮你诊断问题：
>
> ```
> 请帮我排查 Codex-to-im 桥接服务的问题。
> 1. 读取 ~/.Codex-to-im/logs/bridge.log 最近 50 行日志
> 2. 读取 ~/.Codex-to-im/config.env 检查配置是否正确
> 3. 运行 bash ~/.Codex/skills/Codex-to-im/scripts/doctor.sh 并分析输出
> 4. 根据日志和配置给出具体的修复建议
> ```

This approach:
- Reduces maintainer burden by enabling users to self-diagnose
- Leverages the fact that users already have an AI coding assistant installed
- Provides actionable next steps rather than just error explanations

---
> Source: [JiangJingC/claude-to-im-plus](https://github.com/JiangJingC/claude-to-im-plus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
