## outcrop

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A **read-only** web dashboard for inspecting SlateDB databases directly from object storage: a Rust (axum) API server plus a React SPA, shipped as a single binary (`serve` with UI+API/api-only/ui-only modes, plus a `traffic` demo subcommand). DBs are **auto-discovered** by walking the configured store(s) for prefixes with a `manifest/` directory; multiple stores come from a TOML config (`--config`). The dashboard performs zero writes — only manifest reads, SST metadata reads, and object listings. The only thing in this repo that writes is `src/demo.rs` (the `traffic` subcommand), which seeds and churns the local demo DBs.

## Workflow

Always commit after making progress (e.g. after each fix or coherent unit of work), using conventional commit syntax (`fix:`, `feat:`, `chore:`, `docs:`, with an optional scope like `fix(web):`).

## Commands

```sh
# Seed demo DBs (demo-db-1..3) if missing, then churn them concurrently
# until Ctrl-C: varying-rate puts/deletes, embedded compactor + GC enabled
cargo run -- traffic                # --dbs N fleet size; --clean wipes ./demo-data first;
                                    # each DB randomly (stably by name) segments or not (--segments overrides)

# Bulk mode: --target-size switches seeding to unthrottled batched writes
# (embedded compactor + GC running) until the manifest's live bytes reach
# the target; resumable, since progress is measured from the store. Re-runs
# against an existing DB must repeat the same flags (key space isn't
# persisted). SST count ≈ target-size / sst-bytes.
cargo run -- traffic --target-size 50GiB --seed-only   # defaults: --dbs 1,
                                    # --value-bytes 4KiB..64KiB, --sst-bytes 32MiB;
                                    # --no-wal halves seed bytes written.
# Seeding is chunked: writes pause + GC when non-live bytes exceed
# --max-garbage (default 32GiB; the compactor's internal 15-minute
# checkpoints pin replaced SSTs, so pauses can last up to ~15m). Peak disk
# per DB ≈ target-size + max-garbage + in-flight compaction slack.

# Run the server (UI + API on 127.0.0.1:8333; LOCAL_PATH must be absolute).
# DBs are auto-discovered; there is no --path.
CLOUD_PROVIDER=local LOCAL_PATH=$(pwd)/demo-data cargo run
# Also: serve --config stores.toml (multi-store), --root PREFIX (scoped scan),
# serve --api-only (JSON + /metrics, CORS '*' by default), or
# serve --ui-only --api-url http://host:8333 (SPA only, browser calls API)

cargo test                          # backend unit tests
cargo test diff::                   # single module (also: convert::, cache::)
./scripts/smoke.sh                  # curl every endpoint against a running server

npm run dev --prefix web            # Vite dev server on :5173, proxies /api to :8333
npm run build --prefix web          # tsc -b && vite build → web/dist
npx tsc --noEmit                    # typecheck only (run inside web/)

# Single-binary release (frontend embedded via rust-embed)
npm run build --prefix web && cargo build --release
```

In **debug** builds rust-embed serves `web/dist` from disk at runtime, so after `npm run build --prefix web` a running debug server picks up frontend changes without a cargo rebuild. Release builds embed the assets at compile time.

Object store configuration comes from ambient environment variables exactly like `slatedb-cli` (`CLOUD_PROVIDER=local|memory|aws|azure` plus provider-specific vars), or from a `--config` TOML with the same keys lowercased inline per store (`${VAR}` values interpolate from the ambient env; stores are built pre-runtime in `src/config.rs` because slatedb's loaders read the process env).

## Architecture

Request flow: `routes::root_router` (`/api/dbs`, dispatcher, `/metrics`) → `Registry` (`src/registry.rs`: discovery scan via `src/discovery.rs`, lazy per-DB `AppState` + router) → per-DB `src/routes/*` (one file per page/resource, unchanged single-DB handlers, reached by URI-rewriting `/api/dbs/{id}/…` to `/api/…`) → `AppState` (`src/state.rs`) → slatedb `Admin`/`SstReader` or raw object-store listings → DTOs. DB ids are `"{store}:{path}"`; the SPA scopes everything under `/db/{id}/…`.

- **Caching (`src/state.rs`, `src/cache.rs`)** is the core design constraint: every uncached request costs object-store GETs/LISTs, and many viewers may poll. Mutable state (latest manifest, manifest listing, compactor state) sits behind `TtlCache` (default 5s, `--cache-ttl-secs`); its mutex is held across the refresh deliberately, so concurrent callers share one fetch. Objects that are immutable once written (manifest by id, SST detail by ULID) go in `LruMap` and are cached forever.
- **Compacted listing (`CompactedCache` in `src/state.rs`)**: `compacted/` is the one directory that grows with DB size, so it is never fully re-listed per poll. TTL refreshes do a single offset LIST from the last-seen ULID (new SSTs sort after existing ones); deletions reconcile via *background* full sweeps scheduled at `max(60s, 20 × last sweep duration)`, with a recent-write graft so writes racing a sweep don't read as deletions. Consequences to preserve: GC deletion events for compacted SSTs surface on sweep cadence, and the snapshot may briefly list GC'd SSTs.
- **DTO layer (`src/convert.rs` → `src/dto.rs`)**: slatedb domain types are converted to serde DTOs; `web/src/api/types.ts` mirrors `dto.rs` field-for-field and must be kept in sync. DTOs also derive `utoipa::ToSchema` and every handler carries a `#[utoipa::path]` documenting its **public** `/api/dbs/{db}/…` path (the dispatcher rewrite is invisible to the spec); `src/routes/openapi.rs` assembles the document, served at `/api/openapi.json` with a Scalar reference UI at `/api/docs`. New endpoints/DTOs must be added to the annotations and the `ApiDoc` paths list — the in-file spec test enforces path coverage. Rust `Option` fields use `skip_serializing_if`, so the TS side declares them optional (`?`), not `| null`. Keys are always sent as `KeyDto { hex, utf8? }` — `utf8` only when printable.
- **Diff (`src/diff.rs`)**: pure function diffing two manifest DTOs keyed on stable identifiers (L0 view ULIDs, sorted-run ids, checkpoint UUIDs); unit-tested in-file.
- **LSM summary (`src/summary.rs`)**: the LSM page uses `/api/lsm/summary`, not the full `/api/lsm` dump — per-level aggregates plus a 96-bucket max-point-depth coverage histogram, with per-SST detail only for levels ≤64 SSTs, so payloads stay bounded for huge trees. Pure and unit-tested in-file; results cached in an `LruMap` keyed `(manifest_id, segment)`. One segment per response (`?segment=i`; omitted = root, falling to segment 0 when a segmented DB's root is empty). Levels past the detail cap drill down via `/api/lsm/level` (`?run=id&start=hex&end=hex`), which slices one level's SSTs by key range from the cached manifest — pure CPU, no store requests.
- **Errors (`src/error.rs`)**: `ApiError` → JSON `{"error": ...}` with proper status. Do not put underlying object-store errors in client-facing messages — they embed server-side paths; log at `debug` and return a generic message (see `routes/ssts.rs`).
- **Static serving (`src/assets.rs`)**: unknown non-API paths fall back to `index.html` for client-side routing; `/api/*` paths must 404 as JSON instead.
- **List endpoints** cap their `limit` (manifests 500, compactions 200, both clamped to ≥1) and must keep it enforced for every parameter combination — each manifest summary or compactions version in a range costs one object-store GET.
- **Frontend**: react-query hooks in `web/src/api/client.ts` (live pages poll at `LIVE_REFETCH_MS`), pages in `web/src/pages/`, shared rendering in `web/src/components/` (`QueryGate` wraps loading/error states, `KeyDisplay`/`keyText` render `KeyDto`s).

## slatedb crate gotcha

The pinned `slatedb` crates.io release does not match the same-named git tag (e.g. 0.13.1 ≠ `v0.13.1`). When checking what a slatedb API actually does, read the vendored source under `~/.cargo/registry/src/*/slatedb-*/`, not the GitHub repo. To develop against a local checkout, use the commented `[patch.crates-io]` in `Cargo.toml`.

---
> Source: [criccomini/outcrop](https://github.com/criccomini/outcrop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-13 -->
