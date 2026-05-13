## bni-vp-autopilot

> You are resuming work on the operator's personal BNI assistant. Read this file **first**.

# BNI-Masta — onboarding for future Claude sessions

You are resuming work on the operator's personal BNI assistant. Read this file **first**.

## What this is

**BNI-Masta** is a Telegram + LINE + Obsidian assistant for the operator (副主席 in a Taiwan BNI 分會). It orchestrates member CRM, meeting attendance (point: 點名), transcription, PDF reports, and a Google Calendar dedicated to BNI. The source of truth is an **Obsidian vault** structured as a Karpathy LLM-Wiki.

## The two-brain rule (CRITICAL — don't mix them)

| Brain | Model | Job | Never does |
|---|---|---|---|
| **Chat brain** | GPT-5.4 via OpenClaw Codex OAuth | Receives Telegram/LINE messages, orchestrates skills, answers from `wiki/` | Writes `wiki/` directly; compiles raw content |
| **Wiki compiler brain** | Claude via `claude` CLI | Reads `raw/` (immutable sources), rewrites `wiki/` markdown pages with cross-links | Handles chat; edits `raw/` |

When you see "the bot" in user messages it means the chat brain. When you see "Claude" or "compile" it means the wiki compiler (you, probably).

## Layout (on the operator's Mac — these are real paths, not repo paths)

```
~/.openclaw/
├── openclaw.json                              ← live config; GATEWAY on :18801
├── openclaw.json.pre-patch-*                  ← backups before every edit
├── agents/bni-masta/
│   ├── agent/
│   │   ├── SOUL.md                            ← agent personality (chat brain system prompt)
│   │   └── skills/
│   │       ├── pdf-ingest/                    ← chunks PDFs via pdftotext
│   │       ├── ingest-claude/                 ← shells out `claude --print` to compile
│   │       ├── member-upsert/                 ← appends to raw/inbox/
│   │       ├── transcribe-audio/              ← OpenRouter Gemini 2.5 Flash
│   │       ├── zoom-join/                     ← POSTs to Recall.ai /api/v1/bot/
│   │       └── resolve-attendance/            ← exact → Levenshtein → Claude arbitration
│   └── services/
│       └── recall-webhook.mjs                 ← HTTP :18821 — Recall.ai events land here
└── secrets/bni-masta.env                      ← chmod 600 — never commit

~/Documents/BNI AGENT/BNI AGENT/               ← THE VAULT (Obsidian opens this)
├── CLAUDE.md                                  ← LLM-Wiki schema (you read this before compiling)
├── AGENTS.md                                  ← system prompt the chat brain consumes
├── raw/                                       ← IMMUTABLE — never edit
│   ├── handbooks/202101版_領導團隊手冊/...    ← OCR'd chunks
│   ├── transcripts/                           ← Zoom + voice-note transcripts
│   ├── roll_calls/                            ← resolve-attendance output
│   ├── meetings/<date>/participants.jsonl     ← raw Recall.ai participant events
│   ├── visitors/                              ← unmatched display names
│   └── inbox/                                 ← drop-zone for anything the operator sends the bot
├── wiki/                                      ← YOU maintain these
│   ├── index.md · log.md                      ← you update after every compile
│   ├── rules/                                 ← one md per BNI rule/role
│   ├── members/<name>.md                      ← one per BNI member (PII — never commit)
│   ├── chapters/ · meetings/ · events/ · reports/
├── _templates/ · _dashboards/                 ← Obsidian Templater + Dataview

~/.cloudflared/
├── config-bni.yml                             ← bni-webhook tunnel config (managed by LaunchAgent)
└── <your-tunnel-uuid>.json                            ← tunnel credentials

~/Library/LaunchAgents/
├── ai.bnimasta.recall-webhook.plist         ← KeepAlive the webhook service
└── com.cloudflare.bni-webhook-tunnel.plist    ← KeepAlive cloudflared
```

## How to run the wiki compiler manually

```bash
cd "<vault-path>"
claude --print --permission-mode acceptEdits "compile raw/ into wiki/ per CLAUDE.md"
```

or use the skill wrapper:

```bash
bash ~/.openclaw/agents/bni-masta/agent/skills/ingest-claude/compile.sh
```

## Secrets locations (never commit)

- `~/.openclaw/secrets/bni-masta.env` — OpenRouter, Recall.ai, Anthropic, LINE, Telegram, calendar ID
- `~/.openclaw/auth-profiles/openai-codex.json` — OpenAI Codex OAuth (ChatGPT sub)
- `~/.openclaw/auth-profiles/gog.json` — Google OAuth (after the operator runs `gog auth add …`)
- `~/.cloudflared/<your-tunnel-uuid>.json` — Cloudflare tunnel credentials

## BNI domain cheat-sheet (facts the chat brain frequently needs)

- **Traffic lights (Power of One)** — 6-month rolling average: green 70-100, yellow 50-65, red 30-45, black ≤25. Categories: attendance, referrals, visitors, 1-to-1s, CEUs, sponsoring.
- **PALMS**: P=present, L=late (<15min), A=absent (>15min or no-show), M=medical leave (pre-approved), S=substitute. Attendance weight 1.0 for P, 0.5 for L/S.
- **封閉會議**: four monthly committee meetings — 接待組, 導師團, 會員委員會, 領導團隊月會.
- **專員 roles** documented in `wiki/rules/`: 品保, 團隊營造, 申訴協調, 輔導, 會員關心.
- **副主席 暖身期**: 封閉會議後一到兩週.
- Leadership handbook source: `raw/handbooks/202101版_領導團隊手冊/` (OCR'd Traditional Chinese).

## Do-not-touch list

- `raw/` — immutable
- `raw/visitors/*.jsonl` — may contain non-member PII
- `wiki/members/*.md` — member contact info; never commit to any repo
- `~/.openclaw/auth-profiles/` — never commit
- `~/.openclaw/secrets/` — never commit (chmod 600)

## Service topology

```
Telegram/LINE  →  OpenClaw gateway :18801  →  agent `bni-masta` (GPT-5.4 Codex)  →  skills
                                                                                   ├─ shells out `claude` (wiki compiler)
                                                                                   ├─ POSTs OpenRouter (transcription)
                                                                                   └─ POSTs Recall.ai (Zoom bot)

Recall.ai  →  https://<your-webhook-host>/recall-webhook
           →  cloudflared → :18821 (recall-webhook.mjs LaunchAgent)
           →  writes raw/meetings/<date>/participants.jsonl
           →  triggers resolve-attendance → triggers ingest-claude (Claude) → wiki
```

## Current state + open TODOs

This is a public template repo. Track open TODOs in your own fork's HANDOFF.md, GitHub issues, or wherever fits your workflow.

---
> Source: [AlexChanshuo/BNI-VP-Autopilot](https://github.com/AlexChanshuo/BNI-VP-Autopilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
