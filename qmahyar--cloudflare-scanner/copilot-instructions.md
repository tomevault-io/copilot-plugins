## cloudflare-scanner

> go build -ldflags="-s -w" -o Cloudflare-Scanner.exe .

# Cloudflare Scanner — AGENTS.md

## Build & Test

```powershell
go build -ldflags="-s -w" -o Cloudflare-Scanner.exe .
go vet ./...
go test ./...
```

Cross-compile: set `$env:GOOS` / `$env:GOARCH` (linux, darwin, windows × amd64, arm64). CI matrix covers all 6 + termux-arm64.

### Local build scripts

- **`scripts\dev.ps1`** — fast inner dev loop. Rebuilds the UI only when `frontend\src` changed, builds the Go binary into `builds\<platform>\`, drops the matching xray-core sidecar next to it (cached under `builds\.cache\`, downloaded once), and optionally launches it. `builds\` is git-ignored.

  ```powershell
  .\scripts\dev.ps1            # build host platform -> builds\windows-amd64\
  .\scripts\dev.ps1 -Run       # build, then launch (finds xray next to the binary)
  .\scripts\dev.ps1 -SkipUI -Run   # Go-only rebuild + run (fastest)
  ```

- **`scripts\build.ps1`** — release packager: all-platform, re-downloads xray each run, emits release-identical archives to `dist\`. Use for producing distributables, not iterating.

## Key Architecture

- **Single binary + embedded UI**: `//go:embed all:ui/dist` in `httpserver.go` compiles the built Svelte bundle (`ui/dist/`) into the binary. No runtime files. `ui/dist/` is **git-ignored and not committed** — run `cd frontend && npm run build` before `go build` (at least once on a fresh clone, and after any UI change). CI builds the UI before every Go step.
- **Requires `xray.exe`/`xray` co-located** — app exits with download link if missing.
- **Entrypoint**: `main.go` → `startServer(xrayPath)` (random `127.0.0.1` port) → auto-open browser → `waitForShutdown` (blocks on SIGINT/SIGTERM, then cleans temp dirs and exits).
- **No HTTP router**: plain `http.ServeMux` with Go 1.22+ patterns; job IDs via `r.PathValue("id")`.
- **Two xray temp dirs** (both under `os.TempDir()`, one config.json + log per probe, `os.RemoveAll`'d on defer):
  - WARP noise-fallback scan → `os.TempDir()/_xray_work/<tag>/`.
  - Clean-IP scan → `os.TempDir()/_xray_clean/<tag>/`.
  - The default native WARP handshake path (`warp_probe.go`) uses no temp dir.

### Source file map (flat `package main`)

| File | Responsibility |
|------|----------------|
| `main.go` | Entry, xray presence check, browser launch (Termux/win/mac/linux) |
| `httpserver.go` | `http.ServeMux`, CSRF, SSE, job maps + TTL, embed UI, clamps |
| `scan_handlers.go` | WARP scan start/stop/status/results + apply-endpoint |
| `cleanscan_handlers.go` | Clean-IP scan handlers + export |
| `replacer_handlers.go` | Subscription fetch/parse/apply handlers |
| `pickers.go` | Native OS folder picker for output dir |
| `about.go` | Version + GitHub update check |
| `scanner.go` | WARP endpoint testing: native handshake; xray only for noise |
| `warp_probe.go` | Native WireGuard (Noise_IKpsk2) UDP prober |
| `xray.go` | Shared batch xray config + process lifecycle |
| `endpoint.go` | WARP endpoint generator |
| `config.go` | Parse WARP `.conf` |
| `cleanip.go` / `cleanip_gen.go` / `cleanip_measure.go` / `cleanip_validate.go` | Clean-IP pipeline split |
| `iprange.go` | Custom-range parsing + generation |
| `proxy.go` / `proxy_parse.go` / `proxy_share.go` / `proxy_xray.go` | ProxyConfig + parse/share/xray |
| `replacer.go` | Subscription fetch + config replace helpers |
| `noise.go` | UDP noise config + validation |
| `metrics.go` | median/best/jitter, quality score, result sorting |
| `*_test.go` | Offline unit tests (network/xray live paths untested) |

## Module & Conventions

- `go.mod` module = `github.com/QMahyar/Cloudflare-Scanner`.
- Depends on `golang.org/x/crypto` (WARP handshake) and `github.com/quic-go/quic-go` (HTTP/3 probes); otherwise stdlib.
- LDFLAGS `-s -w` strips debug info.
- Hogwarts-style WireGuard configs (S1/S2/S3, Jc, Jmin, H1-H4, I1-I2) are community-specific.
- No env vars — all config through web UI.
- Bilingual docs: `README.fa.md`, `docs/fa/`.

## CI & Releases

- **CI** (`.github/workflows/ci.yml`): matrix of 3 OS × 2 arch, each runs `go vet ./...` → `go build`. `go test ./...` runs on the **linux/amd64** cell only.
- **Release** (`.github/workflows/release.yml`): auto-triggered on `v*` tag. Builds 7 platforms, bundles matching xray-core v26.6.27, uploads `.tar.gz` to GitHub Release.
- Tag: `git tag vX.Y.Z && git push origin vX.Y.Z`

## UI (Vite + Svelte 5)

- Source in `frontend/src/`: one `*.svelte` component per tab in `components/` (`EndpointScanner`, `IpScanner`, `Replacer`, `About`) plus shared helpers in `lib/`.
- All i18n in `frontend/src/locales/en.json` + `fa.json` (identical keys), via `svelte-i18n`; language toggled in `lib/i18n.js`.
- API calls through `lib/api.js` `apiJSON()` wrapper.
- Scan status streams over SSE (`lib/sse.js` `subscribeStatus`, `/api/scan-events` / `/api/clean-events`), with `setInterval` polling fallback; results still poll. Large tables use `components/VirtualTable.svelte` (@tanstack/svelte-virtual).
- Build: `cd frontend && npm run build` → emits `ui/dist/` (git-ignored, embedded by Go at build time). `npm run dev` for a hot-reload dev server.

## Key Architectural Decisions

- **Native WireGuard Handshake** (`warp_probe.go`): Uses `golang.org/x/crypto` (blake2s + chacha20poly1305) for cryptographically valid WARP endpoint validation — no xray process needed. Only falls back to xray when noise/obfuscation is requested.
- **Clean-IP Two-Phase Scan**:
  - **Phase 1**: High-concurrency TCP dial (default NumCPU×64, capped at 500 — 256 on Windows) with retry on timeout only.
  - **Phase 2**: xray validation (SOCKS5 → HTTP/204) + `/cdn-cgi/trace` for colo/country.
  - **Nearby scan**: Expands `/24` (IPv4) or `/64` (IPv6) around working IPs.
- **Two xray Temp Dirs** (auto-cleaned on shutdown):
  - `_xray_work/` — WARP noise-fallback scans
  - `_xray_clean/` — Clean-IP Phase 2 validation
- **No HTTP Router**: Uses `http.ServeMux` with `r.PathValue()` for job IDs.
- **In-Memory Job Maps**: `scanJobs` and `cleanJobs` with 10-min TTL cleanup.
- **SSE Streaming**: Real-time progress via `/api/scan-events` and `/api/clean-events` with polling fallback.
- **Embedded UI**: `//go:embed all:ui/dist` — `npm run build` → `ui/dist/` (git-ignored; built before `go build`, not committed).

## Test Coverage

- **Tested**: `parsers_test.go` (WARP config, proxy URL parsing, share URL round-trip, apply-endpoint path traversal), `cleanip_test.go` (IP generation, weight calculation, IPv6), `iprange_test.go` (CIDR/dash/single parsing, enumerate vs sample).
- **Untested**: Network/xray paths (handshake, xray process management, clean-IP Phase 2 validation, subscription fetch).

## Potential Areas for Improvement

- Expand test coverage to network/xray integration paths.
- Add more context to error paths (e.g., xray startup failures in `cleanip.go`).
- Consider architecture documentation for onboarding.
- Current security: apply-endpoint basenames uploads and validates endpoints; absolute output dirs allowed; folder picker uses native OS dialogs; loopback + CSRF.
- Performance: Native WARP handshake precomputes DH; clean-IP uses high concurrency (Phase-1 default NumCPU×64 capped at 500, 256 on Windows; Phase-2 validates in 16-endpoint xray batches).

---
> Source: [QMahyar/Cloudflare-Scanner](https://github.com/QMahyar/Cloudflare-Scanner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
