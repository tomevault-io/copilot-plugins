## memo-capture

> <!-- current-project-agent-baseline-addendum -->

# AGENTS.md

<!-- current-project-agent-baseline-addendum -->
## Current Project Agent Baseline Addendum

This section carries the current All Standards project-agent baseline. Preserve stronger project-specific instructions elsewhere in this file.

- For any repo setup, maintenance, versioning, or stack-selection work, apply the engineering-project-standard skill from `~/.codex/skills/engineering-project-standard`.
- For any frontend UI design, scaffolding, review, or refinement work, apply the web-app-design-standard skill from `~/.codex/skills/web-app-design-standard`.
- For any Docker, container, image build, image publishing, registry push, or container release work, apply the docker-build-and-publish skill from `~/.codex/skills/docker-build-and-publish`.
- For browser automation, use Chrome unless the user explicitly asks for a different browser or Chrome is unavailable.
- Browser navigation and browser session changes are allowed when they are part of requested or expected verification, testing, debugging, or UI review work.
- Default to Build Mode unless the user explicitly asks for release behaviour.
- Never use `latest`.
- Always use numbered versions.
- When the project is in Git, prefer Git-derived traceability by default.
- When distribution beyond local or dev use is explicitly requested, require publishable images to support both `linux/amd64` and `linux/arm64`.
- Before adding or changing local ports, check `/Users/paulmarshall/Software Development/All Standards/local-port-registry.md`; after updating it, run `python3 "/Users/paulmarshall/Software Development/All Standards/scripts/check-local-port-registry.py"`.
- For technical build work, use Technical Build Logs in `docs/build-logs/YYYY-MM.md`; do not duplicate the same technical work in `docs/completed-tasks.md`.
- If `project-decisions.md` exists, review the relevant recorded decisions before making project changes, and preserve those decisions unless the user explicitly changes direction.


## Core Skill Policy

For any repo setup, maintenance, versioning, or stack-selection work, apply the engineering-project-standard skill from `~/.codex/skills/engineering-project-standard`.

For any frontend UI design, scaffolding, review, or refinement work, apply the web-app-design-standard skill from `~/.codex/skills/web-app-design-standard`.

For any Docker, container, image build, image publishing, registry push, or container release work, apply the docker-build-and-publish skill from `~/.codex/skills/docker-build-and-publish`.

For browser automation, use Chrome for all browser automation unless the user explicitly asks for a different browser or Chrome is unavailable.

## Broad Project Policy

Prefer explicit user intent over convenience defaults. Defaults may suggest values or preselect options, but they are not permission to mutate state, activate features, publish, overwrite files, commit, tag, release, install, delete, send, or navigate/change app or browser state unless the user explicitly chooses or requests that action.

- Default to Build Mode unless the user explicitly asks for release behaviour.
- Never use `latest`.
- Always use numbered versions.
- When the project is in Git, prefer Git-derived traceability by default.
- When the user explicitly asks for distribution beyond local or dev use, require publishable images to support both `linux/amd64` and `linux/arm64`.
- Do not let container distribution work overwrite or weaken existing Codex instructions in this file.

## Repo Workflow Notes

Verified from the current workspace scaffold.

- Install command: `npm install`
- Development command: `npm run dev:desktop`, `npm run dev:api`, and `npm run dev:worker`
- Test command: `npm test` (requires local Docker/Postgres because it runs the Postgres lane first)
- Real Postgres integration test command: `npm run test:postgres`
- Lint or typecheck command: `npm run typecheck`
- Build command: `npm run build`
- Full verification command: `npm run verify`

Dependencies may not be installed yet in a fresh checkout. Run `npm install` before executing npm scripts.

## Runtime Notes

- Product shape: Tauri desktop app with a web UI, backed by a TypeScript API and worker.
- Browser-only desktop dev URL: Vite default `http://localhost:5173` unless Vite prints another port.
- AppLauncher local web URL: `http://127.0.0.1:5177`, reserved in `/Users/paulmarshall/Software Development/All Standards/local-port-registry.md`.
- Tauri desktop dev URL: strict `http://127.0.0.1:5178` from `apps/desktop/src-tauri/tauri.conf.json`.
- API port: `MEMO_CAPTURE_API_PORT`, default `4788`.
- API base URL for desktop: `VITE_MEMO_CAPTURE_API_URL`.
- Data store: Postgres via `DATABASE_URL`.
- Artifact storage: S3-compatible object storage via `OBJECT_STORAGE_*` environment variables.
- Auth: OIDC-compatible provider via issuer, audience, client ID, and JWKS config.
- Background work: API and worker are separate commands; worker claims Postgres-backed processing jobs.
- State Workflow Runtime: API and Desktop target the exact vendored `3.0.0` package family under `vendor/state-workflow-runtime/3.0.0`; Memo owns Runtime persistence in Postgres through its storage adapter, and `npm run doctor` enforces package checksums, family alignment, supported schemas, the `/testing` contract export, installed exports, and the browser-safe debugger boundary.

## Verification Notes

- Prefer `npm run verify` from the repo root after dependencies are installed.
- Use `npm run test:postgres` as the authoritative persistence lane. This command resets and migrates the isolated `memo_capture_test` database in the local `memo-capture-postgres-16-8` container, then runs Postgres-backed integration tests.
- `npm test` intentionally requires local Docker/Postgres by running `npm run test:postgres` before the remaining workspace tests.
- Use real Postgres for repository, API route, migration, transaction, JSONB, constraint, task-route, prompt-setting, file-type-setting, provider-profile-setting, workflow, and job-persistence tests.
- Keep in-memory fakes only for pure mapping or service tests where no SQL contract is being simulated.
- Do not point resettable automated test lanes at the shared `memo_capture` development database. Manual smoke testing may use `memo_capture` when the goal is to inspect the current local dev state.
- Use Chrome for browser validation unless the user asks for another browser.
- Browser automation is not required for backend-only changes.
- Report any script that cannot run because dependencies, Rust/Tauri tooling, Postgres, or object storage are unavailable.

## Documentation Records

- Keep `docs/completed-tasks.md` append-only. After every non-technical task, add a concise completion entry that records what was done, the outcome, and any relevant artifact or follow-up.
- Use `docs/build-logs/YYYY-MM.md` for technical work. After every technical build task that changes code, config, dependencies, tooling, tests, packaging, runtime setup, or verification docs, add one concise build-log entry.
- Do not duplicate the same technical work in `docs/completed-tasks.md`; the build log is the record for technical build history.

## Documentation And State

- Read `docs/design/memo-capture-design-learnings.md` before architecture, schema, workflow, ingestion, AI, or export work.
- Keep product decisions in `docs/design/`.
- Update docs when changing user-facing behavior, workflows, setup, deployment, or verification.

## Project-Specific Constraints

- V1 uses a cross-platform Tauri desktop app, TypeScript backend API, TypeScript worker, Postgres, and S3-compatible object storage.
- Desktop clients must not connect directly to Postgres or object storage.
- Backend settings are canonical; watched-folder and archive paths are desktop-local settings.
- Workflow actions, buckets, and reopen behavior must be driven by the active workflow definition wherever possible.
- The app stores only the active workflow definition bundle; rollback requires re-importing a known-good external bundle.
- V1 blocks workflow activations that require app-code migrations.
- All signed-in users are admins in V1, but authentication is still required.
- AI output consumed by code must be structured JSON and validated before storage.
- CSV export is out of scope for V1.

## Agent Notes

- Inspect relevant files before editing.
- Preserve explicit user requirements and stronger project-local instructions.
- Keep changes scoped to the requested work.
- Do not commit, tag, release, publish, install dependencies, or delete files unless the user explicitly asks.
- Report verification performed and any verification that could not be run.
## Port Registry

Before adding or changing local ports, check and update
`/Users/paulmarshall/Software Development/All Standards/local-port-registry.md`; record project ports in this file's Runtime Notes. After updating, run:

```bash
python3 "/Users/paulmarshall/Software Development/All Standards/scripts/check-local-port-registry.py"
```

---
> Source: [paulmarshall-wfw/memo-capture](https://github.com/paulmarshall-wfw/memo-capture) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
