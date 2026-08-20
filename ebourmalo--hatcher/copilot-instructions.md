## hatcher

> Guidance for AI agents working on this repository.

# AGENTS.md

Guidance for AI agents working on this repository.

## Project Summary

Hatcher is a TypeScript CLI that deploys local AI agent runtimes to cloud sandbox providers.

Supported runtimes:
- `hermes`
- `claude-code`

Supported providers:
- `cloudflare`
- `daytona`

Core contract:
- `hatcher.json` is the source of truth for agent identity, runtime, model, skills, provider, and `deployment.env`.
- `identity.file` is the runtime identity/persona file staged into the image, for example `CLAUDE.md` for Claude Code or `SOUL.md` for Hermes.
- `.hatcher/` is generated deployment output. Do not treat it as source unless debugging a generated artifact.
- `.hatcher/state.json` stores the current deployment/session state.
- Secrets must never be written into `hatcher.json`.

## Common Commands

Use these from the repository root:

```bash
npm run typecheck
npm run build
npm test
npm run coverage
```

Focused tests are preferred while iterating:

```bash
npm test -- tests/docker/cloudflare-worker.test.ts
npm test -- tests/cli/cloudflare-terminal.test.ts
npm test -- tests/tui/operations.test.tsx
npm test -- tests/sandboxes/provider-contract.test.ts
npm test -- tests/e2e/deploy.matrix.test.ts
```

Optional real-local Hermes smoke test:

```bash
npm run test:local-hermes
```

Only run provider commands such as `wrangler deploy`, `wrangler delete`, `daytona sandbox create`, or raw `hatcher deploy` when explicitly requested. They can mutate remote infrastructure and cost real money.

`hatcher test image` builds a local Docker image and runs a readiness smoke test. It does not deploy, but it can be slow and requires Docker, so run it only when image behavior is relevant or explicitly requested.

For changes that touch deploy / probe / destroy semantics, prefer the
end-to-end harness over ad-hoc `hatcher deploy` runs:

```bash
npm run e2e -- --combo claude-code:cloudflare
npm run e2e -- --all
```

The harness lives at `tools/e2e/` (see `tools/e2e/README.md` for the
full lifecycle and flags). Every run writes a `report.json` per cell
plus a `summary_report.json` for the run under `tools/e2e/runs/`, so
you can answer "did it pass and which assertions failed?" with one
`jq` call instead of re-deploying to scrape stdout. Use `--keep` to
retain failed sandboxes for live debugging, and always tear them
down with `hatcher destroy` from the retained cwd when done.

## Repository Map

- `src/cli/`: command implementations and CLI entrypoint.
- `src/tui/`: Ink-based TUI components and operation runner.
- `src/agents/`: runtime detection and runtime-specific local config parsing.
- `src/sandboxes/`: provider implementations for deploy, destroy, connect, and validation.
- `src/docker/`: Dockerfile template resolution, generated Cloudflare Worker source, and `.hatcher/` artifact generation.
- `src/schema/`: `hatcher.json` Zod schema.
- `src/session/`: local deployment state management.
- `docker/`: Dockerfile templates copied into generated deployment contexts.
- `tests/`: unit, integration, e2e, and in-container tests.

## CLI UX Rules

The CLI uses Ink, not `pi-tui`.

For commands other than no-op/help flows:
- Show what the command will do before executing.
- Ask one top-level confirmation unless `--yes` is passed.
- Show a persistent operation list with status markers.
- External commands must be visible before confirmation.
- `--verbose` streams command output below the operation list.
- Non-TTY fallback must still show static operation labels and statuses.

Operation markers:
- pending: `·`
- running: spinner
- succeed: `✓`
- failed: `✗`

Do not reintroduce per-command prompts like `Run: npm install ...?`. External commands are authorized by the single top-level confirmation.

## Deployment Model

`hatcher deploy` should:
1. Load and validate `hatcher.json`.
2. Sync runtime config into `hatcher.json` when needed.
3. Resolve env vars listed in `deployment.env`.
4. Regenerate `.hatcher/`.
5. Check provider auth and existing deployment state.
6. Deploy or update through the provider.
7. Record `.hatcher/state.json`.
8. Validate remote access when supported.

`deployment.env` is the single source of truth for sandbox env injection.

Runtime file mappings live in `src/agents/*` through `RuntimeConfig`. The expected file flow is:

```text
runtime-defined source paths -> .hatcher/ staging -> Docker image runtime paths
```

Dockerfiles must only copy from `.hatcher/`. Do not make Docker templates depend on files in the original runtime source folder or repository root. Required runtime files must fail with explicit errors when missing; optional runtime files may be skipped.

Claude Code runtime files are sourced from `~/.claude` and `~/.claude.json` by default. Hermes runtime files are sourced from `~/.hermes` by default. Optional bundles (e.g. Hermes `~/.hermes/auth.json` for OAuth-only models) are declared in the runtime's `additionalFiles[]` and copied into `.hatcher/` only when the host file is present — never required, never failing.

A directory under the runtime's skills folder counts as a declared skill in `hatcher.json` only when it contains a `SKILL.md` (directly or in any descendant). Category-only directories that hold a `DESCRIPTION.md` and no `SKILL.md` anywhere are skipped — they are not runnable skills and would only create phantom entries.

`hatcher test image` should:
1. Load and validate `hatcher.json`.
2. Resolve env vars listed in `deployment.env`.
3. Regenerate `.hatcher/`.
4. Build the Docker image locally.
5. Run a non-network smoke script inside the image.

The image smoke script should verify runtime binary availability, expected config paths, declared skill files, required env vars, and provider-specific hooks such as the Hermes Cloudflare tmux wrapper. Do not add live LLM calls to the default image test.

Env var resolution should read sources in this order, with the first non-empty value winning:
1. the runtime-specific `.env` file, when available (e.g. `~/.hermes/.env`);
2. the project's `.hatcher/.env`;
3. the current process environment;
4. for the `claude-code` runtime on macOS, `tryExtractClaudeCodeKeychainToken()` in `src/env/local-secrets.ts` — reads the user's `Claude Code-credentials` keychain entry and surfaces the access token as `CLAUDE_CODE_OAUTH_TOKEN`.

`ANTHROPIC_API_KEY` and `CLAUDE_CODE_OAUTH_TOKEN` are alternatives for the `claude-code` runtime: either one satisfies the auth requirement and gets injected into the sandbox under its real name. The alternatives list lives in `CLAUDE_CODE_AUTH_ENV_VARS` (`src/env/deployment.ts`) and is consulted by `envAlternatives()`.

When logging resolved env vars, only show a safe suffix preview, never the full value.

## Cloudflare Notes

Cloudflare deployment generates:
- `.hatcher/wrangler.jsonc`
- `.hatcher/src/index.ts`
- `.hatcher/package.json`
- `.hatcher/hermes-tmux-shell` for Hermes

Cloudflare names use the stable `agent-{identity.name}` convention.

Cloudflare Worker behavior:
- Protect terminal/debug endpoints with `REMOTE_SHELL_AUTH_TOKEN`.
- Return `500 Misconfigured deployment - REMOTE_SHELL_AUTH_TOKEN missing` if the token is absent.
- Use `getSandbox(..., { keepAlive: true })`.
- Apply env vars with `sandbox.setEnvVars(...)`.
- Use one named interactive session, currently `agent`.
- Reset the named terminal session lazily when deployment revision changes.
- Expose authenticated debug routes for env/session diagnostics, including a generic `/debug/exec` that runs a single command via `session.exec()` and returns `{ stdout, stderr, exit }`. All `/debug/*` routes live inside the `HATCHER_DEBUG_START` / `HATCHER_DEBUG_END` markers and are stripped from production builds by `stripDebugSections()`; they only ship in `hatcher deploy --debug` builds. Treat the token check + the marker strip as a single security contract — never weaken either without weakening the other.

Cloudflare terminal behavior:
- `hatcher connect` connects to `/ws/terminal` over WSS.
- The CLI sends local terminal `cols` and `rows` in the WebSocket URL.
- The Worker must pass client-provided `cols` and `rows` into `session.terminal(...)`.
- `--debug` on `hatcher connect` should show WebSocket close/error/reconnect diagnostics.
- Ctrl-C and Ctrl-D should detach locally, not stop the remote Hermes agent.

Hermes on Cloudflare:
- Uses `/opt/hatcher/hermes-tmux-shell`.
- The tmux session owns the Hermes PTY so reconnects attach to the same interactive session.
- The tmux config may improve wheel/copy-mode behavior, but CLI scrollback for full-screen/TUI output is inherently imperfect.

Tmux session naming is a cross-component contract: every runtime's Dockerfile template launches the agent under `tmux new-session -d -s hatcher <agent>` so `hatcher connect` (via `/ws/terminal`) and the e2e harness (via `tmux send-keys` / `capture-pane` over `/debug/exec`) can target it by name. Do not rename the session without updating both the templates and the harness.

## Daytona Notes

Daytona uses a replace-style redeploy strategy.

Daytona connect uses an inherited stdio command path. Keep Cloudflare WebSocket terminal logic separate from Daytona behavior.

## Extension Contracts

To add an agent runtime:
1. Add the runtime id to `AgentRuntimeSchema` in `src/schema/hatcher.ts`.
2. Run `npm run typecheck` and `npm test`.
3. Follow the runtime contract failures until the new runtime is fully implemented.

To add a sandbox provider:
1. Add the provider id to `EnvironmentProviderSchema` in `src/schema/hatcher.ts`.
2. Run `npm run typecheck` and `npm test`.
3. Follow the provider contract failures until the new provider is fully implemented.

The contract tests enforce the shared extension requirements. Add specific tests only for behavior that is unique to the new runtime or provider.

## Docker Notes

Do not assume Dockerfile templates are interchangeable. Runtime/provider combinations have different constraints.

Cloudflare builds from inside `.hatcher/`, so Dockerfile `COPY` paths must be relative to that generated context.

All Dockerfile `COPY` sources must be staged files or directories inside `.hatcher/`. Update the artifact generator and runtime `RuntimeConfig` mapping instead of copying directly from runtime homes such as `~/.hermes`.

Hermes Cloudflare has a debug Dockerfile selected by `hatcher deploy --debug`.

Avoid unpinned deployment tooling. Generated `.hatcher/package.json` should use pinned versions for Cloudflare dependencies.

## Testing Expectations

Add or update tests when changing:
- schema behavior;
- generated Dockerfiles;
- generated Cloudflare Worker source;
- CLI operation plans;
- env var resolution;
- deployment/session state;
- terminal connect behavior.

Useful focused suites:
- Worker generation: `tests/docker/cloudflare-worker.test.ts`
- Docker template generation: `tests/docker/compose.test.ts`, `tests/docker/write.test.ts`
- Terminal client: `tests/cli/cloudflare-terminal.test.ts`
- TUI operation runner: `tests/tui/operations.test.tsx`
- Provider matrix (hermetic, no real cloud): `tests/e2e/deploy.matrix.test.ts`
- Env var resolution + helpers: `tests/env/deployment.test.ts`, `tests/env/local-secrets.test.ts`
- Skill / runtime detection: `tests/agents/detect.test.ts`

When the change touches deploy/probe/destroy semantics end-to-end, also run the live harness (`npm run e2e -- --combo …`) and attach the relevant `tools/e2e/runs/<cell>/report.json` (or `summary_report.json`) to the PR. Do not embed real tokens, sandbox URLs, or account ids from those reports — redact before sharing.

Before handing off substantial changes, run:

```bash
npm run typecheck
npm run build
npm test
```

If the full suite is too broad for the change, run the focused suite and state what was not run.

## Coding Rules

- Keep TypeScript strict and explicit.
- Preserve ESM imports with `.js` extensions for local runtime imports.
- Prefer small, testable helpers over embedding behavior directly in CLI command bodies.
- Keep command execution separate from UI rendering.
- Do not leak secret values in logs, state, generated config, or tests.
- Do not edit generated `.hatcher/` files as the primary fix; update the generator instead.
- Do not change remote infrastructure behavior without updating deploy/destroy/status tests.
- Do not remove safety confirmations unless `--yes` explicitly covers the flow.

## Local Development Safety

This repo may have a dirty worktree. Do not revert unrelated changes.

Never run destructive commands such as `git reset --hard`, `git checkout --`, or provider destroy/delete commands unless explicitly requested.

When changing generated output, update the source generator and its tests, not just the generated artifact.

---
> Source: [ebourmalo/hatcher](https://github.com/ebourmalo/hatcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
