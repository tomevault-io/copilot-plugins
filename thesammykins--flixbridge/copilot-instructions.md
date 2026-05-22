## flixbridge

> This document defines engineering principles, architectural constraints, and contribution patterns for this MCP server. It is intentionally focused on *how* we build (process, quality bar, patterns) rather than *what* we build. Keep it short, boring, and repeatable.

# AGENTS.md – How We Build Here

This document defines engineering principles, architectural constraints, and contribution patterns for this MCP server. It is intentionally focused on *how* we build (process, quality bar, patterns) rather than *what* we build. Keep it short, boring, and repeatable.

---

## 0. Core Constraints

1. Total handwritten runtime TypeScript LOC (excluding config, README, this file) MUST stay ≤ 700 lines, but the entire project can be more than that, any core service is exempt from this rule, but it should be kept as small as possible.
2. Zero heavy frameworks. Prefer the platform (Node 20+ LTS, native `fetch` / `undici`).
3. Minimal dependencies (target set):
   - `typescript@^5.9.x`
   - `zod` (runtime validation, narrow usage)
   - `@biomejs/biome` (formatter + linter; replaces ESLint/Prettier)
   - `tsx` (optional dev runner) OR plain `node` as appropriate
4. No class hierarchies unless polymorphism **cannot** be expressed more simply (object literals + functions preferred).
5. Strict TypeScript (`"strict": true`, include `noUncheckedIndexedAccess`, `useUnknownInCatchVariables`).
6. All external I/O (HTTP) funneled through one tiny helper to enforce:
   - Timeouts
   - Error normalization
   - Consistent logging hooks (single choke point)
7. The repository should always be in a shippable state (main = releasable).
8. Public surface area (exported symbols) stays tiny; internal refactors remain cheap.

---

## 1. High-Level Architecture

```
/src
  index.ts                 -> MCP server bootstrap + tool registration
  core.ts                  -> shared fetch wrapper (retry + timeout)
  debug.ts                 -> debug logging system (Phase 3)
  metrics.ts               -> observability & metrics collection (Phase 3)
  services/
    base.ts                -> shared service & operation type contracts
    shared.ts              -> BaseArrService implementation
    registry.ts            -> service registry: serviceName -> adapter
    arr/
      sonarr.ts            -> Sonarr adapter (conforms to shared contract)
      radarr.ts            -> Radarr adapter (conforms to shared contract)
    downloaders/
      sabnzbd.ts           -> SABnzbd downloader integration (Phase 2)
```

Rationale:
- `services/*` know raw REST shapes (endpoint paths, differences).
- `ops/*` expose normalized *intent-level* functions. They choose the right service adapter & endpoint.
- `index.ts` binds these operations as MCP tools (each tool intentionally coarse-grained).
- `http.ts` centralizes REST calling + error mapping.
- `mapping.ts` allows dynamic extension (future: add `lidarr`, etc.) without changing ops.

---

## 2. Service Abstraction Model

We unify common operations (see differences file) into a normalized capability contract.

```ts
// Pseudocode sketch (not final API)
export interface BaseService {
  id: 'sonarr' | 'radarr' | string;
  baseUrl: string;
  apiKey: string;
  // Raw endpoint path fragments (no query)
  paths: {
    systemStatus: string;             // "/api/v3/system/status"
    queue: {
      list: string;                   // "/api/v3/queue"
      details: string;                // "/api/v3/queue/details"
      status: string;                 // "/api/v3/queue/status"
      grab: (id?: number) => string;  // "/api/v3/queue/grab/{id}" or bulk
    };
    history: {
      base: string;                   // "/api/v3/history"
      since: string;                  // "/api/v3/history/since"
      detail: string;                 // "/api/v3/history/series" | "/api/v3/history/movie"
      failed: (id: number) => string; // POST path
    };
    mediacover: (mediaId: number, filename: string) => string;
    // Divergences captured as simple variant strings
    blocklist: {
      base: string;                   // "/api/v3/blocklist"
      scoped?: string;                // Radarr adds "/api/v3/blocklist/movie"
    };
    lookup: {
      base: string;                   // "/api/v3/series/lookup" or "/api/v3/movie/lookup"
      tmdb?: string;                  // Radarr only
      imdb?: string;                  // Radarr only
    };
  };
}
```

Key points:
- Divergent resource nouns become path variants (e.g. `series` vs `movie`).
- Optional properties indicate feature not present on all services.
- NO inline fetch logic inside adapters: they only produce URLs and supply semantic hints.
- Operation layer decides HTTP verb / body shape (shared semantics).

---

## 3. Operation Normalization

We define an “operation catalog” with stable semantic names. Example subset:

| Semantic Operation | Sonarr Path(s)                             | Radarr Path(s)                              | Notes |
|--------------------|--------------------------------------------|---------------------------------------------|-------|
| systemStatus       | /api/v3/system/status                      | /api/v3/system/status                       | Identical |
| queueList          | /api/v3/queue                              | /api/v3/queue                               | Identical |
| queueGrab          | /api/v3/queue/grab/{id|bulk}               | /api/v3/queue/grab/{id|bulk}                | Identical verbs |
| historyDetail      | /api/v3/history/series                     | /api/v3/history/movie                       | Divergent noun |
| coverImage         | /api/v3/mediacover/{seriesId}/{filename}   | /api/v3/mediacover/{movieId}/{filename}     | Key param name differs |
| lookupGeneral      | /api/v3/series/lookup                      | /api/v3/movie/lookup                        | Divergent noun |
| lookupTmdb         | n/a                                        | /api/v3/movie/lookup/tmdb                   | Radarr only |
| lookupImdb         | n/a                                        | /api/v3/movie/lookup/imdb                   | Radarr only |
| blocklistScoped    | n/a                                        | /api/v3/blocklist/movie                     | Radarr only |

Implementation guidance:
- Each semantic operation is one exported function in `ops/*` returning a typed result.
- Use discriminated union for results when divergent fields exist (e.g. `mediaKind: 'series' | 'movie'`).
- Provide adapter-level type generics only if they reduce duplication; avoid overfitting.

---

## 4. HTTP Layer

`http.ts` exports one function:

```
fetchJson<T>(input: RequestInfo, init?: RequestInit & { timeoutMs?: number }): Promise<T>
```

Responsibilities:
- Inject authentication header or `X-Api-Key` query param (config-driven).
- Enforce default timeout (e.g. 5000 ms) via `AbortController`.
- On non-2xx: parse JSON if possible; throw `ServiceError`.
- `ServiceError` shape:
```ts
{
  service: string;
  status: number;
  code?: string;         // if present in error JSON
  message: string;
  raw?: unknown;         // original parsed payload
}
```
- Retry policy: default 0; operations can request a small bounded retry (e.g. idempotent GETs: 1 attempt with exponential backoff cap 300 ms). The retry mechanism stays under 25 LOC.

Do not add axios or similar: unnecessary overhead.

---

## 5. Typing Strategy

Principles:
1. Define *minimal* response types for fields we actually use (avoid mirroring full upstream schemas).
2. Use `zod` to validate shape where safety matters (e.g. system status, queue items). Convert to typed object via `z.infer`.
3. Wrap unvalidated pass-through payloads in `unknown` and narrow only upon access.
4. Maintain one `types.ts` to host shared small interfaces & unions; keep it lean (< ~80 LOC).
5. Avoid over-modeling dynamic or pass-through arrays—prefer partial structural checks.

---

## 6. Error & Logging Policy

- All thrown errors leaving operation layer are either:
  - `ServiceError`
  - `InternalError` (shape `{ kind: 'internal'; message: string; cause?: unknown }`)
- No raw untyped throws (e.g. `throw "bad"`).
- Logging (if added later) should be injectable (a tiny interface) to keep core testable and under LOC limit. For now, rely on MCP host logging hooks or `console.debug` guarded behind an env flag.

---

## 7. Adding a New Service

Checklist:
1. Create `services/arr/<name>.ts` extending `BaseArrService` and implementing `ServiceImplementation`.
2. Provide required path variants (copy `arr/sonarr.ts` or `arr/radarr.ts` and adjust).
3. Register in `registry.ts` with a key used by configuration (e.g. `"sonarr"`).
4. If the service introduces new divergent endpoints:
   - Add optional fields to `BaseService.paths` only if they fit the pattern.
   - If concept is alien (no semantic overlap), treat as a new operation module (keeps cross-service core stable).
5. Write or adapt a `zod` schema only for data consumed by existing operations; do not preemptively model unused endpoints.

---

## 8. Testing / Verification Strategy

Given LOC constraints, no full test harness initially. Lightweight approach:
- Provide a `scripts/smoke.ts` (excluded from LOC budget if desired) that:
  - Loads local config (JSON)
  - Invokes `systemStatus` and `queueList` for each registered service
  - Prints concise pass/fail summary
- For core utilities (e.g. timeout logic), inline minimal assertions using Node’s `assert`.
- Avoid snapshot tests; they promote schema bloat.

If/when introducing a test runner: prefer `vitest` (lightweight, ESM-friendly) but only if it does not push us over LOC limit for runtime code (test code is separate).

---

## 9. Style & Tooling

- Biome: use formatter + linter with recommended rules.
  - Commands:
    - Format: `npm run format` (or `format:fix`)
    - Lint: `npm run lint` (or `lint:fix`)
    - Check all: `npm run check`
  - Conventions:
    - Avoid `any`; prefer `unknown` + narrowing.
    - Prefer for...of over Array.prototype.forEach for clarity/perf where flagged.
- tsconfig:
  - `"target": "ES2022"`
  - `"module": "ES2022"`
  - `"moduleResolution": "node16"`
  - `"isolatedModules": true`
  - `"verbatimModuleSyntax": true`
  - `"strict": true`
  - `"noUncheckedIndexedAccess": true`
  - `"useUnknownInCatchVariables": true`
- No barrel files (avoid circular risk and hidden exports).
- Named exports only (no default) except possible single default in `index.ts` for MCP bootstrap if required by host.

---

## 10. MCP Tool Design

- Each MCP tool is coarse enough to cover a *semantic operation* (not every REST endpoint).
- Tool naming: underscore_lowercase names (e.g. `list_services`, `system_status`, `queue_list`, `history_detail`).
- **Service Discovery**: The `list_services` tool MUST be called first to discover available services and downloaders before using any other tools.
- Input shape includes:
  - `service: string` (for service-specific tools)
  - Additional parameters (e.g. `ids?: number[]`)
- Output shape:
  - `ok: boolean`
  - `data?: T`
  - `error?: ServiceError | InternalError`
- Guarantee stable output contract to support LLM reasoning.

### Available Tools

- `list_services`: Discover all configured services and downloaders (no parameters required)
- Service-specific tools: All other tools require a valid `service` parameter obtained from `list_services`

---
## 10.1 MCP Server Implementation Details

This section adds MCP‑specific (Model Context Protocol) implementation mechanics beyond generic tool design.

Core runtime shape (request → response):
1. Receive JSON-RPC style MCP request (method = tool name, params = input object).
2. Validate:
   - Tool name matches registered underscore_lowercase names (e.g. `system_status`, `queue_list`)
   - Params schema via lightweight manual checks + optional `zod` (only for fields we actually branch on).
3. Lookup service adapter (`mapping.ts`). If missing: return structured `InternalError` (not generic string).
4. Invoke normalized operation in `ops/*` (pure, side-effect limited to HTTP).
5. Map result to tool response object `{ ok, data? | error? }`.
6. Emit response promptly; never hold open waiting for unrelated async work.

Tool registration:
- Done once in `index.ts` at startup. Provide:
  - `name` (tool id)
  - `description` (succinct, action oriented, <120 chars)
  - `inputSchema` (JSON Schema draft 2020-12 minimal subset; only describe required + optional keys actually used)
- Avoid dynamic tool creation per service instance; a single tool accepts `service` parameter.

Determinism & idempotence:
- GET-like ops (status, queue list, history detail) must be side-effect free.
- Mutating ops (e.g. queueGrab) MUST:
  - Echo inputs back in response metadata for traceability
  - Return a minimal confirmation object (`{ grabbed: number; ids: number[] }`) not the full upstream payload dump.

Versioning:
- Server exposes `serverVersion` constant (semantic version string) included in an optional capabilities or greeting payload (if MCP host supports).
- Breaking change to any tool input/output contract ⇒ bump MAJOR.
- Additive non-breaking fields ⇒ bump MINOR.
- Implementation-only refactors (no contract change) ⇒ bump PATCH.
- Do NOT silently remove or rename fields; use additive then deprecate.
- Maintain a `COMPAT.md` (future) only if contract churn begins; until then this section suffices.

Backward compatibility policy:
- Deprecate by:
  1. Mark field/tool as deprecated in description
  2. Support for ≥ 2 MINOR versions
  3. Then remove at next MAJOR
- Never reuse a removed tool name for a different semantic meaning.

Concurrency & scheduling:
- Hard cap simultaneous in-flight tool executions to a small number (≤ 4) to avoid overwhelming *arr services.
- Use a simple in-memory counter + queue (no external dependency). Each operation yields promptly on completion or error.
- Timeouts enforced centrally in `http.ts`. Operation layer should not introduce per-call custom timers unless justified.

Streaming (future optional):
- Current design returns whole JSON objects (no streaming chunks).
- If adopting streaming, each chunk MUST be a self-contained JSON object with `partial: true` until final chunk with `partial: false`.
- Do not stream for tiny payloads (< 4 KB) — overhead not justified.

Payload size discipline:
- Default: return only fields essential for downstream reasoning (IDs, titles/names, status, error indicators).
- If upstream payloads are large:
  - Provide summarization (counts, truncated arrays)
  - Include a `truncated: boolean` flag when slicing arrays
  - Offer follow-up detail operations rather than overloading a single tool.

Error normalization mapping:
- Network / timeout ⇒ `ServiceError` with `status = 0`, message contains timeout note.
- Upstream 4xx / 5xx ⇒ propagate HTTP status, attempt to surface upstream `error` / `message` field into `message`.
- Internal invariant violation (unexpected shape after validation) ⇒ `InternalError`.
- Never leak raw stack traces to tool consumer; include a short `message` and optionally a `debugId` (UUID v4) when more inspection is needed. Keep an internal (non-exported) map `debugId -> full error` for ephemeral debugging if implemented (not required initially).

Security & validation (MCP specific):
- Reject unknown `service` values early with `InternalError` (kind stable).
- For any user-provided URLs in future tools: validate scheme (`http`/`https`).
- Strip API keys from all error messages.
- Defensive JSON parsing: wrap parse in try/catch; if parse fails on error body, still emit structured `ServiceError` with `raw` omitted.

Observability (lightweight):
- Optional `FLIX_BRIDGE_DEBUG=1` environment flag triggers console debug lines:
  - `tool.invoke start { tool, service, t0 }`
  - `tool.invoke end { tool, service, ms, ok }`
- Keep logging under 10 LOC; do not import logging libraries.

Deterministic output ordering:
- Keys in `data` objects should follow a consistent order: identifiers first (`id`, `service`), then semantic fields, then meta (`truncated`, etc.).
- Arrays should be returned sorted by a clear criterion (e.g. queue list: ascending by `id` or start time). Document the sort rule in code comment.

Schema discipline:
- Keep JSON Schema inline & minimal (type, required, properties).
- Avoid `anyOf` / `oneOf` complexity unless strictly needed; prefer a discriminated union pattern with a `kind` field if divergence arises.

Failure containment:
- A failed call for one service must not impact others—no shared mutable caches that can be poisoned.
- Always clear per-request temporary state even on error (use `finally`).

Resource cleanup:
- AbortController created per HTTP call; ensure `abort()` on timeout path.
- No long-lived timers or intervals (avoid hidden resource leaks).

Deployment assumptions:
- Runs under Node 20+ ESM.
- Single process; scale out horizontally rather than adding in-process complexity if necessary.

Contract example (illustrative):
```json
{
  "ok": true,
  "data": {
    "service": "sonarr",
    "mediaKind": "series",
    "items": 25,
    "queued": [
      { "id": 123, "title": "Show A", "status": "downloading" }
    ],
    "truncated": true
  }
}
```
(Only include representative fields; keep stable naming.)

Quality gates (MCP-specific):
- Tool description must remain ≤ 120 chars to avoid host UI truncation.
- Input schema must reject extraneous top-level keys (validate & strip).
- All date/time fields standardized to ISO 8601 UTC strings (`new Date().toISOString()`); no ambiguous local times.

Adoption checklist when adding a new tool:
1. Define semantic need (not raw endpoint).
2. Extend adapter only if a new path fragment variant is needed.
3. Create `ops/<tool>.ts`, export function with explicit return type.
4. Register tool in `index.ts` with schema + description.
5. Update section 10.1 if new cross-cutting behavior introduced.
6. Run `scripts/smoke.ts` against at least one Sonarr + one Radarr instance.

This section is intentionally prescriptive to keep the surface stable for LLM and human consumers.

---

## 11. Performance & Resilience

- Batch where the API already supports bulk (queue grabs, blocklist deletes).
- Avoid parallel fan-out beyond a safe concurrency (simple limiter: max 4 in-flight).
- No caching until a proven need arises; keep semantics simple.
- All loops over remote calls must have hard caps or rely on provided lists (no unbounded pagination guesses).

---

## 12. Security & Configuration

- API keys read once at service creation; never logged.
- Provide configuration via environment variables (slug-based discovery); no JSON config files or JSON-in-env mappings.
- Do not embed secrets in code or commit sample keys.
- If any user-supplied URL: validate with URL constructor and enforce allowed protocols (`http:`, `https:`).

---

## 13. Documentation & Comments

- Prefer short doc comments over large narrative blocks.
- Any non-obvious decision (like ignoring a field) deserves a one-line rationale.
- Keep this file updated if:
  - LOC threshold changes
  - Architectural layer added
  - Error model expanded

---

## 14. Anti-Patterns (Do Not)

- Add a generic ORM/HTTP client library for trivial GET/POST calls.
- Mirror entire upstream OpenAPI spec (overkill).
- Export raw fetch wrappers per service (breaks normalization).
- Introduce inheritance to model divergence (`class SeriesService extends BaseService`)—use configuration objects.
- Leak upstream domain nouns into normalized responses without tagging (`mediaKind` tag required).

---

## 15. Incremental Evolution Path

1. Phase 1: Implement minimal operations (systemStatus, queueList, historyDetail).
2. Phase 2: Add write action (queueGrab) to validate mutation + error path.
3. Phase 3: Introduce lookups + mediacover retrieval.
4. Phase 4: Extend to additional *arr services (if desired) reusing same contract.

At each phase, re-measure LOC and prune dead code.

---

## 16. Definition of Done (Per Operation)

- Normalized function exported.
- Input & output types defined or reused.
- Minimal validation (zod) for consumed fields.
- Happy path + failure path manually smoke-tested.
- No linter or type errors.
- LOC delta keeps total ≤ 400.

---

## 17. Quick Start (Conceptual)

1. Configure services:
```ts
const services = [
  defineSonarrService({ baseUrl: 'http://localhost:8989', apiKey: process.env.SONARR_KEY! }),
  defineRadarrService({ baseUrl: 'http://localhost:7878', apiKey: process.env.RADARR_KEY! })
];
```
2. Register tools exposing operations referencing `services` registry.
3. Run MCP host; call `system_status` with `{ service: 'sonarr' }`.

---

## 18. Review Checklist (PR Gate)

Before merging:
- [ ] Total runtime LOC check passes.
- [ ] New operation documented in this file (if conceptually new).
- [ ] No untyped `catch (e)` usage (must be `unknown`).
- [ ] Error shapes consistent.
- [ ] No unnecessary dependencies added.
- [ ] Added service does not duplicate logic better expressed in ops layer.

---

## 19. Guiding Principle

Leverage sameness; isolate difference. Every line must justify itself under the LOC budget. If two services share a pattern, encode it once.

---

End of AGENTS.md.

---
> Source: [thesammykins/FlixBridge](https://github.com/thesammykins/FlixBridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
