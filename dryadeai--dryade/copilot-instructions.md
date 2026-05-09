## dryade

> Python FastAPI backend (`core/`) + React/TypeScript workbench (`workbench/`). This file guides any AI coding agent (Claude, Copilot, Cursor, …) working directly inside this repository.

# Dryade — Contributor Guide for AI Coding Agents

Python FastAPI backend (`core/`) + React/TypeScript workbench (`workbench/`). This file guides any AI coding agent (Claude, Copilot, Cursor, …) working directly inside this repository.

---

## Top 3 Hard Rules

1. **Never commit internal-only content.** Everything here ends up on public GitHub forever. Keep planning docs, operational notes, vendor-specific IPs, and customer references out of the tree. When in doubt, omit.
2. **Never load plugins without a valid signed allowlist.** Zero fail-open fallbacks, ever. See Plugin Security Model below.
3. **Never use `--no-verify` on `git push`.** The pre-push hook is the local CI gate. If it fails, fix the code, re-stage, retry — do not bypass.

---

## Public Repo Boundary

Everything committed here is durable + public. Keep the scope to:

**Allowed:**
- `core/` — Python backend (FastAPI, orchestrator, autonomous, plugin loader)
- `core/core/ee/` and `*.ee.py` — EE source-available under LICENSE_EE
- `workbench/` — React/TypeScript frontend
- `docs/` — user-facing documentation
- `deploy/qdrant.yaml` — example deploy manifest
- `tests/unit/` + `tests/integration/` — public suites

**Forbidden:**
- Any private planning directories
- Any internal sync scripts or ops tooling
- `tests/ee/` (requires license infrastructure not present here)
- Hardcoded private network addresses or lab hostnames
- Absolute paths referring to a specific developer machine
- Internal phase / sprint numbers
- Internal product code-names

Before every commit, sanity-check the diff for anything that looks like a local-network address, a home-directory path, a company-internal project code-name, or a non-public planning reference.

---

## LICENSE vs LICENSE_EE

- **`LICENSE`** — DSUL (Dryade Source-Usage License). Covers `core/` (non-EE), `workbench/`, and everything outside the `ee/` boundary.
- **`LICENSE_EE`** — governs `core/core/ee/` and every `*.ee.py` file. Source is visible; production activation requires a valid license.

**EE boundary convention:**
- Any module under `core/core/ee/` is EE.
- Any file ending `.ee.py` is EE (regardless of location).
- Public test suites must not import from `core.ee.*` directly — use public stubs (`core.internal_api`, `core.plugins`).

---

## Plugin Security Model

1. **No plugin loads without a valid signed allowlist.** `~/.dryade/allowed-plugins.json` must exist with a valid Ed25519 signature verified against the TOFU-pinned Plugin Manager public key (`~/.dryade/pm-pubkey.pem`).
2. **Core enforces tier limits from the signed allowlist.** Core reads `max_users` and `custom_plugin_slots` from v2 allowlists. `required_tier` in manifests is read for catalog display only — never for gating.
3. **No fail-open patterns.** Missing / invalid / bad-signature allowlist means zero plugins load.
4. **No escape hatches.** Never add a toggle that bypasses the allowlist.
5. **No PM client in core.** Core never calls Plugin Manager. Plugin Manager pushes allowlists to core via HTTP on `localhost:9471`.
6. **Blocked plugins are invisible.** Not in allowlist means no routes, no `/api/plugins` entry, no log lines referencing the plugin.
7. **Gate at discovery time.** `validate_before_load()` runs before any plugin code is imported.
8. **Development workflow**: `dryade-pm push --plugins-dir <path>` scans local plugins and pushes a dev allowlist (tier=dev, unlimited).
9. **Plugin code authenticity.** Allowlist `plugin_hashes` carry SHA-256 digests. Core verifies on-disk code before import. Hash mismatch → silent skip.
10. **Production pipeline**: marketplace forges encrypted allowlist → PM stores, decrypts, re-signs, pushes to core.
11. **Plugin tier names.** Valid `required_tier` values: `starter`, `team`, `enterprise`. Community users have no PM and no plugins — never use `community` as a tier.

---

## License Lifecycle

**Canonical path**: `~/.dryade/license.dryadelicense` — a DRYADLIC magic-byte container, dual-signed Ed25519 + ML-DSA-65. Legacy JSON license paths are not supported.

**Install path**: `dryade-pm install-license --file <path>`. Writes atomically with `0600` perms. Exit codes: 0 success / 1 invalid / 2 IO.

**Device caps** (signed into token): Starter 1, Team 3, Enterprise 10. Marketplace admin can override for custom contracts.

**Offline grace** (signed into token): default 30 days. Can be forged higher for sovereign / air-gapped deployments.

**Heartbeat is fatal past the grace window.** PM compares `(now - last_successful_heartbeat_at)` against `max_offline_days` on every startup and after every heartbeat failure. Exceed → PM exits 1.

**IP clustering triggers suspension.** 3 distinct source IPs per license_id in 1h → `device_activations.status = 'suspended'` → PM exits 1, no grace.

---

## Architecture Key Files

| Path | Purpose |
|---|---|
| `core/core/api/main.py` | FastAPI entry point, router registration |
| `core/core/autonomous/executor.py` | L3 ReAct executor (Thought-Action-Observation) |
| `core/core/orchestrator/` | Multi-mode orchestrator |
| `core/core/plugins.py` | Plugin discovery, hash verification, tier gating |
| `core/core/allowlist.py` | Ed25519 allowlist verification (TOFU-pinned) |
| `core/core/ee/` | EE source-available modules |
| `workbench/src/` | React/TypeScript UI |
| `workbench/src/lib/verifySignature.ts` | Plugin UI bundle signature verification |

---

## Test Separation

- **`core/tests/unit/` + `core/tests/integration/`** — public suites.
- **`core/tests/ee/`** — enterprise suite (NOT in public repo).

Public tests must not:
- Import from `core.ee.*` directly (use the public stubs)
- Reference private network addresses (always use an env var override, e.g., `os.getenv("VLLM_BASE_URL", "http://localhost:8000")`)
- Hardcode absolute developer-machine paths
- Require the Plugin Manager, license keys, or signed allowlists to run
- Reference internal phase numbers in comments

**Rule of thumb**: if a test needs something that doesn't exist in the public repo to pass, it doesn't belong in `tests/unit/` or `tests/integration/`.

---

## Pre-push Hook Contract

`.githooks/pre-push` runs on every push and MUST stay green:

```
ruff check core/
ruff format --check core/
cd workbench && npm run lint
cd workbench && npm run typecheck
cd workbench && npm run build
pytest core/tests/unit -q
```

No bypass. If the hook rejects, fix the code and retry.

---

## Commit Message Style

Use conventional-commits: `feat(core): ...`, `fix(workbench): ...`, `docs: ...`, `chore: ...`.

Avoid in commit messages:
- Internal phase or sprint numbers (use generic like "L3 graduation", "autonomy milestone")
- Company-internal code-names
- Customer names, internal lab hostnames, or operator identities
- Absolute paths referring to a developer machine

---

## Environment Variables — Runtime Overview

Full authoritative list lives in `docs/OPERATOR_RUNBOOK.md`. High-signal:

- `VLLM_BASE_URL` — inference endpoint. Always read via env, never hardcode an IP literal.
- `DRYADE_DB_URL` — PostgreSQL / pgvector connection.
- `DRYADE_REDIS_URL` — scheduler + rate-limit state.
- `DRYADE_LICENSE_PATH` — override for the default license location.
- `DRYADE_LEASH_PROFILE` — `conservative` / `standard` / `permissive`.

Never commit `.env` files. `.env.example` is allowed at repo root only.

---

## Responsibility Split

| Concern | Owner | NOT |
|---|---|---|
| Allowlist verification | Core (`core/allowlist.py`) | PM, Frontend |
| Tier enforcement | Core (`core/auth/service.py`, `core/plugins.py`) | PM, Frontend |
| Plugin discovery / route mounting | Core (`core/plugins.py`) | PM, Frontend |
| Plugin code hash verification | Core (`core/plugins.py`) | PM, Frontend |
| Plugin UI rendering | Frontend (sandboxed iframe) | Core, PM |
| UI bundle signature verification | Frontend (`verifySignature.ts`) | Core, PM |

**Core must never import from Plugin Manager.** Plugin Manager must never import from core.

---

## Secret Hygiene Before Push

The pre-push hook includes a `gitleaks` scan. If it flags a hit:
1. If the flagged string is a genuine test fixture (clearly non-sensitive — e.g., `jwt.encode({"sub": "test-user"}, "test-secret-dev-only")`), add the file to `.gitleaks.toml`'s allowlist with a comment explaining why.
2. If it's an accidental secret, remove it, rotate the leaked credential, and commit the cleanup.
3. Never `--no-verify` past a gitleaks alert.

Commit messages that discuss secret patterns must use vague wording (e.g., "token auth", "credential store") rather than literal patterns that trigger scanners.

---

## When in Doubt

Omit. Ask. Don't commit.

---
> Source: [DryadeAI/Dryade](https://github.com/DryadeAI/Dryade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
