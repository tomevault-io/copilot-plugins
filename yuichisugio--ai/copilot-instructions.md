## ai

> - Always respond in Japanese

# Rules

- Always respond in Japanese
- Do NOT invent bugs; if evidence is weak, say so and skip.
- Prefer the smallest safe fix; avoid refactors and unrelated cleanup.
- Avoid overconfident root-cause claims; separate “observed” vs “suspected.”
- Reuse existing repo tooling and patterns.
- Break down non-trivial work into sub-agents by phase and perspective to keep the main context clean.
- Assign one focused concern per sub-agent, such as requirements clarification, code search, root-cause analysis, implementation options, test design, or regression review.
- Prefer parallel sub-agents for independent viewpoints, then synthesize their outputs and resolve conflicts in the main thread.
- Always read the software documentation for the tools you'll be using before executing tasks, and actively involve your subagents.

---
> Source: [yuichisugio/ai](https://github.com/yuichisugio/ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
