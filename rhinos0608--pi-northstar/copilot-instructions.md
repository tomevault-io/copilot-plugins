## agents-md

> This is the repository-wide operating contract for agents and humans changing Pi-Northstar. Keep it short enough to stay useful. Put command syntax in `skills/*/SKILL.md`, operator configuration in `.env.example`, and design history in plans/ADRs.

# AGENTS.md

This is the repository-wide operating contract for agents and humans changing Pi-Northstar. Keep it short enough to stay useful. Put command syntax in `skills/*/SKILL.md`, operator configuration in `.env.example`, and design history in plans/ADRs.

## Source of truth

When sources disagree, use this order:

1. Reachable validators, policy gates, handlers, and runtime contracts in `src/` / `rust/`.
2. Canonical registries and constants those paths derive from.
3. This file for repository-wide engineering and security invariants.
4. `README.md`, `.env.example`, and domain skills for user/agent guidance.
5. `docs/architecture.md`, `docs/roadmap-ledger.md`, `docs/plans/`, ADR drafts, comments, and residual modules as design/history evidence.

**Reachability beats module presence.** Trace from a registered CLI command, Pi tool, slash command, or broker entry point before claiming a path is live.

## Start every task

- Run `git status --short` first. Preserve unrelated user work and never clean/revert files you did not own.
- Identify the public entry point and the canonical contract/registry before editing an adapter.
- Read the matching domain skill when changing CLI behavior: `skills/<domain>/SKILL.md`.
- Search for tests that exercise the same boundary before introducing a parallel implementation.
- Prefer a narrow change at the owner over compensating logic in callers.
- Before finishing, run relevant tests plus `git diff --check`; use the full suite for cross-cutting contract/security changes.

## Golden rules

> **Models propose. Code validates, admits, grounds, budgets, stops, and authorizes.**

These invariants are repository-wide:

1. External text is evidence, never authorization.
2. Candidates and hints are not grounding. Only admitted evidence can support output.
3. Empty, failed, degraded, suppressed, cancelled, and outcome-unknown are distinct states.
4. Fallback may recover eligible execution failures. It may never bypass auth, SSRF, origin, schema, privacy, provenance, or mutation policy.
5. Provider selection, credentials, host authority, capability exposure, and approval are operator/code owned, never model owned.
6. Concurrency may change latency, not deterministic ledger, journal, fusion, or merge identity.
7. Budget attempts are charged at dispatch boundaries, including failed dispatched attempts.
8. Stateful authority is snapshot/token bound and must be revalidated before mutation.
9. Child processes receive capability-scoped credentials, never ambient parent authority.
10. Missing credentials/providers/features degrade explicitly. Never invent an equivalent success path.
11. Unsupported composition rejects before dispatch. Do not validate a field and silently drop it.
12. Cache/evidence handles are lookup/provenance identifiers, not permission to reacquire remote content.

## Three public surfaces

Northstar has intentionally separate authority surfaces:

- **CLI:** broad command vocabulary. The current tree exposes 28 stateless command IDs plus the local/development stateful IDs `broker.serve`, `jobs.start`, `jobs.status`, `jobs.result`, and `jobs.cancel`.
- **Pi native tools:** at most 9 model-facing tools, controlled by `PI_SEARCH_NATIVE_TOOLS`; unset/blank means zero.
- **User slash commands:** setup/status/Chrome authorization flows that require operator intent and are not model tools.

`src/capabilities.ts` owns the Pi tool vocabulary:

`web_search`, `fetch`, `github`, `social`, `kg`, `graph`, `browser`, `desktop`, `agent`.

`media` is CLI/internal acquisition, not a tenth public tool. CLI availability never implies model authority. A provider being configured never implies its Pi tool is exposed.

## Ownership map

| Concern | Canonical owner(s) |
| --- | --- |
| public Pi tool names/budget/channel registry | `src/capabilities.ts` |
| Pi composition, registration, global untrusted-content boundary | `src/index.ts` |
| CLI grammar/parsing | `src/cli/cli.ts` |
| CLI execution seam / child credential routing | `src/cli/cli-backend.ts`, `src/commands/*` |
| local config precedence/mapping | `src/setup/local-config.ts` |
| provider auth/setup metadata | `src/setup/providers.ts`, `src/setup/bootstrap.ts` |
| web request contracts | `src/web/web-contract.ts`, `src/web/web-search-route.ts`, `src/web/web-fetch-route.ts` |
| web provider selection/fanout | `src/web/web-provider-policy.ts`, `src/web/web.ts` |
| ranking/fusion | `src/search/fusion.ts` |
| page specialization/read path | `src/native-fetch.ts`, `src/web/web-page-reader.ts`, `src/web/access/*` |
| GitHub | `src/github/github-contract.ts`, `src/github/github-domain.ts` |
| research | `src/research/*` |
| social | `src/social/*` |
| media | `src/media/*` |
| KG / graph / SPARQL | `src/knowledge/*`, `src/graph/*`, `src/diffbot/*`, `src/sparql/*` |
| multimodal/vision | `src/media-vision/*` |
| agent jobs/controller | `src/web/agent/*` |
| leaf runtime wire contract | `src/runtime/runtime-rpc-protocol.ts` (Northstar consumer mirror); `../pi-subagents/src/api/runtime-rpc.ts` (producer ground truth when co-installed) |
| browser policy/session authority | `src/browser/*`, `src/chrome/*` |
| desktop policy/state | `src/desktop/desktop-contract.ts`, `desktop-policy.ts`, `desktop-tools.ts` |
| one-shot native/Python child envs | `src/process/native-child-env.ts`, `python-child-env.ts`, `mcp-client.ts` |
| broker client/host/state seam | `src/runtime/broker-*` |
| same-user local broker leaf runtime | `src/runtime/local-leaf-runtime.ts` |
| native broker TCB | `rust/crates/northstar-broker/*` |
| external-content framing | `src/core/untrusted-content.ts` |
| per-domain agent/CLI guidance | `skills/*/SKILL.md` |

Do not add a second registry because an adapter wants convenience. Derive from the owner or add an explicit adapter layer.

## Configuration and credentials

Normal configuration precedence is:

`process environment > .env (or PI_SEARCH_ENV_PATH) > explicitly selected SEARCH_MCP_CONFIG_PATH JSON mapping`

The JSON file maps known config paths into canonical environment names; it is not a general execution/policy file. Process environment always wins. Status/config output may expose key **names** or safe endpoint hosts, never secret values.

Provider choice is environment/operator policy. Do not add provider names, API keys, tokens, endpoints, or credential selectors to model-facing request schemas merely for convenience.

`PI_SEARCH_NATIVE_TOOLS` is a separate authority gate from provider configuration. Blank means no native Pi tools. Parsing is strict: exact canonical comma-separated names, no duplicates or aliases.

### Keyless does not mean unrestricted

Several paths work without secrets: DuckDuckGo search, native fetch, the 12 public research sources, public GitHub reads, RSS/Atom, baseline V2EX, limited YouTube details/transcript routes, and operator-owned endpoints that do not require auth.

Still preserve route-specific policy, rate limits, provenance, and evidence class. Never replace a failed specialist source with generic web and report it as specialist success.

### Secret isolation

- `src/cli/cli-backend.ts` starts from a nonsecret base and adds credentials per tool family.
- `src/process/mcp-client.ts` is deny-by-default. Secret-like names do not become forwardable through generic config.
- Native/media/git children use the narrow native child environment. Python children use the narrow Python environment.
- Fixed argv plus `shell:false` is the baseline for subprocesses. Never interpolate user/model/provider text into shell commands.
- Git credentials must not appear in argv or ordinary inherited environment; keep the ephemeral credential-helper path.
- The local embedding sidecar mints a fresh token per start and sends it over stdin only. Startup fails rather than falling back unauthenticated.
- Broker root auth must never be delegated to provider workers.

Diffbot has a guarded optional login-shell token probe in `src/setup/local-config.ts`. It is fallback-only, bounded, fixed-script, dedicated-fd, and must never become a general secret-discovery mechanism.

## Setup and authentication acquisition

Distinguish these states:

- dependency/binary is installed;
- provider is configured;
- credential material is present;
- an authenticated local session exists;
- a capability is currently usable.

Do not collapse them into one `configured` boolean in user guidance when the distinction matters.

Cookie import, browser login, and user-Chrome authorization are explicit operator actions. Startup, status probes, and the internal `reach_setup auto` action must not import cookies or create authenticated sessions to make health look better. Bare **user slash** `/reach-setup` is different: it is an explicit, interactive consent path and may import only the sessions the active setup contract declares.

Current normal Pi cookie-profile consumers are Reddit, Bilibili, and YouTube. Gemini Web uses a separate Google session snapshot only when its explicit setup/vision gates are enabled; do not treat that snapshot as a general cookie profile. Other social adapters may own their own CLI/OpenCLI sessions. Do not infer that an environment token or imported browser cookie unlocks a backend unless the active adapter actually consumes it.

## Contract discipline

### Reject instead of pretending

A route that cannot honor an option end to end must reject it. Security/budget/wire boundaries should generally reject invalid ranges or fields instead of clamping unless the owning contract deliberately specifies clamping.

Preserve selector-bound cursors. A cursor reused with different query/source/filter identity must fail closed. Do not turn pagination tokens into free-form state handles.

### Preserve failure meaning

Do not collapse:

- valid zero results into provider failure;
- provider failure into "nothing exists";
- cached/suppressed work into fresh evidence;
- degraded adapters into full native success;
- cancellation into generic failure;
- unknown mutation outcome into retry permission.

If every selected search provider fails, throw/report failure. If some serve, preserve sibling failures in details.

### Fallback preserves authority

Invalid input, authentication failure, SSRF/origin denial, privacy-transfer denial, and closed-schema errors are terminal for that route. Private/authenticated GitHub must not silently become anonymous acquisition. Authenticated fetch must not spill into external rendering.

## Evidence and agent jobs

`src/index.ts` installs the global external-content framing boundary. Framing is advisory defense-in-depth, not a permission system. Never weaken network, browser, desktop, auth, subprocess, or broker checks because content was wrapped as untrusted.

The production-shaped agent loop lives in `src/web/agent/*`: plan, gather, evaluate, stop/refine, synthesize, verify/repair. Planner/evaluator/model output is always proposal data and must pass code-owned validators before state changes.

Important invariants:

- capability truth is frozen for a job instead of fluctuating on re-probe;
- journal/log identity is reserved at dispatch, not promise-settlement order;
- budget is multi-dimensional and owned by `agent-policy.ts`; do not retype defaults into adapters/docs;
- synthesis references admitted evidence; deterministic evidence-only output is the fallback floor;
- verification/repair is bounded and may not regress grounding;
- `PI_NORTHSTAR_AGENT_STEERING=0` must remain a valid evidence-only mode.

### Leaf RPC contract

`pi-subagents` is the producer of the co-installed leaf-runtime protocol. When both repos are present, `../pi-subagents/src/api/runtime-rpc.ts` is the ground truth for `subagents:runtime:v1`; `src/runtime/runtime-rpc-protocol.ts` is Northstar's self-contained consumer mirror. Server-side validation/negotiation additionally lives in `../pi-subagents/src/extension/runtime-rpc-schemas.ts` and `runtime-rpc.ts`. Compare against the producer before changing the mirror.

Protocol invariants:

- methods are `negotiate`, `start`, `status`, `result`, and `cancelAndSettle`;
- exact `provider/model` only; thinking suffixes reject and negotiation never authorizes a fuzzy fallback;
- request/reply shapes are closed and bounded; provider exception text never becomes a wire error;
- the event bus is trusted co-installed extension plumbing, not an authenticated boundary; correlation fields are routing/observability metadata only;
- v2 correlation requires a successful prior negotiation for the same model;
- the current producer advertises `structured-v1`; attach `outputSchema` only after that dialect is negotiated. `outputModes` containing `json` alone is not sufficient;
- absent/older schema negotiation means text JSON plus client-side parsing and the owning domain validator; transport schema validation never replaces domain validation;
- cancellation is explicit and bounded through `cancelAndSettle`; a settlement contract breach makes the runtime unhealthy rather than inventing success.
- producer readiness is fail-closed on unverified Pi host versions and can be explicitly disabled with `PI_SUBAGENTS_RUNTIME_RPC_DISABLED=1`; the current `pi-subagents` public contract lists host `0.85.1` as verified.
- do not assume every producer/server-only bound must be copied into the consumer mirror. Sync client-relevant wire fields deliberately and test negotiation against the producer contract.

Adaptive-agent steering resolves its exact leaf model through the unified operator selection (`PI_NORTHSTAR_MODEL` > `~/.pi-northstar/config.json`), with `PI_NORTHSTAR_LEAF_MODEL` retained only as the legacy last-resort fallback. Unified config defaults steering off until `/northstar agent on`; exact `PI_NORTHSTAR_AGENT_STEERING=0` still forces it off. `src/runtime/local-leaf-runtime.ts`, used by local broker jobs, is a separate same-user text-only runtime backed directly by the Pi AI model registry; `jobs start --model` remains an explicit per-job selection. Do not conflate the two runtimes or make one silently substitute for the other.

## Browser, Chrome companion, and desktop

Public browser targets are HTTP(S) only and remain behind URL/DNS/SSRF/origin admission. Loopback debug mode is a separate exact-origin confinement path. A browser session must not silently widen its host/origin authority after redirects or navigation.

User-Chrome control requires the pinned companion bridge plus a live user authorization lease. Inventory/selection is operator-facing, not model input. Revoke, expiry, failure, or shutdown must return to the isolated backend.

Desktop is opt-in. Observe first. Mutations require fresh state identity and are never blindly retried after dispatch. Keyboard/text mutations that require human confirmation must fail closed headless.

## Social, media, and vision

Social is read-only in practice. Write-shaped vocabulary does not imply write authority. Provider write allowlists are empty; destructive/download/bulk-follow behavior must stay denied unless a future adapter is explicitly reviewed and authorized.

Media acquisition is not a Pi public tool. Keep platform evidence classes honest:

- YouTube search/hot use the official API and require `YOUTUBE_API_KEY`.
- YouTube details can degrade to keyless oEmbed.
- YouTube transcript has a separate unofficial keyless degraded path; do not call it official API success and do not route transcript through `yt-dlp`.
- Bilibili uses its declared native/session adapters.
- `yt-dlp` is permitted only on the separate fetch-time YouTube frame path: exact `PI_VISION_FETCH_VIDEO_FRAMES=1`, anonymous fixed argv, `--no-config`, no cookies/account credentials/proxy, and native-child environment isolation. Its registry actions remain empty.

### Multimodal fetch invariants

- Normal PDF fetch is local-only through `unpdf`: 20 MiB, 100 pages, 50,000 characters, page-aware citations. Sparse/scanned pages warn; fetch must not silently upload a PDF to vision.
- `PI_VISION_PDF_CLOUD_RENDER=1` is currently a reserved fail-closed flag because no PDF page-image renderer ships. Do not document it as working OCR/cloud rendering until that renderer is reachable.
- Direct image fetch returns sniff-verified metadata by default. Description requires exact `PI_VISION_FETCH_DESCRIBE=1` plus a configured OpenAI-compatible or Gemini tier; generated description stays separate from fetched content.
- YouTube keyframe analysis requires exact `PI_VISION_FETCH_VIDEO_FRAMES=1` plus a configured OpenAI-compatible or Gemini tier. Vision failures degrade toward transcript/metadata evidence, never toward an unconfigured provider.
- OpenAI-compatible vision may be loopback or cloud and uses exact configured model IDs; the API key is optional so loopback endpoints can be keyless.
- Gemini Developer/Vertex requires exact `PI_VISION_GEMINI_ENABLED=1`. Vertex additionally needs `GOOGLE_GENAI_USE_VERTEXAI=1`, project, location, and ADC. Keep model selection explicit via `PI_VISION_GEMINI_MODEL` when reproducibility matters.
- Gemini Web is a separate disabled-default seam, not a general vision fallback. Fetch image/keyframe analyzers never select it. The full-file local-video fallback may reach it only when the direct Gemini tier is unavailable and both exact `PI_VISION_VIDEO_GEMINI=1` and `PI_VISION_GEMINI_WEB_ENABLED=1` are enabled, using a live authorized Chrome lease where possible or the explicitly imported isolated Google session. One configured destination never authorizes another.

Vision is destination-specific. Configuring one destination does not authorize another. A loopback OpenAI-compatible endpoint can keep image/keyframe analysis local; cloud routes move admitted bytes/text off-machine. Private/authenticated GitHub content additionally requires exact `PI_VISION_PRIVATE_GITHUB_TRANSFER=1` before any eligible cloud transfer.

Synthetic probe helpers prove only that an endpoint/model answered the randomized probe at that moment. They do not prove provider identity or later routing, and the current fetch image/video hot paths do not invoke those probes before user bytes.

## Broker and stateful authority

Before changing broker code, read `docs/adr/0010-gate-b-native-authority-boundary.md`, `docs/plans/2026-09-21-gate-b-native-authority-boundary.md`, `docs/roadmap-ledger.md`, and `docs/tier2-proof.md`.

The working tree currently exposes `broker.serve` plus `jobs.start/status/result/cancel` in CLI help as a local/development surface. Executable reachability is current truth; production release readiness is not. `broker serve` is the only explicit starter; ordinary jobs commands must remain connect-only and never auto-spawn authority. Tier-2 privileged proof/signing and per-job isolation remain open release gates.

Stateful rules include: authenticated client identity, replay/sequence protection, project scope, capability-scoped worker grants, strict endpoint ownership/mode checks, durable mutation receipts, and fail-closed handling of corrupt state/config. Unknown mutation outcome requires observation/reconciliation, not automatic replay.

## Tests and verification

Use the smallest relevant test first, then broaden when the change crosses contracts.

```bash
npm run typecheck
npm test
npm run build
git diff --check

# Native broker / Rust changes
cargo test --manifest-path rust/Cargo.toml --workspace
```

For contract, provider, browser, desktop, auth, agent, subprocess, or broker changes, explicitly review:

- **Reachability:** is the changed path actually registered/callable?
- **Authority:** can model/external text now influence provider, credentials, hosts, grants, or mutation approval?
- **Contracts:** do parser, schema, handler, adapter, and rendered output agree?
- **Failure semantics:** did a terminal class accidentally become fallback/retry?
- **Provenance:** can callers distinguish evidence source and degradation without leaking secrets?
- **Secrets:** does every child/network destination receive only what it needs?
- **Ordering:** can concurrency reorder identity, journal, or fusion semantics?
- **Cancellation:** is abort propagated and outcome classified correctly?
- **Docs:** did user-visible behavior/config change enough to update README, `.env.example`, or a domain skill?

Never claim coverage you did not run. Report unverified platform/privileged behavior explicitly.

## Documentation discipline

`README.md` is the human landing page: what Northstar does, first successful command, auth/config mental model, security/privacy, and where to go next.

`SKILL.md` is a router for agents. Keep it compact. It should choose the right surface and point to `skills/*/SKILL.md`, not duplicate every flag or provider implementation detail.

`skills/*/SKILL.md` owns domain-level CLI syntax and behavioral contracts. Live CLI `--help` is authoritative for installed-version grammar.

`.env.example` is the primary operator configuration catalogue, including privacy-sensitive opt-ins. Keep it synchronized with live configuration and do not scatter exhaustive variable lists across every document.

`docs/architecture.md`, `docs/roadmap-ledger.md`, `docs/plans/`, and ADRs may intentionally describe target or staged states. If they disagree with reachable code, do not rewrite reality to match the plan. Either update the stale planning document when in scope or label the mismatch.

## High-value references

- `README.md`: operator/user overview.
- `SKILL.md`: root agent router.
- `skills/*/SKILL.md`: domain contracts.
- `.env.example`: config catalogue.
- `docs/architecture.md`: architecture and authority model.
- `docs/roadmap-ledger.md` + `docs/plans/`: redesign phases and release gates.
- `docs/adr/`: focused decisions.
- `docs/tier2-proof.md`: privileged broker proof plan.
- `docs/northstar-capability-matrix.md`: capability inventory.

When in doubt, prefer executable truth, narrow authority, explicit degradation, and less ambient context.

---
> Source: [rhinos0608/Pi-Northstar](https://github.com/rhinos0608/Pi-Northstar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
