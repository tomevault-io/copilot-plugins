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
5. `architecture.md`, `plan.md`, ADR drafts, comments, and residual modules as design/history evidence.

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

- **CLI:** broad command vocabulary. The current tree exposes 28 stateless command IDs plus `broker.serve` and `jobs.status`.
- **Pi native tools:** at most 9 model-facing tools, controlled by `PI_SEARCH_NATIVE_TOOLS`; unset/blank means zero.
- **User slash commands:** setup/status/Chrome authorization flows that require operator intent and are not model tools.

`src/capabilities.ts` owns the Pi tool vocabulary:

`web_search`, `fetch`, `github`, `social`, `kg`, `graph`, `browser`, `desktop`, `agent_poll`.

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
| leaf runtime wire contract | `src/runtime/runtime-rpc-protocol.ts` |
| browser policy/session authority | `src/browser/*`, `src/chrome/*` |
| desktop policy/state | `src/desktop/desktop-contract.ts`, `desktop-policy.ts`, `desktop-tools.ts` |
| one-shot native/Python child envs | `src/process/native-child-env.ts`, `python-child-env.ts`, `mcp-client.ts` |
| broker client/host/state seam | `src/runtime/broker-*` |
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

Cookie import, browser login, and user-Chrome authorization are explicit operator actions. Startup, status probes, and bare auto-setup must not import cookies or create authenticated sessions to make health look better.

Current explicit Pi cookie consumers are Reddit, Bilibili, and YouTube. Other social adapters may own their own CLI/OpenCLI sessions. Do not infer that an environment token or imported browser cookie unlocks a backend unless the active adapter actually consumes it.

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

Leaf runtime transport negotiation never bypasses domain validation. Exact provider/model identifiers, closed request/reply shapes, bounded fields, safe errors, and explicit settlement/cancellation remain part of the wire contract.

## Browser, Chrome companion, and desktop

Public browser targets are HTTP(S) only and remain behind URL/DNS/SSRF/origin admission. Loopback debug mode is a separate exact-origin confinement path. A browser session must not silently widen its host/origin authority after redirects or navigation.

User-Chrome control requires the pinned companion bridge plus a live user authorization lease. Inventory/selection is operator-facing, not model input. Revoke, expiry, failure, or shutdown must return to the isolated backend.

Desktop is opt-in. Observe first. Mutations require fresh state identity and are never blindly retried after dispatch. Keyboard/text mutations that require human confirmation must fail closed headless.

## Social, media, and vision

Social is read-only in practice. Write-shaped vocabulary does not imply write authority. Provider write allowlists are empty; destructive/download/bulk-follow behavior must stay denied unless a future adapter is explicitly reviewed and authorized.

Media acquisition is not a Pi public tool. Keep platform evidence classes honest:

- YouTube search/hot use the official API and require `YOUTUBE_API_KEY`.
- YouTube details can degrade to keyless oEmbed.
- YouTube transcript has a separate unofficial keyless degraded path; do not call it official API success.
- Bilibili uses its declared native/session adapters.
- Do not introduce `yt-dlp` as an undeclared substitution for YouTube or Bilibili.

Vision is destination-specific and explicit opt-in. Configuring one destination does not authorize another. Cloud routes move admitted bytes/text off-machine. Private/authenticated GitHub content additionally requires exact `PI_VISION_PRIVATE_GITHUB_TRANSFER=1`.

Synthetic model probes prove only that the configured endpoint/model answered the probe at that moment. They do not prove provider identity or later routing.

## Broker and stateful authority

Before changing broker code, read `docs/adr/0010-gate-b-native-authority-boundary.md`, `plan.md` Gate B, and `docs/tier2-proof.md`.

The working tree currently exposes `broker.serve` and `jobs.status` in CLI help, while the plan text still records them as unregistered. Executable reachability is current truth; release readiness is not. Tier-2 privileged proof/signing remains an open release gate until those artifacts are updated together.

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

`architecture.md`, `plan.md`, and ADRs may intentionally describe target or staged states. If they disagree with reachable code, do not rewrite reality to match the plan. Either update the stale planning document when in scope or label the mismatch.

## High-value references

- `README.md`: operator/user overview.
- `SKILL.md`: root agent router.
- `skills/*/SKILL.md`: domain contracts.
- `.env.example`: config catalogue.
- `architecture.md`: architecture and authority model.
- `plan.md`: redesign phases and release gates.
- `docs/adr/`: focused decisions.
- `docs/tier2-proof.md`: privileged broker proof plan.
- `northstar-capability-matrix.md`: capability inventory.

When in doubt, prefer executable truth, narrow authority, explicit degradation, and less ambient context.

---
> Source: [rhinos0608/Pi-Northstar](https://github.com/rhinos0608/Pi-Northstar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
