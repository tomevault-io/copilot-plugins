## kidschat-admin

> > **Available Stack:** `/home/services/.claude/available-stack.md` — shared VPS services (endpoints, env var patterns, API docs).

# KidAI — Codebase Orientation

> **Available Stack:** `/home/services/.claude/available-stack.md` — shared VPS services (endpoints, env var patterns, API docs).

KidAI is the **parents admin panel** for a kids AI chat service built on LibreChat. The admin
(this repo) handles parent accounts, child quotas, billing controls, and email alerts.
The LibreChat chat frontend is a **separate service**, still running on Railway.

## Stack

| Layer | Technology |
|---|---|
| Framework | Next.js (App Router, TypeScript) |
| Database | MongoDB 7 |
| Auth | NextAuth.js |
| Email | Resend |
| Image search | image-search-mcp sidecar (custom MCP server) |

## Deployment

**This repo (kidai-admin)** migrated to Hetzner VPS on 2026-04-25.
**LibreChat** deliberately stays on Railway — the admin manages it via Railway API.

| Service | Host | Detail |
|---|---|---|
| `kidai-admin` | Hetzner VPS | Port 3002, Docker container |
| `kidai-mongo` | Hetzner VPS | MongoDB 7 Docker container |
| `image-search-mcp` | Hetzner VPS | Port 8080, Docker sidecar |
| **LibreChat** | **Railway** | Managed via Railway API — `RAILWAY_PROJECT_ID`, `RAILWAY_ENVIRONMENT_ID`, `RAILWAY_SERVICE_LIBRECHAT_ID` are intentional and required |

**Compose file:** `/home/services/hetzner-vps/docker-compose.yml`

## Key Integration: Railway API for LibreChat

The admin uses `RAILWAY_API_TOKEN` + `RAILWAY_PROJECT_ID` + `RAILWAY_SERVICE_LIBRECHAT_ID` to
start/stop LibreChat on demand (quota enforcement, cost control). These Railway env vars are
correct and necessary — LibreChat is not planned for Hetzner migration.

Do not remove or question the Railway env vars in this project. They are load-bearing.

## Auto-deploy

GitHub Actions → SSH → `docker compose up --build -d`

## GSD Tools

Planning files in `.planning/`. GSD tools at `/data/home/.claude/get-shit-done/bin/gsd-tools.cjs`.

## GSD Workflow

- After every plan execution completes, run `/gsd-verify-work` before reporting done — do not wait for a dashboard trigger.

---

## Verbosity Contract

These rules apply to every terminal session in this project. They reduce what Claude says in the terminal so the tmux pane stays readable.

1. **Skip CONTEXT.md interrogation when CONTEXT.md already exists.** If `.planning/phases/{phase}/{phase}-CONTEXT.md` is present, do not re-interview the user about the phase — proceed directly to planning.
2. **Name the phase in plain English in the first line of the session report.** Instead of "I will now begin Phase 56", write "Starting CLI Verbosity Contract + Portfolio Feed work." One line, present tense, specific.
3. **Don't repeat what the user just said.** If the user said "plan phase 56", do not echo back "You asked me to plan phase 56." Begin the work.
4. **Prefer one-line status updates.** Instead of a paragraph explaining what you are about to do, emit a single line: "Reading roadmap." "Writing plan 01." "Done." Reserve multi-line output for actual results (lists of tasks, file paths, errors).
5. **Active voice, present tense.** Write "Creating feedStore.js" not "feedStore.js will be created" and not "I am in the process of creating feedStore.js".

<!-- GSD:non-programmer-contract-start source:templates/claude-md.md -->
## Non-Programmer Contract

Claude must never ask the user to perform a programmer action that Claude can do itself. Technical decisions are made by Claude using its own judgment, documented in the session report, and reversible by the user in plain English.

| Forbidden | Replacement |
|-----------|-------------|
| Asking user to open/view/read code | Read it yourself; summarise findings in plain English |
| Asking user to paste git diffs or logs | Read them yourself with `git diff`, `git log`, or file reads |
| Asking user to edit a config/.env/any file | Edit it yourself; use the Global Env Editor (Dashboard) if credentials are missing |
| Asking user to run a terminal command | Run it yourself |
| "Deploy started, check back in a few minutes" | Run the deploy, wait for it, verify it's live, then ping the user |
| Asking user to run the tests | Run them yourself; only report after they pass (or after a real failure needing a decision) |
| Asking user a technical architecture decision in jargon | Decide yourself; state the decision in plain English; offer to change course |
| Asking user to review code before commit | Commit yourself after verify-work passes |
| "You'll need to do X manually after this finishes" | Don't finish until X is done, or add X to the plan |
| "I'll leave this for you to configure" | Configure with a sensible default; document in the session report |
| Technical disambiguation questions mid-plan | Use CLAUDE.md defaults; only escalate if truly stuck, framed in plain English |
| Asking user to paste an API key in the terminal | Use the Global Env Editor panel (Dashboard) |
<!-- GSD:non-programmer-contract-end -->

---
> Source: [mirkanu/kidschat-admin](https://github.com/mirkanu/kidschat-admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
