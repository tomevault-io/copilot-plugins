## sublane

> Before significant changes, read `PRODUCT.md` for scope and status, `DESIGN.md` for interface rules, and [the architecture guide](https://sublane.dev/docs/architecture) for module ownership and implementation boundaries.

# Agent instructions

## Context

Before significant changes, read `PRODUCT.md` for scope and status, `DESIGN.md` for interface rules, and [the architecture guide](https://sublane.dev/docs/architecture) for module ownership and implementation boundaries.

Use English for code comments, `PRODUCT.md`, and primary developer documentation. Keep the English and Simplified Chinese UI dictionaries complete. English is the default interface language.

## Work style

Usage, operations, and architecture guides are maintained in [the documentation repository](https://github.com/murongg/sublane-website). Update guides there instead of duplicating them here; see [the documentation index](docs/README.md) for references retained locally.

- Make the smallest complete, scoped change; reuse existing mechanisms instead of speculative frameworks, packages, or services. Report assumptions and actual verification results concisely; live subscription and desktop compatibility require separate evidence.
- Do not create commits, publish a repository, or deploy without a user request.

## Releases

- Publishing a version is not complete until `CHANGELOG.md` includes that tag. The tag workflow generates GitHub Release notes but does not update the repository file.
- After the release workflow succeeds, use `git-cliff 2.14.2` to run `make changelog-check` and `make changelog`. Review the new version section and comparison link, then commit the changelog and merge it into `main`.

## Boundaries

- Follow the architecture guide's ownership boundaries. Keep allocation accounting separate from provider IO, browser authorization, and billing; membership, policy, storage, and version policy must not depend on SDK types.
- Backups use read-only SQLite snapshots, bounded archives, integrity/key validation, and atomic publication to new paths. Never overwrite live data or expose backups to unauthenticated users or members.
- New pools grant no member access until configured. Gateway keys have immutable pool bindings and never authenticate browser sessions. Secret disclosure is owner-only and audited; revocation removes encrypted values while legacy hash-only keys remain usable without disclosure.
- Keep one-to-one metadata on its owner (`api_keys`, `memberships`, `settings`); separate counters, relations, and history when lifecycle or cardinality requires it.
- Production embeds the client-rendered React app in Go; it must not require a Node.js server.
- Use pinned public CLIProxyAPI SDK executors through `internal/upstream`. Keep the SDK credential store empty, watcher inert, loopback routes blocked, and automatic refresh disabled. Never register live credentials or pass refresh tokens to execution auth. Only the explicit Antigravity refresh in `accounts.Prepare` may receive a refresh token; persist its result before execution. Never import upstream `internal` packages.

## Backend

- Use standard Go conventions, explicit dependency injection at test seams, and `net/http`-compatible handlers; domain services must not depend on chi.
- Default to loopback listening. Protect workspace management routes with administrator authentication; platform settings and backups require the platform owner. Enforce roles on frontend routes and backend requests without bypasses. Browser roles come from active membership; gateway workspace identity comes from the key, never the browser selection header. See [authentication](https://sublane.dev/docs/authentication).
- Use chi `Route`, method registrations, and router-level `Use` for session/role/gateway authentication, including 404/405 responses. Reserve `With` for endpoint-specific checks. Never dispatch methods or gateway paths inside handlers or serve HTML for API errors.
- First-run username/password setup atomically creates the initial workspace owner with a single-administrator guard. Password changes atomically revoke browser sessions; session creation rechecks the verified password hash. OAuth attempts remain session-bound, with PKCE where supported.
- Write application queries in `internal/storage/queries/` and run `make generate`. Never hand-edit `internal/storage/db/`; keep generated code with its SQL changes. Migration bootstrap SQL remains in storage.
- Domain services own transactions; use `queries.WithTx(tx)` throughout, never hold transactions during network IO or generation, and keep database rows separate from public responses. Successful mutation audit events share the domain transaction; automatic refresh is not manual authorization.
- Preserve existing SQLite migrations; use ordered additive migrations for subsequent schema changes.
- Serialize account credential changes and preserve upstream identity on reauthorization. Persist rotated credentials, model/quota snapshots, and version policy before publishing them. Official release checks stay bounded, respect manual pins, and never install executables. Quota cache reads check account enablement; stale values retain their observation time.
- Keep gateway candidates and models within the key's pool. Every request and WebSocket turn, including local prewarm, rechecks group access, model allowlists, key expiry, and enablement. Native conversations use one cross-provider affinity binding; existing conversations must never switch accounts after disablement, deletion, pool removal, saturation, or cooldown. Sessionless requests create no affinity.
- Model discovery respects account lifecycle revisions. Unknown or over-age catalogs never imply unrestricted support. Catalogs and inference use the same capabilities and current pool policy; recheck membership/policy after discovery IO and preserve administrator allowlists. Public catalogs use native IDs; select by capability and recheck legacy provider-scoped rules against the selected account.
- Hold model-request leases until body closure and release exactly once on cancellation. Share member rate/concurrency admission across keys and transports within a workspace; release member leases with the model observation. Commit aggregates and history together, with bounded model labels and explicit token coverage.
- Personal history and usage derive identity from enabled sessions and filter by workspace in SQL. Hide subscription account identities from personal history; full history and workspace summaries require workspace administrators.
- Bound OAuth states, bodies, stream events, WebSocket history, discovery, and concurrency. Preserve cancellation and graceful shutdown; shared background refreshes use process context and workers join before SQLite closes. Streaming needs explicit timeout/resource policies, not blanket response buffering.

## Frontend

- Do not use the JavaScript `void` operator, including to discard promises. Return or await promises where appropriate and handle failures explicitly. TypeScript `void` return types are allowed.
- Use PascalCase component filenames and lowercase domain folders. Follow TanStack Router conventions for routes.
- Reuse the adapted Shadcn Admin components under `web/src/components/ui/`.
- Use TanStack Query for remote state and validate API responses at the boundary.
- Keep user-facing text in `web/src/locales/en.ts` and `zh.ts`, including accessibility labels and error messages.
- Follow `DESIGN.md` for color, accessibility, responsive behavior, themes, and honest loading/failure/empty states. Never add fake account or usage data.

## Tests and verification

- Use TDD for behavior changes, with synthetic fixtures, fake upstreams, and temporary SQLite databases only. Keep meaningful tests near the code; no tests solely for text/nonfunctional edits, deleted content, or implementation details. Comment non-obvious invariants and ordering constraints beside critical logic after verification.
- Run `make check` before completing a broad implementation. For a small change, start with affected tests and run the relevant build or checks.
- For UI changes, verify the affected flows in the browser at desktop and narrow widths, in both themes, and with English and Chinese where relevant.
- For deployment changes, validate Compose and build the image when Docker is available. State clearly when the Docker daemon is unavailable.

## Credentials and licensing

- Never automatically read local Codex credentials or commit databases, credentials, `.env` files, auth caches, or upstream request bodies. Never return full keys in metadata or cache disclosed keys in the frontend.
- Audit only bounded management metadata and actor context, never credentials, request bodies, raw URLs, or error contents. Request history excludes prompt/response/error bodies and credentials; do not log tokens or model prompt/response content by default.
- Preserve AGPL-3.0-only licensing and all third-party notices. Adapted Shadcn Admin components retain their MIT attribution.
- Do not invent repository URLs, maintainer contact addresses, support promises, or performance benchmarks.

---
> Source: [murongg/SubLane](https://github.com/murongg/SubLane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-29 -->
