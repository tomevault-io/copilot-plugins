## git-attribution

> No AI co-author trailers in commits or PR descriptions


# Git attribution

Do not add a `Co-Authored-By:` trailer naming an AI tool (Cursor, Claude,
Copilot, Codex, ChatGPT, etc.) to any commit message. Do not add a
"Generated with <tool>" footer to any commit message or PR description.

This repo's CI (`no-ai-coauthor` check, required on `main` and `redesign`)
rejects any PR containing such a trailer — write commits as if authored
by the human running this session.

---
> Source: [yuviz-ai/yuviz](https://github.com/yuviz-ai/yuviz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
