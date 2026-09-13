## agents-md

> Pi-Northstar is the Pi coding agent's browser/desktop automation and web search extension. It provides CDP-based browser control, agent-browser integration, desktop automation (via Cua Driver MCP), hybrid search (BM25 + vector embedding + RRF fusion), social/reach tools, and cookie/auth management.

# Agent Reference: Pi-Northstar

Pi-Northstar is the Pi coding agent's browser/desktop automation and web search extension. It provides CDP-based browser control, agent-browser integration, desktop automation (via Cua Driver MCP), hybrid search (BM25 + vector embedding + RRF fusion), social/reach tools, and cookie/auth management.

## Sister Repos

### No protocol-level dependencies on the other Pi repos.
Pi-Northstar is self-contained. It does not consume `@rhinos0608/pi-workspace-protocol`, Pi-SmartRead, or Pi-SmartEdit directly.

## Operational Contracts and Invariants

### Application SSRF guards (Scope A)
Public user-controlled fetch/browser URLs use `src/network-policy.ts` and reject private/reserved literals, metadata/local hostnames, credentials, and private DNS answers. Browser navigation also freezes allowed domains and performs system-DNS preflight. This is defense-in-depth, not complete SSRF containment; container egress remains authoritative.

Configured local SearXNG, Ollama, embedding, sidecar, CDP/setup paths remain operator-owned and bypass public validation (`unsafeFetchJson` is intentional). Loopback browser access is only through `browser-tools` → `LoopbackProxy`.

Residual risks: DNS rebinding, Chromium DNS TOCTOU, redirects, and debug-server outbound proxying. See ADR 0003.

### Python child processes MUST use the shared env allowlist
`src/process/python-child-env.ts` exports `buildPythonChildEnvironment()` — a sanitized environment with an allowlist of benign system/PI vars and a `BLOCKED_PATTERN` excluding TOKEN/KEY/SECRET/COOKIE/PASSWORD/API_KEY/API_SECRET/AUTH/BEARER and NODE_OPTIONS, NODE_PATH, PYTHONPATH, GIT_CONFIG_, SSL_CERT_, LD_PRELOAD, DYLD_ patterns.

**All three Python spawn sites use it:**
- `src/web/access/scrapling-bridge.ts:474` — `spawnProcess` (per-fetch Python bridge)
- `src/web/access/scrapling-bridge.ts:623` — `oneShotCommand` (health check)
- `src/sidecar/sidecar-manager.ts:82` — `SidecarManager.start` (embedding sidecar)

**When adding a new Python child process:** always pass `env: buildPythonChildEnvironment()`. Never default to inheriting `process.env` — full environment leakage exposes API keys, cookies, and tokens to arbitrary code running in those child processes.

**Tests exist:** `test/process/python-child-env.test.ts` verifies the allowlist logic and confirms sentinel secrets do NOT leak through any of the three spawn sites (via mocked child processes). If the allowlist changes, these tests must pass.

### Desktop policy enforcement
`src/desktop/desktop-policy.ts:validatePolicy` now directly calls `isDeniedUpstreamTool` as defense-in-depth (not relying on upstream callers to check it). `test/desktop/desktop-policy.test.ts` covers all branches: allowed/denied actions, confirmation checks, pid/windowId validation, screenshot restrictions, stateId requirements.

### desktop-contract.ts constants (ground truth)
- `MAX_AX_NODES = 1000` (not 5000 — the README was wrong and has been fixed)
- `MAX_AX_DEPTH = 32` (not 50)
- `MAX_DIMENSION = 10000` (not 2048 — this is the desktop screenshot cap; browser screenshots have a separate `SCREENSHOT_MAX_DIMENSION = 8000` in `src/browser/agent-browser.ts:181`)

### Browser tool schema
The registered `browser` tool in `src/index.ts` exposes `compact`, `semanticAction`, `job`, and `batch` parameters in its schema, matching the capabilities handled by `src/browser/browser-policy.ts:validateBrowserRequest`.

### semanticAction uses `verb` and `query` fields (not `action` and `role`)
`src/browser/browser-policy.ts:SemanticActionRequest` shape: `{ locator, query, verb, name?, index?, value?, exact? }`. The README code example was fixed to match; double-check any new docs or agent prompts that reference the old `{ action, role }` field names.

## Architecture Notes
- `src/native-tools.ts` is an ~900-line dispatcher mixing web search backends, semantic crawl, academic research, GitHub API, and embedding pipeline. This is the most coupled file in the codebase — refactoring it is deferred technical debt, not a quick fix.
- `src/browser/cdp.ts` is a self-contained CDP implementation (793 lines, zero external deps). WebSocket failure and protocol-error CDP responses are covered by `test/browser/cdp.test.ts` (timeout, onclose rejection, error-result propagation).
- The desktop control stack (`src/desktop/desktop-tools.ts` → `src/desktop/cua-client.ts`) is cleanly separated from search and browser modules with no cross-imports.

## Residual Risks
- **No integration test verifies container network isolation.** Application guards are defense-in-depth; container egress remains outer boundary.
- **DNS rebinding / Chromium DNS TOCTOU** can occur after preflight.
- **Redirects** may reach targets not covered by initial validation in unrestricted fetch paths.
- **Debug-server outbound proxying** can make loopback server an egress relay.
- **CLI subprocess overhead (measured 2026-09-14, M1 Pro/10-core):** Every `web_search`/`fetch` call spawns a child process via `CliSearchBackend` at ~300 ms spawn tax per call (Node boot + tsx compile; in-process same work ≈ 0 ms). Throughput plateaus at ~16 spawns/s; 32-way parallel spawns degrade per-call p50 to ~1.9 s. One `web_search` fans out to up-to-3 providers by default (max 8 explicit), so a 32-agent pool means ~96 concurrent spawns. `SEARCH_BACKEND=mcp` selects the persistent single-transport `SearchMcpClient` (no per-call spawn) — prefer it for pool deployments. Harness: `npm run bench:cli-spawn`, baseline in `bench/README.md`. No global semaphore yet (deliberate: re-run the harness on the deployment host before adding one). Performance concern, not correctness.
- **Untrusted-content framing is advisory, not enforcement.** `src/core/untrusted-content.ts` fences external tool text (`web_search`, `fetch`, `github`, `social`, `media`, `browser`) with per-result tokens and heuristic flags; it never redacts visible text and cannot guarantee prompt-injection prevention — a model may still follow malicious page text. `tool_result` and `before_agent_start` hooks apply the framing.
- **Loopback-only debug mode confines browser network to exact origin.** When navigating to a loopback address, a local enforcing proxy pins DNS at startup and blocks HTTP/WebSocket/CONNECT to non-matching origins. `AGENT_BROWSER_ALLOWED_DOMAINS` blocks cross-domain sub-resources. CDP backend fails closed on loopback. Batch/job commands cannot target loopback URLs. Container egress remains the outer defense. Integration test with real agent-browser deferred.
- **WSS (WebSocket Secure) not wrapped in TLS.** The loopback proxy uses plaintext `net.connect` for WebSocket upgrades. `wss:` targets on non-443 ports will fail. This is acceptable for local debug servers which typically use plain `ws:`.

---
> Source: [rhinos0608/Pi-Northstar](https://github.com/rhinos0608/Pi-Northstar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
