## streamarena

> Single full-stack streaming app: Rust/Axum backend (port **5173**) + SolidJS/Vite MPA. Two SQLite files: durable accounts/user data in `cache/users.sqlite` (never auto-wiped), regenerable cache in `cache/resolver-cache.sqlite` (self-heals from corruption). No Docker in-repo. See `README.md` for architecture and API surface.

# AGENTS.md

## Cursor Cloud specific instructions

### Product

Single full-stack streaming app: Rust/Axum backend (port **5173**) + SolidJS/Vite MPA. Two SQLite files: durable accounts/user data in `cache/users.sqlite` (never auto-wiped), regenerable cache in `cache/resolver-cache.sqlite` (self-heals from corruption). No Docker in-repo. See `README.md` for architecture and API surface.

### Toolchain (first-time / VM notes)

- **Rust:** `Cargo.toml` uses **edition 2024**. Run `rustup default stable` so `cargo` is ≥ 1.85. The image may expose an older `/usr/local/cargo/bin/cargo` until rustup’s stable toolchain is default.
- **PATH:** `export PATH="$HOME/.bun/bin:/usr/local/cargo/bin:$PATH"` (Bun is installed under `~/.bun` on fresh VMs).
- **System:** `ffmpeg` and `ffprobe` must be on `PATH` (used for remux/HLS/uploads).
- **Config:** `cp .env.example .env` once per workspace. Full TMDB/Real-Debrid resolve needs real keys in `.env` or environment secrets; auth, health, static UI, and mocked frontend smoke work without them.

### Commands (standard scripts in `package.json`)

| Task | Command |
|------|---------|
| Install JS deps | `bun install` |
| Playwright Chromium (smoke / embed helpers) | `bun run bench:playback:install` |
| Full-stack dev (build + server) | `bun run dev` → http://127.0.0.1:5173 |
| Backend only | `bun run dev:rust` / `cargo run` (needs `dist/` or use after `bun run build`) |
| Frontend HMR + API proxy | `cargo run` on 5173 + `bun run dev:vite` → http://127.0.0.1:4173 |
| Lint (JS syntax) | `bun run lint:frontend` |
| Rust tests | `bun run test:rust` |
| Frontend smoke (mock API, no Rust) | `bun run test:frontend` |
| Full CI-style check | `bun run check` |

### Running the server in Cloud Agent VMs

Use **tmux** for long-lived `bun run dev` (see portal tmux config under `/exec-daemon/tmux.portal.conf`). First startup compiles Rust dependencies and can take several minutes.

### Frontend smoke / sample media

`bun run test:frontend` expects sample files under `assets/videos/` (gitignored). If the directory is empty, create tiny placeholders with `ffmpeg` or symlink local test media; some player smoke cases (source switch / auto-fallback) can be timing-sensitive in headless CI.

### Optional integrations

TMDB home/bootstrap, resolver, and remote playback need `TMDB_API_KEY` and `REAL_DEBRID_TOKEN`. Sports/embed native HLS paths need Playwright Chromium installed.

---
> Source: [fightingentropy/streamarena](https://github.com/fightingentropy/streamarena) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
