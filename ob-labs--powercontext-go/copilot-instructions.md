## powercontext-go

> - Preserve observable compatibility with the authoritative OpenAPI contract,

# PowerContext Go Engineering Rules

## Architectural commitments

- Preserve observable compatibility with the authoritative OpenAPI contract,
  current domain behavior, persistence formats, and concurrency semantics.
- `openapi/powercontext.yaml` is the HTTP source of truth. Files generated into
  `api/v1` must never be edited by hand.
- Core packages do not read environment variables, open databases, start
  goroutines, or own process lifecycle.
- `internal/runtime` owns scope validation, operation lifecycle, per-scope write
  serialization, scheduler coordination, and use-case orchestration.
- Inference must run outside SQL transactions. Artifact or Candidate writes and
  their associated cursor CAS must commit atomically.
- SQL projections and indexes are rebuildable; immutable Artifact revisions and
  Memory manifests remain authoritative.

## Dependency direction

```text
source / artifact / trigger / inference
  -> artifact families
  -> internal/review / internal/contextpack / internal/stats
  -> internal/work / internal/handoffreport
  -> internal/runtime
  -> internal/endpoint
  -> internal/httpapi / internal/mcpapi / internal/webui
  -> server / cmd
```

- `internal/sqlstore` may import public and internal domain packages but must
  not import `internal/runtime`, `server`, or transport packages.
- Domain packages must not import SQL, HTTP, MCP, environment configuration, or
  provider-specific implementations.
- Interfaces belong to their consumers unless they are deliberate public Core
  extension contracts.
- A new top-level Go package must define a deliberate public extension or
  product contract. Product-only domain and orchestration code belongs under
  `internal`; directory creation is not a substitute for cohesive same-package
  files.
- Do not introduce `pkg`, `src`, `common`, `utils`, `helpers`, or global
  `models`, `services`, and `repositories` packages.

## Go conventions

- Put `context.Context` first on every operation that can block or perform I/O.
- Prefer explicit constructors and narrow interfaces. Do not use a DI framework.
- Keep domain errors typed and map them to wire errors only in
  `internal/endpoint`.
- Use `log/slog` through `internal/observability/logging`; do not create a
  parallel logging interface.
- Log an error once at the operation-owning boundary. Never log Source or
  Memory content, queries, prompts, vectors, credentials, raw scope IDs, full
  database URLs, or unredacted local paths.
- Keep unit tests beside packages. Put cross-backend, differential, and process
  tests under `test`.

---
> Source: [ob-labs/powercontext-go](https://github.com/ob-labs/powercontext-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
