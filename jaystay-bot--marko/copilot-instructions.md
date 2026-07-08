## marko

> MARKO is a local Python-based growth engine.

# CLAUDE.md — MARKO ENGINE ARCHITECTURE

## SYSTEM OVERVIEW

MARKO is a local Python-based growth engine.

It consists of:

- CLI interface (main.py)
- Command logic (commands.py)
- Dashboard UI (dashboard.py)
- JSON data storage

---

## CORE FILES

- cli.py → CLI entry point
- main.py → Vercel/WSGI entrypoint (exposes Flask `app`; no CLI logic)
- commands.py → MARKO command logic
- dashboard.py → Flask UI
- campaigns.json → campaign tracking
- leads.json → lead storage
- config.json → settings
- marko_log.json → send logs

---

## COMMAND SYSTEM

Commands must include:

- run → create campaign
- add_lead → add new lead
- send → generate outreach batch
- log → track sends
- analyze → assign verdict (SCALE / HOLD / PIVOT / KILL / BLOCKED)
- report → display campaign data

---

## DATA RULES

All commands must read/write the SAME JSON structure.

Campaign fields must include:
- name
- project
- status
- sends
- open_rate
- replies
- signups
- verdict
- last_action
- next

---

## DASHBOARD

- Flask-based
- Reads JSON files
- Buttons trigger CLI commands
- Runs locally only

---

## SYSTEM GOAL

Create a minimal working system that:

1. Creates campaigns
2. Generates outreach
3. Tracks sends
4. Analyzes performance
5. Displays results

---

## CONSTRAINTS

- No external APIs
- No paid tools
- No automation of outreach
- No unnecessary features

---

## FINAL RULE

Build only what is required for the system to run end-to-end.

Do not expand scope.


CLAUDE.md has been created.

Proceed with the build using:
- CLAUDE.md for architecture
- CLAUDER.md for execution rules

Do not expand scope.

---
> Source: [jaystay-bot/MARKO](https://github.com/jaystay-bot/MARKO) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
