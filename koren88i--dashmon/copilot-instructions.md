## dashmon

> This file provides guidance to OpenAI Codex when working with code in this repository.

# AGENTS.md

This file provides guidance to OpenAI Codex when working with code in this repository.

## What this project is

Dashboard SRE — a service that takes a Grafana dashboard JSON and produces:
1. A **probe engine** that actively monitors that dashboard's health
2. A **meta-dashboard** (Grafana JSON) — "the dashboard for the dashboard"
3. **Alert rules** (YAML) for each detectable failure mode

We monitor the **user experience of a specific dashboard** — detecting when a customer would open it and see something wrong (blank panels, stale data, slow loads, broken variables). We do NOT monitor the Grafana stack itself.

The repo also ships a **self-contained demo** with a mock Prometheus backend, fault injection, and a UI simulator.

## Tech stack

- **Python 3.11+**, **FastAPI** (probe engine + mock backends)
- **Playwright** (optional browser render probe)
- **Single-file HTML/JS** for UI simulator (no build step)
- **Docker Compose** — everything runs with `docker compose up`
- **Prometheus exposition format** for probe metrics (`/metrics`)
- **Grafana Alerting YAML** (Grafana 9+ provisioning format)
- Config: YAML for probe config, JSON for dashboard input/output

## Commands

```bash
# Run everything (Docker)
docker compose up --build

# UI simulator (after compose up)
open http://localhost:8080/simulator.html

# Probe engine health endpoint (JSON summary)
curl http://localhost:8000/health

# Probe engine metrics (Prometheus format)
curl http://localhost:8000/metrics

# Mock backend health
curl http://localhost:9090/-/healthy

# Inject a fault (docker or local)
curl -s -X POST http://localhost:9090/faults/inject \
  -H "Content-Type: application/json" \
  -d '{"type":"no_data","target":"http_requests_total","duration_seconds":60}'

# Clear all faults
curl -s -X POST http://localhost:9090/faults/clear \
  -H "Content-Type: application/json" -d '{"target":"all"}'
```

## Ports

| Port | Service |
|---|---|
| 3000 | Grafana (anonymous auth, provisioned dashboards + alerts) |
| 8080 | Demo UI simulator (nginx) |
| 8000 | Probe engine (`/health`, `/metrics`) |
| 9091 | Real Prometheus (scrapes probe engine) |
| 9090 | Mock Prometheus backend + fault injection |
| 8012 | Browser render probe (`/health`, `/metrics`) |

Override via `.env` (copy from `.env.example`): `GRAFANA_PORT`, `PROMETHEUS_PORT`, `SIMULATOR_PORT`, `PROBE_ENGINE_PORT`, `MOCK_BACKEND_PORT`.

## Architecture

```
Grafana Dashboard JSON
        │
        ▼
   parser.py ──→ ProbeSpec objects
        │
        ├──→ engine.py (probe loop, 30s interval, concurrent)
        │       │
        │       ├──→ query_probe.py      (NO_DATA, QUERY_TIMEOUT, SLOW_QUERY, PANEL_ERROR)
        │       ├──→ staleness_probe.py  (STALE_DATA)
        │       ├──→ variable_probe.py   (VAR_RESOLUTION_FAIL)
        │       └──→ cardinality_probe.py(CARDINALITY_SPIKE, METRIC_RENAME)
        │       │
        │       ▼
        │   metrics.py ──→ /metrics (Prometheus) + /health (JSON for UI)
        │
        ├──→ meta_dashboard.py ──→ Grafana dashboard JSON (importable)
        └──→ alert_rules.py    ──→ Grafana alerting YAML (provisionable)
```

**mock_backend/** — FastAPI app mimicking Prometheus HTTP API with fault injection (`POST /faults/inject`, `POST /faults/clear`, `GET /faults/active`). The probe engine talks to it the same way it would talk to real Prometheus.

**demo/simulator.html** — single HTML file. Top: target dashboard with live sparklines. Middle: meta-dashboard polling `/health`. Bottom: fault injection buttons + issue log. Faults must be detected within ≤30s.

## Key data structures

- `PanelProbeSpec` — panel_id, panel_title, datasource_uid, datasource_type, queries (raw PromQL), expected_min_series
- `VariableProbeSpec` — name, datasource_uid, query, is_chained, chain_depth

## Failure modes detected

`NO_DATA`, `STALE_DATA`, `METRIC_RENAME`, `QUERY_TIMEOUT`, `VAR_RESOLUTION_FAIL`, `SLOW_QUERY`, `SLOW_DASHBOARD`, `CARDINALITY_SPIKE`, `PANEL_ERROR`

## Design constraints (do not change)

- Local/unit fixtures may run without Grafana, but Docker probe configs must exercise Grafana's panel request path through `/api/ds/query`
- Probe engine is datasource-agnostic (Prometheus HTTP API; other types out of scope but architecture allows adding them)
- Meta-dashboard JSON must be importable into real Grafana
- Alert rules YAML must be valid Grafana Alerting provisioning format
- UI simulator works in modern browser with no build step
- Docker Compose must work on Mac and Linux
- Probe engine handles errors gracefully — one panel failure doesn't crash others
- No external databases, no heavy frameworks
- Mock APIs must implement the spec, not just the subset our own code uses. Real consumers (e.g. Grafana) will exercise different parts of the contract (e.g. POST instead of GET). If we only test against our own code, we miss what breaks for everyone else.
- A green raw datasource probe is not enough to claim a Grafana dashboard is healthy. User-facing dashboard contracts must include the Grafana datasource/plugin path, because variables, range queries, plugin serialization, and non-JSON errors can fail after Prometheus itself succeeds.
- Issue event logs must be diagnosis-aware, not just status-aware. A panel that changes from `no_data` to `slow_query` while still degraded needs a new event; repeated probes of the same `(probe_type, error_type)` diagnosis should not spam the log.
- Grafana variable dropdowns may resolve `label_values(metric, label)` through `/api/v1/series`, not only `/api/v1/label/<label>/values`. Variable-failure simulations must cover both paths or the SRE variable probe can go red while the real dropdown still looks healthy.
- Grafana alert rule UIDs must be ≤40 characters; rule titles must not contain `$` (interpreted as template variables)
- Stateful systems don't forget what you told them. If your code changes what it writes (metric labels, DB rows, cache keys, file paths), it must also clean up what it previously wrote — old state outlives the code path that created it.

## Step verification approach

Every implementation step must be independently verifiable before moving on. No throwaway test files.

- **Steps 1–6** (backend): verify via curl commands against running services
  - Mock backend: `curl localhost:9090/api/v1/query?query=up` → non-empty result
  - Fault injection: `POST /faults/inject` then re-query → observe degraded response
  - Probe engine: `curl localhost:8000/health` → JSON with health_score, panel statuses, issues
  - Generators: validate output with `python -m json.tool` / `yaml.safe_load`
- **Step 7+** (demo UI): `simulator.html` becomes the permanent visual verification surface
  - Inject fault via button → panel degrades visually + meta-dashboard goes red within 30s
  - Clear all → everything green within 30s
- **Step 8** (Docker): `docker compose up` then repeat all above checks against containerized services

**Timing budget:** probe_interval=15s + UI poll=5s → worst-case ~20s detection (within 30s requirement).

## Doc hierarchy

Use the repo docs this way:

- `README.md` for current setup and operator usage
- `ARCHITECTURE.md` for current system behavior and topology
- `AGENTS.md` for repository working conventions
- `BROWSER_RENDER_PROBE_PLAN.md` for browser render probe implementation notes and future render-fault follow-up

Historical context only:

- `DASHBOARD_SRE_BRIEF.md`

Do not treat the historical docs as the current implementation contract.

## Session management
End a session after completing and verifying a coherent slice of work.
To close gracefully, use the `session-close` skill.

## Purpose (engineering behavior)
Keep it short, stable, and high-signal. Put reusable deep playbooks in `.agents/skills/*/SKILL.md`.

## Engineering mindset

Before planning or implementing any feature, think like a **senior engineer on the platform infra team of a 300-developer company**:

- **Assume multi-tenancy from day one.** Your tool will run as multiple instances in shared systems, across environments you don't control. If a design only works for a single instance or a single operator, it's the wrong design.

- **Identity must be derived, not declared.** Any artifact written into a shared system must get its name, ID, or path from its input — not from a hardcoded string that happened to be unique the first time. Ask: "what breaks when a second instance runs alongside this one?"

- **Config owns the environment; code owns the logic.** Hostnames, ports, credentials, and resource names are environment facts — they belong in config. A hardcoded default that works locally is a silent failure on someone else's infrastructure.

- **Design for the operator, not the author.** Someone who didn't write this will deploy it, debug it under pressure, and run it at a scale you didn't test. Names, logs, and error messages should make their life easier.

- **Think day-2.** What happens when a second tenant is added? When one is removed? When a new version is deployed over the old one? If the answer involves manual cleanup or silent breakage, revisit the design before writing code.

## Default working style
- Optimize for simplicity, readability, and maintainability over cleverness.
- Prefer small, understandable changes over large sweeping rewrites.
- Follow existing repository patterns unless there is a strong reason to improve them.
- When a request is large, risky, or vague, break it into smaller deliverable slices before implementing.
- Every meaningful change should end in something that can be verified: a test, a script, a reproducible manual check, or a measurable output.

## Coding standards
- Write code so a new team member can understand it quickly.
- Use clear names for variables, functions, classes, and files.
- Keep functions focused on one job.
- Prefer explicit data flow over hidden side effects.
- Avoid unnecessary abstraction. Introduce layers only when they remove duplication or complexity.
- Prefer constants, configuration, and schema-driven behavior over magic numbers and hard-coded values.
- Document non-obvious decisions near the code or in lightweight docs.
- Do not mix unrelated refactors into a bug fix unless necessary for safety.

## When to split work into smaller parts
Split the task before coding when one or more of these are true:
- The request touches multiple subsystems.
- The acceptance criteria are unclear.
- The change is hard to verify end-to-end.
- The implementation would be easier to review as separate commits or steps.
- A safe intermediate state can be delivered first.

When splitting, define:
1. the smallest useful slice,
2. how it will be verified,
3. what remains for the next slice.

## Bug-handling default
For bugs and regressions, use the `bug-investigation` skill.

Default flow:
1. Explain the bug clearly.
2. Reproduce it.
3. Narrow the cause.
4. Add or identify a failing test when practical.
5. Make the smallest safe fix.
6. Run tests / verification.
7. Refactor only after the bug is understood and protected.
8. Capture a reusable lesson by updating `AGENTS.md` or a skill when the lesson is likely to matter again.

## Refactoring default
For non-trivial cleanup, use the `refactor-safely` skill.
Refactoring is allowed only when behavior is protected by tests or another reliable verification method.

## Deliverable quality bar
A task is not complete unless the result is testable or otherwise verifiable.

For each deliverable, provide:
- what changed,
- how to verify it,
- what is still not covered,
- risks or follow-ups if any.

Use the `deliverable-verification` skill when you need a stronger verification checklist.

## Hard-coding policy
Avoid hard-coded:
- business rules that may change,
- environment-specific values,
- secrets, tokens, URLs, and file paths,
- thresholds/timeouts/limits without named constants,
- duplicated literal values spread across files.

Allowed exceptions:
- stable protocol values or standards,
- tiny local literals whose meaning is obvious,
- test fixtures where inline values improve readability.

If a literal is important, name it.

## Output expectations
When implementing:
- state assumptions,
- mention the verification method,
- call out missing information or untested edges honestly.

When debugging:
- explain cause before proposing broad cleanup,
- prefer evidence over guesswork.

## Git conventions

### Commit messages
Use [Conventional Commits](https://conventionalcommits.org/en/v1.0.0/):

```
<type>(<scope>): <description>

[optional body — explain WHY, not WHAT]
```

**Types:** `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`, `ci`, `build`, `style`

**Scopes** map to project subsystems: `mock-backend`, `parser`, `probe`, `engine`, `metrics`, `generator`, `demo`, `docker`, `config`

**Rules:**
- One logical change per commit. Do not bundle unrelated changes.
- Write the "why" in the message body; the diff shows the "what".
- Breaking changes: append `!` before colon (`feat!: ...`) or add `BREAKING CHANGE:` footer.
- Always include `Co-Authored-By` trailer when AI generates or substantially writes the code.

**Examples:**
```
feat(probe): add staleness detection for panel data

Checks max timestamp vs now() to detect stale data that would
show outdated values to a dashboard viewer.

Co-Authored-By: OpenAI Codex <noreply@openai.com>
```
```
fix(mock-backend): prevent fault injection from crashing on empty target
```

### Branching — one branch per plan step
- **Trunk-based with short-lived feature branches.** Keep `main` always in a working state.
- Branch naming: `<type>/<short-description>` (e.g., `feat/mock-backend`, `feat/parser`, `fix/timeout-handling`).
- **One branch per major plan step**, not per sub-step. Sub-steps within a step are tightly coupled and share files. Example: `feat/mock-backend` covers 1A + 1B + 1C.
- Merge to `main` when the full step is verified (all sub-steps working together). Delete the branch after merge.

### Committing — after each verified sub-step
- Complete a sub-step → run its verification → commit. One commit = one verified slice.
- **Bug found during implementation?** Fix in a separate `fix()` commit. Do not fold it into the feature commit.
- **Refactoring triggered by a step?** Separate `refactor()` commit after the feature lands.
- This keeps history honest and makes `git bisect` useful.

### Pushing — after every commit
- Push after each commit. For a solo project, there is no reason to batch. Pushing is cheap insurance against losing work.

### Pull requests — one per plan step
- One PR per major plan step. PR title follows conventional commit format: `feat(mock-backend): mock Prometheus API with fault injection`.
- PR body must include: summary (what + why), verification results, and anything still not covered.
- Self-review the diff (`git diff main...HEAD`) before merging.
- PRs serve as documentation milestones — a record of what each step accomplished.

### Bug fixes on main (post-merge)
- Create a hotfix branch (`fix/<description>`), fix, verify, merge back. Same trunk-based workflow.

### Workflow example
```
main ─────●────────────────●────────────────●───
          │                │                │
          └─ feat/mock-backend              └─ feat/query-probe
             ├─ commit: 1A (mock API)          ├─ commit: Step 3
             ├─ commit: 1B (fault injection)   └─ merge → main
             ├─ commit: 1C (grafana stub)
             └─ merge → main
                           └─ feat/parser
                              ├─ commit: 2A (example dashboard)
                              ├─ commit: 2B (parser)
                              └─ merge → main
```

### Safety rules
- Never force-push to `main`.
- Never use `--no-verify` to skip hooks.
- Never commit secrets, `.env` files, or credentials.
- Prefer creating a new commit over amending, especially after hook failures.
- Stage files explicitly by name — avoid `git add .` or `git add -A` which can catch unintended files.

### .gitignore
The repo `.gitignore` covers Python, Docker, and IDE artifacts. Keep it updated when adding new tooling.

## Tests

```bash
# Install test deps (once)
pip install -r tests/requirements.txt

# Run by layer
pytest -m unit
pytest -m integration
pytest -m e2e
pytest
pytest --collect-only -q
```

Treat the collected tests and the files under `tests/` as the current source of truth for coverage.

## Skills available
- `.agents/skills/bug-investigation/SKILL.md`
- `.agents/skills/refactor-safely/SKILL.md`
- `.agents/skills/deliverable-verification/SKILL.md`
- `.agents/skills/docker/SKILL.md` — port management, resource sizing, networking, Dockerfile rules, dev/prod patterns
- `.agents/skills/session-close/SKILL.md` — end-of-session checklist for docs, memory, git, and handoff

---
> Source: [koren88i/dashmon](https://github.com/koren88i/dashmon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
