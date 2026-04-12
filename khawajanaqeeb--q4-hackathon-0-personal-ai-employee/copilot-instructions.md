## q4-hackathon-0-personal-ai-employee

> This is a **Personal AI Employee** (Digital FTE) — **Platinum Tier** ✅

# Personal AI Employee — Claude Code Instructions

## Project Overview

This is a **Personal AI Employee** (Digital FTE) — **Platinum Tier** ✅
You are an autonomous agent that manages personal and business affairs by reading from
and writing to the Obsidian vault at `AI_Employee_Vault/`.
Platinum adds a Cloud Agent running 24/7 on a VM and a Git-synced vault for
seamless Cloud/Local coordination.

## Vault Location

```
AI_Employee_Vault/
├── Dashboard.md          ← Your real-time status board (read/write — Local only)
├── Company_Handbook.md   ← Your rules of engagement (read only)
├── Business_Goals.md     ← Revenue targets and KPIs (read/write)
├── Inbox/                ← Drop zone: files land here first
├── Needs_Action/         ← Watcher outputs; your primary work queue
├── In_Progress/          ← Claim-by-move protocol (Platinum Tier)
│   ├── cloud/            ← Cloud Agent claims (move files here to claim)
│   └── local/            ← Local Agent claims (move files here to claim)
├── Done/                 ← Completed tasks (move files here)
├── Plans/                ← Your reasoning plans (create here)
├── Logs/                 ← Structured audit log (append here)
├── Pending_Approval/     ← Human-in-the-Loop queue (CLOUD_DRAFT_* + local)
├── Approved/             ← Human-approved actions (orchestrator monitors)
├── Rejected/             ← Rejected actions (archive here)
├── Briefings/            ← CEO briefings (create here)
├── Accounting/           ← Financial data (read/write)
└── Signals/              ← Cloud writes status signals; Local merges into Dashboard
```

## Core Rules (ALWAYS follow)

1. **Read Company_Handbook.md first** before taking any action.
2. **Never delete files** — move them to `/Done/` or `/Rejected/`.
3. **Never send external communications** without a file in `/Approved/`.
4. **Never store credentials** in vault files — use `.env` only.
5. **Log every action** to `Logs/<today_date>.json`.
6. **Update Dashboard.md** after completing any batch of work.
7. **Always create Plan.md** in `/Plans/` before executing multi-step tasks.
8. **Ralph Wiggum**: When activated, loop until Needs_Action/ is empty (output `<promise>TASK_COMPLETE</promise>` when done).

## Workflow

```
Inbox → Watcher → Needs_Action → Claude reads → Plans/ → Pending_Approval/
  → (Human approves) → Approved/ → Orchestrator → Action → Done/
```

## Agent Skills (Slash Commands)

| Command | Description | Tier |
|---------|-------------|------|
| `/process-inbox` | Process all items in Needs_Action | Bronze |
| `/update-dashboard` | Refresh Dashboard.md with current stats | Bronze |
| `/morning-briefing` | Generate CEO briefing with weekly audit | Bronze |
| `/start-watcher` | Start the file system watcher | Bronze |
| `/linkedin-post` | Draft a LinkedIn business post (HITL) | Silver |
| `/approve-pending` | Review and approve/reject pending actions | Silver |
| `/run-orchestrator` | Start the Master Orchestrator | Silver |
| `/social-post` | Draft multi-platform social post (HITL) | Gold |
| `/weekly-audit` | Generate weekly CEO business briefing | Gold |
| `/ralph-loop` | Autonomous multi-step task loop | Gold |
| `/odoo-query` | Query Odoo accounting/ERP system | Gold |
| `/sync-vault` | Git pull/push vault sync + report | Platinum |
| `/cloud-status` | Show Cloud Agent activity + pending drafts | Platinum |

## Python Watchers

```bash
# ── Bronze Tier ──────────────────────────────────────────────────────────────
# Filesystem watcher (no API key needed)
python3 watchers/filesystem_watcher.py --vault AI_Employee_Vault

# Gmail watcher (requires Google API credentials)
python3 watchers/gmail_watcher.py --vault AI_Employee_Vault

# ── Silver Tier ──────────────────────────────────────────────────────────────
# LinkedIn watcher (requires Playwright + credentials in .env)
python3 watchers/linkedin_watcher.py --vault AI_Employee_Vault

# WhatsApp watcher (requires Playwright; run --setup first for QR code)
python3 watchers/whatsapp_watcher.py --vault AI_Employee_Vault --setup
python3 watchers/whatsapp_watcher.py --vault AI_Employee_Vault

# Master Orchestrator (watches /Approved/ + handles scheduling)
python3 orchestrator.py --vault AI_Employee_Vault

# ── Gold Tier ─────────────────────────────────────────────────────────────────
# Twitter/X watcher (run --setup first)
python3 watchers/twitter_watcher.py --vault AI_Employee_Vault --setup
python3 watchers/twitter_watcher.py --vault AI_Employee_Vault

# Facebook watcher (run --setup first)
python3 watchers/facebook_watcher.py --vault AI_Employee_Vault --setup
python3 watchers/facebook_watcher.py --vault AI_Employee_Vault

# Instagram watcher (run --setup first)
python3 watchers/instagram_watcher.py --vault AI_Employee_Vault --setup
python3 watchers/instagram_watcher.py --vault AI_Employee_Vault

# Process watchdog (monitors & restarts all watchers)
python3 scripts/watchdog.py --vault AI_Employee_Vault

# Process watchdog (monitors & restarts all watchers)
python3 scripts/watchdog.py --vault AI_Employee_Vault

# Weekly business audit + CEO briefing
python3 scripts/weekly_audit.py --vault AI_Employee_Vault

# ── Platinum Tier ─────────────────────────────────────────────────────────────
# Cloud Agent (run on Cloud VM with AGENT_MODE=cloud)
AGENT_MODE=cloud python3 scripts/cloud_agent.py --vault AI_Employee_Vault

# Vault Sync daemon (git pull/push every 5 min)
python3 scripts/vault_sync.py --vault AI_Employee_Vault --interval 300

# Single vault sync (for cron or manual trigger)
python3 scripts/vault_sync.py --vault AI_Employee_Vault --once

# Merge cloud signals into Dashboard.md (local only)
python3 scripts/merge_signals.py --vault AI_Employee_Vault

# Full Platinum demo (end-to-end flow)
bash scripts/demo_platinum.sh
```

## MCP Servers (Silver + Gold Tier)

```bash
# Email MCP Server
python3 mcp_servers/email_server.py --test

# Social Media MCP Server (Gold Tier)
python3 mcp_servers/social_media_server.py --test

# Odoo Accounting MCP Server (Gold Tier)
python3 mcp_servers/odoo_server.py --test
```

## Scheduling

```bash
# Set up cron jobs (Linux/WSL2) — Silver tier
bash scripts/setup_cron.sh

# Set up Gold tier cron (adds weekly audit Monday 7am)
bash scripts/setup_cron_gold.sh

# Set up Platinum tier cron (vault sync + signal merge + odoo backup)
bash scripts/setup_cron_platinum.sh

# Start with PM2 (recommended — always-on, auto-restart)
pm2 start scripts/pm2.config.js           # Local: all watchers + vault-sync
pm2 start scripts/cloud_pm2.config.js     # Cloud VM: cloud-agent + gmail + vault-sync
pm2 save && pm2 startup

# Cloud VM setup (Ubuntu 22.04 / Oracle Cloud Free Tier)
bash scripts/setup_cloud.sh

# Odoo Community with HTTPS (cloud VM)
ODOO_DOMAIN=odoo.yourdomain.com bash scripts/odoo_cloud_setup.sh

# WSL2: Start cron service
sudo service cron start
```

## Security

- Never commit `.env` files
- Credentials go in `.env` (already in `.gitignore`)
- All sensitive actions require human approval (HITL)
- Payments > $100 always require approval regardless of automation level
- LinkedIn/WhatsApp/Twitter/Facebook/Instagram sessions stored locally (never synced to git)
- Rate limits: max 10 emails/hr, max 3 social posts/hr, max 3 payments/day

## Gold Tier: Error Recovery

The system now includes:
- `watchers/retry_handler.py` — exponential backoff + circuit breaker + rate limiter
- `scripts/watchdog.py` — process health monitor with auto-restart
- All watchers handle `TransientError` for graceful degradation

## Gold Tier: Ralph Wiggum Loop

The Stop hook (`scripts/ralph_wiggum_hook.py`) enables autonomous persistence:
- Configured in `.claude/settings.local.json`
- Activates when `/ralph-loop` skill is used
- Loops until `Needs_Action/` is empty OR `<promise>TASK_COMPLETE</promise>` is output
- Max 10 iterations as safety guard

## Gold Tier: Odoo Accounting

Odoo Community integration via `mcp_servers/odoo_server.py`:
- Works in mock/demo mode without Odoo installed (DRY_RUN=true)
- Set ODOO_URL, ODOO_DB, ODOO_USERNAME, ODOO_PASSWORD in `.env` to connect
- Docker setup: `docker run -p 8069:8069 odoo:17`

## Platinum Tier: Cloud + Local Architecture

### Cloud/Local Work-Zone Specialisation

| Domain | Cloud Agent | Local Agent |
|--------|-------------|-------------|
| Email triage | Draft reply → Pending_Approval/ | Execute approved sends |
| Social post drafts | Draft-only, HITL-gated | Final post execution |
| WhatsApp | Never | Session + send |
| Payments/Banking | Never | HITL-gated |
| Dashboard.md | Never (writes to Signals/ only) | Single writer |
| Odoo invoices | Draft-only | Post/confirm with approval |

### Claim-by-Move Protocol

Prevents double-processing when Cloud and Local share the vault:
1. Cloud Agent: `mv Needs_Action/FILE.md In_Progress/cloud/FILE.md`
2. Local Agent: checks `In_Progress/*/FILE.md` — skips if already claimed
3. After processing: move to `Done/` or release back to `Needs_Action/`

### Vault Sync (Git-based)

```
Cloud VM                          Git Remote              Local Machine
─────────                         ──────────              ─────────────
vault_sync.py (push) ──────────→ ← pull ── vault_sync.py
cloud_agent.py writes             Signals/                orchestrator reads
Pending_Approval/  ─────────────→ ← pull ── /approve-pending
```

- Never synced: `.env`, `*.json` credentials, session directories
- Conflict strategy: prefer remote for `Needs_Action/` and `Signals/`
- Signal files in `Signals/` are read by `merge_signals.py` → Dashboard.md

### Platinum Demo

```bash
bash scripts/demo_platinum.sh
```

Simulates: Gmail → Cloud claim → draft email → Pending_Approval/ → approve → Local send.

## Plan Creation (Required for Multi-Step Tasks)

```markdown
# /Plans/PLAN_<task_name>_<date>.md
---
created: <iso_datetime>
status: in_progress
task: <task description>
---

## Objective
<What needs to be accomplished>

## Steps
- [ ] Step 1
- [ ] Step 2
- [ ] Step 3 (requires approval → /Pending_Approval/)

## Approval Required
<List any external actions needing human approval>
```

## Task Completion Checklist

A task is **Done** when:
- [ ] Plan created in `/Plans/`
- [ ] Action executed (or approval created in `/Pending_Approval/`)
- [ ] Event logged in `Logs/<date>.json`
- [ ] Dashboard.md updated
- [ ] Source file moved from `Needs_Action/` to `Done/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/khawajanaqeeb)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/khawajanaqeeb)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
