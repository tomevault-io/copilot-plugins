## lumen

> Single-binary Rust/Axum service that supervises per-agent Chromium or Quickshell

# AGENTS.md

Single-binary Rust/Axum service that supervises per-agent Chromium or Quickshell
desktop sessions and serves a live viewer. Node exists only for Playwright E2E
tests and the container healthcheck.

## Verification gates

CI (`.forgejo/workflows/ci.yml`) runs, in order:

```bash
cargo fmt --all -- --check
cargo clippy --all-targets --locked -- -D warnings
cargo test --locked
cargo build --release --locked
make ui-test
```

Keep `Cargo.lock` in sync — CI builds `--locked`. Run the same order locally before committing, or just `make ci`, which mirrors it.

- Single Rust test: `cargo test host_matching_ignores` (name substring).
- Single E2E test: set the env from `make ui-test`, then `npm run test:e2e -- -g "pattern"`.
- Dependency audit: `npm audit --omit=dev`. There is no `cargo-audit`/`cargo-deny` on this host.

## The 8899 port trap

A production `lumen` container usually runs on 8899 with host networking and live sessions, so a test run must never assume whatever answers on a port is the service under test.

- Always run E2E via `make ui-test`: `bin/ui-test.sh` picks a free port starting at `LUMEN_TEST_PORT` (default 18899), writes a per-run config with its own data directory, and starts its own server. `playwright.config.js` sets `reuseExistingServer: false`, so the suite can only exercise the worktree it just started.
- Only test the production service by explicitly setting `LUMEN_URL`.
- Do not restart the production container to "refresh" tests; it owns real sessions. `bin/smoke.sh` runs against the live service and honors `LUMEN_PORT`/`LUMEN_CONTAINER`.

## Config and deployment facts

- Config layering in `src/config.rs::Config::load`: `LUMEN_CONFIG` file (default `config/lumen.toml`), then `LUMEN_PORT`, `LUMEN_CHROME`, `LUMEN_SWAY`, `LUMEN_QUICKSHELL`, `LUMEN_WTYPE`, `LUMEN_DBUS`, `LUMEN_ATSPI_REGISTRYD`, `LUMEN_TUI_COLS`, and `LUMEN_TUI_ROWS` env overrides.
- `compose.yaml` must forward every config surface the service reads and the healthcheck must probe the same port — these were once out of sync. The container's log level is `LUMEN_LOG`; never interpolate the host's `RUST_LOG`, which leaks in from the operator's shell.
- Caller-supplied session paths — Quickshell QML, Qt binaries, and terminal/ratatui executables — are absolute host paths that the service resolves in its own filesystem. They must therefore live under `LUMEN_PROJECTS_ROOT` (default `/var/lib/lumen/projects`), which `compose.yaml` bind-mounts read-only at the identical path; a host path outside that root simply does not exist inside the container.
- `bin/up.sh` converges: it recreates the container only when the running image's `org.opencontainers.image.revision` label differs from the checkout's, because compose otherwise keeps an old container serving a stale binary. Image ids cannot be compared directly — every rebuild produces a new one.
- Container runs with `network_mode: host`: pages reach host dev servers at `http://127.0.0.1:<port>`; `host.containers.internal` / `host.docker.internal` are mapped to loopback via `extra_hosts`.
- `bin/common.sh` sources `.env` and wraps `lumen`/container-runtime helpers (`LUMEN_RUNTIME`: podman or docker, podman-preferred auto-detect; `LUMEN_DOCKER_SUDO`: auto/1/0 controls sudo elevation when the user cannot reach the docker daemon); host helper scripts honor `LUMEN_PORT`. `make` targets reach the runtime through `bin/ctr.sh`, never a bare `docker`/`podman` call.

## Code invariants

- `Policy::check` (`src/config.rs`) is the single navigation-policy predicate, enforced at two points: Lumen's HTTP preflight (403) and a per-tab CDP `Fetch.enable` interception (`ERR_BLOCKED_BY_CLIENT`). Never add a second check. Enforcement scope (covers every tab Lumen mediates; not a sandbox for agent-created tabs) is documented in README "Security" — read it before reasoning about "bypass".
- Exactly one managed page per `CdpSession` (`src/cdp.rs`); tab activation replaces it and stops the old screencast. Activation happens through the API and through the supervisor watcher that adopts tabs the browser opens itself (`target=_blank`, popups). `ViewHub::rebind` must follow every managed-page swap, wherever it happens.
- Session profiles are ephemeral and owned by one instance: `<data_dir>/run/<name>-<suffix>` exists only while that browser or desktop session is alive. Shutdown, reaping, and startup reconciliation remove them; the path is service-generated so deletion never derives from API input. Desktop Wayland sockets use a short service-generated symlink because Unix socket paths have a platform length limit.
- `chromiumoxide::Page::close(self)` consumes the page — clone `target_id` before closing if you need it afterwards.
- Quickshell paths are executable QML supplied by the caller. Validate that they are absolute and existing, but do not treat the check as sandboxing.
- A Qt session path is an absolute executable plus arguments, validated like a terminal command. The session starts a private `dbus-daemon` and starts `at-spi2-registryd` against it **eagerly** (`src/desktop.rs`), then exports `QT_ACCESSIBILITY=1` and `QT_LINUX_ACCESSIBILITY_ALWAYS_ON=1` to the app. Do not replace the eager registry with lazy activation: when the host runs systemd, the bus launcher delegates activation to systemd, which cannot reach a private bus, and the tree goes empty. `src/accessibility.rs` reads and walks the tree; `Environment`-level trees are per session by construction. Quickshell renders Qt Quick but publishes no accessibility tree, so it stays screenshot-only.
- UI assets are embedded via `rust-embed` (`src/http.rs`): debug builds read `ui/` from disk, release builds embed. Verify UI changes under `cargo run`, rebuild the image for release behavior.

## Conventions

- Conventional commit subjects with scope: `fix(cdp): …`, `test(viewer): …`, `docs(policy): …`.
- Releases use `commit-and-tag-version`; `.versionrc.cjs` owns every version surface (Cargo.toml, Cargo.lock, the image-tag defaults, `.env.example`) and `CHANGELOG.md`. Never edit a changelog or version file by hand — run `npx commit-and-tag-version --release-as vX.Y.Z`.
- Do not push or publish unless asked. Two remotes: `origin` is the local Forgejo (`http://vigilance:3002/blackopsrepl/lumen.git`) and `blackopsrepl` is GitHub. Both run the same gates, and `.github/workflows/ci.yml` declares `workflow_call` so `.github/workflows/release.yml` reuses it — a pushed `v*` tag publishes a GitHub Release with generated notes, gated on those checks.

---
> Source: [blackopsrepl/lumen](https://github.com/blackopsrepl/lumen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-28 -->
