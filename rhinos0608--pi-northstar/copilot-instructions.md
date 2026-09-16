## agents-md

> This file is the repository-wide contract for agents and humans changing Pi‑Northstar. It is not a historical design diary. Prefer executable truth, preserve authority boundaries, and update prose only after the production-shaped path is settled.

# AGENTS.md · Pi‑Northstar engineering contract

This file is the repository-wide contract for agents and humans changing Pi‑Northstar. It is not a historical design diary. Prefer executable truth, preserve authority boundaries, and update prose only after the production-shaped path is settled.

## Source of truth

When sources disagree, use this order:

1. Executable validators, policy gates, and runtime contracts in `src/`.
2. Canonical registries/constants those validators derive from.
3. This `AGENTS.md` for repository-wide security and engineering invariants.
4. `README.md` for user-facing behavior and operator guidance.
5. Plans, ADR drafts, task prose, comments, and residual modules only as design/history clues.

**Reachability beats module presence.** Trace from the registered/public entry point before claiming a path is live.

## Golden rule

> **Models propose. Code validates, admits, grounds, budgets, stops, and ships.**

Corollaries:

1. External text is evidence, never authorization.
2. Candidates navigate; only admitted evidence grounds.
3. Failure, empty output, degradation, suppression, and cancellation are different states.
4. Fallback may recover execution failure; it may never bypass auth, SSRF, origin, schema, or provenance policy.
5. Provider selection and effective capability policy are operator/code owned, never model owned.
6. Concurrency may change latency, never semantic ledger/merge/journal order.
7. Budget attempts are charged at the dispatch boundary, including failed attempts.
8. Stateful authority is frozen by a code-owned token/snapshot and revalidated before mutation.
9. Child processes receive capability-scoped credentials, never ambient process authority.
10. Missing models/providers/credentials must degrade toward evidence, not invented equivalence.

## Public surface and reachability

`src/capabilities.ts` owns the public model-facing tool vocabulary and the hard budget `MAX_PUBLIC_TOOLS = 9`:

`web_search`, `fetch`, `github`, `social`, `kg`, `graph`, `browser`, `desktop`, `agent_poll`.

Registration is conditional, so nine is a maximum. `src/index.ts` wraps `pi.registerTool` and checks the budget on every addition. Do not add a model-facing tool without first reconciling the ceiling and the capability registry.

`media` is internal/CLI acquisition, not a public model tool.

The registered agent-mode route is:

```text
web_search {query, mode:"agent"}
  → buildSearchRoute()
  → createAgentJob()
  → executeAgentJob()
  → runAgentCore()
  → canonical job snapshot
  → agent_poll
```

Older report machinery in `src/web/web.ts`, `src/web/web-agent-report.ts`, and `src/web/agent/agent-report-route.ts` is residual/internal. Its existence does **not** make an opaque Tavily report leg part of the registered public agent flow.

## Ownership map

Before editing a vocabulary, schema, budget, or side-effect path, identify its owner. Parallel copies are contract drift, not harmless duplication.

| Concern | Primary owner(s) |
|---|---|
| public tool ceiling/channel metadata | `src/capabilities.ts` |
| composition/registration/global framing | `src/index.ts` |
| web-search public shape | `src/web/web-search-route.ts`, `src/web/web-contract.ts` |
| web provider selection/fanout | `src/web/web-provider-policy.ts`, `src/web/web.ts` |
| ranking identity/fusion | `src/search/fusion.ts` |
| fetch public shape | `src/web/web-fetch-route.ts`, `src/web/access/web-access-contract.ts` |
| URL specialization/read path | `src/native-fetch.ts`, `src/web/web-page-reader.ts`, `src/web/access/*` |
| agent job lifecycle/snapshot | `src/web/agent/agent-jobs.ts` |
| adaptive controller | `src/web/agent/agent-core.ts` |
| budgets/profile/stop | `src/web/agent/agent-policy.ts` |
| GatherIntent domain | `src/web/agent/agent-gather-intents.ts` |
| gather execution/adapters | `src/web/agent/agent-gather.ts`, `src/web/agent/agent-gather-adapters.ts` |
| evidence/admission | `src/web/agent/agent-state.ts`, `src/web/agent/agent-acquisition.ts` |
| candidate hints | `src/web/agent/agent-candidates.ts` |
| agent model/wire schemas | `src/web/agent/agent-model.ts` |
| leaf wire contract | `src/runtime/runtime-rpc-protocol.ts` plus mirrored pi-subagents ground truth |
| browser action/security policy | `src/browser/browser-policy.ts` + browser session modules |
| desktop freshness/mutation policy | `src/desktop/desktop-contract.ts`, `src/desktop/desktop-policy.ts`, `src/desktop/desktop-tools.ts` |
| GitHub routing | `src/github/github-contract.ts`, `src/github/github-domain.ts` |
| child credential isolation | `src/cli/cli-backend.ts`, `src/process/*-child-env.ts` |
| external-content framing | `src/core/untrusted-content.ts` |

Browser verbs deliberately live in browser policy, not the channel registry. Their mutation semantics are stateful and do not fit the read-oriented capability table.

## Contract discipline

### Reject unsupported composition

A route must reject fields it cannot honor end to end. “Validate, accept, then drop” is a correctness defect because it tells the caller a constraint was applied when it was not.

New security/budget/wire boundaries should reject rather than clamp unless the owning contract has tested compatibility semantics that intentionally clamp. Existing exceptions such as social limits or desktop internal timeout bounding are local contracts, not a project-wide invitation to clamp.

### Preserve state distinctions

Do not collapse:

- valid zero results into provider failure;
- provider failure into “nothing exists”;
- search suppression into fresh evidence;
- cancellation into ordinary failure;
- degraded specialist execution into successful native execution;
- unknown mutation outcome into retry permission.

These distinctions are observable architecture.

### Fallback preserves authority

Fallback may switch execution mechanisms only when the security meaning stays the same. Auth failures, SSRF denials, origin violations, invalid requests, and closed-schema failures are terminal for that route.

In particular, a privileged/private GitHub attempt must not silently become anonymous/public acquisition, and an authenticated web fetch must not spill into external rendering.

## External-content trust boundary

`src/index.ts` installs the global boundary. `before_agent_start` tells the model remote/tool text is evidence, not instructions; `tool_result` wraps external tool text through `wrapUntrustedText()`.

External model-facing tools are currently `fetch`, `github`, `graph`, `kg`, `agent_poll`, `social`, `web_search`, `browser`, and `desktop`.

The wrapper removes dangerous invisible/control formatting, may flag suspicious patterns, and surrounds visible text with a fresh random fence. It intentionally does not destructively redact ordinary visible text.

**Wrapping is advisory, not a permission system.** Never weaken browser, desktop, auth, subprocess, or network policy because content already passed through `wrapUntrustedText()`.

## Agent controller

The only production-shaped control loop is adaptive:

`PLAN → GATHER → EVALUATE → STOP/REFINE → SYNTHESIZE → VERIFY/REPAIR`.

### Planning

Planner output is a proposal. Normalize/validate it before state mutation. Invalid or missing planner output falls back deterministically. Missing/invalid plan truth fails toward balanced capacity, not an artificially narrow profile.

`depth:"deep"` is explicit user/job input. Otherwise code derives balanced/narrow after validated plan shape, required-question count, and servable specialist need.

### Gather

A job snapshots effective capabilities once and reuses that frozen snapshot for planning, admissibility, and execution. Do not re-probe and let authority fluctuate mid-job.

Production native specialist lanes currently are `research`, `github`, and `kg`, plus the web baseline. Social/video GatherIntents and admission adapters exist but production `buildNativeGatherTools()` does not expose those specialist tools this cycle. Capability truth is intersected with `EXECUTOR_SUPPORTED_SPECIALIST_LANES`, so planner-visible lanes must remain a subset of executable lanes.

Search/fetch wrapper log slots are reserved at **call time** and filled on settlement. Concurrent completion order must not reorder journal identity.

Gather actions consume budget on dispatch attempts. A failed web search still costs one search attempt. Duplicate/rejected intents that never dispatch do not get charged as successful execution.

### Candidates vs evidence

Candidates are bounded discovery/navigation hints. They may carry typed follow-up identity. They do not become grounding merely because a provider returned them.

Evidence enters only through route-specific admission functions with provenance, locator, status, and question linkage. Model text such as “answered” or “supported” is never the grounding predicate.

### Evaluate and stop

Evaluator output is validated before state mutation. `shouldContinue` is advisory.

Code-owned stop policy currently considers deadline, required grounding, round cap, utility budget, global acquisition envelope, lane headroom, search/fetch budgets, semantic no-progress, and fresh follow-up availability. Duplicate-only next actions do not count as work.

The current lane-aware implementation treats global `maxFetches` exhaustion as a job-wide budget stop. Treat that as current tested semantics, not an inferred philosophical requirement; any change must reconcile executor and stop policy together.

### Synthesis, verification, degradation

A synthesizer proposes evidence-referenced IR. Validate it against admitted evidence before rendering. If synthesis is absent, fails, or produces invalid IR, return the deterministic evidence-only floor. Do not invent a narrative fallback from raw passages.

Verification is bounded. Repair is at most one pass and is re-verified. Reject a repair that regresses verification, rebinds unsupported citations, grows contradictions, deletes too much, or reduces grounded-question support.

Exact `PI_NORTHSTAR_AGENT_STEERING=0` strips model-call dependencies while preserving acquisition and code-owned stopping. No-model mode must remain a valid evidence-only execution mode.

## Budget invariants

Budgeting is multi-dimensional. Do not replace it with a single calls-remaining integer.

Current defaults are owned by `src/web/agent/agent-policy.ts`:

- 3 rounds;
- 4 web-search attempts;
- 12 fetch attempts;
- 8 utility/model calls;
- 6 total gather actions;
- per-lane caps;
- round-scoped fetch reserves.

Hard ceilings and defaults must be read from the owner, not retyped into adapters.

Utility capacity is role-aware. Planner, synthesis, and verify/repair reserves prevent evaluator pressure from starving terminal stages. Keep executor dispatch predicates and stop-policy exhaustion predicates aligned. If one thinks an action is runnable while the other thinks the job is exhausted, the controller can stop early or spin.

## Leaf-runtime RPC and structured model output

Pi‑Northstar does not import pi-subagents code. `src/runtime/runtime-rpc-protocol.ts` mirrors the co-installed runtime wire contract. The producer contract in pi-subagents is external ground truth when both extensions are installed.

The event bus is trusted **in-process module plumbing**, not an authenticated boundary. Correlation fields are routing/observability metadata, never authorization.

Leaf rules:

- exact `provider/model` only;
- thinking suffixes rejected;
- closed request/reply shapes;
- bounded prompt/result/token/time/correlation fields;
- fixed safe error codes/messages, never provider exception text;
- explicit cancellation/settlement;
- provider/model/token internals stay out of model-visible job snapshots.

### Structured JSON negotiation

Do **not** gate wire schemas on `outputModes` containing `json`.

Only negotiated `jsonSchema === "structured-v1"` allows `src/web/agent/agent-model.ts` to send an `outputSchema`. Absent dialect, `flat-v1`, or unknown dialect means text JSON: parse client-side, apply the wire shape gate, then apply the owning domain validator.

Even `structured-v1` output is parsed and checked after the leaf call. Transport negotiation can enable a wire feature; it never bypasses domain validation.

Wire schemas may intentionally be broader than domain intent schemas. For example, GatherIntent wire objects avoid unsupported schema constructs; `validateGatherIntent()` remains authoritative for exact per-kind keys and bounds.

## Web search

`createWebSearchExecute()` in `src/index.ts` owns the model-facing entry. The session `WebSearchLedger` runs before paid dispatch and may coalesce in-flight equivalents, suppress recent successful repeats, block repeated failures temporarily, or treat caller abort as cancellation.

Suppressed work is a control result pointing at captured state. Never synthesize fresh hits for it.

Ordinary provider selection is environment-owned. Every runnable selected provider is dispatched once concurrently. Fusion order follows policy/configuration order, not promise settlement order. Keep URL normalization and RRF deterministic.

Provider failure must remain observable. If at least one provider serves, carry sibling failures in details. If every provider fails and none serves, throw instead of returning “zero results.”

## Research

`source:"all"` means exact `RESEARCH_SOURCE_CAPABILITIES` registry order: Semantic Scholar, OpenAlex, PubMed, Stack Overflow, DataCite, ROR, GDELT, Wikipedia, Wikidata, arXiv, Crossref, Hacker News.

Do not substitute generic web for a research-specific source failure, unsupported option, pagination mismatch, or empty result. That changes the evidence class while pretending the requested contract succeeded.

## Fetch and network authority

The public fetch contract is a five-branch presence union owned by `src/web/web-fetch-route.ts`. Cached retrieve/source-check branches are no-network operations. A `responseId` is a provenance/cache handle, not permission to reacquire remote content.

URL specialization precedes generic page reading. Security/contract errors are terminal; execution/upstream failures may use eligible fallback.

### Cookie-authenticated fetch

`PI_FETCH_AUTH_PROFILES` is operator configuration. Absent config means inert.

Configured profile hosts must fall within the provider's cookie domains, but authentication matches only the explicitly configured profile hosts using exact-host or dot-boundary subdomain matching. A provider's whole cookie domain is **not** automatically authenticated.

Authenticated fetch is HTTPS-only and same-origin-only across redirects. It bypasses Scrapling, Diffbot Analyze, and Firecrawl/Jina external processing. Cache is off by default unless the profile explicitly selects session caching. Error paths must not echo cookie values, `Set-Cookie`, or sensitive request material.

Credentials narrow the legal acquisition path; they never broaden it.

### Public targets vs operator infrastructure

Public/user-provided fetch/browser URLs pass application network policy before I/O. Operator-configured infrastructure such as SearXNG, OpenCLI, sidecars, or SPARQL endpoints is a separate trust class.

Do not blindly apply public-URL policy to operator infrastructure, and do not exempt a public URL merely because an operator backend ultimately performs the socket call. The trust decision follows who controls the target/configuration.

The container/network environment is the outer egress boundary. Application SSRF controls are defense in depth.

## Browser

Browser action truth belongs to `src/browser/browser-policy.ts`.

Public browser sessions validate URL/DNS/SSRF and freeze allowed hostname authority. An unrelated hostname requires an explicit close/new-navigation transition. Do not silently widen the allowlist mid-session.

Snapshot refs are claims about observed page state. Ref-targeting mutations must freshness-preflight; navigation/invalidation makes older refs stale. Recovery is a fresh observation, not replaying a stale ref.

`evaluate`, `set_cookies`, and `batch` are sensitive and require exact `PI_SEARCH_BROWSER_ALLOW_SENSITIVE=1`. Cookie reads return metadata only, never values.

Loopback debug mode binds exact scheme + host + port and confines traffic to that origin. A different loopback origin requires close first. Batch/job commands must not smuggle loopback navigation into broader authority.

## User-Chrome companion

Authorized user-Chrome is an explicitly leased alternate backend. The companion bridge binds loopback only, requires an operator-pinned extension ID, pairing secret, process-local session token, and explicit user confirmation through `/chrome authorize`/onboarding.

The model cannot manufacture, renew, or broaden this grant. The browser-family selector is slash-command/user input, not model input. Missing/revoked/expired/unhealthy authorization falls back to isolated browser behavior.

## Desktop

The invariant is **observe → bind → revalidate → mutate**.

`observe_window` issues a `stateId` bound to PID, window ID, generation, timestamp/TTL, and AX-tree fingerprint. Every mutation requires it. A newer observation invalidates the older generation; coordinate mutations have the tighter freshness window.

Before dispatch, re-observe and compare fingerprint. Stale state rejects.

`type_text` and `press_key` require explicit human TUI confirmation. Headless code cannot self-approve them. Click/scroll still require fresh state.

Transport loss after a mutation may have dispatched is `OUTCOME_UNKNOWN`, not retry permission. Mutations are serialized per target resource.

Desktop output bounds are policy: AX depth/nodes plus screenshot bytes/dimensions. Screenshots may contain sensitive information even when capture is read-only.

Cua Driver is resolved as the fixed `cua-driver` command from `PATH`; do not document `CUA_DRIVER_PATH` unless implementation support is added and tested.

## GitHub

Action preference is canonical in `GITHUB_BACKEND_PREFERENCE`:

- `repo`: clone → REST;
- `tree`: clone → REST;
- `file`: REST → clone;
- remaining actions: REST.

Do not collapse “REST-first” into “REST-only.”

The clone backend must remain hostile to ambient machine configuration: random private root, fixed argv, `shell:false`, disabled hooks/LFS/submodules/file protocol, bounded refs/paths, live size monitoring, final scan, unconditional cleanup.

`gh` must not receive the repository token. Raw git may receive token material only through the private ephemeral credential-helper path, never argv or ordinary inherited environment.

Clone fallback is selective. Invalid request and authentication failures surface directly. Backend unavailability/upstream/malformed execution may fall back according to the action chain. Authentication failure must not become an anonymous request.

The live size ceiling is polled, so brief overshoot is possible before abort. The final scan prevents serving an oversized completed clone; this is not strict filesystem quota isolation.

## Child processes and backend boundaries

Default `SearchBackend` is one-shot CLI; `SEARCH_BACKEND=mcp` selects MCP. Backend choice must not change public contracts.


### Child environment rules

`src/cli/cli-backend.ts:buildCliEnvironment()` starts from a nonsecret base and adds credentials by tool family. Unknown tools receive the base only. A web-search child must not inherit GitHub/Reddit/graph credentials merely because they exist in the parent process.

`src/process/mcp-client.ts` is deny-by-default. Only its explicit benign/provider allowlists plus names admitted by `SEARCH_MCP_FORWARD_ENV_JSON` may cross. Secret-like names and non-benign `SEARCH_MCP_*` internals must continue to reject rather than “forward for convenience.”

Native/media/git children use `src/process/native-child-env.ts`. Python children use `src/process/python-child-env.ts`. Both are narrow capability environments, not sanitized copies of `process.env`.

**Every new Python spawn must call `buildPythonChildEnvironment()`.** Do not rely on a reviewer noticing ambient secret leakage later.

The local embedding sidecar is a special case with a stronger rule: `SidecarManager.start()` mints a fresh 256-bit token per start and sends it over child stdin only. It must not appear in argv, inherited env, or logs. Failed token delivery is startup failure, never unauthenticated fallback.

Fixed argv and `shell:false` are baseline subprocess requirements. Do not interpolate model/provider/user text into a command string.

## Social and media

Social public vocabulary is canonical in `src/social/social-contract.ts` and registry metadata in `src/capabilities.ts`. Unknown/legacy actions reject before backend dispatch.

Social is read-only in practice. Future write vocabulary does not imply write authority. `src/social/social-write-policy.ts` defaults writes off, provider allowlists are empty, and write-shaped requests must remain deny/dry-run only until a reviewed adapter is explicitly allowlisted.

Media is internal acquisition, not a tenth public model tool. Canonical media actions belong to `src/media/media-contract.ts`; legacy spellings such as `video`/`subtitle` must not be reintroduced as aliases.

Platform fallback must preserve evidence class and auth semantics. YouTube search/hot fail closed without the official key; details may use keyless oEmbed; transcript is the separately declared degraded adapter. Do not add generic-web substitution and call it platform-native success.

Cookie ingestion/login is explicit user/operator action. Startup and bare auto-setup may discover capability or install allowed dependencies; they must not import browser cookies or create authenticated sessions to make status look healthier.

## Multimodal transfer

Vision eligibility is destination-specific and fail-closed. Configuring one destination never authorizes another.

Synthetic probes establish only that the configured endpoint/model identifier answered the randomized challenge at probe time. They do not prove later routing or model identity. Keep this limitation visible in docs and risk analysis.

Private/authenticated GitHub material has an independent cloud-transfer gate: exact `PI_VISION_PRIVATE_GITHUB_TRANSFER=1`. Public vision configuration alone must never authorize private repository bytes.

Anonymous YouTube frame extraction is also separately gated by exact `PI_VISION_FETCH_VIDEO_FRAMES=1` plus vision eligibility. `yt-dlp`/`ffmpeg` children must remain credentialless, proxyless, `--no-config`, fixed argv, `shell:false`, and under the native child environment.

## Setup and credential acquisition

Setup policy belongs to `src/setup/*` plus the capability/provider registries. Separate these concepts:

- capability exists;
- capability is installed;
- credentials are configured;
- a live authenticated session exists;
- an action is currently usable.

Status should report those distinctions instead of manufacturing “configured” from a stale env hint or available binary.

User-authorized credential acquisition must stay opt-in. Never make first-start bootstrap, status probes, or automatic dependency setup perform cookie import/login as a side effect.


## Documentation and public descriptions

Public tool descriptions, schema descriptions, README examples, `.env.example`, and package metadata are part of the contract surface. Keep them synchronized with the registered path, not with residual modules.

Before claiming a capability:

1. trace the registered entry;
2. trace validation and policy;
3. trace the selected execution adapter;
4. trace normalization/admission;
5. verify fallback and failure semantics;
6. verify tests exercise that same path.

If a module remains for internal/CLI compatibility but the registered public route bypasses it, say so explicitly. Do not let “code exists” become marketing language.

Examples must use canonical request shapes. Prefer an executable CLI/test fixture over hand-written pseudo-JSON when one exists.

## Change review checklist

For any contract, provider, agent, browser, desktop, auth, or subprocess change, review all of the following:

- **Shape:** Are unknown fields/actions rejected? Are bounds reject-vs-clamp semantics intentional?
- **Authority:** Can model/external text influence provider selection, credentials, hosts, side-effect grants, or stop/budget ownership?
- **Failure:** Are empty, degraded, failed, cancelled, suppressed, stale, and unknown-outcome states still distinct?
- **Fallback:** Could a privileged/security failure fall into a broader or anonymous path?
- **Ordering:** Does concurrency preserve caller/registry/ledger order?
- **Budget:** Is spend charged at the intended attempt boundary on both success and failure?
- **Provenance:** Can model-visible output distinguish where evidence came from without leaking provider secrets?
- **Secrets:** Does every child/network destination receive only credentials it actually needs?
- **State:** Does every mutation revalidate a fresh code-owned state token/snapshot?
- **Docs:** Did public prose change with the executable contract, and were stale examples removed?

## Validation before commit

At minimum for repository-wide work:

```bash
npm run typecheck
npm test
```

For focused changes, also run the closest contract/policy tests. Agent-mode work should usually include `test/web/web-search-agent-seam.test.ts`, the relevant `test/web/agent/*` files, and runtime RPC tests. GitHub routing work should include `test/github/github-contract.test.ts` and `test/github/github-domain.test.ts`.

Do not invent package scripts in documentation. If a special test command matters enough to document, add the script or document the exact existing `node --import tsx --test ...` invocation.

A passing unit suite is not proof of deployment isolation. There is currently no integration test that proves container/network egress confinement end to end.

## Residual risks to keep visible

- Application SSRF checks still have DNS-rebinding/redirect/Chromium TOCTOU residuals; deployment egress remains authoritative.
- Loopback debug mode confines browser traffic, not arbitrary behavior of the debug server itself.
- Untrusted-content framing is advisory and cannot prove prompt-injection resistance.
- Clone live-size enforcement is polled, so brief overshoot before termination is possible.
- Synthetic vision probes prove a challenge response at probe time, not durable model identity.
- CLI one-shot isolation carries measurable startup cost; MCP is the persistent alternative for pool workloads.
- Social/video typed agent intents are ahead of current production specialist execution and must stay capability-intersected until native wiring lands.

When these risks change, update tests, README, and this file together. Do not quietly delete the warning because the happy path improved.

---
> Source: [rhinos0608/Pi-Northstar](https://github.com/rhinos0608/Pi-Northstar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
