## mini-on-ai

> mini-on-ai is an AI-powered digital product factory running on a Mac mini.

# mini-on-ai — Claude Operating Instructions

## What This Project Is

mini-on-ai is an AI-powered digital product factory running on a Mac mini.
It continuously generates, packages, and publishes small digital products to a public showcase website at `mini-on-ai.com`.

Products are sold via Gumroad ($9–$49). Visitors can also generate a custom product on-demand via the "Build Your Own" page (Stripe, $9).

## Quick Context Recovery

The memory system is the authoritative state source — read these in order:
1. `~/.claude/projects/-Users-minion-Dev-mini-on-factory/memory/factory_state.md` — products, commands, infrastructure
2. `~/.claude/projects/-Users-minion-Dev-mini-on-factory/memory/distribution_state.md` — channel status
3. `~/.claude/projects/-Users-minion-Dev-mini-on-factory/memory/scripts_manifest.md` — all scripts

Do NOT rely on `docs/current-state.md` — it is stale.

## Project Structure

```
mini-on-factory/
├── CLAUDE.md                    ← you are here
├── data/
│   ├── product-catalog.json     ← all published products
│   ├── idea-backlog.json        ← idea candidates
│   ├── blog-posts.json          ← published blog posts
│   └── pipeline-log.json        ← run history
├── scripts/                     ← all Python pipeline scripts
│   └── lib/                     ← shared utils, claude_cli wrapper
├── worker/
│   ├── generate.js              ← CF Worker: Build Your Own (generate/checkout/download)
│   └── subscribe.js             ← CF Worker: Brevo email subscribe proxy
├── skills/                      ← skill specifications
├── products/                    ← one folder per product
│   └── {id}/
│       ├── meta.json
│       ├── assets/
│       └── package.zip
└── site/                        ← static showcase website
    ├── index.html               ← rebuilt by update_site.py
    ├── style.css                ← Dark Premium design system
    ├── build.html               ← Build Your Own page
    ├── _headers                 ← Cloudflare Pages security headers (CSP etc.)
    ├── blog/
    └── products/{id}.html
```

## Product Categories (all active)

- `prompt-packs` — 20–30 ready-to-use prompts
- `checklist` — structured action/decision list
- `swipe-file` — copy-ready examples
- `mini-guide` — concise practitioner guide
- `n8n-template` — ready-to-import automation workflow
- `claude-code-skill` — Claude Code skill configuration pack

## Pipeline

All scripts are Python 3. Orchestrated by `scripts/run_pipeline.py`.

```
run_pipeline.py
  → trend_scan.py       writes to data/idea-backlog.json
  → idea_rank.py        marks one idea as selected:true
  → generate_product.py writes to products/{id}/assets/
  → package_product.py  creates products/{id}/package.zip
  → update_site.py      updates site/ and data/product-catalog.json
  → telegram_notify.py  sends Telegram message
```

Triggered via `/run [seed]` in Telegram or the background daemon.

## Site Rebuilding

After any change to product data or templates:
```bash
python3 scripts/update_site.py --rebuild-all
```
This regenerates all 57+ product pages, index, and blog index. Then `git push` deploys to Cloudflare Pages.

## Cloudflare Workers

Two workers deployed (not in this repo, managed in Cloudflare dashboard):
- `worker/generate.js` — Build Your Own: generate (Anthropic Haiku) → Stripe checkout → ZIP download. Deployed as `mini-on-ai-generate`.
- `worker/subscribe.js` — Brevo subscribe proxy. Deployed as `mini-on-ai-subscribe`.

Both use `ALLOWED_ORIGIN = "https://mini-on-ai.com"` CORS. Secrets in CF environment.

## Conventions

- Product IDs: `{prefix}-{kebab-title}-{YYYYMMDD}` e.g. `prompts-marketing-hooks-20260312`
- All pipeline scripts are Python 3, no build step required
- Environment variables: see `.env.example`
- Never commit `.env` file
- Never explain the factory on the public website — products only
- After any edit: commit + push immediately (user preference)

## Active Workflow Skills

Read these at the start of the relevant task:

- `skills/pattern-detect.md` — at the start of any new planning session: scan git log for repeated patterns, propose up to 2 new skills
- `skills/sales-counsel.md` — when user asks about sales/distribution: fetch live data and give concrete next actions

## Environment Variables Required

```
ANTHROPIC_API_KEY=         # Claude API
TELEGRAM_BOT_TOKEN=        # Telegram bot
TELEGRAM_CHAT_ID=          # Your Telegram chat
SITE_URL=                  # https://mini-on-ai.com
BREVO_SUBSCRIBE_URL=       # CF Worker URL for email subscribe
GUMROAD_API_TOKEN=         # Gumroad API
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/abenjelloun1989) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
