## mkhlab

> Arabic-first OpenClaw plugin — 60 skills, 2 messaging channels, 3 programmatic tools.

# مخلب (Mkhlab) — Project Instructions

## What is this?
Arabic-first OpenClaw plugin — 60 skills, 2 messaging channels, 3 programmatic tools.
Plugin architecture (NOT a fork) following the openclaw-china pattern.

## Structure
- `skills/*/SKILL.md` — OpenClaw skill definitions (YAML frontmatter + markdown instructions)
- `extensions/mkhlab-core/` — Plugin entry with registered tools (greeting, arabizi, hijri-today)
- `extensions/whatsapp-channel/` — WhatsApp Business Cloud API channel (scoped service bot)
- `extensions/telegram-channel/` — Telegram Bot API channel (full access)
- `SOUL.md` — Arabic persona and behavior rules
- `IDENTITY.md` — مخلب branding (name, emoji, theme)

## Skill format
All skills use OpenClaw's SKILL.md format:
```yaml
---
name: skill-name
description: "Arabic description — when to trigger"
metadata: {"openclaw": {"emoji": "X", "requires": {"bins": ["tool"]}}}
---
```
- `metadata` MUST be single-line JSON (not nested YAML)
- Descriptions should be in Arabic with English after the dash
- Skills that need CLI tools use `requires.bins`, API keys use `requires.env`

## Languages
- Skill descriptions and instructions: Arabic
- Code (TypeScript): English
- README: Bilingual (Arabic + English)
- Comments in code: English

## Key APIs used
- Aladhan API: prayer times, Hijri dates (no key needed)
- AlQuran Cloud API: Quran search (no key needed)
- Hadith API (fawazahmed0): Hadith search (no key needed)
- WhatsApp Cloud API: messaging (needs Meta Business credentials)
- Telegram Bot API: messaging (needs BotFather token)
- Wathq API: commercial registration, legal docs (key needed)
- ZATCA Fatoorah: e-invoicing (sandbox free)
- National Address API: address lookup (key needed)
- Open-Meteo: weather (free, no key)
- data.gov.sa: open data (free, no key)
- Salla API: ecommerce (OAuth2)
- Lean Technologies: open banking (key needed)
- Unifonic: SMS/WhatsApp CPaaS (key needed)

## CLI tools (installed via pipx from ~/Projects/)
arabench, khalas, sarih, bidi-guard, qalam, artok, majal, safha, raqeeb

## Testing
- `openclaw skills list` — verify all skills discovered
- `openclaw` then `/prayer-times` — test individual skills
- Skills with `requires.bins` show "needs setup" if the binary isn't in PATH

---
> Source: [Moshe-ship/mkhlab](https://github.com/Moshe-ship/mkhlab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
