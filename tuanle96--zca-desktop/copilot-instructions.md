## zca-desktop

> Cross-platform **Zalo desktop client** (personal-use, unofficial). Rust **Tauri v2**

# AGENTS.md — zca-desktop

Cross-platform **Zalo desktop client** (personal-use, unofficial). Rust **Tauri v2**
core hosting concurrent [`zca-rust`](https://github.com/tuanle96/zca-rust) sessions;
**SvelteKit / Svelte 5 + Tailwind v4 + shadcn-svelte** frontend; **bun** package
manager. An optional Rust **axum + sqlx** cloud backend (`server/`) provides email
magic-link auth, server-hosted accounts, and an encrypted sync/media store.

## Stack & layout
- **Rust core** — `src-tauri/src/`, layered `types → config → store → zalo → session → command` (forward-only: a module may only depend on layers earlier in that list).
- **Cloud backend** — `server/`: axum + sqlx (Postgres + S3). The app talks to it over HTTP (default `http://127.0.0.1:37880`). Run native (`cargo run`) or via Docker — see [`server/README.md`](./server/README.md).
- **Frontend** — `src/` (SvelteKit SPA, `ssr=false`) calls the core via Tauri `invoke`/`listen`. shadcn-svelte components live in `src/lib/components/ui/`; `cn` in `src/lib/utils.ts`; theme in `src/app.css`.

## Commands
- Dev app: `bun run tauri dev`  ·  Frontend build: `bun run build`  ·  Type-check: `bun run check`
- Cloud backend (dev, hot-reload): `docker compose -f server/docker-compose.dev.yml up -d --build` — brings up Postgres + MinIO + MailHog + the server at `:37880`.
- Rust build/test/lint: `cargo build|test|clippy --manifest-path src-tauri/Cargo.toml -- -D warnings` (cloud backend: same with `--manifest-path server/Cargo.toml`).
- Add a shadcn component: `bunx shadcn-svelte@latest add <name>`.

## Hard rules
- **Secrets.** Zalo credentials (`imei` + `cookie` + `userAgent`) are bearer tokens. Keep them in the OS keychain (desktop) or under per-user encryption (server); never log or echo their values, and never commit them — `.gitignore` blocks `*.cred.json` and `cookies.json`.
- **Layering.** Respect the forward-only layer order; don't add or rename a layer without a clear rationale in your PR.
- **Verify before done.** Run the relevant build/test/lint gates; don't claim a change works without checking it.
- **Posture.** Unofficial Zalo API, personal use only — avoid spam-like or bulk automation (ban risk for users).

## Codex Agent Notes
- This file is the Codex runtime boundary for repository work. Codex-facing instructions live here; local harness implementation files live under `.harness/`, `.agents/`, and `.codex/`.
- Do not assume Claude-only runtime behavior; Codex must use the same task contracts, evidence bundles, review gates, and readiness checks as every other agent runtime.
- Use the feature list in `.harness/feature_list.json` to understand planned/completed work before changing feature scope.
- For non-trivial changes, bind work to a task contract in `.harness/task-contracts/`. A task contract identifies allowed files/layers, risk, acceptance checks, and required reviewers.
- Evidence bundles live in `.harness/evidence/`; the evidence bundle docs are `.harness/docs/evidence-bundle.md` and the evidence bundle schema is `.harness/schemas/evidence-bundle.schema.json`.
- Review decisions live in `.harness/reviews/<taskId>/`; the review decision schema is `.harness/schemas/review-decision.schema.json`.
- Passing review decisions must bind to the task with `taskId`, bind to the feature with `featureId`, and list `checkedFiles` for review coverage.

## Mandatory Advisor Protocol
- Security-touching, public API, cross-layer, high-risk, or claim-done work requires the advisor protocol unless the active task contract explicitly says otherwise.
- The advisor decision artifact is `.harness/reviews/<taskId>/advisor-decision.json`; it must use reviewer `advisor`, include `taskId`, `featureId`, `checkedFiles`, and record a pass decision before claiming done.
- Do not weaken hooks, bypass review, or mark work complete when required advisor/readiness artifacts are missing.

## Readiness Gates
- Run `npm run harness:check` for the structural harness gate.
- Run `npm run harness:doctor` when validating Codex runtime surfaces.
- Run `npm run harness:readiness` before declaring repo-level readiness.
- Code/test green is not the same as harness green; report both when they differ.

## Pointers
- Cloud backend (run · deploy · env vars) → [`server/README.md`](./server/README.md)
- Contributing, CLA, PR checklist → [`CONTRIBUTING.md`](./CONTRIBUTING.md)
- Security policy → [`SECURITY.md`](./SECURITY.md)  ·  Usage caveats / ban risk → [`DISCLAIMER.md`](./DISCLAIMER.md)

---
> Source: [tuanle96/zca-desktop](https://github.com/tuanle96/zca-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
