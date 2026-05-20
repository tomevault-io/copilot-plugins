## windsurf-rules

> When generating windsurf rules, stored in .windsurf/rules


Do not modify Windsurf rule files unless the user explicitly asks for rule changes or a PR that updates rules.

When the user only asks for rule suggestions, return the proposed rule content in chat, wrapped in code blocks. Escape the ` character inside those code blocks.

When the user explicitly asks for a PR or direct rule cleanup, edit `.windsurf/rules/*.md` directly and keep each rule focused on one concern.

---
> Source: [architects-toolkit/SmartHopper](https://github.com/architects-toolkit/SmartHopper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
