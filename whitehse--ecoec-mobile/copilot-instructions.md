## ecoec-mobile

> **ecoec-mobile** is the Edge Platform **Track 3** monorepo:

# AGENTS.md — ecoec-mobile

## What this is

**ecoec-mobile** is the Edge Platform **Track 3** monorepo:

- Shared **C core** (`mobile_core`): offline `.wmap` package store, map session
  (tile counts + camera events), auth session SM, `apply_config`
- **Employee** and **customer** app shells (phase-1 offline stubs)
- **Android / iOS** hosts: phase-1 **WebView offline** package (not full native
  WebGPU)

Program design: `~/edge-platform-program-design.md` (Track 3).

## Layout

```
core/           — pure-C mobile_core + host_alloc + YAML apply
apps/employee/  — offline employee UI stub (HTML/JS for WebView)
apps/customer/  — offline customer UI stub
platform/android/  — WebView host (P3.6)
platform/ios/      — WKWebView host (P3.7)
packages/       — lab fixture basemap subset (unsigned OK phase 1)
tests/ fuzz/ docs/decisions/
```

## Language / build

- C11 core; CMake ≥ 3.20
- Sibling: **libyaml** (`$HOME/libyaml` or `SIBLING_ROOT`)
- Optional: **libsim** (P3.5b soft dep)

```bash
cmake -B build -S .
cmake --build build
ctest --test-dir build --output-on-failure

./build/mobile_core_demo --config config/mobile.example.yaml
./build/mobile_core_demo --fixture packages/fixture_basemap
```

## Directives

- **Must** keep mobile_core free of TLS and platform UI kits; hosts own sockets/UI.
- **Must** use a **single** `mobile_core_apply_config` path for file reload, UI
  “Reload config”, mtime-on-foreground, and tests (HUP-equivalent).
- **Must not** silently malloc after create for core-owned buffers; use
  `NEED_ALLOC` events + `mobile_host_alloc`.
- **Must** treat phase-1 map success as **tile counts + camera events**, not pixels.
- **Must** allow lab packages with `packages.require_signature: false`; field
  inventory requires signatures later.
- **Auth profiles**: `employee` vs `customer` claims match edgehost ADR-013 roles
  (bearer token storage in core; no crypto verify in phase-1 lab).
- **Phase-1 hello map**: **WebView offline SPA package** (chosen over clear-color
  GL/Metal HUD). Documented in P3.6/P3.7.

## Definition of done (Track 3)

- [x] Monorepo scaffold + ADRs M001–M008
- [x] Event queue + host_alloc + apply_config
- [x] Offline package store + fixture tile count API
- [x] Auth session SM profiles
- [x] Fuzz apply_config + store; optional libsim
- [x] Android + iOS WebView offline hello shells
- [x] Employee + customer offline UI stubs
- [x] Package sync helper + signature flag

## Related

| Path | Use |
|------|-----|
| `~/libyaml` | Config load |
| `~/libwebmap` | Package format + demo SPA subset (WebView assets) |
| `~/libsim` | Optional class-B/C sim (P3.5b) |
| `~/edgehost` | Online package sync target (P3.10) |

---
> Source: [whitehse/ecoec-mobile](https://github.com/whitehse/ecoec-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-01 -->
