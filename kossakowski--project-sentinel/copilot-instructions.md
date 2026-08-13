## project-sentinel

> Real-time bot that scans PL/EN/UA/RU media for military attacks/invasions targeting Poland and

# Project Sentinel — Military Alert Monitoring System

Real-time bot that scans PL/EN/UA/RU media for military attacks/invasions targeting Poland and
the Baltic states and alerts via Twilio **phone call + SMS** plus an optional Expo **push**. Each
SMS-tier (5–8) carries a per-tier `channel` (`sms` / `push` / `both`, default `both`) so the
operator can route that tier to SMS, push, or both; the urgency 9–10 call also fires an additive
push. **Live in production on a Hetzner VPS.**

A separate read-only **Article Dashboard** (`dashboard/`) is local-only and NOT part of the
monitoring runtime — its rules live in [`dashboard/CLAUDE.md`](dashboard/CLAUDE.md) and load when
you work in that subtree.

## ⚠️ Critical rules (always honor)
- **NEVER modify production server files** without explicit user permission — ask first. Read-only
  commands (logs, health, DB queries) on the server are fine.
- **SSH only as `deploy@`**: `ssh -p 2222 deploy@178.104.76.254`. `root@`/`kossa@` trigger fail2ban bans.
- **Run and test locally by default.** Use local env vars for Twilio/Anthropic/Telegram credentials.
- **No quiet hours** — this is a life-safety alert system; call at any hour. Never miss urgency 9–10.
- **Alerts are in Polish.** Source scanning covers PL/EN/UA/RU.
- **Don't spam.** One phone call per event, then SMS for updates.
- **Corroboration required** before a phone call (independent source). Tunables live in
  `config/config.yaml` — see [config reference](docs/reference/config-reference.md) and
  [`.claude/rules/corroboration.md`](.claude/rules/corroboration.md). Do not restate the numbers here.
- **Nothing is hardcoded.** All keywords, sources, countries, thresholds, and URLs live in `config/config.yaml`.
- **Classification model: Claude Haiku 4.5** (`claude-haiku-4-5-20251001`) — cost-efficient.
- **Every phase passes its tests** before the next begins.

## Quick reference
- Config: `config/config.yaml` (template `config/config.example.yaml`)
- Run: `./run.sh` · once: `--once` · dry run: `--dry-run` · health: `--health`
- Diagnostic: `./run.sh --diagnostic` (writes `data/diagnostic.html`) · log level: `--log-level DEBUG`
- Test one headline: `./run.sh --test-headline "…"` · headline file (YAML with a `headlines:` list): `--test-file FILE`
- Classifier eval harness: `./run.sh --eval [PATH]` (default `tests/fixtures/eval_set.yaml`; scored, CI gate)
- Test a real alert: `./run.sh --test-alert` (phone call) · `--test-alert sms` · `--test-alert push`
- Tests: `.venv/bin/pytest tests/ -v`
- Full CLI: [docs/reference/cli.md](docs/reference/cli.md)
- Stack: YAML config · SQLite · APScheduler dual-lane (fast 3 min: Telegram + priority-1 RSS + Google
  News; slow 15 min: all enabled sources — GDELT is currently disabled).

## Docs (link, don't restate — `docs/` follows Diátaxis)
- **Index:** [docs/README.md](docs/README.md)
- **Server ops / deploy / troubleshooting:** [docs/how-to/server-runbook.md](docs/how-to/server-runbook.md) — **read first for anything server-related**
- Architecture → [docs/explanation/architecture.md](docs/explanation/architecture.md) · Pipeline → [docs/explanation/pipeline.md](docs/explanation/pipeline.md) · Mobile app → [docs/explanation/mobile-app.md](docs/explanation/mobile-app.md)
- Config params → [docs/reference/config-reference.md](docs/reference/config-reference.md) · Sources → [docs/reference/sources.md](docs/reference/sources.md) · CLI → [docs/reference/cli.md](docs/reference/cli.md)
- Local setup → [docs/tutorials/getting-started.md](docs/tutorials/getting-started.md) · API setup → [docs/how-to/api-setup.md](docs/how-to/api-setup.md) · Testing → [docs/how-to/testing.md](docs/how-to/testing.md) · Mobile push verification → [docs/how-to/mobile-push-setup.md](docs/how-to/mobile-push-setup.md) · Mobile inbox verification → [docs/how-to/mobile-inbox-verification.md](docs/how-to/mobile-inbox-verification.md) · VPS hardening → [docs/how-to/security/vps-hardening.md](docs/how-to/security/vps-hardening.md)
- Dashboard spec (living) → [SPEC.md](SPEC.md) · Backlog → [TODO.md](TODO.md) · Historic specs → [docs/archive/](docs/archive/README.md)

## Gotcha: project rename history
Renamed twice (`twilio-playground` → `sentinel` → `project-sentinel`). If imports fail with old
paths, recreate the venv: `rm -rf .venv && python -m venv .venv && pip install -r requirements.txt`
and clear `__pycache__` dirs.

---
> Source: [kossakowski/project-sentinel](https://github.com/kossakowski/project-sentinel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
