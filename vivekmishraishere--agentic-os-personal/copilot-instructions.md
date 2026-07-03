## agentic-os-personal

> This repo ships project-level Cursor skills for web scraping and automation. Agents should discover and use them from `.cursor/skills/`.

# Agentic OS — Agent Guide

This repo ships project-level Cursor skills for web scraping and automation. Agents should discover and use them from `.cursor/skills/`.

## Skill suite

| Skill | When to use |
|-------|-------------|
| [hostinger-deploy](.cursor/skills/hostinger-deploy/SKILL.md) | Deploy or fix Node.js apps on Hostinger via MCP |
| [scraper-intake](.cursor/skills/scraper-intake/SKILL.md) | **Always first** for new scrape/automation work — routes script vs dashboard |
| [scraper-script](.cursor/skills/scraper-script/SKILL.md) | CLI/headless scripts, cron jobs, one-off extracts |
| [scraper-dashboard](.cursor/skills/scraper-dashboard/SKILL.md) | Full browser OS with job queue, live logs, history, export |

## Shared reference kit

Common patterns live in `.cursor/skills/scraper-kit/` (not a skill — linked from the skills above):

- `nodemaven.md` — proxy setup, API flow, Puppeteer wiring
- `env-and-ports.md` — `.env.example` template, port allocation
- `anti-bot.md` — UA hardening, block detection, retry strategy

## Workflow

1. Run **scraper-intake** before writing code
2. Confirm NodeMaven (or search alternative)
3. Route to **scraper-script** or **scraper-dashboard**
4. Only create/update `.env.example` — never read the user's `.env`

---
> Source: [vivekmishraishere/agentic-os-personal](https://github.com/vivekmishraishere/agentic-os-personal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->
