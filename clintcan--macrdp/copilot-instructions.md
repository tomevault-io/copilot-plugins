## macrdp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Status

Functional v0. RDP clients (mstsc, Microsoft Remote Desktop, FreeRDP) can:
- Connect over TLS to the Mac on port 3390 with a local Mac username/password.
- See the primary display at native resolution with incremental damage-region updates.
- Optionally stream the display as **H.264 over EGFX** (`--enable-h264`, AVC420, Annex-B framing, VideoToolbox-encoded) — far less bandwidth than legacy bitmaps. Verified rendering on mstsc, on FreeRDP built with H.264 decode, and on the macOS Windows App / Microsoft Remote Desktop client (it decodes AVC420 over EGFX — only its *legacy* bitmap-codec list is NSCodec-only). Clients that genuinely don't advertise AVC420 decode (e.g. a decoder-less FreeRDP build) fall back to legacy BitmapUpdate automatically. **Caveat:** reconnecting *mstsc* to a still-running macrdp can show a blank screen (mstsc-specific EGFX surface-handling quirk — confirmed not a server bug, since FreeRDP reconnects cleanly); workaround is to restart macrdp or fully reopen the mstsc window. See the H.264 quirk note below.
- Drive keyboard and mouse, including modifier keys (per-side L/R tracking with NX_DEVICE bits, Caps Lock as a toggle, MS-RDPBCGR Synchronize lock-state reconciliation), mouse buttons, and wheel.
- Forward macOS symbolic hotkeys that WindowServer's dispatcher refuses to fire for user-space CGEventPost: Cmd+Tab / Cmd+Shift+Tab cycle apps via Accessibility API (per-bundle dedup with MRU, dead-pid filtering via `kill(pid, 0)`), Cmd+\` / Cmd+Shift+\` cycle windows of the current app (AXRaise + window AXMain + app AXMainWindow for Electron compatibility), Cmd+Space invokes Spotlight via AppleScript, Cmd+Shift+3/4/5 shell out to `/usr/sbin/screencapture` or open Screenshot.app.
- See the real macOS cursor shape (I-beam, hand, etc.) overlaid by the client.
- Copy/paste UTF-8 text and images (CF_DIB ↔ PNG) between Mac and remote.
- Mac→Windows file copy, including whole folders: copying a file or directory in Finder and pasting on Windows produces a real file/tree in Explorer. The pasteboard walk recurses into directories (skipping symlinks, capped at 10 000 descriptors per copy) and emits one FILEGROUPDESCRIPTORW entry per leaf with `relative_path` set so upstream's wire encoder reconstructs the right `MyFolder\sub\file.txt` cFileName. Bytes stream via MS-RDPECLIP `FileContentsRequest` SIZE + RANGE chunks (4 MiB per chunk). Reaches upstream `Cliprdr::initiate_file_copy` via the vendored `ServerEvent::ClipboardFileCopy(Vec<FileDescriptor>)` variant — that's the only API that populates `local_file_list`, without which upstream short-circuits every byte fetch with CB_RESPONSE_FAIL. Finder hands out *file-reference* URLs (`/.file/id=...`); we resolve them through `NSURL::URLByResolvingSymlinksInPath` because `std::fs::metadata` can't stat them directly.
- Windows→Mac file copy (one or more files; recursive folder copy via Ctrl-C does *not* work — see caveat below): when Windows announces a `FileGroupDescriptorW` we **eagerly** download every entry to `/tmp/macrdp-paste-<pid>-<nanos>/` via parallel `FileContentsRequest` chunks (1 MiB × 8 in flight), recreating any directory structure encoded in each descriptor's `relative_path`, then publish the top-level entries to NSPasteboard as real `NSURL`s. The eager approach is forced because Cocoa's `NSFilePromiseProvider` / `NSFilePromiseReceiver` is drag-and-drop-only — Finder's Cmd-V never calls into a promise delegate. `resolve_dest` path-sanitizes every `relative_path` component (rejects `.`, `..`, embedded `/`) so a malicious remote can't escape the temp sandbox. When the download lands we play `/System/Library/Sounds/Glass.aiff` (`afplay` bypasses notification permissions; `osascript display notification` was silently suppressed because macOS attributes the banner to the unsigned macrdp binary) and, *only if Finder is the frontmost app*, fire `Cmd-V` via System Events so the paste the user attempted finishes automatically. A `SelfChangeCount` atomic stops our own pasteboard write from being rebroadcast to Windows by the change-count poller.

  **Ctrl-C on a folder in Windows Explorer is a known no-op** — not our bug, and not fixable from the server side. Explorer puts `CFSTR_SHELLIDLIST` (Shell IDList Array) on the clipboard as the primary format and delay-renders `FileGroupDescriptorW` only when a shell-aware receiver asks. mstsc doesn't request the delayed format, so it never forwards anything via CLIPRDR — `cliprdr=debug` shows zero PDUs for the folder copy attempt. Workaround for the user: enter the folder in Explorer, `Ctrl-A` then `Ctrl-C` to copy the contents (with directory descriptors for any subfolders) — that path uses `FileGroupDescriptorW` directly and forwards correctly. True drag-from-Windows folder copy would need drive redirection (a different RDP feature, not clipboard).
- Forward macOS system audio to the remote (RDPSND, 44.1 kHz stereo 16-bit PCM; SCK captures at 48 kHz and the capture loop resamples via `rubato`).
- NLA / CredSSP authentication — no more "type username before Connect" mstsc workaround.
- Optionally attach a **headless virtual display** (`--virtual-display --width W --height H`) and serve that to the client instead of mirroring the primary panel — behaves like plugging in an external monitor, so the local Mac screen stays available while the remote session has its own desktop at any requested resolution. Backed by undocumented `CGVirtualDisplay*` private API; see the maintenance note below.
- Optionally go **fully headless while a client is connected** via one of two mechanisms (mutually exclusive):
  - `--virtual-display ... --detach-primary`: disables every active physical display at the WindowServer level once the first RDP client actually connects (private `CGSConfigureDisplayEnabled`). Backlight off, no menu bar, cursor can't cross over. Cleaning a stale detach if macrdp dies hard happens automatically — the detach uses `CGConfigureForAppOnly` so SIGKILL / panic / power loss trigger an OS-level revert with no logout required. **Caveat:** on some macOS versions / displays the disable transaction succeeds but the panel keeps showing the desktop; if that's the case, use `--capture-primary` instead.
  - `--virtual-display ... --capture-primary`: takes exclusive `CGDisplayCapture` of every physical display once a client connects AND forces each panel's gamma LUT to map every input to black via `CGSetDisplayTransferByFormula(_, 0,0,1, 0,0,1, 0,0,1)`. Capture alone doesn't visually blank modern macOS panels (the "fill with black on capture" semantic disappeared around 10.10) — the gamma trick is what actually makes the panel render solid black while the WindowServer keeps compositing the desktop to it. Backlight stays on, cursor sunk by the capture. Both gamma changes and capture tokens are process-scoped, so SIGKILL / panic auto-restores. Uses only public CG symbols — no private SkyLight surface, no `CGError 1001` window.
  Either way, the original layout is restored the moment the last client disconnects; local Mac usage is normal whenever no one is connected.

Not yet implemented: multi-monitor (client-side multi-display), non-US keyboard layouts, drive/printer redirection.

## Project goal

A native RDP server for macOS written in Rust on top of [`ironrdp`](https://github.com/Devolutions/IronRDP). Functionally analogous to `xrdp` on Linux: Windows / cross-platform RDP clients connect to the Mac and see its desktop, with keyboard/mouse forwarded back.

Not a client, not a VNC bridge, not a proxy — the server terminates the RDP protocol itself and renders/feeds the local macOS session.

## Architecture

```
src/main.rs       CLI, TCC preflight, TLS cert mgmt, RdpServer assembly
src/auth.rs       Startup PAM auth against the macOS account (libpam FFI)
src/capture.rs    ScreenCaptureKit → BgrA32 BitmapUpdate, dirty-rect driven
src/cursor.rs     NSCursor → RGBAPointer, hashed for change detection
src/input.rs      RDP scancodes/mouse PDUs → CGEvent synthesis (US ANSI),
                  per-side modifier state with NX_DEVICE bits, Caps Lock
                  toggle, AX-driven symbolic-hotkey workarounds
                  (Cmd+Tab app cycle, Cmd+` window cycle, Spotlight,
                  screencapture) since WindowServer's symbolic-hotkey
                  dispatcher won't fire for CGEventPost
src/clipboard.rs  CLIPRDR ↔ NSPasteboard (CF_UNICODETEXT + CF_DIB
                  + Mac↔Windows file copy via FileGroupDescriptorW
                  and FileContentsRequest streaming)
src/file_promise.rs  Windows→Mac eager download to /tmp + NSPasteboard
                     publish + Glass-chime auto-paste into Finder
src/audio.rs      RDPSND ← second SCK stream with system-audio capture,
                  rubato 48→44.1 kHz resample, latency-bounded
src/virtual_display/    Opt-in headless display via undocumented
  mod.rs                CGVirtualDisplay* private API. Public Rust
  private_api.rs        surface is `VirtualDisplay::new(w,h,hz)` +
                        display_id/origin_pts/size_pts; ALL touches to
                        private Obj-C classes/symbols are confined to
                        private_api.rs (the maintenance boundary —
                        when Apple changes the API in a future macOS,
                        update only that file).
src/h264.rs       EGFX/H.264 video pipeline (opt-in via --enable-h264).
                  Bridges the VideoToolbox encoder (src/videotoolbox.rs) to
                  upstream's GraphicsPipelineServer: per SCK frame, encode →
                  non-blocking drain → AVC420 (Annex-B framing) → DRDYNVC →
                  ServerEvent::Egfx. Each connection uses a fresh, monotonic,
                  temp-file-persisted surface id (no DeleteSurface — see the
                  H.264 quirk note below). Falls back to legacy BitmapUpdate
                  for clients that don't advertise AVC420 decode.
src/videotoolbox.rs  VideoToolbox H.264 encoder (AVCC NALs + SPS/PPS).
                  Feeds VT a full-range BT.709 NV12 (420f) buffer it builds
                  from the captured BGRA — VT would otherwise emit video-range
                  YUV, which mstsc renders washed-out. The BGRA→NV12 conversion
                  is vImage (Accelerate/NEON) accelerated, ~24-32x over the
                  scalar reference kept as a fallback + benchmark baseline.
build.rs          Bakes Xcode Swift-runtime rpath into the final binary

vendor/ironrdp-server/    Local fork of ironrdp-server 0.10.0, pulled in
                          via [patch.crates-io] in Cargo.toml. Single
                          targeted fix in dispatch_server_events: keep the
                          NEWEST queued waves on per-batch overflow instead
                          of the oldest (upstream 0.10.0 keeps oldest, which
                          bakes any dispatch stall into a permanent audio
                          offset). Submitted upstream — delete this vendor
                          dir once it lands in a released version.

vendor/ironrdp-egfx/      Local fork of ironrdp-egfx (same upstream rev as the
                          git-pinned siblings), pulled in via [patch.crates-io].
                          TWO divergences: (1) adds
                          GraphicsPipelineServer::create_surface_with_id so
                          src/h264.rs can drive surface ids from an external
                          monotonic counter (fresh id per session); (2)
                          CapabilitySet::decode tolerates unknown capability
                          versions (keeps them as Unknown) instead of erroring
                          fatally — without this, Windows App for Mac can't
                          connect with --enable-h264 (see quirk note). Drop once
                          both are upstreamed.
```

Cross-cutting:
- **TLS** terminates inside the acceptor; `rustls` with a self-signed cert at `~/Library/Application Support/macrdp/{cert,key}.pem` (generated on first run, persisted thereafter for stable client TOFU). `RdpServerSecurity::Hybrid` is used so the negotiation response advertises CredSSP — the public-key bytes handed to ironrdp are the raw `subjectPublicKey` BIT STRING from the X.509 cert (not the SPKI sequence, not the keypair-derived bytes), since that's what sspi hashes client-side.
- **Auth** at startup: `--username` (defaults to `$USER`) + interactive password prompt → PAM `checkpw` service → set as the static credential ironrdp_server checks per-connection. `--skip-auth` bypasses for dev.
- **Session model** — by default macrdp attaches to the console session of the logged-in user (single session, mirrors the primary panel). With `--virtual-display --width W --height H`, the server instead allocates a headless `CGVirtualDisplay` and serves *that*; the local Mac screen is untouched and the remote sees its own desktop at the requested resolution. The CG-side display is owned by `main()`'s scope, registered via `[CGVirtualDisplay initWithDescriptor:]` + `applySettings:`, and torn down on normal exit (signal-driven `std::process::exit(0)` skips Drop, but macOS reaps the registration when the owning process dies). Capture / input / cursor all parameterize on `(displayID, origin_pts, size_pts)` so they target the right surface regardless of which path is in effect.
- **Signal handling** — `main.rs` spawns a task that awaits SIGINT/SIGTERM and `std::process::exit(0)`s. Without it, ScreenCaptureKit's framework threads can leave the process unkillable by Ctrl-C once an SCStream is active.
- **Audio rate** — SCK only supports 8/16/24/48 kHz, so capture is at 48 kHz, but `src/audio.rs` resamples to 44.1 kHz via `rubato` before sending. 44.1 matches the native rate of most Windows audio endpoints, so the client plays directly without internal resampling — which used to cause a ~20% sustained over-feed and multi-second audio backlogs on mstsc. The advertised RDPSND `AudioFormat` is therefore 44.1 kHz / 2 ch / 16-bit.
- **Single capture loop** — `MacRdpsnd` (the audio factory) holds an `Arc<AtomicU64>` generation counter shared with every backend it builds. Each `start()` claims a fresh generation; older capture loops observe the bump on their next iteration and exit. Without this, an mstsc cert-prompt reconnect leaves the first capture loop running while the second starts, both feeding the shared event channel → ~2× audio reaching the client.

When adding a feature, locate it in one of those modules first; if it spans them (e.g., a new virtual channel), it belongs in a dedicated module alongside `clipboard.rs`, driven by `ironrdp_server`'s factory traits.

## macOS-specific gotchas

- **Screen Recording permission** (TCC) is required for ScreenCaptureKit. Granted in System Settings → Privacy & Security → Screen Recording.
- **Accessibility permission** is required to post synthetic keyboard/mouse events via `CGEventPost`. Granted in System Settings → Privacy & Security → Accessibility. Without it, posted events are silently dropped.
- **TCC grants are path-keyed AND signature-keyed.** `target/debug/macrdp` and `target/release/macrdp` are tracked separately. An *unsigned* rebuild at the same path also invalidates the grant — every fresh link gets a different identity. Ad-hoc sign the release binary (`codesign -s - --force target/release/macrdp`) to get a stable code-signature identity so the grant survives rebuilds. Cargo doesn't have a post-link hook, so do it manually or via a wrapper script.
- **Posting events to the login window or secure-input contexts** (password fields, lock screen) is blocked by the OS and cannot be worked around — document the limitation rather than fighting it.
- **Default RDP port 3389 is privileged**; bind 3389 only with elevated rights, otherwise default to 3390 in dev.
- **OpenPAM, not Linux-PAM.** `checkpw` uses `use_first_pass`, so `pam_opendirectory` reads the password from `pam_set_item(PAM_AUTHTOK, ...)` and never invokes the conv callback. See `src/auth.rs`.
- **`CGVirtualDisplay*` is a moving target.** macOS 26 (and presumably forward) replaced the older C-function surface (`CGVirtualDisplayCreate` / `ApplySettings` / `GetDisplayID`, which BetterDisplay's wrappers document) with an Obj-C class (`-[CGVirtualDisplay initWithDescriptor:]` / `applySettings:` / `displayID`). `private_api.rs` is written against the new shape only; if a future release renames a class or selector again, error messages from `VirtualDisplay::new` will name what's gone. BetterDisplay's open-source Swift wrappers are the canonical reference when adapting. Symbols are resolved via `AnyClass::get(...)` so missing-class failures are clean runtime errors, never link-time breakage.
- **Virtual-display refresh rate must be ≥ 24 Hz.** `applySettings` rejects modes below the real-monitor floor; the `VirtualDisplay::new` callers pass a hard-coded 60 Hz regardless of `--fps` (capture cadence is independent — `--fps` controls the SCK stream, refresh rate is metadata as far as the RDP server is concerned).
- **macOS 14+ killed every "activate this app" path except Accessibility for unsigned headless callers.** `NSRunningApplication.activateWithOptions` silently no-ops (front-app-only enforcement, `NSApplicationActivateIgnoringOtherApps` deprecated). `osascript tell application "X" to activate` requires TCC Automation, which an unsigned binary running without a UI session can't get prompted for. `/usr/bin/open -b <bundle>` doesn't activate AND re-launches just-quit apps. The one path that works: `AXUIElementSetAttributeValue(appRef, kAXFrontmostAttribute, true)` — macrdp already holds the Accessibility TCC grant (CGEventPost needs it), so no extra permission. Pair it with `AXRaise` on `kAXFocusedWindow` so the correct window of a multi-window process comes forward. For Electron apps (VSCode, Slack, Discord, Cursor) AXRaise on a window is often a no-op — layer `AXMain=true` on the target window + app's `AXMainWindow=<window>` to actually move the stacking. `NSWorkspace.frontmostApplication` is NOT trustworthy as live state after we've AX-activated something — it keeps returning the pre-activation frontmost; for "what app is the user in" use the internal `LAST_CYCLE_PID` cursor in `input.rs`.

### Known behavioural quirks

- **Server-side scaling forces full-frame updates + fills the frame.** Passing `--width N --height M` when N×M ≠ the Mac's native size makes SCK scale internally, and SCK's default behaviour leaks both ways: dirty-rects come in *source* (native) coords (clients that strictly respect tile coordinates render a black canvas with only the cursor sprite — mstsc/RemoteFx is the worst offender) and aspect-mismatch causes pillarbox/letterbox padding (the captured Mac content sits in a sub-rectangle of the output, so client-predicted cursor sprite drifts proportionally from the click position as you move away from screen center). `capture.rs` works around both: detects size mismatch up front, skips dirty-rect emission (full BitmapUpdate every tick — the upstream encoder's framebuffer diff still keeps unchanged-region cost low when SCK returns exactly the configured size), and sets `scalesToFit=true` on the SCK config so source content stretches to fill the requested frame instead of getting padded. The cost at non-native sizes: higher bandwidth, plus non-uniform scaling on aspect mismatch (e.g., 16:10 MacBook panel into a 16:9 frame shows ~13.5% vertical compression). Prefer native if you can.
- **No server PointerPosition forwarding (intentional).** mstsc and Microsoft Remote Desktop local-predict the cursor from the user's mouse input; any `PointerPositionAttribute` PDU we send arrives one encode-plus-network round-trip late and snaps the cursor back to a stale position on fast moves. The current design (`poll_shape` only, `poll_position` defined but unwired) is deliberately the right one for all interactive use. *Do not* simply wire `poll_position` up — that's the bug, not the fix. The only thing this design misses is Mac-side *programmatic* cursor moves (an app calling `CGWarpMouseCursorPosition`, rare); fixing that properly requires emitting position only when the Mac cursor diverges from where the last RDP mouse event left it and enough time has passed since that event, which needs shared state between `input.rs` and `cursor.rs`.
- **Legacy codec mismatch with the macOS client (only without `--enable-h264`).** In the legacy bitmap path, Microsoft Remote Desktop / Windows App on macOS offers only NSCodec while the server advertises RemoteFx/QOI, so they fall back to raw/RLE BitmapUpdate — works, but bandwidth-heavy. With `--enable-h264` that same client negotiates AVC420 over EGFX and renders H.264 (verified), which is far better — prefer it for the macOS client.
- **EGFX capability parsing must tolerate unknown versions (Windows App fix).** Windows App for Mac (`com.microsoft.rdc.macos`) advertises an EGFX capset version the pinned ironrdp_egfx didn't recognize; upstream's `CapabilityVersion::try_from` returned `Err` on it, which the CapabilitiesAdvertise decode propagated as a *fatal* error — killing the whole connection during negotiation, before the AVC-vs-legacy fallback could run. So `--enable-h264` made Windows App unable to connect at all (it connects fine without). Fix: `vendor/ironrdp-egfx` `CapabilitySet::decode` now keeps an unrecognized version as `CapabilitySet::Unknown(raw)` instead of erroring. This is the SECOND vendor-egfx divergence (alongside `create_surface_with_id`) and a clean upstream candidate.
- **H.264 wire format must be Annex-B, and mstsc retains EGFX surfaces.** Two hard-won facts behind `src/h264.rs` (`--enable-h264`): (1) the AVC420 payload must be **Annex-B** (start codes + in-band SPS/PPS) — Microsoft's decoder gives zero frame-acks for length-prefixed AVCC, so the default is Annex-B (`MACRDP_H264_LENGTH_PREFIXED=1` flips it for ironrdp-decoder interop). (2) **mstsc retains EGFX surfaces by id for its whole process lifetime** (across reconnects and even macrdp restarts) and no-ops a `CreateSurface` for an id it already holds → frames decode into a stale surface → black screen + live cursor on reconnect. We work around it by using a fresh, monotonic, temp-file-persisted surface id per session (so the id is never one mstsc cached). **Do NOT try to fix this with `DeleteSurface`** — deleting a surface mstsc doesn't currently hold (or one mapped to the output) breaks its GFX channel entirely (no acks, then disconnect). This was confirmed twice; the fresh-unique-id approach is the only safe lever. FreeRDP (with H.264) has none of these quirks — it renders and reconnects cleanly, which is how we proved the server output is spec-correct. The residual mstsc reconnect blank is therefore a documented client limitation, not a server bug.
- **Cursor sprite shape is process-global on macOS, not per-display.** `NSCursor::currentSystemCursor()` reflects whatever the foreground application set in *this process*'s perspective, not "the cursor on display X." When `--virtual-display` is active, the RDP client sees the cursor shape that whatever app is foreground locally has set (typically the arrow), regardless of what an app on the virtual display might want. Probably fine in practice — the cursor *position* on the virtual display is correct because `input.rs` translates RDP coords by the vdisplay's origin in global space; only the visual shape is shared. Fixing this properly needs a per-display NSCursor query, which AppKit doesn't expose.
- **mstsc gates Win-combos by full-screen by default.** mstsc → Local Resources → Keyboard → "Apply Windows key combinations" defaults to "Only when using the full screen", so a windowed mstsc session eats `Win+Tab` (the Cmd+Tab we want to forward) locally as Task View — macrdp never sees the press. The #1 false-positive when debugging "Cmd+Tab doesn't work" is the user being windowed. Set it to "On the remote computer" or go full-screen. `xfreerdp` is more permissive by default and a useful cross-check.
- **Symbolic-hotkey dispatcher needs kernel HID; CGEventPost can't wake it.** WindowServer's internal Cmd+Tab / Cmd+Space / Cmd+Shift+3/4/5 / Mission Control dispatcher only fires on kernel-injected HID events. User-space CGEventPost cannot trigger it regardless of source state or tap location. A `src/virtual_hid.rs` module that registered a virtual USB-style HID keyboard via `IOHIDUserDeviceCreate` was tried for this and reverted — didn't unblock the dispatcher (likely needs entitlements / signing we don't have) and added a parallel scancode-mapping table to maintain. The current design instead reimplements each symbolic combo in user space (`cycle_apps` via AX, `invoke_spotlight` via osascript, `screencapture` via the binary). Do not resurrect IOHIDUserDevice without new evidence.

## Commands

```bash
cargo build                    # debug build
cargo build --release          # release build (LTO, ~30s)
cargo run                      # prompts for password, runs against PAM
cargo run -- --skip-auth --password test  # bypass PAM for quick tests
cargo run -- --virtual-display --width 1920 --height 1080  # headless remote desktop, local screen untouched
cargo test                     # run all tests
cargo clippy --all-targets -- -D warnings  # lint as errors
cargo fmt                      # format
RUST_LOG=debug cargo run       # crank logging for troubleshooting
```

Useful CLI flags (see `src/main.rs::Args` for the full set):
```
--bind 0.0.0.0:3390       # listen address
--username NAME           # default: $USER
--password PASS           # avoid the interactive prompt (logs are warned)
--skip-auth               # bypass PAM (also skips password validation)
--width  / --height       # override autodetected display size
--fps N                   # default 15
--enable-h264             # stream H.264 over EGFX (AVC420) instead of legacy bitmaps
--cert-dir PATH           # default ~/Library/Application Support/macrdp
```

Testing against the server:
```bash
# FreeRDP — easiest to script and get verbose logs from.
xfreerdp /v:127.0.0.1:3390 /u:$USER /cert:ignore /log-level:DEBUG

# Microsoft Remote Desktop / Windows App.app — closest to real-user UX.
# Windows mstsc: just enter the computer and click Connect — NLA/CredSSP
# is enabled, mstsc will prompt for credentials in its own dialog.
# Expect one "Broken pipe" error in the log on the first attempt: that's
# mstsc's cert-trust prompt closing and reopening the socket. The next
# attempt succeeds.
```

When iterating on the capture/encode path, prefer FreeRDP with `/log-level:DEBUG` — its PDU traces are far more useful than mstsc's silent failures.

## Conventions worth keeping

- Keep `ironrdp` as the only crate that touches RDP wire format. Wrappers around it are fine; parallel parsing/emitting of PDUs is not.
- Per-platform code (capture, input, cursor, clipboard) is feature-gated via `#[cfg(target_os = "macos")]` so the protocol layer remains cross-compilable on Linux CI. Each module has a non-macOS stub for that reason.
- Errors that originate from macOS APIs (`OSStatus`, `CGError`, TCC denials, PAM error codes) should be wrapped with enough context that the user knows *which permission or service* is missing — those are the #1 support question.
- Direct FFI via `extern "C"` is preferred over heavyweight wrapper crates when the call surface is small (see `src/auth.rs::pam_impl`).
- Default log level is `info`; reach for `RUST_LOG=debug` when investigating, don't make debug the default.

---
> Source: [clintcan/macrdp](https://github.com/clintcan/macrdp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
