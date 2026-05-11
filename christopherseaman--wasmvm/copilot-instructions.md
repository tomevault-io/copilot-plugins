## wasmvm

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository status

**Spec-only repository.** There is no buildable code yet — no Xcode project, no package manifests, no tests, no CI. All material lives under `spec/`:

- `spec/README.md` — project overview, goals, non-goals, document map (read first)
- `spec/01-architecture.md` through `spec/08-investigation.md` — drill-downs by section
- `spec/code/*.swift` — **reference sketches** for core Swift components (VMHost, LocalWSServer, NetBridge, NinePServer). These are illustrative, not buildable; they show the intended API shape and protocol framing.

When implementation begins, the working tree layout described in `spec/02-ios-wrapper.md` (`WebVMApp/...`) is the target structure. Until then, treat the numbered spec docs as the source of truth.

## What this project is

A self-contained iPad app that runs unmodified i386 Linux binaries locally via a patched [CheerpX](https://github.com/leaningtech/cheerpx) (WASM x86 virtualization) inside a WKWebView. The Swift wrapper is a *services provider* to the VM, not just chrome around it. It supplies the three things a raw browser-hosted WebVM cannot:

1. **Networking** without Tailscale or any external control plane
2. **Persistent storage** not subject to Safari's opportunistic eviction
3. **File exchange** with iOS Files (Working Copy, iCloud Drive, etc.)

End-user goal: run LazyVim + standard shell toolchain (git, curl, python, build-essential, ripgrep, fd, tmux) on iPad with Files-app integration and no remote backend.

## Architectural shape (one-screen summary)

Single iOS app process. Inside it:

- **WKWebView** loads bundled HTML via a custom `webvm://` scheme handler that sets COOP/COEP headers (required for SharedArrayBuffer). Hosts patched CheerpX.
- **VMHost** (Swift `@MainActor`) — coordinator; owns both WS server lifecycles and the security-scoped bookmark for the user-selected shared folder.
- **NetBridge** on `ws://127.0.0.1:8080/net` — translates raw-socket-over-WS frames into `NWConnection` calls (Network.framework). Replaces CheerpX's Tailscale/WireGuard transport.
- **NinePServer** on `ws://127.0.0.1:8081/9p` — implements 9P2000.L; exposes the security-scoped folder at guest `/mnt/host`.
- **Patched CheerpX** (JS/WASM) — stock except (a) networking transport replaced with the WS raw-socket client and (b) a `9p` mount type added that speaks to the NinePServer.

Block device stack:
```
/          ext2   OverlayDevice(HttpBytesDevice("webvm:///disk/base.ext2"), IDBDevice("root-overlay"))
/home      ext2   OverlayDevice(empty 1GB ext2, IDBDevice("home-overlay"))
/mnt/data  ext2   HttpBytesDevice("webvm:///datasets/...")   [read-only]
/mnt/host  9p     ws://127.0.0.1:8081/9p                     [security-scoped folder]
```

Split `/` and `/home` overlays are intentional — "Reset VM" wipes one without destroying the other. See `spec/05-storage.md`.

## Milestone-based development

The spec defines a strict sequential plan in `spec/07-milestones.md`. Respect the ordering — each milestone has explicit in-scope and out-of-scope items, and later milestones assume earlier ones are complete.

| | Scope summary |
|---|---|
| **M0** | Run upstream WebVM in a WKWebView; validate the platform |
| **M1** | Custom disk image + `WebVMSchemeHandler` with COOP/COEP and Range support; split overlays |
| **M2** | Raw-socket NetBridge + CheerpX networking patch (TCP only; UDP/LISTEN out of scope) |
| **M3** | NinePServer + CheerpX 9P mount patch + security-scoped folder picker |
| **M4** | Pause/resume hooks tied to iOS app lifecycle |
| **M5** | Bundled read-only datasets at `/mnt/data/...` |
| **M6** | **Investigation only** — instrument PoC, run workload suite, write `FINDINGS.md`. No implementation in this milestone. See `spec/08-investigation.md`. |

If a request would skip ahead or expand a milestone's scope, surface that explicitly before proceeding.

## Hard constraints (do not propose alternatives)

- **i386 guests only** — CheerpX does not support x86_64. Accepted, not a TODO.
- **iOS 17+** — required for reliable COOP/COEP via `WKURLSchemeHandler`. Custom response headers prior to 17 are not dependable.
- **WKWebView only** — no BrowserEngineKit (would be EU-only); no JIT entitlement.
- **No XPC / app extensions** in the PoC arc. Single-process app.
- **No background execution claimed** (`UIBackgroundModes` = `[]`). The VM suspends with the app; reconnection on resume is the design.
- **No external services** — no Tailscale, no remote control plane, no cloud backends. The whole point is local-only.

## Wire protocols (load-bearing, keep stable)

Both transports use little-endian binary framing on WS binary messages:

- **NetBridge frame** (`spec/03-net-bridge.md`): `op(1) | conn_id(4 LE) | length(4 LE) | payload`. Ops: `0x01 CONNECT`, `0x02 DATA`, `0x03 CLOSE`, `0x04 CONNECT_OK`, `0x05 CONNECT_ERR`, plus `0x06–0x0A` LISTEN/ACCEPT/RESOLVE (out of PoC scope).
- **9P2000.L** (`spec/04-ninep-server.md`): standard `size(4 LE) | type(1) | tag(2 LE) | body`. One WS message = one 9P message; no fragmentation either direction. Required PoC opcodes are tabled in that doc.

Errors from 9P always go via `Rlerror` with a Linux errno; mapping table is in `spec/04-ninep-server.md`.

## CheerpX fork policy

Patches live in a fork of `leaningtech/cheerpx`. Categories tracked in `spec/06-cheerpx-fork.md`:

- **P1** Networking transport replacement (required, M2)
- **P2** WS-based 9P mount type (required, M3)
- **P3** `window.webvm.pause()` / `resume()` hooks (required, M4)
- **P4** Swift-backed block device (optional; only if M6 investigation justifies Direction A)

Do **not** patch ext2, JIT, IDBDevice, OverlayDevice, HttpBytesDevice, or non-networking syscall emulation. Pin to a specific upstream commit; record it in `CHEERPX_VERSION.md` when the fork lands.

## Specs convention

The spec docs are deliberately terse and implementation-oriented. They assume familiarity with CheerpX/WebVM, 9P2000.L, iOS app lifecycle, and Network.framework. If editing them:

- No narrative process descriptions, no time estimates, no user stories.
- Prefer tables and code blocks over prose.
- Each milestone doc states explicit in-scope and out-of-scope items; preserve that structure.

## Reference sketch code (`spec/code/`)

These files exist to pin down API shape and framing details, not to be compiled:

- `VMHost.swift` — coordinator + minimal SwiftUI shell + WebVM `WKWebView` wrapper
- `LocalWSServer.swift` — `NWListener`-based localhost WS server, plus `wsReceive`/`wsSend` helpers
- `NetBridge.swift` — implements CONNECT/DATA/CLOSE only; LISTEN/ACCEPT/RESOLVE are TODO
- `NinePServer.swift` — 9P2000.L request dispatcher and FID table

When implementation begins, these are the starting point — but expect to replace `loadFileURL:allowingReadAccessTo:` in `VMHost.swift` with the `webvm://` scheme handler (the spec explicitly warns the file URL path does not reliably set COOP/COEP).

## Investigation milestone (M6) is special

`spec/08-investigation.md` defines four candidate directions (A: Swift-backed block overlay + File Provider, B: heavily cached 9P, C: bidirectional sync, D: accept the status quo) for resolving the "where should real project work live?" question. M6 produces a `FINDINGS.md` and a recommendation — it does **not** implement any direction. Subsequent milestones (M7+) are defined only after M6 chooses.

---
> Source: [christopherseaman/wasmvm](https://github.com/christopherseaman/wasmvm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
