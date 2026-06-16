## otto

> Build Otto as a secure, debuggable remote browser automation platform (controller -> relay -> extension node).

# AGENTS

## Mission
Build Otto as a secure, debuggable remote browser automation platform (controller -> relay -> extension node).

## Invariants
- Always require `targetNodeId` on commands. If there's only one connected node, CLI can auto-select it.
- Preserve terminal command outcomes: `completed`, `failed`, `timed_out`, or `cancelled`.
- Keep per-tab session execution serial and cross-tab execution parallel.
- Apply pre-ingress redaction for logs.
- Keep `chrome.debugger` features behind explicit opt-in flags.
- Keep command execution site-scoped and validate tab URL before running command logic.
- Keep `command.test` streaming command-native via returned stream listener manifests; avoid site-specific runtime listener managers.
- Keep runtime listener infrastructure generic (for example `network.http_intercept`) and keep site-specific stream parsing/fallback policy in command modules.
- Validate declared command input metadata before command execution; command handlers should receive sanitized input.
- For `otto test`, support optional command-level test hooks with execute fallback for simple commands.
- Never automate user credential submission; use explicit manual login handoff (`manual_login_required`).
- Keep listener lifecycle deterministic: `listener.subscribe` and `listener.unsubscribe` are terminal commands, while async `listener_update` events are correlated by the original subscribe `requestId`.
- Keep controller-to-node authorization node-owned: registered controller clients require explicit per-node ACL grants from the authenticated node before command routing.
- Keep `otto test` teardown signal-safe: Ctrl+C/SIGTERM should trigger `command_cancel` for active stream tests and attempt auto-opened `primitive.tab.close` before exit.
- Keep orphan cleanup owner-scoped: when controller sessions disconnect/time out, close only tabs owned by that controller identity.

## Canonical Commands
- Install deps: `npm install`
- Build all: `npm run build`
- Check types: `npm run check`
- Lint all: `npm run lint`
- Test all (where present, concise output): `NODE_OPTIONS='--test-reporter=dot' npm run -ws --if-present --silent test`
- Start relay daemon: `otto start`
- Start relay attached with logs (dev): `otto start --attached`
- Restart relay daemon: `otto restart`
- Stop relay daemon: `otto stop`
- Start MCP server: `otto mcp`
- Register with agent framework: `otto agent install <runtime>`
- Agent framework status: `otto agent status`
- Manual E2E harness: `npm run e2e:manual`
- Run relay dev: `npm run dev:relay`
- Run CLI dev: `npm run dev:cli`
- Run extension dev: `npm run dev:ext`
- Run docs dev: `npm run docs:start`
- Build docs site: `npm run docs:build`

## CI and Release Workflows
- Use Conventional Commits (`fix:`, `feat:`, `docs:`, etc.) so release-please can generate changelogs and version bumps correctly.
- CI package quality gates live in `.github/workflows/ci.yml` and should keep per-package checks independent.
- Release Please plus publish/asset automation lives in `.github/workflows/release-please.yml` and `release-please-config.json`.
- The release-please workflow includes a reconcile step that ensures merged release PRs always get their git tags and GitHub releases, even when release-please-action v4 skips tag creation. If a merged PR still has `autorelease: pending`, the reconcile step creates the tag and release and relabels the PR to `autorelease: tagged`.
- Publish and extension asset upload jobs use a `determine-release-tag` passthrough so they trigger from both release-please-created releases and reconciled (recovered) releases. The gate outputs an empty `release_tag` on ordinary pushes to main (no new release), which causes publish and upload jobs to skip. This prevents spurious publishes on non-release pushes.
- Keep semver-tagged extension assets attached to `v<version>` releases from the consolidated release workflow so `otto setup` download contract remains valid.
- Docs site deploy to GitHub Pages on docs path changes lives in `.github/workflows/docs-pages.yml`.

## Required Validation After Any Update
- Always run, in order, at the end of any code update:
- `npm run check`
- `npm run lint`
- `npm run build`
- `NODE_OPTIONS='--test-reporter=dot' npm run -ws --if-present --silent test`
- If a command fails, fix the issues and re-run until all pass.
- Write new tests for any new features or edge cases, and ensure they pass reliably.
- Update the documentation extensively to reflect any changes in behavior, architecture, or developer experience.
- For big changes, where it makes sense, also make concise updates to AGENTS.md.
- Never leave debug artifact files anywhere in the repo. This includes files like `test_results.txt`, `*.log`, scratch `*.json`, or any `*.html`/`*.md` outside `docs/` — at any directory level (repo root, `extension/`, `packages/`, etc.). Delete them immediately after use and do not commit them.

## Where To Change Things
- Protocol contracts: `packages/shared-protocol/src/index.ts`
- Relay routing and locks: `packages/relay/src/index.ts`
- CLI UX and command entrypoint: `packages/cli/src/index.ts`
- Extension SW/offscreen runtime: `extension/entrypoints/background.ts`, `extension/src/runtime/offscreen-client.ts`
- Extension network interception listeners: `extension/src/runtime/network-intercept/listener.ts`, `extension/src/runtime/listener-managers.ts`
- Extension popup onboarding UI: `extension/entrypoints/popup.html`, `extension/src/runtime/popup-ui.ts`, `extension/src/runtime/onboarding/ui.ts`
- Extension command runtime orchestration: `extension/src/runtime/command-runtime.ts`
- Extension command bundles and registry: `extension/src/commands/**`
- Extension settings UI: `extension/entrypoints/options.html`, `extension/src/runtime/options-ui.ts`

## Safety
- Do not add remote code execution paths.
- Do not log tokens/cookies/raw sensitive payloads by default.
- Enforce auth and role checks before routing commands.
- Keep command input handling bounded and deterministic; avoid unbounded page scraping loops.
- Preserve legacy command compatibility aliases only when explicitly documented and tested.

## File Naming Convention
- Use kebab-case for all source and test filenames.
- Prefer directory namespacing over prefix namespacing for feature families (for example `onboarding/ui.ts` over `onboarding-ui.ts`).
- Keep NodeNext ESM import specifiers explicit and aligned with moved paths (for example `./feature/file.js`).

## Local Dev Log Streaming (Extension -> Relay)
- In local development flows, extension runtime can stream structured extension logs to relay when `localDevLogStreamingEnabled` is set in extension storage.
- Start condition: after relay URL is saved in popup/options, extension starts queueing log objects immediately and flushes them after socket auth succeeds.
- Relay stores these as `source: node` events and concatenates them with relay-native logs for `otto logs` APIs and follow streams.
- CLI log source filters now support `relay|controller|node|all`; use `node` to inspect extension-originated runtime logs.
- Relay operation logs are persisted in day-windowed JSONL files with size-based spillover (`OTTO_LOG_MAX_FILE_BYTES`) and 14-day file retention.
- Keep this feature for local debugging only; do not enable broad sensitive payload logging.

## Agent Debugging Guidance
- When debugging extension-relay behavior locally, prefer `otto logs follow --source all` in one terminal while reproducing in extension UI.
- For extension-only traces, use `otto logs follow --source node` or `otto logs list --source node --latest 50`.
- Validate that early extension logs appear before `authenticated_connected` transitions during relay URL + pairing workflows.
- For autonomous/CI agents, prefer non-TTY invocation and `--json` output for machine parsing.
- Treat `otto logs follow` as unbounded: enforce a caller-side timeout window for live capture sessions.
- For long-running controller stream sessions (for example `otto test` streaming commands), keep websocket heartbeat `ping`/`pong` active; stale controllers should be treated as disconnected and cleaned up by relay.
- Correlate failures by `requestId` first, then narrow by source (`all` -> `node` or `relay`) to isolate root cause.

## Debugging Tools That Worked Best
- `otto listener subscribe-network`
	- Use case: prove raw network capture before debugging command stream plumbing.
	- Example: `otto listener subscribe-network --tab-session <id> --site reddit.com --request-host matrix.redditspace.com --pattern 'https://matrix.redditspace.com/_matrix/client/v3/*' --mode network --max-body-bytes 200000`
- `otto test <site> <command> --json`
	- Use case: validate command-level stream manifests and controller-visible frames.
	- Example: `otto test reddit.com getChatMessages --stream-follow-ms 45000 --json`
- `otto test ... --stream-probe`
	- Use case: force immediate traffic after subscribe to validate listener wiring.
	- Example: `otto test reddit.com getChatMessages --stream-probe --stream-follow-ms 45000 --json`
- `otto logs list --source node --latest <n>`
	- Use case: inspect extension-side runtime events during stream failures.
	- Example: `otto logs list --source node --latest 300`
- `otto logs follow --source all`
	- Use case: correlate relay/controller/node events live by `requestId`.
	- Example: `otto logs follow --source all`
- `otto cmd --action primitive.tab.open`
	- Use case: mint a fresh managed `tabSessionId` after reloads and avoid stale-session false negatives.
	- Example: `otto cmd --action primitive.tab.open --payload '{"url":"https://chat.reddit.com"}'`
- `otto commands list`
	- Use case: quick health check for auth, routing, and node reachability before deeper debugging.
	- Example: `otto commands list`

## CLI TUI Guidance
- For any CLI TUI updates, prefer `@inkjs/ui` components and patterns over bespoke terminal rendering.
- Keep non-interactive output deterministic and machine-readable; TUI polish must not change JSON contracts.
- Retain existing keyboard escape affordances (`q`, `Esc`, `Ctrl+C`) in interactive screens.
- Keep custom Ink rendering only where needed for streaming behavior (for example, `Static` log follow output).

## Docs Site Guidance
- Docusaurus site root: `website/`
- Docs content source for site navigation: `docs/**` (repository root)
- Keep docs information architecture aligned with sections: Getting Started, Guides, Reference, Technical, Contributing.
- Keep `website/src/css/custom.css` aligned with Snoopy docs theme values unless a deliberate design update is requested.

## Reference Docs
- Architecture: `docs/overview.md`
- Protocol semantics: `docs/protocol.md`
- Pairing/auth lifecycle: `docs/guides/pairing-auth.md`
- Extension runtime behavior: `docs/extension-runtime.md`
- Relay operations: `docs/relay-operations.md`
- Testing matrix: `docs/testing.md`
- Security controls: `docs/security.md`
- Docusaurus docs home source: `docs/index.md`

## MCP and Agent Integration Sync Policy
- When MCP tool surface changes, update in the same change: `packages/cli/src/mcp/tools.ts`, `packages/cli/src/mcp/server.ts`, `docs/for-agents/mcp-server.md`, and `skill/otto-cli/SKILL.md`.
- When agent install targets change, update `packages/cli/src/agent/install.ts`, `docs/for-agents/agent-setup.md`, and `skill/otto-cli/SKILL.md`.
- Keep `skill/otto-cli/references/command-catalog.md` synchronized with actual CLI command surface.

---
> Source: [telepat-io/otto](https://github.com/telepat-io/otto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
