## arc

> This file provides context for AI agents working on this codebase.

# Agent Context — ARC (Agent Remote Control)

This file provides context for AI agents working on this codebase.

**IMPORTANT FOR AGENTS:** Update this file on EVERY commit that changes features,
architecture, CLI commands, configuration, build steps, or known issues.
This is the canonical context document — future agents and humans rely on it.
If you're unsure whether to update it, update it.

## Project Overview

ARC is a universal remote control for AI agent frameworks. It connects any agent to a browser-based interface for real-time monitoring, interaction, and remote command injection via WebSocket relay.

**Architecture:** Agents → WebSocket Relay → Web Viewers. Commands flow back the same path.

**Beta relay:** `arc-beta.axolotl.ai` — open access with prefix-based tokens (`axolotl_beta_*`).

## Repository Structure

```
├── relay/                     # OSS relay server (Python/FastAPI)
│   ├── relay.py               # Main app: WebSocket relay + create_app() factory + web viewer serving
│   ├── beta_app.py            # Beta relay entrypoint (Redis session store + prefix auth)
│   ├── protocols.py           # Extension interfaces (AuthProvider, SessionStore, SessionPolicy, LifecycleHooks)
│   ├── defaults.py            # OSS defaults (token auth, prefix tokens, in-memory store)
│   ├── models.py              # Session, SessionInfo dataclasses (includes e2e field)
│   └── tests/                 # Unit tests (test_relay.py, test_prefix_auth.py)
├── packages/                  # TypeScript (npm workspaces, @axolotlai scope)
│   ├── protocol/              # @axolotlai/arc-protocol — wire types, RemoteControlClient, BaseAdapter, crypto
│   ├── cli/                   # @axolotlai/arc-cli — `arc` command
│   ├── web-client/            # Static React SPA viewer (auto-connect from URL params, E2E decrypt)
│   ├── adapter-hermes/        # @axolotlai/arc-adapter-hermes (unused — native plugin is primary)
│   ├── adapter-deepagent/     # @axolotlai/arc-adapter-deepagent
│   └── adapter-openclaw/      # @axolotlai/arc-adapter-openclaw
├── hermes-plugin/             # Native Hermes Agent plugin (PRIMARY Hermes integration)
│   ├── arc-remote-control/    # Plugin: tools, lifecycle hooks, clarify monkeypatch, E2E encryption
│   │   ├── __init__.py        # Main plugin code (ArcRelay, hooks, clarify patch, E2E)
│   │   └── plugin.yaml        # Hermes plugin manifest
│   └── tests/                 # Plugin tests (test_reconnect.py, test_e2e.py)
├── hosted/                    # Hosted SaaS version (see hosted/AGENTS.md)
│   ├── backend/               # Python: WorkOS auth, Stripe billing, Postgres, Redis pub/sub
│   │   └── store.py           # RedisSessionStore with pub/sub for multi-instance
│   ├── deploy.sh              # Fly.io deploy script (beta/staging/prod)
│   ├── Dockerfile.beta        # Beta: relay + Redis + web viewer (multi-stage Node + Python)
│   ├── fly.beta.toml          # Beta Fly.io config (arc-beta.axolotl.ai)
│   └── AGENTS.md              # Hosted-specific agent context
├── .github/workflows/
│   ├── ci.yml                 # Lint (ESLint + ruff) + parallel Node/Python test jobs
│   ├── deploy-web-client.yml  # Deploy web client to GitHub Pages on push to main
│   ├── publish-npm.yml        # Publish @axolotlai/* packages on vX.X.X tags
│   └── publish-pypi.yml       # Publish arc-relay to PyPI on vX.X.X tags
├── eslint.config.mjs          # TypeScript linter config
├── ruff.toml                  # Python linter config (black-compatible)
├── .husky/pre-commit          # Pre-commit: lint + build
├── install.sh                 # curl | sh installer (--local flag for dev installs)
├── docker-compose.yml         # Local dev: relay + Postgres 16 + Redis 7
└── pyproject.toml             # PyPI config (setuptools-scm for version from git tags)
```

## CLI Commands

```bash
arc setup [--hermes|--self-hosted]
    # Defaults to beta relay (arc-beta.axolotl.ai), generates prefix token
    # --hermes: installs plugin, skill, patches hermes entrypoint
    # --self-hosted or ARC_ENV=dev: local relay with auto-start

arc connect [--json] [--quiet] [--framework hermes] [--name my-agent]
    # Start a remote control session
    # Writes ~/.arc/session.json and ~/.arc/viewer-url
    # Copies viewer URL to clipboard, opens browser
    # --json outputs machine-readable JSON

arc update
    # Local: git pull + npm install + npm run build + reinstall skills
    # Global: npm install -g @axolotlai/arc-cli@latest + reinstall skills
    # Re-patches hermes entrypoint, restarts local relay if dev mode

arc install-skill [framework]    # Install /remote-control skill
arc status                       # Show current config
```

## Key Design Patterns

### Extension Protocol
The relay is extensible via four protocol interfaces in `relay/protocols.py`:
- **AuthProvider** — authenticate agents, viewers, admins
- **SessionStore** — persist session state
- **SessionPolicy** — enforce session limits
- **LifecycleHooks** — react to session events

### E2E Encryption — See `hosted/AGENTS.md`
E2E encryption is a hosted feature. The wire protocol supports encrypted envelopes
(`EncryptedField`, `encrypted: true` flag) and the relay passes them through without
inspection. Implementation details are in the hosted docs.

### Auth Model
- **Fixed token** (OSS): `AGENT_TOKEN` env var
- **Prefix tokens** (beta): `AGENT_TOKEN_PREFIX=axolotl_beta_` — any token matching prefix + 43 chars. Per-agent session isolation via SHA-256 user_id.
- **Session secrets**: per-session, preserved in Redis across agent reconnects

### WebSocket Protocol
- Agents: `register` → relay returns `registered` with session secret
- Viewers: `subscribe` with session secret → replay traces from Redis, then live stream
- Traces: agent → relay (publish_trace) → Redis pub/sub → all instances → local viewers
- Commands: viewer → relay (publish_command) → Redis pub/sub → agent's instance
- Encrypted envelopes: `{ kind: "trace", event: {ciphertext, nonce}, encrypted: true }`
- Relay skips command type validation for encrypted commands

### Multi-Instance Support — Hosted Only
See `hosted/AGENTS.md`. The OSS relay uses `InMemorySessionStore` (single instance).
The relay code uses `hasattr()` checks to detect hosted store capabilities — zero
hosted dependencies in the OSS code.

### Web Viewer
- Served at `/viewer` by the relay (StaticFiles mount)
- Auto-connects from URL params: `?session=...&s=...`
- Auto-reconnects with exponential backoff (2s → 30s cap)
- Stops reconnecting only on auth errors (invalid secret)
- Keeps retrying on "session not found" (waits for agent to re-register)
- Sends pings every 30s to prevent Fly idle timeout
- Clears error traces on successful reconnect
- E2E: buffers encrypted traces until key is derived, skips undecryptable
- Distinct visual styles for user (navy) vs assistant (indigo) messages

### Hermes Agent Integration
- **Native Python plugin**: `hermes-plugin/arc-remote-control/` — the primary integration
- **Lifecycle hooks**: `pre_llm_call`, `post_llm_call`, `pre_tool_call`, `post_tool_call`
- **Clarify tool**: Monkeypatches `tools.clarify_tool.clarify_tool` at first use (lazy load)
  to forward choices to viewer as `waiting_for_input` with choices. Viewer answer goes
  directly into CLI's `_clarify_state["response_queue"]` to dismiss prompt_toolkit picker.
- **Clarify bypass**: Hermes special-cases clarify in `run_agent.py` — `pre_tool_call` hook
  never fires for it. That's why we monkeypatch the function directly.
- **Skills**: `~/.hermes/skills/devops/remote-control/SKILL.md` (agentskills.io format)
- **Tools registered**: `arc_start_session`, `arc_stop_session`, `arc_session_status`
- **`hermes` entrypoint patch**: adds `PluginContext.inject_message` + `_cli_ref` for
  viewer→agent messaging. Pending upstream PR: NousResearch/hermes-agent#3778
- **Plugin auto-installs**: `websocket-client` + `cryptography` into Hermes's venv
- **Reconnect loop**: while-loop with exponential backoff (2s→30s, 20 max attempts)
- **Trace buffer**: queues up to 200 traces while disconnected, flushes on reconnect
- **E2E**: derives AES-256-GCM key via HKDF (uses `cryptography` lib with manual fallback)
- **Debug logging**: writes to `~/.arc/plugin.log` (prompt_toolkit suppresses stdout)

### Build System
- **Build order**: protocol → adapters/cli/web-client (`npm run build` handles this)
- **Module resolution**: `nodenext` for Node packages, `bundler` for Vite frontends
- **Linting**: ESLint (TypeScript) + ruff (Python, black-compatible)
- **Pre-commit hook**: husky runs lint + build on every commit
- **CI**: lint + build + test (parallel Node/Python jobs)
- **npm scope**: `@axolotlai` (packages: arc-protocol, arc-cli, arc-adapter-*)
- **PyPI**: `arc-relay` (version from git tags via setuptools-scm)

## Config Files

- `~/.arc/config.json` — CLI config (relayUrl, agentToken, framework, hosted, e2e). Mode `0o600`.
- `~/.arc/session.json` — Current session info (sessionId, sessionSecret, viewerUrl)
- `~/.arc/viewer-url` — Plain text viewer URL (for agent consumption)
- `~/.arc/relay.pid` — PID of relay started by `arc setup`
- `~/.arc/plugin.log` — Hermes plugin debug log (tail -f to debug)
- `~/.hermes/skills/devops/remote-control/SKILL.md` — Hermes skill
- `~/.hermes/plugins/arc-remote-control/` — Hermes plugin (symlink to repo)

## Development

```bash
# First-time setup
npm install && npm run build
pip install -r relay/requirements.txt

# Or use the CLI
./install.sh --local && arc setup --hermes

# Run relay locally
ARC_ENV=dev arc setup --self-hosted
# or: python -m relay
# or: docker compose up relay

# Run web viewer (hot reload)
npm run dev:viewer

# Lint
npm run lint                    # ESLint + ruff

# Tests
npm run test:ts                 # TypeScript (protocol + crypto)
npm run test:py                 # Python (relay + hosted + plugin + integration)
npm run test:plugin             # Hermes plugin tests only
npm test                        # All tests

# Deploy beta
./hosted/deploy.sh beta --skip-tests
```

## Test Coverage

Python tests:
- `relay/tests/test_relay.py` — OSS relay unit tests
- `relay/tests/test_prefix_auth.py` — Prefix-based token auth (12 tests)
- `hosted/backend/tests/test_hosted.py` — Hosted provider unit tests
- `hosted/backend/tests/test_redis_store.py` — Redis store persistence + pub/sub (19 tests)
- `hermes-plugin/tests/test_reconnect.py` — Plugin reconnect loop (6 tests)
- `hermes-plugin/tests/test_e2e.py` — Plugin E2E encryption (12 tests)
- `tests/test_api_contracts.py` — Protocol interface contract enforcement
- `tests/test_websocket_relay.py` — End-to-end WebSocket relay integration

TypeScript tests:
- `packages/protocol/tests/test-helpers.mjs` — Wire type helpers
- `packages/protocol/tests/test-base-adapter.mjs` — BaseAdapter validation
- `packages/protocol/tests/test-crypto.mjs` — E2E encryption (16 tests)

## Known Issues / Next Steps

### Hermes integration
- **`/` slash commands from viewer** may silently fail for CLI-internal commands
- **`/` autocomplete in viewer** — needs `capabilities` trace event
- **`hermes update` overwrites ARC entrypoint patch** — run `arc update` to re-apply
- **Clarify from viewer** works but CLI picker stays visible briefly until response_queue resolves

### Relay
- **Security hardening**: no per-IP connection limit, no auth timeout, X-Forwarded-For trusted unconditionally
- No "Active Sessions" dashboard in hosted frontend
- No push notifications for "agent needs input"

### Remaining roadmap
- Mobile app (iOS/iPadOS)
- Multi-agent dashboard view
- Canvas/A2UI rendering in web client (for OpenClaw)
- Persistent trace storage (long-term history)
- Publish `@axolotlai/arc-cli` to npm
- Publish `arc-relay` to PyPI

## Instructions for Agents

1. **Read this file first** to understand architecture, conventions, and current state.
2. **Build before committing**: `npm run build` (pre-commit hook enforces this).
3. **Write tests for new features**: every major feature or behavioral change must include
   unit tests. The pre-commit hook runs lint + build but NOT tests —
   run tests manually: `npm run test:ts && npm run test:py`.
4. **Update this file** when you change features, commands, config, build, or known issues.
5. **Hermes clarify**: bypasses `pre_tool_call` — patched via `tools.clarify_tool.clarify_tool`
   monkeypatch (deferred to first use since module is lazy-loaded).
6. **E2E encryption** is a hosted feature — see `hosted/AGENTS.md` for implementation details.
7. **Viewer URL** must use `/viewer` path prefix. Assets use relative paths (`./`).
8. **Debug the plugin**: `tail -f ~/.arc/plugin.log` — prompt_toolkit suppresses stdout.
9. **Multi-instance** is a hosted feature — see `hosted/AGENTS.md`.

---
> Source: [axolotl-ai-cloud/arc](https://github.com/axolotl-ai-cloud/arc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
