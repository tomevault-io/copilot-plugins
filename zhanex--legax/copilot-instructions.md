## legax

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Commands

This is a **dependency-free Node.js project** (no `node_modules`, no bundler). Everything runs against the Node 18+ standard library.

```bash
npm run ci                            # full CI gate: npm test + check:docs
npm test                              # local code gate: check:node + check:shell + test:e2e
npm run check:node                    # node --check on every script (syntax only)
npm run check:docs                    # docs/i18n/secret/BOM gate (scripts/check-docs.mjs)
npm run check:shell                   # bash -n on installer + service scripts
npm run test:e2e                      # node --test over tests/e2e/*.e2e.mjs (excludes real-clients)
npm run test:e2e:real                 # opt-in: requires LEGAX_REAL_CLIENTS=1 and real codex/Codex/gemini CLIs on PATH
npm run test:real:messages            # real-local-message smoke test
```

Run a single E2E file: `node --test tests/e2e/<name>.e2e.mjs`. Run a single test inside it: `node --test --test-name-pattern '<substring>' tests/e2e/<name>.e2e.mjs`.

`npm run ci` is the contract enforced by CI; if you add a new script or test file, also append it to the corresponding `check:node` / `test:e2e` list in `package.json` — the lists are explicit, not glob-driven.

Dry-run the supervisor without spawning real CLIs: `node scripts/legax-daemon.mjs --dry-run`.

## Contribution and commits

Follow `.github/CONTRIBUTING.md` when preparing changes for commit:

- Treat `npm run ci` as the full merge gate (`npm test` plus `check:docs`). For targeted bugfix work, run the narrow regression tests first, then the relevant broader gate, and state any full-gate failures explicitly instead of implying a clean CI.
- Documentation-only commits may use `npm run check:docs` locally, but code commits should at least run the affected E2E file(s), `npm run check:node`, and any daemon/transport integration tests touched by the change.
- Keep documentation pairs in the same commit: English `.md` with `.zh-CN.md`, and `config.example.yaml` with `config.example.zh-CN.yaml`.
- Before staging, inspect `git status --short` and the full diff. Do not stage local runtime artifacts, large binaries, `data/` residue, machine-specific config, lock files, or tracked secrets.
- Commit subjects should be short imperative sentences. Conventional Commits are not required; a topic prefix is preferred when it helps scanning, for example `telegram: refresh active pinned context`, `daemon: route opencode launches`, or `docs: clarify relay pairing`.
- Use a commit body only when the *why* is not obvious from the subject and diff.

## Architecture

The project is a local-first relay layer that bridges coding-agent CLIs (Codex, Claude Code, Gemini CLI, OpenCode) to a phone via self-hosted relay, Telegram, or webhook. Read `docs/ARCHITECTURE.md` first — the design uses **three planes** that must not be conflated when changing code:

- **Control plane (CLI adapters)** — owns process lifecycle, session selection/continuation, and structured-output parsing. One adapter per agent under `scripts/`: `codex-app-server-link.mjs` (JSON-RPC over WebSocket app-server), `claude-code-link.mjs` (`claude -p` stream-json), `gemini-cli-link.mjs` (`gemini` stream-json), `opencode-link.mjs` (OpenCode HTTP server). Each adapter is a long-lived process; the daemon supervises them.
- **Capability plane (MCP)** — `scripts/mcp-server.mjs` is a generic stdio MCP server exposing `legax_send/poll/request_permission/status`. `scripts/claude-permission-mcp-server.mjs` is a Claude-specific permission-prompt MCP that mirrors permission asks to the phone and returns the decision through Claude's permission hook. MCP is a **capability layer, not a lifecycle manager** — never use it to start/stop processes.
- **Communication plane (transports)** — `scripts/lib/outbound-transports.mjs` (relay POST, Telegram sendMessage, generic webhook) and `scripts/lib/inbound-transports.mjs` (Telegram parsing/routing helpers). The daemon owns relay `/api/messages` polling and Telegram `getUpdates` while running, writes messages into per-agent inbox queues, and adapters launched by the daemon only drain their inbox. Standalone adapters use a single-poller fallback.

The **unified daemon** (`scripts/legax-daemon.mjs`) is what users normally run. It reads one `config.yaml`, owns remote inbound routing, supervises all enabled adapters, restarts crashed ones with bounded backoff, writes per-adapter MCP config (`mcpAutoConfigure`) before launch, and handles **on-demand launches** for `autoStart: false` adapters via launch requests in runtime state. If only an individual adapter is running, it cannot start siblings.

### Cross-process coordination via runtime state

`scripts/lib/runtime-state.mjs` is the **single source of truth shared between the daemon and every adapter**. It persists adapter cursors, dynamic modes, Telegram chat selections, selected Codex thread metadata, per-agent inbound queues, and pending launch requests to `data/runtime-state.json` (configurable via `runtimeStatePath`). Writes use a temp-file + retrying atomic rename to tolerate concurrent writes on Windows (EPERM/EACCES/EBUSY). When you add new cross-process state, extend this module — do not invent a sibling state file. The relay's own state lives separately at `data/relay-store.json`; the generic MCP server's state at `data/mcp-state.json`.

### Per-adapter session model (these differ and matter)

- **Codex**: existing-session mode connects to a shared `codex app-server --listen ws://...` over WebSocket (`cliBackend: app-server-ws`). Local visibility requires the user to start `codex --remote ws://127.0.0.1:18779`; the desktop app's embedded stdio app-server is **not** a shared backend. Approval uses JSON-RPC `requestApproval`.
- **Claude Code**: launches `claude -p` with stream-json. Existing-session mode adds `--continue` or `--resume <id>` and discovers sessions from local persisted JSONL history. Permission is delivered through `claude-permission-mcp-server.mjs` configured as Claude's permission-prompt MCP.
- **Gemini CLI**: one headless turn per phone message via `--prompt`; resume is `--resume latest` or a configured id; sessions come from `gemini --list-sessions`. Daemon/headless runs in untrusted directories require `trustWorkspace: true` (sets `GEMINI_CLI_TRUST_WORKSPACE=true`).
- **OpenCode**: connects to `opencode serve` through `cliBackend: server-http`; `serverMode: connect-or-start` can start the server on demand. Sessions come from `GET /session`; phone text goes to `POST /session/:id/message`. OpenCode-native permission callback bridging is not implemented yet.

Phone approval decisions are honored only in `interactive` and `approval-only` modes. `paused` is a hard stop and cannot be cleared by selecting an adapter — only `/mode <agentId> interactive` clears it. Selecting an adapter from Telegram/phone activates `interactive` for non-paused adapters and returns its session list.

### Hand-rolled YAML parser

There is **no YAML library**. Repo scripts use the shared hand-rolled parser in `scripts/lib/yaml.mjs`; the standalone deployable relay keeps an inline copy because the installer can copy just `self-hosted-relay/server.mjs`. The parser supports the subset the example config uses: top-level scalars, nested objects two levels deep, the `transports:` list with sub-lists, and inline comments outside quotes. **JSON config is intentionally not supported** — users must use YAML. If you need a new config shape, extend the shared parser and any standalone inline copies in lockstep, then add coverage.

## Documentation rules (enforced by `check:docs`)

- Every prose `.md` and every `config.example*.yaml` file must ship as a **language pair**: `*.md` (English) + `*.zh-CN.md` (Simplified Chinese), `config.example*.yaml` + matching `config.example*.zh-CN.yaml`. When you change one, change the other in the same commit.
- Files must be UTF-8 **without BOM**. Do not mix English prose into a `.zh-CN.md` (or vice versa) beyond short cross-links, product names, CLI commands, config keys, and code blocks.
- The doc gate also scans for accidentally committed Telegram bot tokens, OpenAI-style `sk-...` keys, and GitHub `gh[pousr]_...` tokens. Use placeholders (`replace-with-a-long-random-secret`, `YOUR_RELAY_HOST`, `TELEGRAM_BOT_TOKEN`).
- See `docs/DOCUMENTATION.md` for the full review checklist.

## Testing notes

- E2E tests spawn real adapter processes and a fake relay/Codex app-server (`tests/e2e/fixtures/`). They share helpers in `tests/e2e/helpers.mjs` for spawning, port allocation, and temp config generation.
- `tests/e2e/real-clients.e2e.mjs` is gated by `LEGAX_REAL_CLIENTS=1` and only runs when actual `codex`, `Codex`, and `gemini` binaries are on PATH; it is not part of `npm test`.
- On Windows, when direct `codex` calls are blocked by execution policy, set `codex.command: codex.cmd` in test/dev config.

## Permission model (do not violate)

Legax **mirrors** native approval prompts to the phone and returns the decision through the agent's structured callback when the adapter supports one (Codex JSON-RPC, Claude permission-prompt MCP, Gemini's own approval mode). It must never simulate UI clicks, auto-approve a native prompt, or bypass an agent's security policy. TUI/PTY backends are a fallback only and must be treated as high-trust remote terminal control.

---
> Source: [zhanex/legax](https://github.com/zhanex/legax) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
