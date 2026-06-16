## legends-github

> Legends GitHub exposes GitHub repository optimization workflows as Gemini CLI agent skills.

# Legends GitHub

Legends GitHub exposes GitHub repository optimization workflows as Gemini CLI agent skills.

Use the bundled skills from `skills/`:

- `github-audit`
- `github-legal`
- `github-community`
- `github-release`
- `github-seo`
- `github-meta`
- `github-readme`
- `github-empire`

Run these workflows from the root of the repository you want to optimize. The skills use local source files, git history, GitHub CLI metadata, and optional DataForSEO/KIE.ai credentials to make recommendations.

Follow the GARE pattern in each workflow: gather data, analyze the current state, recommend changes, and execute only after the user approves live GitHub or filesystem mutations.

---
> Source: [avalonreset/legends-github](https://github.com/avalonreset/legends-github) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
