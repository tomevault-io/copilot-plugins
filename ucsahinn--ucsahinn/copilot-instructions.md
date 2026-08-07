## ucsahinn

> This repository powers a public GitHub profile README. It should present

# AGENTS.md

## Repository Purpose

This repository powers a public GitHub profile README. It should present
the profile owner as a security product builder working on MyVuln, VaultPilot,
Codex Chef, AI/Codex workflow systems, public-safe release surfaces, and
Turkish cybersecurity content.

## Working Rules

- Keep the first screen direct: identity, focus, flagship projects, and links.
- Prefer concise, public-safe messaging over long marketing copy.
- Preserve the current product signals: MyVuln, VaultPilot, Codex Chef,
  AI Project Starter, AI Skill Create, Prompt Architect, Codex CLI Operator
  Handbook, and SiberDergi.
- Do not add private repo details, operational secrets, customer data, tokens,
  internal URLs, environment values, or machine-local paths.
- Keep generated badge and card URLs readable and stable.

## Important Files

- `README.md` is the profile page source.
- `assets/profile/*.svg` are public profile cards and may be referenced from
  the README.
- `.github/workflows/snake.yml` publishes the contribution snake to the
  `output` branch only when manually dispatched. Do not add schedule or push
  triggers without explicit approval.
- `.serena/` is local agent state and should not be committed unless explicitly
  requested.

## Verification

- For README edits, check that headings, tables, image URLs, and external links
  still render as valid Markdown/HTML.
- For workflow edits, keep `permissions: contents: write` scoped to the snake
  publishing workflow and avoid adding broad secrets.
- Before commit or push, run `git status --short`, stage explicit files, inspect
  `git diff --cached`, and avoid blanket `git add .`.

## Preferred Tone

Direct, technical, and trust-focused. The profile should make it easy for a
visitor to understand what to star, what to try, and why the work is credible.

---
> Source: [ucsahinn/ucsahinn](https://github.com/ucsahinn/ucsahinn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
