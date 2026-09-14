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

### Deny-by-default child environments (CLI/MCP/native)
`src/cli/cli-backend.ts:buildCliEnvironment` gives every CLI child the nonsecret base config only, plus per-tool-family credentials (`CLI_TOOL_CREDENTIALS`) — a `web_search` child never carries GitHub/Reddit/graph secrets and vice versa; unknown tool names get base config only. `src/process/mcp-client.ts:toProcessEnvironment` is deny-by-default: only listed provider credentials, benign client config, and names in the explicit `SEARCH_MCP_FORWARD_ENV_JSON` allowlist forward — there is no `SEARCH_MCP_*` wildcard, and `parseForwardedEnvironmentKeys` rejects secret-like names (`SECRET_LIKE_NAME_PATTERN`) and non-benign `SEARCH_MCP_*` internals. `src/process/native-child-env.ts:buildNativeChildEnvironment` (git/ffmpeg/media CLIs) takes a minimal OS-spawn allowlist only — no tokens/keys/cookies, no proxy config, no interpreter/linker/git-config/cert overrides — with fixed argv arrays and `shell: false` (documented, test-enforced).

### Local sidecar stdin-token auth
`src/sidecar/sidecar-manager.ts:SidecarManager.start` mints a fresh 256-bit token per start (`randomBytes(32)`) and delivers it over the child's stdin pipe only (`SIDECAR_TOKEN=<hex>\n`, via `writeAuthToken`) — never argv/env (the child env is allowlisted and would strip it anyway), never logged. Delivery is mandatory: missing stdin or a write failure is a startup failure, never an unauthenticated running sidecar. The token clears on `stop()`/exit/crash; restarts mint fresh. `src/sidecar/embedding-client.ts` takes the token via explicit `apiToken` or a per-request `apiTokenProvider` (wins fresh on every request, so long-lived clients survive restarts); `EMBEDDING_SIDECAR_API_TOKEN` env fallback is for external sidecars only (`EMBEDDING_SIDECAR_BASE_URL` Bearer health check).

### Leaf-runtime RPC: fixed safe errors, reject-not-clamp, provider-opaque DTOs
`src/runtime/runtime-rpc-protocol.ts:RUNTIME_RPC_ERROR_MESSAGES` is the closed set of safe messages — `wireErrorToSafe` in `src/runtime/leaf-runtime-client.ts` maps unknown codes to `provider_error`; provider exception text never crosses. `safeLeafCode` in `src/web/agent/agent-jobs.ts` allowlists `[a-z_]{1,64}`, else `provider_error`. Out-of-range `timeoutMs`/prompt bytes/`maxOutputTokens` reject, never clamp (`asTimeoutMs`, `runLeaf`); clone ceilings are operator-lower-only (`resolveGithubClonePolicy`); the MCP forward list rejects secret-like/invalid names; vision eligibility never broadens on failure (`eligibleTiersAfterFailure`). Provider-opaque DTOs: `runLeaf` resolves to `{ text }` only (runId stays internal, metadata redacted via `redactProvenance`); job snapshots carry `transport` + safe `reason` only — never provider/model identity (`src/web/agent/agent-rpc.ts`, `negotiateLeafTransport`).

### Event-bus trust boundary
The leaf-runtime `LeafEventBus` (`src/runtime/leaf-runtime-client.ts`) is an in-process seam for trusted co-installed extension modules only (provider registered via `setLeafRuntimeProvider`, wired in `src/index.ts` when `PI_NORTHSTAR_LEAF_MODEL` is set) — it is not an authenticated channel. The `RuntimeCorrelationV1` metadata (`owner: 'northstar'`, bounded ASCII `correlationId`/`stage`, bounded `queryIndex`/`attempt`, closed role set) is routing/observability metadata with exact-keys validation, not auth: unknown fields reject, but nothing in it proves caller identity.

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
- `src/native-tools.ts` is a ~560-line dispatcher over web search backends, semantic crawl, academic research, and the embedding pipeline — still the most coupled file in the codebase, but GitHub now delegates to its own domain (`src/github/github-domain.ts` owns all GitHub HTTP + normalization; `native-tools.ts` delegates). Refactoring it further is deferred technical debt, not a quick fix.
- The desktop control stack (`src/desktop/desktop-tools.ts` → `src/desktop/cua-client.ts`) is cleanly separated from search and browser modules with no cross-imports.

## Residual Risks
- **Child-env allowlists are static pins without revocation.** `PYTHON_CHILD_ALLOWLIST` / `NATIVE_CHILD_ALLOWLIST` are module-level sets — there is no per-spawn revocation or rotation; an allowlisted value that turns secret-capable passes for the process lifetime (the native allowlist already excludes proxy URLs for exactly this reason).
- **Live clone-cap overshoot window.** `runCloneChild` (`src/github/github-clone.ts`) polls clone-root size on a bounded interval and aborts past `maxRepoBytes`, but up to one poll interval plus the directory-walk time plus the SIGTERM grace can elapse first — a clone can briefly exceed the ceiling; the post-clone scan is the final safeguard and still rejects before serving.
- **Synthetic-probe proof gap.** `runVisionProbe` (`src/media-vision/probe.ts`) attests that the configured endpoint answers the randomized challenge for a model-ID string at probe time; it cannot prove the endpoint routes that model ID to a real vision model on later calls (model IDs are operator-exact strings the endpoint claims to serve).
- **No integration test verifies container network isolation.** Application guards are defense-in-depth; container egress remains outer boundary.
- **DNS rebinding / Chromium DNS TOCTOU** can occur after preflight.
- **Redirects** may reach targets not covered by initial validation in unrestricted fetch paths.
- **Debug-server outbound proxying** can make loopback server an egress relay.
- **CLI subprocess overhead (measured 2026-09-14, M1 Pro/10-core):** Every `web_search`/`fetch` call spawns a child process via `CliSearchBackend` at ~300 ms spawn tax per call (Node boot + tsx compile; in-process same work ≈ 0 ms). Throughput plateaus at ~16 spawns/s; 32-way parallel spawns degrade per-call p50 to ~1.9 s. One `web_search` fans out to up-to-3 providers by default (max 8 explicit), so a 32-agent pool means ~96 concurrent spawns. `SEARCH_BACKEND=mcp` selects the persistent single-transport `SearchMcpClient` (no per-call spawn) — prefer it for pool deployments. Harness: `npm run bench:cli-spawn`, baseline in `bench/README.md`. No global semaphore yet (deliberate: re-run the harness on the deployment host before adding one). Performance concern, not correctness.
- **Untrusted-content framing is advisory, not enforcement.** `src/core/untrusted-content.ts` (`EXTERNAL_TOOL_NAMES`: `fetch`, `github`, `graph`, `kg`, `agent_poll`, `social`, `web_search`, `browser`, `desktop` — no `media`) fences external tool text with per-result tokens and heuristic flags; it never redacts visible text and cannot guarantee prompt-injection prevention — a model may still follow malicious page text. `tool_result` and `before_agent_start` hooks apply the framing.
- **Loopback-only debug mode confines browser network to exact origin.** When navigating to a loopback address, a local enforcing proxy pins DNS at startup and blocks HTTP/WebSocket/CONNECT to non-matching origins. `AGENT_BROWSER_ALLOWED_DOMAINS` blocks cross-domain sub-resources. CDP backend fails closed on loopback. Batch/job commands cannot target loopback URLs. Container egress remains the outer defense. Integration test with real agent-browser deferred.
- **WSS (WebSocket Secure) not wrapped in TLS.** The loopback proxy uses plaintext `net.connect` for WebSocket upgrades. `wss:` targets on non-443 ports will fail. This is acceptable for local debug servers which typically use plain `ws:`.

---
> Source: [rhinos0608/Pi-Northstar](https://github.com/rhinos0608/Pi-Northstar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-14 -->
