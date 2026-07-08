## skills

> Public-facing workflow packages for marketing and product agents. Read this file before editing any skill, fixture, or example in this repository.

# Agent instructions — Roots & Fruit skills repo

Public-facing workflow packages for marketing and product agents. Read this file before editing any skill, fixture, or example in this repository.

---

## Scope and audience

- Skills are **domain-agnostic** and **client-agnostic**. They must work for any user who installs them.
- Do **not** embed real client names, project codenames, internal URLs, or engagement-specific context in tracked files.
- Session context from a live client audit must **not** leak into skill edits, fixtures, or examples.

---

## Examples and fixtures

- Use **`example.com`** or other clearly fictional domains in `SKILL.md`, `EXAMPLES.md`, fixtures, and scorecards.
- Real domains belong in **user prompts only**, not in committed artifacts.
- Fixture filenames and content stay generic (e.g. `example-saas`, `example-cloudflare-managed-origin-append`).

---

## Skill structure

| Path | Role |
|------|------|
| `marketing/skills/{skill}/SKILL.md` | Primary agent workflow (installable skill entry) |
| `marketing/skills/{skill}/REFERENCE.md` | Handoff schema, rubrics, templates |
| `marketing/skills/{skill}/REQUIREMENTS.md` | Data prerequisites |
| `marketing/skills/{skill}/EXAMPLES.md` | Sample prompts and output shapes |
| `marketing/skills/{skill}/scripts/` | SSOT parsers and verifiers |
| `marketing/skills/{skill}/AGENTS.md` | Skill-specific maintainer notes (optional pointer here) |

Companion modules (e.g. `CLOUDFLARE-MANAGED.md`) live in the same skill folder and are invoked by the parent `SKILL.md` — they are **not** separate installable skills.

---

## Verification before merge

Each skill documents its regression commands in `SKILL.md`. Run them after substantive changes. Do not claim completion without command output or explicit manual verification steps.

---

## Secrets

Never commit API keys, `.env` values, or customer credentials. Use placeholders in docs and `${VAR}` in MCP examples.

---
> Source: [Roots-and-Fruit/skills](https://github.com/Roots-and-Fruit/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
