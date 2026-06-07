## hermes-gbrain-honcho

> You are helping a user wire Hermes + Honcho + GBrain together.

# Agent instructions

You are helping a user wire Hermes + Honcho + GBrain together.

This repository is a guide, not a replacement for any upstream project. Your job is to inspect the user's machine, explain what is already present, and apply only the safe steps the user approves.

## What you are building

```text
Hermes = agent runtime, tools, gateways, cron, MCP client
Honcho = relationship/session/user memory
GBrain = source-backed corpus, search, graph, citations, synthesis
```

Recommended local endpoints:

```text
Hermes gateway/API     http://127.0.0.1:8642
Honcho API             http://127.0.0.1:8000
GBrain HTTP MCP        http://127.0.0.1:3131/mcp
```

The main operational rule:

```text
Run one persistent GBrain HTTP service.
Let that service own the GBrain database.
Point Hermes at it over MCP.
```

This avoids local PGLite lock contention from multiple short-lived `gbrain serve` processes.

## Read these files first

1. `README.md` — mental model and high-level flow.
2. `INSTALL_FOR_AGENTS.md` — short operational runbook when another agent is executing setup.
3. `docs/installation.md` — safe install/bootstrap path.
4. `docs/configuration.md` — Honcho + GBrain config shapes.
5. `docs/routing.md` — when the agent should use Honcho, GBrain, both, or neither.
6. `docs/security.md` — secrets, localhost defaults, and tool exposure.
7. `docs/operations.md` — health checks, restarts, backups.
8. `docs/production-readiness.md` — release gates and the guarded/autopilot boundary.
9. `docs/reflex-automation.md` — optional n8n heartbeat/webhook layer.
10. `docs/eval-harness.md` — repeatable transport/eval checks.
11. `docs/troubleshooting.md` — common failure modes.
12. `docs/upstream-alignment.md` — what this guide claims about upstream projects.

## Safe execution order

### 1. Inspect before changing anything

Run read-only checks first:

```bash
pwd
git status --short || true
command -v hermes || true
command -v gbrain || true
command -v bun || true
curl -fsS http://127.0.0.1:8642/health || true
curl -fsS http://127.0.0.1:8000/health || curl -fsS http://127.0.0.1:8000/openapi.json >/dev/null || true
curl -fsS http://127.0.0.1:3131/health || true
```

If this repo is already checked out locally, run:

```bash
bash install.sh --dry-run
bash install.sh --check-only
```

If the user gave you only the GitHub URL, clone the repo or download `install.sh`, then run the same dry-run/check-only commands.

### 2. Explain what is missing

Report clearly:

- Hermes present or missing;
- Honcho reachable or not;
- GBrain CLI present or missing;
- Bun present or missing;
- whether a GBrain HTTP MCP service already exists;
- whether Hermes already has a GBrain MCP config;
- what side effects the next step would have.

Do not bury this in logs. Give a short status table and recommended next step.

### 3. Apply only conservative changes by default

The default bootstrap is intentionally cautious:

```bash
bash install.sh
```

It may create a guide checkout, a markdown corpus directory, placeholders, and an optional service file. It does not start services, overwrite existing service files, or install missing runtimes unless flags are passed.

Use opt-in flags only when the user wants them:

```bash
bash install.sh --install-missing
bash install.sh --start-services
bash install.sh --force-overwrite
```

### 4. Configure Hermes

Merge the templates manually. Do not blindly overwrite a user's live config.

Relevant templates:

```text
templates/honcho.json
templates/hermes-mcp-gbrain.yaml
templates/gbrain-http.service
templates/agent-routing-policy.md
```

After changing Hermes config, verify with:

```bash
hermes mcp list
hermes mcp test gbrain
curl -fsS http://127.0.0.1:8642/health
```

Restart only the affected service. Do not restart every layer unless necessary.

### 5. Verify end to end

A successful setup should pass these checks:

```bash
curl -fsS http://127.0.0.1:8642/health
curl -fsS http://127.0.0.1:8000/health || curl -fsS http://127.0.0.1:8000/openapi.json >/dev/null
gbrain doctor
hermes mcp list
hermes mcp test gbrain
```

Then run one real user-level test:

- Ask a preference/session question that should use Honcho.
- Ask a source/corpus question that should use GBrain.
- Ask a mixed question that should use both.

## Routing policy for the finished agent

Use Honcho for:

- stable user facts and preferences;
- relationship/session continuity;
- recent conversations;
- corrections that should shape future behavior.

Use GBrain for:

- markdown notes and documents;
- research trails;
- source-backed project knowledge;
- citations, graph links, contradictions, and gap analysis.

Use both when the answer needs personal context plus evidence.

Use neither for one-off commands, temporary calculations, or raw logs unless the user explicitly asks to store the result.

## Safety rules

- Keep default services bound to `127.0.0.1`.
- Never commit real API keys, tokens, `.env` files, database dumps, or private corpus content.
- Do not expose GBrain write/admin tools to autonomous loops until backups, audit logs, and rollback are in place.
- With PGLite, prefer one persistent GBrain service owner. Stop the service before CLI maintenance if lock contention appears.
- Prefer additive edits and backups over overwrites.
- If upstream docs conflict with this guide, prefer upstream for command details and patch this guide afterward.

## If you are unsure

Stop after `--dry-run` and `--check-only`, then tell the user exactly what you found and what you would change next.

---
> Source: [elkimek/hermes-gbrain-honcho](https://github.com/elkimek/hermes-gbrain-honcho) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-07 -->
