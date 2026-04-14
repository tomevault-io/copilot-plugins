## mq-studio-site

> **Work is NOT done until real-world validation is complete.**

# CLAUDE.md — MQ Studio

## 🚨 MANDATORY: Real-World E2E Validation Gate (DEC-195)

**Work is NOT done until real-world validation is complete.**

Before declaring any feature, task, or phase complete, you MUST provide at least one of:
- **Screenshots** of live pages (via agent-browser or mcp__chrome-devtools) showing the feature working
- **Passing curl checks** against `http://localhost:3000` (dev) or the production URL
- **Playwright test output** with passing results
- **Explicit manual walkthrough** documented in the session

Absence of evidence = not done. This gate applies to every agent in every session.

> **Why this exists:** Agents repeatedly declared work complete without testing against a live server. This is a structural rule, not a suggestion. (ISSUE-2063, ISSUE-2065, DEC-195)

---

## Project

MQ Studio: content management website for Professor Moura Quayle.
**Tech Stack**: Next.js 15 + Payload CMS v3 + PostgreSQL + Cloudflare R2 + Tailwind CSS 4
**PRD**: `implementation-track/phase-1/PRD-content-management-interface.md` (v2.2, source of truth)

## Git Remotes

```
origin      → mq-studio/mq-studio-dev.git      # Development (safe to push)
production  → mq-studio/mq-studio-site.git     # Production → Railway (test first)
```

## Commands

```bash
npm run dev          # Development server
npm run build        # Production build
npm run lint         # ESLint
npm run test:unit    # Jest unit tests
npm run verify       # lint + test + validate
```

## Design System

- **Colors**: Rice Paper `#FDFCF8`, Moura Teal `#00A8A8`, Scholar Blue `#2C5985`, Living Pink `#E91E63`
- **Typography**: Montserrat (headings), Lora (body)
- **Layout**: Mobile-first responsive grid, 8px radius

## Code Conventions

- TypeScript, 2-space indent, trailing commas
- PascalCase components, camelCase utilities
- Tailwind classes in JSX; minimal custom CSS
- ES modules (`import`/`export`)

## Key Directories

- `app/` — Next.js App Router pages
- `src/collections/` — Payload CMS collection definitions
- `src/globals/` — Payload CMS globals
- `src/access/` — Access control logic
- `src/components/` — Reusable UI components
- `lib/` — Utilities, content services, types

## Related Docs

- `AGENTS.md` — Build/test/style guidelines
- `docs/` — Architecture, implementation decisions
- `background/` — Design references, research

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mq-studio)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mq-studio)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
