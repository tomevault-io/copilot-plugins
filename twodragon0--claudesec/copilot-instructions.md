## markdown-quality

> Markdown docs — frontmatter, code blocks, links, markdownlint


# Markdown Quality (ClaudeSec)

## Frontmatter

Every Markdown doc must have YAML frontmatter with:

- `title` — short document title
- `description` — one-line summary
- `tags` — list of tags (e.g. `["devsecops", "github"]`)

```yaml
---
title: GitHub Actions Security
description: Secure CI/CD with GitHub Actions
tags: [github, ci-cd, security]
---
```

## Code Blocks

- Always set the **language** (e.g. ` ```bash `, ` ```yaml `).
- Code examples must be **tested and runnable**; avoid pseudocode unless explicitly noted.

## Links & Lint

- Links must be **valid** (no broken references).
- All Markdown must pass **markdownlint** before commit.

---
> Source: [Twodragon0/claudesec](https://github.com/Twodragon0/claudesec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
