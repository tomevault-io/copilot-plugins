## junto

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
go build -o junto ./cmd/junto   # build the binary
go test -race ./...              # run all tests (CI requirement)
go test -race ./internal/transfer/... -run TestFallback  # run a single test
go vet ./...                     # static analysis (CI requirement)
gofmt -l .                       # check formatting; must be empty to pass CI
gofmt -w .                       # auto-format
```

Version info is injected at release time via `-X main.version=...` ldflags (see `.goreleaser.yaml`). Dev builds report `version = "dev"`.

## Architecture

junto is a leaderless watch-party coordinator. Two roles: **host** (`junto create`) and **joiner** (`junto join`). The host serves media files; joiners stream them P2P while watching in sync.

### Control plane: Nostr relays

All coordination (room hello/goodbye, playback state heartbeats, WebRTC signals) travels as NIP-44-encrypted ephemeral events over public Nostr relays (`internal/nostrx`). Each session generates a throwaway keypair; the room code encodes a 128-bit secret from which both the public room ID (relay filter tag) and the NIP-44 conversation key are deterministically derived (`internal/room`).

### Data plane: WebRTC

Media files transfer over a single persistent WebRTC data channel per file (`internal/transfer`). Signaling travels through the Nostr transport. ICE is **half-trickle** (`iceReader` in `transfer.go`): the offerer still sends a complete offer (it waits for gathering) but advertises `Signal.Trickle`; a capable answerer then sends its answer immediately and trickles its candidates as separate `"candidate"` signals, so the answer-side gather is off the connect path. Against a peer that doesn't advertise trickle (an old build), the answerer falls back to the non-trickle path (wait for gathering, one complete answer, no candidate messages) — the capability gate is required because old peers reject the unknown `"candidate"` signal kind. Offer-side trickle is a later phase (`Role` tag is already on the wire for it). The serving peer answers byte ranges on demand; the fetching peer requests ranges via JSON control frames on the data channel (schema documented on `ctrl` in `transfer.go`). NAT fallback: if STUN hole-punching fails, the downloader silently retries with `FallbackRelays` (populated at session start by `transfer.InitBuiltinRelay`).

Every byte is verified per 256 KiB chunk group against the host's announced BLAKE3/Bao root before it reaches disk (`internal/transfer/verify.go`; the host computes root + outboard tree at create, joiners fetch the tree once over the channel). Rooms whose `FileMeta` lacks the `Bao*` fields (old hosts) fall back to the legacy whole-file SHA-256 check.

**Swarm** (`internal/transfer/swarm.go`): peers advertise verified piece coverage on heartbeats (`Have`/`HaveDone` on `MsgState`); a streaming joiner runs one primary source (the host) plus up to two aux sources fetching piece-sized claims — window assist behind the primary's stride, rarest-first beyond the window. A claim table on `fileEntry` deconflicts sources; per-source preempt channels handle seeks. Any peer serves what it verifiably has (`ServeFromStore`, coverage-aware with `nak` for missing ranges); a completed streaming joiner turns host-eligible via `syncer.Deps.CanHostDynamic`. With no advertising peers the path is identical to the single-source downloader.

**Upload fairness** (`internal/transfer/fairness.go`): a serving node runs one `UploadFairness` across its concurrent viewers. Each viewer reports buffer depth (bytes ahead of its playhead) in additive `buf` data-channel frames (`Downloader.reportCushion`); a comfortably-buffered `sender` calls `fair.pace` before each chunk and briefly yields while a same-file viewer is close to stalling (capped, file-scoped, self-expiring). No-op with one viewer or when viewers don't report; degrades to the un-prioritized path. Stacks on top of the swarm (same node, same policy, more sources).

### Streaming playback path

When a joiner has no local files: a local HTTP range server (`internal/streamserver`) exposes in-progress downloads at `http://127.0.0.1:<ephemeral>/0`, `/1`, … and blocks reads on bytes not yet downloaded. mpv is pointed at these URLs. The downloader (`transfer.Downloader`) follows the playhead with an adaptive window (32–512 MB) and redirects on seek.

### Sync engine

`internal/syncer` is the session loop. It runs on both host and joiner. Design: **leaderless last-writer-wins** — every explicit user action (pause, play, seek, speed, audio track, subtitle track/delay, playlist jump) carries a version (wall-clock ms). The action with the highest version wins everywhere; ties break on largest pubkey so corrections flow one way. Between actions, 2-second heartbeats carry `PlayState` and double as drift beacons. Clock skew is estimated NTP-style on join and subtracted from every position projection. Drift > 0.5 s triggers a hard seek.

Each synced mpv property is observed (`obsPause`, `obsSpeed`, `obsSid`, `obsAid`, `obsSubDelay`, `obsPlaylistPos`, `obsPausedForCache`) and has an echo-suppression `expectation` in `internal/syncer/drift.go` so applying a remote command doesn't rebroadcast as a local action. Adding a new synced property means touching all four spots: the `obs*` const, the `Engine` field + `*Init` flag, the observation map in `Run`, the `case` in `handlePropertyChange`, `snapshot()`, the `applyRemoteState` block, and the suppression struct. `aid` (audio track) is the worked example. The readiness gate counts observed-property echoes (`len(seenProps) >= 7`); bump it when adding a property.

### Observability

- `internal/debug` — a nil-safe JSON-line `Logger` enabled by `-debug`. `l.E("event", "k", v, ...)` is a no-op on a nil `*Logger`, so it's threaded everywhere (`Transport.SetLogger`, `Downloader.SetLogger`, `ServeFile(..., log)`, `syncer.Deps.Log`) without guarding call sites. Writes to `~/.cache/junto/debug-<timestamp>.log`.
- `internal/telemetry` — opt-in anonymous ping. `Collector` accumulates metrics during a session; `Fire(version, outcome, reason)` POSTs once at the end in a goroutine (3 s timeout, silent failure). `ClassifyError` reduces errors to a fixed category set (no raw text). The endpoint is a single `const`. Opt-out: `config.Config.NoTelemetry` (`telemetry = false`) or `--no-telemetry`. It must never carry PII, room secret, or file names.

### Preflight & guided failure

`cmd/junto/main.go` runs `checkMpvEarly` before any relay work (fast failure on a missing player) and wraps session errors with `hintDoctor` (suggests `junto doctor` on relay/connect/NAT errors). `internal/doctor` is also callable standalone via `Run`; it ends with a `streamingSummary` verdict and reports built-in-relay and telemetry status.

### Key data-flow wiring

`cmd/junto/main.go` → `internal/app/app.go` (`Create` / `Join`) → `runSession` → `syncer.Engine.Run`.

`syncer.Deps` is the dependency-injection struct — it receives an `mpv.Client`, a send function, the relay inbox channel, and hooks (`OnFileReq`, `OnSignal`, `Buffer`, `DownloadProgress`). The engine loop is the sole consumer of the relay inbox; it forwards WebRTC signals to the downloader via `OnSignal`.

### Terminal UI (`internal/tui`)

On a real TTY, sessions run inside a `bubbletea` TUI; otherwise output is byte-for-byte the old plain path. There is **one** session body per command: `Create`/`Join` are thin gates (`tui.UseTUI()`) around `createInner`/`joinInner`/`joinStreaming`/`runSession`, all parameterized on `tui.Hooks` (Printf/ShowProgress/ClearProgress/OnSnapshot/SessionInfo/Lines). Each inner shadows the package `printLine` with `h.Printf`, so the body is identical either way. `plainHooks()` supplies the old `term`-backed writers; `tui.RunSession` supplies bridges that `tea.Program.Send` typed messages (safe from any goroutine — required because `Printf` fires from the sync loop, sender loop, and swarm downloader concurrently). `RunSession` owns the terminal via `p.Run()` while the session runs on a goroutine; typed Ctrl-C arrives as `KeyCtrlC` (raw mode, not SIGINT) → `tea.Quit` → `cancel()` → the normal `ctx.Done()` cleanup. The live party panel comes from the nil-safe `syncer.Deps.OnSnapshot` hook off `buildSnapshot()`, which is also the single source the text `/peers` formats from. `programOpts` is a package-var test seam (pipe-backed I/O for headless `RunSession` tests). When adding session output, route it through `h`/`Deps`, never write to stdout directly — a raw write corrupts the alt-screen.

### Config and paths

- Config file: `~/.config/junto/config.toml` (XDG-aware). The parser is hand-rolled (flat key = value, no TOML library). A `telemetry-notice` sentinel file sits alongside it (written after the first-run telemetry disclosure).
- Cache: `~/.cache/junto/` — yt-dlp downloads, `.part` + `.junto` resume sidecars, and `-debug` logs.
- Unknown config keys are a hard error (not silently ignored) so typos surface immediately.

### Distribution

- `scripts/install.sh` — the `curl junto.watch/install | sh` bootstrap (OS/arch detection, GitHub release download + SHA-256 verify, mpv via system package manager). `scripts/build-pkg.sh` — builds the macOS `.pkg` (no-op off Darwin); invoked by an explicit step in `release.yml`, not goreleaser (its `after` hooks run post-publish, too late to upload). Both must stay executable (`chmod +x`).
- The release workflow runs on **macOS** so `pkgbuild` is available; the Linux binaries are still cross-compiled (`CGO_ENABLED=0`). After goreleaser publishes the release, a step builds the `.pkg` and uploads it with `gh release upload`. CI (`ci.yml`) shellchecks `scripts/*.sh`.

### go-nostr pin

`go-nostr` is archived upstream (read-only since Jan 2026) and pinned at `v0.52.3`. Do not upgrade. The roadmap tracks replacing it with a hand-rolled minimal client.

### Testing patterns

- `transfer_test.go` runs a real WebRTC loopback (no mocks). It sets `minWindowBytes`, `tailPrefetch`, and `progressInterval` to small values to keep tests fast.
- `FallbackRelays` and `builtinCredsURL` (in `turn_creds.go`) are package-level vars specifically so tests can override them without build tags.
- The doctor package tests use `gatherCandidateTypes` against a real STUN server; they require network access.

## After every major change

Run these checks and update these files before considering a task done.

### CI / release sanity

```bash
go build -o junto ./cmd/junto   # binary must compile
go test -race ./...              # all tests must pass
go vet ./...                     # no vet warnings
gofmt -l .                       # must be empty (auto-fix with gofmt -w .)
```

Review `.github/workflows/ci.yml` and `.github/workflows/release.yml` for any steps that touch files you changed (goreleaser config, install scripts, shellcheck targets). If you added a new flag, config key, or binary behaviour, check that the workflow assumptions still hold.

### CHANGELOG.md

Add an entry under the `## Unreleased` section (or create one if absent). Use the existing format: bullet points grouped by type (`### Added`, `### Changed`, `### Fixed`). One concise sentence per item; no implementation details.

### ROADMAP.md

If the change implements or partially implements a roadmap item, mark it done or update its description. If it introduces a known limitation or follow-up work, add a new item with a brief rationale.

### README.md

Update if the change affects anything a user reads before installing: CLI flags, commands, configuration keys, supported platforms, or the feature list. Do not add implementation details to the README.

### Debug instrumentation (`internal/debug`)

The `debug.Logger` (`l.E("event", "k", v, ...)`) is threaded through every major component. After a major change, add structured log events for anything that would help diagnose problems in the field:

- **New code paths that can fail silently** — log the entry and outcome (e.g. `l.E("host_promoted", "self", pub)`, `l.E("nudge_complete", "restored_speed", s)`).
- **New async goroutines** — log start and any non-nil error on exit.
- **New state transitions** — log when the system enters and leaves a meaningful state (e.g. buffer gate open/closed, host changed, election started).
- **New retry / fallback logic** — log each attempt with enough context to reconstruct the sequence from the log alone.

Use the existing call-site pattern: `e.d.Log.E(...)` is a no-op on a nil `*Logger`, so no nil-guard is needed. Event names are snake_case verbs (`peer_join`, `drift_corrected`). Key names are short lowercase words. Values should be scalars — avoid logging file paths, room codes, or any user content.

---
> Source: [swayam-mishra/junto](https://github.com/swayam-mishra/junto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
