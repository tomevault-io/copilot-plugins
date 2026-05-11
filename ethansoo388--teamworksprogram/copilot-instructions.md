## teamworksprogram

> Do NOT make changes to static HTML mockup files (e.g. `jess/*.html`, or any `.html` files in the mockup directories).

# Project Rules

## Static HTML Mockups

Do NOT make changes to static HTML mockup files (e.g. `jess/*.html`, or any `.html` files in the mockup directories).

All code changes go into the React/TypeScript codebase only (`src/`, `scripts/`, etc.).

**Exception:** Only touch static HTML files when the user explicitly instructs a new mockup.

## Zip File Exports

When creating a zip of the codebase, always exclude:
- `public/` (regenerated on build)
- `node_modules/`

The Bash tool's working directory persists between commands. Always verify `pwd` is the worktree root before zipping, then use `.` as the source:

```
# 1. Verify cwd is the worktree root
pwd
# Should be: /Users/ethansoo/Code/teamworks/Teamworksprogram/.claude/worktrees/sad-keller

# 2. Zip using relative paths
zip -r ~/Downloads/Claude<NNN>_<what-changed>.zip . --exclude "public/*" --exclude "node_modules/*" --exclude ".git/*"
```

---
> Source: [ethansoo388/Teamworksprogram](https://github.com/ethansoo388/Teamworksprogram) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
