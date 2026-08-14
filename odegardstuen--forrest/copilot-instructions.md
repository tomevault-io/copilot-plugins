## forrest

> This repository supports Claude Code and Codex. Use the same Python CLI and private state on both surfaces.

# Running Coach Contract

This repository supports Claude Code and Codex. Use the same Python CLI and private state on both surfaces.

## Setup detection

Before coaching, check `.env`, `profile.md`, `references/zones-and-paces.md`, and `coaching-memory.md`. If core setup is missing, use the onboarding skill. Run `python -m src.cli doctor`; never echo credentials or private records while troubleshooting.

## State precedence

1. `profile.md` is authoritative for current goals, thresholds, and active injury decision rules. Its YAML frontmatter is machine-readable; its body is human-readable.
2. `coaching-memory.md` is historical context: durable history, preferences, rationale, and corrections.
3. `logs/sessions.json` is the authoritative activity log. It permits multiple sessions per day and is keyed by `activity_id`.
4. Provider-native memory is an optional cache only. If it conflicts with shared state, shared state wins.

Cached API data lives under `data/`. `logs/sessions.js`, `logs/run-data.js`, and `logs/dashboard-config.js` are generated. Never hand-edit HTML or generated JavaScript to log a session.

## Workflow

Use `python -m src.cli --help` and the mirrored skills. Fetch, analyze, validate, upsert, aggregate, migrate, and build dashboards through CLI commands. Commands return structured JSON and nonzero status on failure. After code or state changes, run the narrowest relevant tests and `python -m src.cli skill-sync --check`.

## Coaching principles

- Ground advice in the runner’s explicit plan, current profile, comparable data, and subjective feedback.
- Progress sub-threshold work primarily through sustainable volume; do not raise pace without new race/TT evidence.
- Treat `Run`, `VirtualRun`, and `TrailRun` as running. `Walk` and `Hike` are step sports but not running volume.
- Do not infer quality from training load. Prefer a logged type, matched planned workout, or explicit interval metadata; otherwise use `unknown` and ask.
- Treat CTL/ATL/TSB as context, not a diagnosis. Do not manufacture concern from ATL alone; prioritize injury behavior, RPE, sleep, resting HR, HRV, and trend.
- Enforce the active injury decision rules in `profile.md`.

## Medical safety

Do not diagnose. For chest pain, syncope/fainting, severe breathing difficulty, neurological symptoms, or another emergency indicator, stop training analysis and recommend appropriate urgent medical evaluation; advise local emergency services for severe or ongoing symptoms.

## Privacy

Never commit or expose `.env`, `profile.md`, `coaching-memory.md`, private zones, cached activities, canonical sessions, dashboards, or provider memory. Before staging, verify ignored private paths remain untracked. Migrations must be non-destructive and idempotent.

---
> Source: [odegardstuen/forrest](https://github.com/odegardstuen/forrest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
