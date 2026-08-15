## agents-md

> Pi-Atlas is the Pi coding agent's browser/desktop automation and web search extension. It provides CDP-based browser control, agent-browser integration, desktop automation (via Cua Driver MCP), hybrid search (BM25 + vector embedding + RRF fusion), social/reach tools, and cookie/auth management.

# Agent Reference: Pi-Atlas

Pi-Atlas is the Pi coding agent's browser/desktop automation and web search extension. It provides CDP-based browser control, agent-browser integration, desktop automation (via Cua Driver MCP), hybrid search (BM25 + vector embedding + RRF fusion), social/reach tools, and cookie/auth management.

## Sister Repos

### No protocol-level dependencies on the other Pi repos.
Pi-Atlas is self-contained. It does not consume `@rhinos0608/pi-workspace-protocol`, Pi-SmartRead, or Pi-SmartEdit directly.

## Operational Contracts and Invariants

### ❌ OPEN: Containerization verification for network containment
**This is an unresolved item — do NOT resolve it silently.**

- `src/http.ts:validatePublicHttpUrl` (now aliased as `validateHttpUrl`) performs **no** private/reserved-IP blocking. SSRF protection was *deliberately removed* (commits `a0fad0e`, `04f373d`) with the intent that network containment is handled by external containerization.
- `src/browser-policy.ts:checkDomainAllowed`, `validateAllowedDomainsDns`, and `dnsPreflight` are all **intentional no-ops** with JSDoc stating "containerization handles containment."
- `docs/adr/0001-use-agent-browser-for-web-automation.md` documents the domain/SSRF removal as SUPERSEDED.

**What this means:** If Pi-Atlas is ever run *outside* a container with egress restrictions, there is zero protection against SSRF (metadata endpoints, localhost, RFC1918, link-local). If Pi-Atlas is running inside a properly configured container, this is fine.

**Action required:** Explicit confirmation from the repo owner on whether:
1. Pi-Atlas is **always** deployed in a container with egress restrictions, OR
2. A follow-up network-policy project is needed to restore defense-in-depth IP/domain blocking.

**Until confirmed, do NOT restore SSRF/domain blocking as a 'fix'** — the existing posture is deliberate, not an oversight. If you believe blocking should be restored, escalate as a product-scope decision, not a routine code change.

### Python child processes MUST use the shared env allowlist
`src/python-child-env.ts` exports `buildPythonChildEnvironment()` — a sanitized environment with an allowlist of benign system/PI vars and a `BLOCKED_PATTERN` excluding TOKEN/KEY/SECRET/COOKIE/PASSWORD/API_KEY/API_SECRET/AUTH/BEARER and NODE_OPTIONS, NODE_PATH, PYTHONPATH, GIT_CONFIG_, SSL_CERT_, LD_PRELOAD, DYLD_ patterns.

**All three Python spawn sites use it:**
- `src/scrapling-bridge.ts:384` — `spawnProcess` (per-fetch Python bridge)
- `src/scrapling-bridge.ts:528` — `oneShotCommand` (health check)
- `src/sidecar-manager.ts:117` — `SidecarManager.start` (embedding sidecar)

**When adding a new Python child process:** always pass `env: buildPythonChildEnvironment()`. Never default to inheriting `process.env` — full environment leakage exposes API keys, cookies, and tokens to arbitrary code running in those child processes.

**Tests exist:** `test/python-child-env.test.ts` verifies the allowlist logic and confirms sentinel secrets do NOT leak through any of the three spawn sites (via mocked child processes). If the allowlist changes, these tests must pass.

### Desktop policy enforcement
`src/desktop-policy.ts:validatePolicy` now directly calls `isDeniedUpstreamTool` as defense-in-depth (not relying on upstream callers to check it). `test/desktop-policy.test.ts` covers all branches: allowed/denied actions, confirmation checks, pid/windowId validation, screenshot restrictions, stateId requirements.

### desktop-contract.ts constants (ground truth)
- `MAX_AX_NODES = 1000` (not 5000 — the README was wrong and has been fixed)
- `MAX_AX_DEPTH = 32` (not 50)
- `MAX_DIMENSION = 10000` (not 2048 — this is the desktop screenshot cap; browser screenshots have a separate `SCREENSHOT_MAX_DIMENSION = 8000` in `src/agent-browser.ts:74`)

### Browser tool schema limitations
The registered `browser` tool in `src/index.ts` does not expose `compact`, `semanticAction`, `job`, or `batch` parameters in its schema — but `src/browser-policy.ts:validateBrowserRequest` handles all of them. Agents using tool schemas for planning won't discover these features. This is a known gap.

### semanticAction uses `verb` and `query` fields (not `action` and `role`)
`src/browser-policy.ts:SemanticActionRequest` shape: `{ locator, query, verb, name?, index?, value?, exact? }`. The README code example was fixed to match; double-check any new docs or agent prompts that reference the old `{ action, role }` field names.

## Architecture Notes
- `src/native-tools.ts` is an ~900-line dispatcher mixing web search backends, semantic crawl, academic research, GitHub API, and embedding pipeline. This is the most coupled file in the codebase — refactoring it is deferred technical debt, not a quick fix.
- `src/reach-tools.ts` has a `case 'browser'` in `dispatchReachTool` that is **dead code** — the `browser` tool is registered independently in `src/index.ts` and never flows through this dispatcher. Do not add logic there expecting it to execute.
- `src/cdp.ts` is a self-contained CDP implementation (793 lines, zero external deps). It has no WebSocket connection-failure or protocol-error tests — if CDP behavior changes, those paths need coverage.
- The desktop control stack (`desktop-tools.ts` → `cua-client.ts`) is cleanly separated from search and browser modules with no cross-imports.

## Residual Risks
- **Unconfirmed containerization posture** (see above — OPEN).
- **No integration test verifies container network isolation.** The SSRF posture relies on external infrastructure that is untested in this codebase.
- **CLI subprocess overhead:** Every `web_search`/`fetch` call spawns a child process via `CliSearchBackend`. For high-frequency use, this is a performance concern, not a correctness one.

---
> Source: [rhinos0608/Pi-Atlas](https://github.com/rhinos0608/Pi-Atlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-15 -->
