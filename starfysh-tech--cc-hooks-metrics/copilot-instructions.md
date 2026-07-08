## cc-hooks-metrics

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

`cc-hooks-metrics` gives Claude Code users a **fast, actionable overview of hook health** — so you can immediately see what's broken, slow, or regressing without wading through raw data. Designed to be shareable, not just personal.

## Working in this repo

Discoveries outside the current task go to `TODO.md`, not into the code. Add a one-line description under the appropriate section (Backlog for ideas, Prioritized for approved work). Do not implement without approval.

## Running the report

```bash
# Default: traffic lights, overhead waterfall, action items, trends, guardrails
~/.claude/hooks/hooks-report.sh

# Verbose: adds perf table, WoW summary, top projects, FIXED/GONE trends, + 7 legacy sections
~/.claude/hooks/hooks-report.sh --verbose

# OTel-aligned JSON export
~/.claude/hooks/hooks-report.sh --export

# Show recent sessions
~/.claude/hooks/hooks-report.sh --sessions

# Drill into a specific step
~/.claude/hooks/hooks-report.sh --step audit-logger

# Pipe to Claude for analysis
~/.claude/hooks/hooks-report.sh --export | claude -p "Analyze and suggest next steps"
```

Output is always Rich static. `--export` outputs JSON. `--verbose` adds legacy detail sections.

## Deployment

Scripts are installed to `~/.claude/hooks/` — the copies in this repo are the source of truth. See `TODO.md` for planned distribution improvements (Homebrew, install script).

```bash
# Deploy Python package + wrapper
rsync -a --delete hooks_report/ ~/.claude/hooks/hooks_report/
install -m 755 hooks-report.sh ~/.claude/hooks/hooks-report.sh
```

The database path defaults to `~/.claude/hooks.db` and can be overridden with `CLAUDE_HOOKS_DB`.

## Architecture

**Data flow**: Claude Code event → `hook-metrics.sh` (wrapper) → downstream hook script → `hooks.db`

### Data ingestion scripts (unchanged bash)

`hook-metrics.sh` is a **passthrough wrapper** — takes `EVENT:STEP_NAME` as `$1` and the actual hook script + args as remaining args. Captures wall-clock timing via `/usr/bin/time -p`, git context, and exit code, then inserts a row into `hook_metrics`. The wrapped script's exit code is always preserved.

`audit-logger.sh` reads the Claude tool-use JSON payload from stdin, extracts `tool_name`, `tool_input`, and `session_id`, and inserts into `audit_events`. Echoes stdin through so it can be chained.

`db-init.sh` is sourced by all scripts. Owns the schema and provides:
- `_init_hooks_db` — creates tables or runs `ALTER TABLE` migration (idempotent)
- `_db_exec sql` — write-only helper with `PRAGMA busy_timeout=1000`, stdout suppressed
- `_q sql` — **report-only** read helper using `sqlite3 -separator '|'` with **no** busy_timeout (adding it would emit a result row corrupting pipe reads)
- `_sql_escape` — single-quote doubling via `sed`
- `_maybe_prune_hooks_db` — 1% probabilistic pruning of rows older than 30 days

### Python package: hooks_report/

Rewrite of the original 1331-line bash report in Python (Rich 14.3.3).

```
hooks_report/
  __init__.py       # empty
  __main__.py       # entry: export/static dispatch
  cli.py            # argparse: --export, --export-spans, --verbose, --static, --db, --include-sensitive
  config.py         # STEP_TIMEOUTS, SEMANTIC_EXIT_STEPS, thresholds, SKIP_HOOKS_PATTERN, OTLP constants
  db.py             # HooksDB: typed dataclasses + SQLite queries
  otlp.py           # OTLP/HTTP JSON export: build_otlp_payload(), send_spans(); zero external deps
  render.py         # Rich helpers: fmt_dur, bar_chart, trend_badge, pct_change, traffic_light_grid
  spans.py          # OTel span model: Span dataclass, hook_metric_to_span, audit_event_to_span, spans_to_dict
  static.py         # Rich Console output: compact sections + verbose sections + export_json
  advisor.py        # Tuning suggestions, hot sequences, periodic summaries
```

**hooks-report.sh** is a Python wrapper with venv detection:
```bash
#!/usr/bin/env bash
DIR="$(cd "$(dirname "$0")" && pwd)"
PYTHON="${DIR}/.venv/bin/python3"
[ -x "$PYTHON" ] || PYTHON=python3
PYTHONPATH="$DIR" exec "$PYTHON" -m hooks_report "$@"
```

### Output structure

**Default mode** (~35-50 lines):
1. Traffic-light grid (Reliability / Performance / Broken Hooks / Regressions / Review Gate)
2. Overhead waterfall — top 5 steps by total overhead with proportional bars
3. Action items grouped by step — one entry per step with all issues deduplicated (or "All clear")
4. `section_wow_compact()` — REGR/SLOW trend lines only (FIXED/GONE suppressed in default)
5. Guardrails table with top block reason per guardrail

**--verbose mode** adds compact sections + 7 legacy detail sections:
- `section_perf_compact()` — per-step performance table (last 7d, avg ≥500ms or has timeout, max 12 rows)
- `section_wow_compact()` — full WoW summary + REGR/FIXED/SLOW/GONE trend lines
- `section_projects_compact()` — top 5 repos by overhead
- 7 legacy detail sections

**--export mode** — OTel-aligned JSON, schema `claude.hooks.trends/v1`, metric names `claude.hooks.*`, attributes `hook.step` / `vcs.repository`.

**--export-spans mode** — OTel span JSON, schema `claude.hooks.spans/v1`. One span per hook_metrics row (`hook.{step}`, kind=3 CLIENT) and per audit_events row (`tool.{tool_name}`, kind=1 INTERNAL). Redacts sensitive fields by default; `--include-sensitive` disables redaction. Skip warnings on corrupt rows go to stderr. If `HOOKS_METRICS_OTLP_ENDPOINT` is set, also POSTs spans to the OTLP endpoint before printing JSON (`otlp.py`); `HOOKS_METRICS_OTLP_HEADERS` sets auth headers (`key=value,key2=value2`).

## Key conventions

- `codex-review` uses semantic exit codes (exit 1 = findings, not failure) — excluded from failure counts via `step NOT IN ('codex-review')`, tracked separately; `SEMANTIC_EXIT_STEPS` set in `config.py` controls this for span export too
- OTel SpanKind: hooks → `kind=3` (CLIENT — spawn external processes); tools (audit events) → `kind=1` (INTERNAL — Claude-internal operations)
- `SKIP_HOOKS` regex (`fake-fail|ok-step|echo|test-hook|main`) filters noise from coverage gap detection — use `re.fullmatch()` not `re.search()`
- `ROUND(...,0)` in SQLite returns float — use `int(round(float(val)))` not `int(val)`
- NULL failure_rate → `None` in Python, `null` in JSON (not `0`)
- `CLAUDE_HOOKS_DB` env var overrides DB path
- Empty/missing DB: auto-init schema on first connect, returns zero rows (no crash)
- All Rich content must be `rich.text.Text` objects, not markup strings

## Adding a new hook step

1. Create the hook script following the `mermaid-lint.sh` pattern (read JSON from stdin, exit 0 on no-op)
2. Wire it in `~/.claude/settings.json` using `hook-metrics.sh EVENT:STEP_NAME /path/to/script`
3. Add its timeout to `config.STEP_TIMEOUTS` in `hooks_report/config.py` if it has a configured timeout

## Security & operations

- [SECURITY.md](SECURITY.md) — coverage table for the [OWASP Top 10 for Agentic Applications (2026)](https://genai.owasp.org/), per-ASI mapping, honest limits.
- [RUNBOOK.md](RUNBOOK.md) — kill-switch drill, audit-chain verification, archive policy.
- Claude Code CVEs the floor in `version-requirements` defends against: [CVE-2025-59536](https://nvd.nist.gov/vuln/detail/CVE-2025-59536) (`.claude/settings.json` / `.mcp.json` code injection, fixed 1.0.111) and [CVE-2026-21852](https://nvd.nist.gov/vuln/detail/CVE-2026-21852) (`apiUrl` auth-header exfil, fixed 2.0.65).

### Opt-in security toggles

All default off; existing users see no behavior change unless enabled.

- `HOOKS_AUDIT_CHAIN=1` — write tamper-evident chain (`prev_hash` + `row_hash`) into `audit_events`. Adds ~30-80ms per event (Python startup). Verify with `hooks-report.sh --verify-audit-chain`.
- `HOOKS_METRICS_OTLP_ALLOWED_HOSTS=otel.local,otel.cloud` — restrict where the OTLP exporter may POST. Bare hostnames, no port, no scheme.
- `GUARD_SECURITY_ALLOW=blocks_force_push_main,...` — skip specific `guard-security.py` predicates per-machine. Names are the function names in `guardrails/_patterns.py`.

### CLI additions

- `hooks-report.sh --verify-audit-chain` — walks `audit_events`, reports the first hash divergence (exit 1) or `OK` (exit 0). Pre-migration DBs return exit 0 with a note.

## Guardrails

Optional guardrail scripts live in `guardrails/`. All use plain `python3` (stdlib only) for portability.

| Script | Event | Purpose |
|--------|-------|---------|
| `guard-security.py` | PreToolUse | Hard-blocks destructive Bash + `.env` access; soft-blocks `rm` against non-`/tmp` paths via `permissionDecision: "ask"` |
| `guard-python-lint.py` | PostToolUse | Runs `ruff check` on `.py` Write/Edit |
| `guard-python-typecheck.py` | PostToolUse | Runs `ty check` on `.py` Write/Edit |
| `guard-ts-typecheck.py` | PostToolUse | Runs `tsc --noEmit` on `.ts`/`.tsx` Write/Edit |
| `guard-auto-allow.py` | PermissionRequest | Auto-allows read-only tools |

All guardrails exit 2 + stderr to block (Claude self-corrects), exit 0 to allow. Wired via `hook-metrics.sh` for tracking. See `settings-guardrails-example.json` for copy-paste wiring.

`GUARDRAIL_STEPS` in `config.py` controls reporting queries. `event-log` step is already in `SKIP_HOOKS_PATTERN`.

### Hook Protocol

- **PreToolUse**: stdin `{tool_name, tool_input}`. Exit 2 + stderr = hard-block. Exit 0 + stdout JSON `{hookSpecificOutput: {hookEventName: "PreToolUse", permissionDecision: "ask", permissionDecisionReason: "..."}}` = soft-block (surfaces user confirmation prompt even when `skipDangerousModePermissionPrompt` is set).
- **PostToolUse**: stdin `{tool_name, tool_input, tool_use_id}`. Exit 2 + stderr = block.
- **PermissionRequest**: stdout JSON `{hookSpecificOutput: {hookEventName, decision: {behavior: "allow"}}}`. No output = defer to user.

### Naming convention

Guardrail steps use `guard-` prefix (e.g., `guard-security`, `guard-python-lint`).

## Design Context

### Users
AI/DevOps enthusiasts — conference/meetup attendees, community members, and developers curious about AI agent observability. Technical audience comfortable with terminals, dashboards, and metrics, but not necessarily deep in this specific tooling.

### Brand Personality
**Bold, data-driven, precise.** The tool earns trust through real numbers, not marketing polish. It should feel like looking at a well-built control room — impressive density of information, immediately legible.

### Aesthetic Direction
- **Visual tone:** Bold and impressive — "wow factor" for demos while remaining data-dense and functional
- **Reference:** Grafana / Datadog — dense observability dashboards with strong data hierarchy, meaningful color, and panels that reward close inspection
- **Anti-reference:** Generic SaaS landing pages, excessive whitespace, decorative illustrations with no data
- **Theme:** Dark and light mode. Dark is primary (matches terminal-native identity); light mode for accessibility and varied contexts
- **Typography:** `Space Grotesk` for headlines, `IBM Plex Mono` for data/code. Terminal report uses system monospace (`SF Mono`, `Cascadia Code`, `JetBrains Mono`, `Fira Code`)

### Color System
Semantic palette rooted in observability conventions:
- **Red** (`#f85149`) — critical, failures, blocks
- **Yellow** (`#d29922`) — warnings, degraded performance
- **Green** (`#3fb950`) — healthy, success, all clear
- **Blue/Cyan** (`#58a6ff` / `#79c0ff`) — information, accent, data viz primary
- **Purple** (`#bc8cff`) — secondary emphasis, highlights
- **Orange** (`#f0883e`) — tertiary warning, attention
- Neutrals: dark surfaces (`#0d1117` → `#1c2129`), muted text (`#8b949e`), borders (`#30363d`)

### Design Principles
1. **Data density over decoration** — every pixel should carry information or improve legibility. No ornamental elements.
2. **Color means something** — never use color for aesthetics alone. Red = bad, green = good, yellow = caution. Consistent everywhere (terminal, web, export).
3. **Terminal-native first** — the CLI report is the canonical experience. Web pages extend it, not replace it. Monospace, dark backgrounds, and data tables are features, not constraints.
4. **Impressive at first glance, useful on inspection** — the "wow" comes from real data, well-organized. Waterfall charts, traffic lights, and trend badges should be visually striking AND immediately actionable.
5. **Accessible without compromise** — light/dark mode, sufficient contrast, semantic HTML. Accessibility is a requirement, not a nice-to-have.

---
> Source: [starfysh-tech/cc-hooks-metrics](https://github.com/starfysh-tech/cc-hooks-metrics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
