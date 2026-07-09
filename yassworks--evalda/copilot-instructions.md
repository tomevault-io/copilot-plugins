## evalda

> - Keep code clean and lightly documented with minimal comments.

# Guidelines

- Keep code clean and lightly documented with minimal comments.
- Follow existing conventions and patterns in the codebase.
- Do not reimplement components or utilities that already exist.

Don't assume. Don't hide confusion. Surface tradeoffs.

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.
- If you write 200 lines and it could be 50, rewrite it.

# Git Commits & PR Titles

All git commit messages and PR titles must follow "Conventional Commits" format:

```
<type>: <description>
```

Common types: `feat`, `fix`, `refactor`, `chore`, `docs`, `test`, `perf`, `ci`.

# Backend

- Use `uv` not `python`.
- Always run tests after every major change using `uv run pytest -q` in the relevant directory.

# Frontend

- Use shadcn UI for all frontend components.
- Follow the established layout, spacing, and styling patterns.
- Do NOT put em dahses in the UI in the middle of text anywhere, use ", " instead.

# Infra

- Everything is on AWS.
- We use Packer + Ansible to build the base AMI for all servers.
- We use Terraform to provision all infrastructure.

---
> Source: [YassWorks/Evalda](https://github.com/YassWorks/Evalda) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
