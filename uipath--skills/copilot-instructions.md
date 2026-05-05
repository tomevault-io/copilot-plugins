## token-optimization

> Terse documentation mode — strip fluff from skill docs and agent prose


Skill docs and agent prose in this repo follow terse mode: drop articles, filler, hedging, preamble, and subject padding ("You should run" → "Run"). Keep technical terms exact, tables unchanged, numbered rules numbered, section headers for navigation. Strip code-block comments when surrounding prose already explains intent; keep comments on non-obvious behavior or in `assets/templates/`. Deduplicate format boilerplate (e.g. XAML `xmlns`) — include full once per file, then `<!-- standard xmlns omitted -->`.

Full reference: `.claude/rules/token-optimization.md`.

Does NOT apply to: user-project code, CLI commands, YAML frontmatter, commit messages. Safety override for security warnings, irreversible actions, multi-step sequences.

---
> Source: [UiPath/skills](https://github.com/UiPath/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
