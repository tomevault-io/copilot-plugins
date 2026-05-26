## livectf-defcon34

> Infrastructure for the LiveCTF bot-game competition at DEFCON 34. Teams submit bots (compiled to a custom architecture) that compete in an automated game. The system handles bot submission, queueing games, running them in sandboxed containers, and publishing results and replays.

# LiveCTF DEFCON34 — Codebase Guide

## What this is

Infrastructure for the LiveCTF bot-game competition at DEFCON 34. Teams submit bots (compiled to a custom architecture) that compete in an automated game. The system handles bot submission, queueing games, running them in sandboxed containers, and publishing results and replays.

## Repo structure

```
/
├── bot-game/           # All Rust code, Docker images, deployment config, and frontend
├── ansible/            # Server provisioning (run once after Terraform)
├── infrastructure/     # Hetzner Cloud Terraform (servers, network, firewalls, DNS)
├── infrastructure-bootstrap/  # Terraform for the TF state bucket (run once ever)
├── docs/               # cicd.md — deployment design decisions
├── Makefile            # Top-level shortcuts (infra, ansible, compose)
└── README.md           # Deployment runbook
```

## Frontend (`bot-game/frontend/`)

Vue 3 + Pinia SPA built with Vite and pnpm. Serves the team-facing UI (leaderboard, games, bots, profile, watch mode). The WASM game viewer is embedded at `/game/` as a separate static asset served by the same nginx container.

- Package manager: **pnpm** (not npm). Use `pnpm install`, `pnpm run dev`, `pnpm run build`.
- Dev server proxies `/api` to `http://localhost:3000` (configured in `vite.config.ts`).
- `.env.development` sets `VITE_USE_GAME_SHIM=true` and `VITE_USE_MOCKS=true` — MSW intercepts all API calls in dev so no backend is needed.
- `GameViewer.vue` dynamically imports either `GameViewerShim.vue` (dev) or `GameViewerWasm.vue` (prod) based on `VITE_USE_GAME_SHIM`. With no `.env.production`, the shim branch is constant-folded out of prod builds, so `GameViewerShim.vue` and its `lz4js` dependency are fully tree-shaken (zero prod bundle cost). Both viewers auto-detect the LZ4-frame magic and handle compressed or plain replays.
- The WASM viewer calls `import('/game/game.js')` → `init('/game/game_bg.wasm')` → `start(canvasSelector, replayUrl)`. These are typed via `src/wasm.d.ts` with `paths` in `tsconfig.json`.
- `Dockerfile.frontend` has a `vue-builder` stage (node:22-alpine + pnpm) that runs before the WASM build stages; the Vue dist is copied to nginx root, WASM to `/game/`.

## Cargo workspaces

The code is split into **five independent Cargo workspaces** under `bot-game/`. There is no workspace at the `bot-game/` level itself.

| Workspace | Path | Crates |
|---|---|---|
| backend | `bot-game/backend/` | `api`, `entities`, `migration` |
| cli | `bot-game/cli/` | `cli` (binary: `livectf`) |
| coordinator | `bot-game/coordinator/` | `coordinator` (single-crate workspace) |
| game | `bot-game/game/` | `game` (single-crate workspace) |
| runner | `bot-game/runner/` | `compiler`, `compiler-bin`, `driver`, `emulator`, `game-common`, `game-engine` |

Always `cd` into the relevant workspace before running `cargo` commands. There is no top-level workspace to run them from.

## Crate overview

| Crate | Workspace | Description |
|---|---|---|
| `api` | backend | Axum REST API for teams and admins. SeaORM + PostgreSQL, object storage via `object_store`. Checks for pending migrations at startup and refuses to start if any exist. |
| `entities` | backend | SeaORM database entity models, shared by `api` and `migration`. Includes `Phase` (phase1–phase6) and `RunStatus` (pending/queued/running/finished/failed) as PostgreSQL native enums. **Auto-generated** — do not hand-edit the files in `src/entities/`. Manual trait impls (Display, FromStr, JsonSchema) live in `src/impls.rs`. |
| `migration` | backend | SeaORM schema migrations. Run with `sea-orm-cli migrate` or the `migration` binary. |
| `cli` | cli | Command-line client for the API (`livectf`). Covers every endpoint. Config via `Config.toml`, overridable with `--api-url` and `--token`. |
| `coordinator` | coordinator | Polls the API for queued games, auto-detects its phase by running `driver --phase`, downloads bots, runs `driver` under nsjail, reads back replay/scores files, uploads artifacts and final scores. No internal path dependencies. |
| `game` | game | Bevy-based game visualizer, compiles to WASM for browser playback. Path-depends on `game-common` from the runner workspace. Exports `start(canvas_selector: &str, replay_url: &str)` via `wasm_bindgen`; uses `#[wasm_bindgen(main)]` as the wasm32 entry point. |
| `compiler` | runner | Compiles the custom bot language to emulator bytecode. Depends on `emulator`. |
| `compiler-bin` | runner | CLI wrapper around `compiler`: `--source-path <dir> --binary-path <dir>` compiles every `.bot` in the source dir to `<stem>.bin`. Used by the runner Dockerfile to compile NPC bots. |
| `driver` | runner | Runs a single game: executes bots on the `emulator`, drives `game-engine`, writes replay/log/score **files** into `--output-path`. Feature-gated per competition phase. Supports `--phase` / `--version`. See "Game execution pipeline". |
| `emulator` | runner | Custom CPU architecture emulator. No internal dependencies. |
| `game-common` | runner | Shared types (positions, directions, game state) used by `driver`, `game-engine`, and `game`. |
| `game-engine` | runner | Core game logic and simulation. Depends on `game-common`. C++ bindings via bindgen/cmake. |

### Internal dependency graph (runner workspace)

```
emulator ──┬──► compiler ──► compiler-bin
           │
           └──► driver ◄─── game-engine ◄─── game-common
                                │
                         game ──┘ (via path dep to runner/game-common)
```

`coordinator` and `game` have no internal deps within their own workspaces.

## Running tests

Integration tests in the backend workspace require a running PostgreSQL instance and the `integration` feature flag:

```sh
cd bot-game/backend
cargo test -p api --features integration
```

Unit tests (no database needed):
```sh
cd bot-game/backend
cargo test -p api
```

The `driver` crate has phase-gated features. To test a specific phase:
```sh
cd bot-game/runner
cargo test -p driver --no-default-features --features phase1
```

## Docker images

Eight container images are built from `bot-game/`:

| Image | Dockerfile | Build context | Description |
|---|---|---|---|
| `bot-game-api` | `deploy/Dockerfile.api` | `bot-game/backend/` | API server |
| `bot-game-frontend` | `deploy/Dockerfile.frontend` | `bot-game/` | Game WASM + nginx reverse proxy |
| `bot-game-runner-phase{1-6}` | `deploy/Dockerfile.runner` | `bot-game/` | coordinator + driver (one image per phase) |

Build all images from `bot-game/`:
```sh
cargo make docker-build
```

Build individually:
```sh
cargo make docker-build-api
cargo make docker-build-frontend
cargo make docker-build-runner-phase1   # through phase6
```

The runner Dockerfile has two independent cargo-chef build chains (coordinator and driver) combined into one runtime image. The driver is built with `--no-default-features --features phase<N>` — omitting `--no-default-features` causes a feature conflict at compile time. The `driver-builder` stage also builds `compiler-bin` and compiles `runner/driver/npcs/` → `.bin`. The runtime image bakes in the world map (`/opt/bot-game/world/`) and NPCs (`/opt/bot-game/npcs/`). A separate `driver-export` (`FROM scratch`) stage is the **participant handout**: `cargo make docker-export-driver-phase{N}` tars it to `dist/driver-phase{N}.tar` containing `driver`, `libGameEngine*.so`, `world.txt`, `Honk1.png`, and `npcs/*.bin` (whatever is in that stage is the handout — no Makefile change needed to add files).

The frontend Dockerfile has three stages: `vue-builder` (pnpm build of `bot-game/frontend/`), WASM build stages for the game crate, and the nginx runtime that combines both. The `wasm-bindgen-cli` version is pinned via `ARG WASM_BINDGEN_VERSION` in the Dockerfile — keep it in sync with the version in `bot-game/game/Cargo.lock`. `wasm-opt` comes from a pinned upstream binaryen release (`ARG BINARYEN_VERSION`), not Debian's apt package: bookworm's binaryen v105 finalizes the indirect function table with `max == min`, breaking wasm-bindgen's startup `Table.grow()` ("failed to grow table by N" at runtime). Do not switch back to the apt `binaryen`. The nginx config source is `bot-game/frontend/config/nginx.conf`; it uses envsubst to inject `API_URL` at container start, served from `/etc/nginx/templates/`.

## Local dev (docker compose)

```sh
cd bot-game
cargo make compose-up      # start postgres + minio + api + frontend + runner
cargo make compose-logs    # follow all logs
cargo make compose-down
```

Config files for local compose live in `bot-game/deploy/config/`.

## API config

The API reads `Config.toml` (must exist — uses `Toml::file_exact`). Template at `bot-game/backend/api/Config.toml.tpl`. Key fields:

- `server_host` — bind address (`127.0.0.1` to accept only local connections, `0.0.0.0` for all interfaces)
- `server_port` — TCP port (default 3000)
- `paseto_secret` — 32-byte hex-encoded PASETO v4 key
- `admin_password` — password for the built-in admin account
- `[storage]` — S3-compatible object store (GCS in production, MinIO locally)
- `[external]` — **optional** integration with the external competition system ("bbb-api"): `base_url`, `bearer_token`, `jwt_public_key` (Ed25519 PEM), and an `[external.extra_headers]` map (staging needs `X-Staging-Token`). When the whole section is omitted (`Config.external` is `None`), JWT team login and score reporting return an error and the scheduler skips roster filtering. See "External competition system" below.

The Ansible template for the API is at `ansible/roles/bot-game-api/templates/Config.toml.j2`. The coordinator template is at `bot-game/coordinator/Config.toml.tpl`.

## Game lifecycle

Games are created in `Pending` status with an optional `start_at` timestamp. A background scheduler in the API (`api/src/scheduler.rs`, spawned with `tokio::spawn` in `main.rs`) ticks every 10 seconds and promotes ready games:

1. Query all `Pending` games where `start_at IS NOT NULL AND start_at <= now()`.
2. For each game, run a single `DISTINCT ON (team_id)` query (`scheduler::eligible_bots`) to get the latest bot per team submitted before `start_at`.
3. Add each eligible bot as a participant via `GameService::add_participant`.
4. Transition the game to `Queued` via `GameService::update_status` — **even with zero participants** (a scheduled game runs at its time regardless; `GameService::start`'s 0-participant guard is intentionally bypassed here and kept only for the manual admin start endpoint).

A game only auto-promotes if it has `start_at` set (CLI: `livectf games create --start-at <rfc3339>`); otherwise it stays `Pending` until `POST /games/{id}/start`. The "which teams" logic in `scheduler::eligible_bots` is currently all teams, designed to extend with enrollment filtering later. The coordinator then claims `Queued` games and runs them (see next section).

## Game execution pipeline (driver ↔ coordinator)

The driver is **file-based**, not stdout-based. Contract:

- Args are kebab-case: `--bots-path`, `--npcs-path`, `--world-path`, `--output-path` (all required), `--random-seed`, `--tick-count`, `--compress-output`.
- `--bots-path` / `--npcs-path` are directories; the driver loads only `*.bin` files and uses each file's **stem as its internal `team_id`**.
- `--output-path` is a directory the driver writes into: `game.replay` (LZ4-frame compressed unless `--compress-output false`), one `<team_id>.replay` per bot, and `scores.json` (`[{team_id, score}]`, NPCs excluded).
- stdout is only a progress bar — no result data.

Coordinator `run_game` flow: the `/games/queue/claim` response already carries `(bot_id, team_id, team_name)` per participant via a JOIN (no follow-up `get_bot` calls) → download each participant bot as `<bot_id>.bin` (so the driver's `team_id` == the API `bot_id`) → write a `teams.json` sidecar in the bots dir (`{ "<bot_id>": "<team_name>", … }`) that the driver and post-game tooling can both read to resolve player names without a DB lookup → run driver under nsjail → read `game.replay` + `scores.json` (both mandatory; missing ⇒ requeue) → upload `game.replay` as the `replay` artifact → `finish_game` with scores from `scores.json` (`map_scores`, every participant gets an entry, missing defaults to 0). Per-bot `<bot_id>.replay` uploads are gated by the `UPLOAD_BOT_LOGS` constant in `runner.rs` (currently `false`); when enabled, the coordinator additionally reads each per-bot replay (best-effort) and adds it to the upload under `log_<team_id>` (remapping bot_id → the real DB team_id via `BotInfo` populated from the claim response).

NPC bots live in `runner/driver/npcs/` (`.bot` source), compiled to `.bin` by `compiler-bin` in the runner Dockerfile and baked into the image at `/opt/bot-game/npcs/` (and into the handout).

### nsjail sandbox

`runner/nsjail.cfg` declares static read-only mounts for system libs, `/opt/bot-game/lib` (game-engine `.so`), and `/opt/bot-game/npcs`, plus `envar: "LD_LIBRARY_PATH"` (name-only passthrough — value lives only in `Dockerfile.runner`'s `ENV`). The coordinator injects per-game mounts via flags: driver `-R`, world dir `-R`, bots dir `-B` (rw), and the output dir `-B` (rw — must be a host bind, not the jail tmpfs, so files survive jail exit). **The runner container runs `--privileged`** (systemd unit + compose) — required for nsjail's user/mount/PID namespaces and pivot_root inside Docker; nsjail is the actual sandbox boundary for untrusted bot code, the container is a single-purpose deployment unit.

## Authentication & tokens

`tokens::create_token` takes a **required** `TimeDelta` — there are no non-expiring tokens. rusty_paseto's `PasetoBuilder` injects a default 1-hour expiry unless an `ExpirationClaim` is set explicitly, and `PasetoParser::default()` rejects tokens with no/expired `exp`, so every token sets an explicit expiry. Service tokens: `api generate-token` → **180 days** (use this for the coordinator/runner; principal `game-manager` is authorized for all coordinator endpoints). Interactive admin login (`livectf auth login --username admin`) → **7 days**. Admin-minted impersonation tokens (`POST /auth/impersonate/{team_id}`, exposed via `livectf auth impersonate <team_id>`) → **24 hours**, principal `Team{team_id}`, indistinguishable from a normal team session. Team login via `POST /auth/team` exchanges an external **bbb-api EdDSA JWT** for a 7-day `Team` session token (see "External competition system"). Auth failures return specific messages (`TokenError::{Expired,Malformed,InvalidClaims}`, `AuthenticationError::{MalformedHeader,InvalidScheme,ExternalAuth,ExternalUnavailable}`); the coordinator surfaces the API's error body via `ensure_ok`.

## External competition system ("bbb-api")

Integration with the external platform, all of it in `api/src/external.rs`. Active only when the `[external]` config section is present (`AppState.external: Option<ExternalIntegration>`).

- **Team login** — `POST /auth/team` takes a bbb-api **EdDSA JWT** (`{"token": "<jwt>"}`). `verify_team_jwt` checks the signature against the configured Ed25519 public key and validates `iss == "bbb-api"`, `aud == "livectf"`, `exp`; `iat` must be present but its value isn't range-checked. `sub` (a string) is the external team id. The handler resolves the team from the external roster, **auto-provisions** a local team (`TeamService::upsert_external`, keyed on `teams.external_id`), and mints a normal `Team` session token. The JWT's `gen` claim is checked against the roster entry's `authGeneration`: a token whose `gen` is below the team's current `authGeneration` is rejected as superseded (bbb-api bumps `authGeneration` to revoke previously issued tokens). Team login **fails closed** — if the roster fetch fails, membership and generation can't be verified, so the login is rejected (the opposite of the scheduler's fail-open roster filter).
- **`teams.external_id`** — nullable string; `NULL` for internal/test teams, set for bbb-api teams. Unique index.
- **Roster helper** — `ExternalClient::list_teams()` (`GET {base}/api/team/all`, bare JSON array) is reused by team login and by the scheduler. When the scheduler promotes a game Pending→Queued it filters participants to teams still on the roster (internal/`NULL`-external_id teams always kept); a roster-fetch error is logged and **fails open** (all teams included).
- **Score reporting** — `POST /external/scores/{phase}` (admin-only) pushes a phase's per-team totals to bbb-api (`PUT {base}/api/scores/livectf-{phase}`, body `{"score": {"<external_id>": <n>}}`). Scores pass through `external::rescale_scores` (a pass-through hook for future score normalization) first; teams without an `external_id` are skipped.
- All external requests carry `Authorization: Bearer <token>` plus the configured `extra_headers`, baked into the `reqwest::Client` as default headers.

## Handouts & phases

Per-phase "handout" files (the driver tarball teams need to build bots) are distributed through public, release-time-gated endpoints.

- **Handout bucket** — a second per-environment object-storage bucket (`bot-game-<env>-handouts`), separate from game state. Created by Terraform (`infrastructure/modules/environment/main.tf`); the API reuses the `[storage]` credentials/endpoint and only needs the extra bucket name (`storage.handout_bucket` in `Config.toml`). Read-only from the API via `storage::HandoutStorage`.
- **`Phases.toml`** — an optional config file loaded alongside `Config.toml` (`Toml::file`, so a missing file just yields an empty phase list). Each `[[phase]]` has `id`, `title`, `body`, `release_time`, and a list of `[[phase.download]]` entries (`id`, `label`, `object` = handout-bucket key). Rendered per environment by Ansible (`roles/bot-game-api/templates/Phases.toml.j2`) — phase content is identical across environments and authored in the template; only `release_time` differs, supplied by `phase_release_times` in `group_vars/<env>/vars.yml`.
- **Endpoints** (`api/src/routes/phases.rs`, public — no token required):
  - `GET /phases` — lists every phase whose `release_time` has passed, with its text and download links.
  - `GET /phases/download/{id}` — streams a handout file from the handout bucket. Returns 404 if the download id is unknown **or** its phase is unreleased (an unreleased handout must not be revealed).
- **CI** — the `export-handouts` job in `staging.yml` / `release.yml` builds the `driver-export` (`FROM scratch`) stage of `Dockerfile.runner` per phase, gzips the tar, and uploads `driver-phase<N>.tar.gz` to the environment's handout bucket. Needs `HANDOUT_BUCKET` / `STORAGE_ENDPOINT` vars and `STORAGE_ACCESS_KEY_ID` / `STORAGE_SECRET_ACCESS_KEY` secrets.
- **Frontend** — the public `/phases` page (`PhasesView.vue`) renders each released phase as a post with download buttons.

## Logging

Both `api` and `coordinator` use `tracing_subscriber` with `EnvFilter` honoring `RUST_LOG`. Defaults when unset: API `info,axum=warn,sea_orm=warn`; coordinator `info`. The systemd units set `Environment=RUST_LOG=…` and pass `-e RUST_LOG` into the container (the processes run containerized, so a bare `Environment=` isn't visible without `-e`).

## API response conventions

Responses are wrapped by `ApiJson` (`#[serde(tag = "status")]` + `#[serde(flatten)]` data). Because the envelope owns the `status` key, the `Game` model serializes its run status as **`game_status`** (not `status`) to avoid the flatten clash — CLI and frontend types mirror this. The leaderboard is **per-phase**: `GET /leaderboard` returns `{ phases: [{ phase, entries: [...] }] }` (scores grouped by the phase a game was played in; no cross-phase total).

## Phases

Games are associated with a phase (`phase1`–`phase6`) stored as a PostgreSQL native enum. The coordinator auto-detects its phase by running `driver --phase` at startup and passes `?phase=<phase>` when polling for games. Each runner image is built for exactly one phase via `--no-default-features --features phase<N>`.

## CLI (`livectf`)

The `livectf` binary covers every API endpoint. It reads `Config.toml` in the working directory (overridable with `--config`). An empty `token = ""` in config is treated as no token (no `Authorization` header sent). Build and run:

```sh
cd bot-game/cli
cargo build -p cli
./target/debug/livectf auth login --username admin --password <password>
./target/debug/livectf teams list
./target/debug/livectf games claim --phase phase1
```

## CI/CD

| Workflow | File | Trigger |
|---|---|---|
| Rust CI | `.github/workflows/build.yaml` | Push / PR to main |
| Staging deploy | `.github/workflows/staging.yml` | Rust CI passes on main |
| Production release | `.github/workflows/release.yml` | GitHub Release published |

`build.yaml` runs fmt, clippy, build, and test for every crate across all phases. Each matrix entry declares which workspace it belongs to via the `workspace` field — this is what sets `working-directory` for all cargo commands. Concurrent runs for the same branch are cancelled automatically.

The `game` crate is built for the **native host target** in CI, not `wasm32-unknown-unknown`. Running `cargo test` against a WASM target on an x86 runner is not possible without a WASM runtime, and the game has no meaningful unit tests anyway. WASM correctness is instead gated by `Dockerfile.frontend`, which does the full `wasm32-unknown-unknown` compile, `wasm-opt`, and `wasm-bindgen` steps. Do not add `--target wasm32-unknown-unknown` back to the CI matrix.

`staging.yml` builds all 8 images and pushes to ghcr.io, then SSH-deploys to staging servers. It checks that `API_HOST`, `RUNNER_HOST`, and `HETZNER_SSH_KEY` are set before attempting deployment.

`release.yml` re-tags already-built images by git SHA (no rebuild) and deploys to production after manual approval.

## Infrastructure

Hetzner Cloud, managed by Terraform in `infrastructure/`. Four servers: staging API, staging runner, production API, production runner. DNS for `livectf.com` is managed by the Cloudflare provider inside the `environment` Terraform module — each environment gets `api.<subdomain>.livectf.com` (A), `<subdomain>.livectf.com` (CNAME → api), and `runner.<subdomain>.livectf.com` (A). Staging uses `subdomain = "staging"`, production uses `subdomain = "play"`.

Ansible in `ansible/` provisions each server (Docker, systemd units, secrets). Docker logs go to journald (`--log-driver=journald`) so `journalctl -u bot-game-*` works. Images are pulled in `ExecStartPre` before each service start.

See `README.md` for the full deployment runbook and `docs/cicd.md` for architecture decisions.

---
> Source: [Live-CTF/LiveCTF-DEFCON34](https://github.com/Live-CTF/LiveCTF-DEFCON34) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-26 -->
