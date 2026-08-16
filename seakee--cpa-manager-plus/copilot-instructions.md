## cpa-manager-plus

> For all agents. Shared approval semantics: `/Users/seakee/.codex/prompt-policy.md`; repo rules may only tighten them.

# Repository Rules

For all agents. Shared approval semantics: `/Users/seakee/.codex/prompt-policy.md`; repo rules may only tighten them.

## Operation

- Language: English unless the user asks otherwise.
- Dev flow: requirements -> read-only analysis -> plan -> explicit approval -> edit -> verify.
- Pre-approval: read files/history and run non-mutating commands only.
- Edit only user-named files or approved-plan files; dependent files may be read.
- Preserve unrelated user changes. No reset/checkout/delete/stage/commit/push/remote mutation without explicit approval.
- Prefer GitHub MCP `github` namespace tools for GitHub-related operations:
  repository reads, refs, branches, tags, releases, issues, PRs, checks,
  commits, contents, and remote mutations. Examples include `get_me`,
  `pull_request_read`, `create_pull_request`, and `create_branch`; legacy
  runtimes may expose the same tools as `mcp__github__*`. Use local `git`
  freely for local worktree and local history inspection. If the current
  session cannot actually call the required GitHub MCP tool, or if MCP cannot
  perform a required GitHub operation, state the MCP limitation and request
  explicit approval before using a scoped fallback such as `gh`, `git fetch`,
  `git pull`, `git push`, `git ls-remote`, or direct GitHub HTTP/API calls.
- Local CPA source checkout: `/Users/seakee/WorkSpace/Golang/src/github.com/seakee/CPA`.
  For CPA upstream code and history analysis, inspect this checkout first. Use
  GitHub MCP only when the required ref, issue/PR context, or remote metadata is
  unavailable locally.
- Search with `rg` / `rg --files`; manual edits with `apply_patch`.
- `docs/plan/` is a local ignored planning workspace. Use it for issue/epic
  plans, keep `docs/plan/roadmap.md` as the index, move near-term work to
  `active/`, keep unscheduled work in `backlog/`, and archive completed or
  superseded plans. Follow `docs/plan/README.md`.
- `AGENTS.md` / `CLAUDE.md` are collaboration files; commit them only on explicit request.

## Layout

| Path | Purpose |
|---|---|
| `apps/web/` | React 19 + Vite management panel; root npm scripts forward here |
| `apps/manager-server/` | Go 1.24 backend, `modernc.org/sqlite`, no CGO, embeds built panel |
| `tests/` | repo Vitest tests for architecture/source integrity |
| `docs/`, `img/` | user docs, migration notes, README assets |
| `docs/plan/` | ignored local planning workspace: roadmap, active plans, backlog, archive |
| `bin/release/` | release packaging |
| `bin/tmp/` | scratch: `bin/tmp/{mod}/{feat}/{desc}.{ext}`; clean stale files |

Release outputs: Docker images, native Linux/macOS/Windows packages for amd64/arm64, standalone `management.html`.

## Commands

| Task | Command |
|---|---|
| install | `npm install` |
| dev/build/preview | `npm run dev` / `npm run build` / `npm run preview` |
| type/lint/format | `npm run type-check` / `npm run lint` / `npm run format` |
| web + repo tests | `npm run test` |
| backend tests | `npm run manager-server:test` |
| backend run | `cd apps/manager-server && go run ./cmd/cpa-manager-plus` |
| backend race | `cd apps/manager-server && go test -race ./...` |
| manager stack | `docker compose -f docker-compose.manager.yml up --build` |

Focused tests: `npm --workspace apps/web run test -- <path>` or `cd apps/manager-server && go test -run TestName ./internal/...`. If `origin/main...HEAD` is missing for `repoSourceIntegrity`, set `CPA_MANAGER_CHANGED_FILES_BASE`.

## Architecture

- Supported modes: Full Docker and CPA Panel. Auth, setup, proxy, collector, panel hosting, and shared config changes must preserve both.
- Full Docker: Manager Server serves `/management.html`; login uses `cmp_admin_...`; CPA Management Key is encrypted server-side in SQLite.
- CPA Panel: CPA serves the panel; browser holds CPA Management Key; Manager Server accepts CPA-key-compatible access.
- Backend flow: `model -> repository -> service -> controller -> router -> httpapi -> cmd/cpa-manager-plus`.
- Backend placement: business in `internal/service/<domain>`, HTTP parsing in `internal/http/controller/<domain>`, routes in `internal/http/router/router.go`; controllers stay thin.
- Frontend direction:

```text
pages -> features / components / entities / services / stores / hooks / utils
features -> components / entities / services / stores / hooks / utils
components -> entities / services / stores / hooks / utils
```

`features/` and `components/` must not import `@/pages`; enforced by `tests/frontendArchitectureBoundaries.test.mjs`. Use `@/` for `apps/web/src`.

## Build And Style

- Vite uses `vite-plugin-singlefile`; JS/CSS/assets must inline into `apps/web/dist/index.html`.
- Do not add external chunks or sibling runtime files: dynamic `import()`, worker files, `new URL('./asset', import.meta.url)`.
- Embedded panel HTML at `apps/manager-server/internal/httpapi/web/management.html` is generated during packaging; do not hand-edit it.
- `__APP_VERSION__`: `VERSION` -> `git describe` -> non-`0.0.0` package version -> `dev`.
- Frontend: TypeScript, 2 spaces, single quotes, semicolons, Prettier, ESLint, React Hooks/Refresh rules, i18n for visible text.
- Zustand stores live in `apps/web/src/stores/`; test non-trivial behavior locally.
- Go: `gofmt`; domain logic stays out of controllers.

## Security And CPA Runtime

- Never commit secrets, admin keys, CPA Management Keys, SQLite data, generated runtime files, or local config.
- CPA Management Key is encrypted with `data.key`; backups need SQLite files plus `data.key`.
- Data defaults: Docker `/data/usage.sqlite`, native `./data/usage.sqlite`.
- `fail_body` and `raw_json` stay local; APIs/JSONL exports expose sanitized summaries only.
- Monitoring depends on CPA usage publishing and queue retention.
- CPA retention defaults to 60s, caps at 3600s; `pollIntervalMs` must not exceed retention.
- Only one Manager Server should consume a CPA queue; RESP pop is destructive.
- RESP Pub/Sub/pop connect directly to CPA API port, commonly `8317`; HTTP proxies do not work for RESP.
- HTTP queue can use an HTTP proxy. Recommended CPA `v7.1.18+`; HTTP queue minimum `v6.10.8+`. Optional metadata must be absent-safe.

## Data Migration Availability (Mandatory)

- Data migrations must not block normal service startup. The HTTP listener, management page, and core health endpoints must become available without waiting for work whose cost grows with stored data volume.
- Pre-listener migrations are limited to bounded metadata and schema operations. Large-table scans, backfills, clears, rebuilds, index creation, FTS rebuilds, and `VACUUM` must run after the listener starts or in an explicit maintenance mode.
- Online migrations must use bounded batches and short transactions, be idempotent and checkpointed, resume safely after interruption, expose start/progress/failure/completion logs, and keep a correctness-preserving read fallback while derived data is incomplete.
- A release that changes data migrations must validate 100k+ rows, interruption, restart/resume, and listener availability before rebuild completion. Failure of any of these checks blocks release.
- `usage_events` is authoritative migration input and must never be deleted or rewritten as part of rebuilding derived data.

## Verification

| Change | Verify |
|---|---|
| frontend behavior | `npm run type-check`, `npm run lint`, `npm run test` |
| frontend build/single-file | add `npm run build` |
| backend behavior | `npm run manager-server:test` |
| backend concurrency/worker/collector | add `cd apps/manager-server && go test -race ./...` |
| auth/setup/proxy/collector/monitoring | cover Full Docker and CPA Panel semantics |
| packaging/Docker | verify relevant release or Docker build path |

## Git And PRs

- No direct commits to `main`.
- Conventional subjects: `feat(web): ...`, `feat(manager-server): ...`, `fix(login): ...`, `docs: ...`.
- Keep commits scoped; no AI markers.
- PRs: purpose, tests, linked issues, UI screenshots/recordings when relevant, affected modes (`frontend-only`, `CPA panel`, `full Docker`, `native packages`).

---
> Source: [seakee/CPA-Manager-Plus](https://github.com/seakee/CPA-Manager-Plus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
