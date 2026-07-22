## pcairplay

> **The user-facing name is AirPlayPC everywhere** (rebranded 2026-07-20):

# AirPlayPC (repo `pcairplay`) — context for Claude Code

## What this is

**The user-facing name is AirPlayPC everywhere** (rebranded 2026-07-20):
window title, tray, MessageBoxes, shortcuts, installer, README, launcher file
names (`AirPlayPC.vbs` / `AirPlayPC.cmd`), release artifacts. Deliberately NOT
renamed: the repo/URL (`pcairplay`), `%LOCALAPPDATA%\pcairplay` (settings and
log continuity), the firewall rule names (`PCAirPlay - …`; renaming would
strand rules on existing installs), the `gbulog.pcairplay` AppUserModelID
(pinned-taskbar identity), shared function names (`*-PCAirPlay*`), mutex
names, and `pcairplay.ico` / `pcairplay.iss` file names.

A thin Windows wrapper around **UxPlay**, an open-source AirPlay mirroring
receiver. The goal: mirror an iPhone screen to a Windows PC using **native iOS
Screen Mirroring**, with **no app installed on the iPhone**.

This repo contains no protocol code. UxPlay does the hard part (mDNS
advertisement, RTSP, the FairPlay handshake, pair-verify, H.264 over RTP). What
we add is the Windows glue that is otherwise fiddly and undocumented: install,
firewall, mDNS prerequisites, sane low-latency defaults, and diagnostics.

Machine-specific status (test-machine hostnames, LAN details, in-progress
debugging state) lives in **`CLAUDE.local.md`**, untracked and per-machine —
if it exists next to this file, read it too. Keep anything personal or
machine-identifying THERE, never here: this file is public.

## Target environment

- **Home PC**: Windows, wired Ethernet.
- **iPhone**: on Wi-Fi, same router / same subnet as the PC.
- This wired-PC / Wi-Fi-phone split is the intended setup and works, provided
  both are in one broadcast domain. See *Known pitfalls*.

## Files

| File | Purpose |
|---|---|
| `uxplay-common.ps1` | Dot-sourced by every script below. Owns engine discovery, environment setup, LAN IP, rival-receiver detection, the firewall rule names, **and the engine argv** (`Build-UxPlayArgs`). **Put shared logic here.** |
| `setup.ps1` | One-time, **needs Administrator**. Installs UxPlay **1.72.1-3** (see *The 1.x / 2.x split*), checks Bonjour, opens firewall ports. `-WhatIf` previews without elevation. |
| `start-airplay.ps1` | Starts the receiver. Normal user, no elevation. `-DryRun` prints the argv without launching. |
| `airplay-ui.ps1` | WPF desktop UI over the same engine. Owns the framed view's lifecycle (opens it with the UI, closes it on exit), persists settings, displays the PIN, single-instance. `-SelfTest` builds the window, asserts layout and the generated argv, and exits — run it after any edit here. |
| `AirPlayPC.vbs` | **Preferred double-click launcher for the UI** — zero window flashes (WScript starts the powershell console hidden from birth; a `.cmd` cannot avoid flashing its own). |
| `AirPlayPC.cmd` | Fallback launcher for the UI, for machines with Windows Script Host disabled. Flashes a console briefly. |
| `frame-mirror.ps1` | iPhone-chassis "simulator look" around the live mirror window: bezel, rounded screen corners, side buttons, shadow. Attaches to the engine's video window whenever one exists; single-instance; remembers position/zoom; has its own `-SelfTest`. Normally launched and closed **by the UI** (the "iPhone frame" switch). |
| `Framed Mirror.cmd` | Double-click launcher for the framed view standalone — only needed next to a `start-airplay.ps1` console session; the UI manages the frame itself. |
| `Diagnostics.cmd` | Double-click launcher for `doctor.ps1`, so diagnostics can't be blocked by ExecutionPolicy. |
| `doctor.ps1` | Read-only diagnostics. Run this first whenever mirroring fails. Exit code 0 = clean, 1 = problems. |
| `installer/pcairplay.iss` | Inno Setup **net-installer**: ships ONLY this repo's scripts + Start Menu shortcuts, offers to run `setup.ps1` post-install (the engine download happens there), uninstall runs `setup.ps1 -Uninstall`. Built by CI; `installer/Output/` is gitignored. See *Distribution*. |
| `.github/workflows/` | `ci.yml` — the real `setup.ps1` flow (engine download + firewall + teardown), both `-SelfTest`s, `-DryRun`, `-WhatIf` and an installer smoke-build on every push. `release.yml` — tag `v*` → build installer + scripts zip + `SHA256SUMS.txt`, publish the GitHub release. |
| `pcairplay.ico` | The app icon — window/taskbar/tray, both shortcut sets, the setup exe, Apps & Features. **Regenerate with `tools/make-icon.ps1`, never hand-edit.** |
| `tools/make-icon.ps1` | Vector-draws the icon with WPF and packs a proper multi-size .ico (256 px as a PNG entry, the rest as 32-bit BGRA BMP entries with AND masks). |
| `tools/make-installer-art.ps1` | Vector-draws the installer's wizard bitmaps (`installer/wizard-*.bmp`: dark background, white AirPlay glyph, wordmark) that the .iss references. **Regenerate, never hand-edit the BMPs.** |

`uxplay-common.ps1` is dot-sourced by all five scripts (including
`frame-mirror.ps1`, which runs it under StrictMode 3 — keep it strict-clean)
and owns the answers that used to be duplicated (and drift): `Find-UxPlay`,
`Initialize-UxPlayEnvironment`, `Get-LanIPAddress`,
`Get-CompetingReceiverProcess`, `Build-UxPlayArgs`, `Resolve-UxPlayDeviceName`,
and the UI↔frame coupling (`Test-FramedMirrorRunning`,
`Close-FramedMirrorWindow`, `Show-PCAirPlayUiWindow`). **Put shared logic
there, not in a fourth copy.**

**`Build-UxPlayArgs` is the only place the engine's argv is constructed**, and
`Resolve-UxPlayDeviceName` the only place the advertised name is normalised.
`start-airplay.ps1` and `airplay-ui.ps1` each used to build both privately, and
they drifted in ways neither side could see — see *The argv fork* below. Do not
re-fork them.

`Build-UxPlayArgs` returns `.Args` (the token array) **and `.Notes`** — the sink
fallback has something to say, and the function must not assume a console exists
to say it in. The CLI prints them all; the UI shows each as a MessageBox before
launch, so a share-safe request that falls back to D3D11 is reported in both.

`Find-UxPlay` **searches** for `uxplay.exe` — under the known install roots *and*
whatever `InstallLocation` the uninstall registry reports — then derives the
GStreamer plugin dir from wherever the exe turned up. Never hardcode
`_internal\bin` again; that is what makes this survive the v2.x layout change.

**The iPhone used for all testing runs iOS 26.5.2** — inside the confirmed-working
range (see *iOS compatibility*). If mirroring fails, it is not an iOS-version
problem.

## First-time setup on a new PC

```powershell
# 1. Elevated PowerShell, in the repo directory:
powershell -ExecutionPolicy Bypass -File .\setup.ps1

# 2. Normal PowerShell — verify:
.\doctor.ps1

# 3. Start it:
.\start-airplay.ps1
```

Then on the iPhone: Control Center → Screen Mirroring → pick the PC.

## Key facts established while building this

These were verified by inspecting an actual install — don't re-derive them.

- Upstream is **[leapbtw/uxplay-windows](https://github.com/leapbtw/uxplay-windows)**,
  a prebuilt packaging of UxPlay — so **no MSYS2 or GStreamer toolchain is
  needed**. The 1.x line ships an Inno Setup `.exe`; 2.x ships `.msi`/`.zip`.

### The 1.x / 2.x split — read this before touching `setup.ps1`

**UxPlay 2.x cannot be used by this repo, and installing it breaks everything.**

`uxplay-windows` 2.x is a ground-up Qt6 rewrite that links uxplay in as a
*library*. Verified on 2026-07-20 by reading **all 170 entries** of the published
`2.0.0.1736` zip central directory (HTTP range request, no full download): the
only executables in it are `uxplay-windows.exe`, `mDNSResponder.exe`, and
`uxplay-bluetooth-beacon.exe`. **There is no `uxplay.exe` and no command line.**

Every script here drives the engine by command line. So a 2.x install produces
an engine that `Find-UxPlay` cannot use, and the old code reported
"UxPlay is not installed" while an install plainly existed.

`setup.ps1` **used to default to `/releases/latest`, which resolves to 2.x** —
i.e. the documented default install path installed the one build the toolkit
structurally cannot drive. It now queries `/releases`, filters to stable `1.*`,
and picks the newest by publish date (**1.72.1-3**,
`uxplay-windows-installer-v1.72.1-3.exe`). A `2.x` tag passed via `-Tag` is
rejected outright. `Find-UxPlay` reports `Kind='GuiOnly'` for a 2.x install so
every caller can say something true instead of "not installed".

The winget package `leapbtw.uxplay` is pinned to **1.72.1.3** — the *same
generation*, so `-UseWinget` is a legitimate fallback, not a downgrade. The old
framing of winget as "the older build with the wake-from-sleep bug" was
backwards: it is the correct one, and the GitHub "latest" path was the broken one.

What is genuinely lost by staying on 1.x: the bundled mDNSResponder, the
wake-from-sleep fix, and Bluetooth discovery. Those live only in 2.x, and 2.x has
no CLI. Wrapping 2.x's GUI would mean giving up every tuned flag
(`-vsync no`, `-fps 60`, `-vs d3d11videosink`, `-nohold`) — i.e. this repo's
entire value-add. **That trade-off is unresolved and worth a deliberate decision.**

### Apple Bonjour is REQUIRED, not optional

Also verified 2026-07-20, and it contradicts what this file used to say.
`uxplay.exe` 1.72.1-3 **imports `dnssd.dll` and calls `DNSServiceRegister`** —
that is Apple's Bonjour SDK. It does not self-advertise. Only the 2.x rewrite
compiles in its own mDNSResponder, and 2.x has no CLI.

So on the build these scripts use: **no Bonjour means no mDNS advertisement, and
the PC never appears on the iPhone.** Both machines tested so far passed only
because they happened to have Bonjour already (iTunes installs it; the work PC
had Bonjour 3.1.0.1 with `dnssd.dll` in System32 and SysWOW64, and `mDNSResponder`
holding UDP 5353). A clean PC without iTunes would have failed, while the old
`doctor.ps1` cheerfully reported *"Apple Bonjour absent — fine, UxPlay v2
supplies its own mDNS."*

`setup.ps1` now treats a missing Bonjour as a hard failure and points at
<https://support.apple.com/kb/DL999>.

- Install layout (observed on 1.72.x — **informational only; no script hardcodes
  it.** `Find-UxPlay` searches for the exe and derives the plugin dir from where
  it lands):
  - GUI wrapper: `C:\Program Files (x86)\uxplay-windows\uxplay-windows.exe`
  - **Real engine**: `...\uxplay-windows\_internal\bin\uxplay.exe`
  - GStreamer plugins: `...\uxplay-windows\_internal\lib\gstreamer-1.0`
- `uxplay.exe` is **not on PATH**. `start-airplay.ps1` locates it and sets
  `PATH` + `GST_PLUGIN_PATH`. Launching the engine directly *without* setting
  `GST_PLUGIN_PATH` starts the process but produces **no video**, because the
  sinks and decoders silently fail to load. This is the most likely cause of a
  "it runs but the window is black" report.
- Bundled plugins confirmed present: `d3d11` (video sink), `wasapi` (audio
  sink), `nvcodec` (NVIDIA hardware H.264 decode).

## Tuning decisions (and how to reverse them)

Defaults are tuned for **demos / presentations / app screen-sharing**, i.e.
responsiveness over perfect sync:

- `-vsync no` — disables A/V timestamp sync. Big latency win, but audio can
  drift on long video playback. **For watching video, run
  `.\start-airplay.ps1 -Sync`** to restore correct lip-sync.
- `-fps 60` — UxPlay's own default is 30. Lower it with `-Fps 30` if the network
  is congested and playback stutters.
- `-h265` — sets AirPlay features bit 42 (SupportsScreenMultiCodec). **Not
  optional in practice**: a phone that elects to send H.265 — always at 4K,
  and observed live at 1440p (iPhone16,1, iOS 26.5.2) — is otherwise rejected
  mid-stream with "connected but no video" while the phone shows the
  tickmark. Root-caused 2026-07-20; see the resolved stall under *Status*.
  Flag verified against `uxplay -h` on 1.72.1-3; `d3d11h265dec` confirmed
  present in the bundled d3d11 plugin.
- `-vs d3d11videosink` — hardware-accelerated Windows presentation, and the only
  bundled sink that handles `-fs` (fullscreen) correctly.
- `-nc` — keeps the window open after the phone disconnects.
- `-nohold` — a newly connecting phone takes over from the current one, which
  suits multi-presenter demos.

### Re-sharing the mirror into a Teams / Meet / Zoom call

A frequent use: mirror the iPhone to the PC, then screen-share *that* into a
call. Two things to know.

- **Share the whole screen, not the single window.** Desktop capture composites
  everything, including hardware-overlay video. Window capture is the fragile
  path.
- If the window must be shared and remote participants see a **black rectangle**
  where the iPhone should be, that is `d3d11videosink` presenting through a
  hardware overlay that the capture path cannot read. Run
  `.\start-airplay.ps1 -ShareSafe` to switch to `glimagesink`, which draws into
  the window normally. Verified to start cleanly on 1.72.1-3. It gives up
  hardware presentation, so use it only when black actually appears — **the
  local mirror looks perfect in this failure mode**, so it can only be caught by
  asking someone on the call.

**iOS notifications do appear in the mirror** — mirroring is a copy of the phone
screen, so banners land in it like anything else. What suppresses them is on the
phone, not here: a **Focus / Do Not Disturb** mode silences banners entirely, and
that is the usual reason someone reports "notifications don't show up when
mirroring". Check the Focus state on the phone first. Note this cuts both ways —
mirroring to a shared display means every incoming message is shown to the room.

**Apple-DRM video (Apple TV+, Netflix) will never mirror** to an open-source
receiver, regardless of sink.

`-Port <n>` pins the receiver to n, n+1, n+2 instead of letting the OS pick an
ephemeral port. The iPhone doesn't care either way: it reads the port out of the
mDNS record.

**It is *not* the fix for a competing receiver**, which is what this file and
`start-airplay.ps1` both used to claim. Started without `-p`, UxPlay never binds
7000 at all (see *Ports used*), so a rival app holding 7000 cannot collide with
it. The real damage a rival does is the **duplicate mDNS entry** — the phone
lists two devices and the wrong one gets tapped — and `-Port` does nothing about
that. Close the rival instead.

Use `-Port 7000` only when a restrictive third-party firewall ignores
program-scoped rules and you need the fixed 7000/7001/7100 rules to be the ones
doing the work.

## The framed mirror (added 2026-07-20)

`frame-mirror.ps1` wraps the engine's bare video window in an iPhone-style
chassis for demos and screen-sharing. Design constraints discovered while
building it — don't rediscover them:

- **A WPF window with `AllowsTransparency=True` is a layered window, and
  layered windows do not render child HWNDs.** Reparenting the video window
  into the WPF frame makes the video invisible. That is why the frame is a
  three-window sandwich (bezel below, untouched-but-borderless video in the
  middle, an input overlay with the perimeter ring mask on top) glued with
  `GWLP_HWNDPARENT` ownership, which makes Windows itself maintain the z-order.
- The video window only exists while a phone is mirroring, so the frame polls
  and attaches/detaches; closing the frame restores the video window's original
  style, owner and position.
- GStreamer's d3d11 sink window class starts with `GSTD3D11`; the finder
  prefers `GST*` classes and must never match `ConsoleWindowClass`.
- **The engine's video window routinely appears minimized at session start.**
  A minimized window's rect is a 160x28 stub at -32000, so any size filter
  rejects it and the frame "only works after you maximize the window". The
  finder admits iconic windows and the attach path restores them first.
- **The stream bakes the phone's display shape in** (rounded content, black
  corners in the rectangle). Four corner masks matched to a guessed radius
  left black crescents showing; the overlay now draws ONE perimeter ring mask
  with an inset rounded-rect hole, which swallows the baked black at any size.
- **The video rectangle overhangs the chassis' rounded outline at the
  corners.** The chassis curve cuts ~0.29·OuterR deep at 45°, the video sits
  only one bezel-width inside, so its (black) corners poked past the curve
  onto the desktop as hard black squares. Two-part fix, both required: the
  ring mask is chassis-shaped with a ROUNDED outer boundary (a square one is
  itself an overhang), and the video window is clipped with
  `SetWindowRgn`/`CreateRoundRectRgn` — its jagged GDI edge is invisible
  because it lands under the overlay ring. The region is removed on detach.
- The UI only *controls* the engine IT started: an engine launched from
  `start-airplay.ps1` mirrors happily while the UI shows "Ready". Since
  2026-07-20 evening the UI at least *names* this (a ~12 s poll of
  `Get-UxPlayEngineProcess` puts "A receiver started outside this app is
  running" in the sub-status), so the mismatch no longer reads as a broken UI.
  Controlling an adopted engine is still future work.
- **If the UI gets launched elevated** (right-click → run as administrator),
  its engine is elevated too, and no normal-privilege tooling can stop either
  (`Stop-Process`/`CloseMainWindow` → access denied). Happened in normal use
  on 2026-07-20 and cost a cleanup round. Nothing here needs elevation after
  `setup.ps1`; don't run the launchers as admin.
- **When sharing into a call, share the whole screen** — the sandwich is three
  windows, and window-capture of the d3d11 sink can be black anyway (see
  `-ShareSafe`).
- **The frame detects an elevated engine instead of spinning (added 2026-07-20
  afternoon).** UIPI makes `ShowWindow`/`SetWindowLongPtr` against an elevated
  engine's window fail *silently* from a normal-privilege frame, so the old
  code re-tried `SW_RESTORE` forever — "mockup shows, no image" while the
  video played minimized. Now five failed restores, or an adoption whose
  read-back does not match, blacklists that window and puts the cause on the
  chassis. The placeholder also names the session stall: engine running + an
  established TCP connection + no video window for 12 s → "stop Screen
  Mirroring and start it again".
- **The frame is its own app and must be RUNNING to get the mockup.** Bitten
  in normal use 2026-07-20: mirroring came up in a plain full-height window
  and read as "the mockup is gone" — the frame simply wasn't running.
  Launching it mid-session adopted the already-streaming video window within
  seconds (verified), so starting it late is fine; starting it never is the
  trap. **Closed the same evening: the UI now owns the frame's lifecycle.**
  The "iPhone frame" switch (default on, persisted) opens it with the UI and
  on every Start; unchecking closes it; closing the UI always closes it
  (verified live: frame mutex released within a second of the UI's WM_CLOSE).
  The switch is deliberately NOT locked while running, and a ~12 s poll snaps
  it back off if the frame is closed by hand. Coupling is via
  `Test-FramedMirrorRunning` (mutex probe) and `Close-FramedMirrorWindow`
  (WM_CLOSE to the bezel by exact title — never a kill; see the owned-window
  lesson below) in `uxplay-common.ps1`, which `frame-mirror.ps1` now
  dot-sources for the shared mutex name/title.
- **2026-07-20 evening polish — all three user-reported.** (1) *Fat black
  edge, corners that don't nest*: the aspect locked at attach was the
  WINDOW's (0.486 for an advertised 2560x1440), not the stream's (0.461), and
  the letterbox measurement called the resulting ~2.4%-per-side pillars
  "full" at its 0.97 threshold. The sample grid hits the exact scan edges, so
  a genuinely full axis measures ~1.0 — the threshold is 0.985 now, and thin
  pillars get corrected. (2) *Wheel resize dead once attached*: the overlay
  was `WS_EX_TRANSPARENT`, so over the video the wheel went to the ENGINE
  window — resize only ever worked before the first attach. The overlay now
  carries a 1/255-alpha input sheet (layered windows hit-test per pixel) and
  owns wheel resize, left-drag move, and the right-click scale menu; stealing
  the mouse costs nothing because UxPlay does not forward touches. (3) *Fake
  Dynamic Island removed* — the real one arrives inside the mirrored image,
  and two pills read as a glitch.
- **The frame is single-instance (named mutex) — required, not nice-to-have.**
  A double launch put two frames in a MoveWindow fight, the chassis
  flip-flopping between two aspects as each applied its own measurement.
  Related hard lesson: **never hard-kill an ATTACHED frame** — the video
  window is OWNED by the bezel, Windows destroys owned windows with their
  owner, and the live mirror session dies with it (observed: engine left
  ERRORing about missed client feedback). Close the frame window instead; its
  Closing handler detaches and restores first.
- **The video can NEVER be *born* inside the frame — only adopted fast.** The
  engine creates its own top-level window (GStreamer's d3d11 sink), the uxplay
  CLI has no way to hand it a pre-made HWND, and `d3d11videosink` exposes no
  `window-handle` property a `-vs` string could set (GstVideoOverlay is
  programmatic only, inside uxplay's C code). So a short bare-window phase is
  structural; what was shipped (2026-07-20 late) shrinks it below perception:
  the finder **hides the window in the same tick it is first seen** (it used
  to stay bare for 1–2 ticks ≈ "appears for a second"), and the poll drops to
  **150 ms while a window is imminent** — armed by the engine's established
  TCP connection, which precedes window birth by seconds — or mid-adoption
  (settle = 5 stable 150 ms samples ≈ 750 ms, wall-clock stricter than the
  old 2×700 ms). Measured with a real `d3d11videosink` window: flash ~1 ms,
  birth → styled-in-chassis ~1.0 s. Idle states stay at 700 ms. True zero
  would mean patching uxplay to pass a prepared HWND — not worth it.
- **Wheel-resize "only resizes the frame" (reported 2026-07-20) could not be
  reproduced at any layer** — verified with real input (SendInput) against
  the live frame adopting a real `GSTD3D11`-class `d3d11videosink` window
  (driven via P/Invoke `gst_parse_launch` under a process renamed uxplay.exe,
  since gst-launch doesn't ship): zoom → layout → MoveWindow → sink content
  rescale all track, and the read-back matches. The two states that DO
  produce exactly that symptom: wheeling **before adoption** (bare window +
  placeholder chassis — now nearly gone, see above) and a **missing overlay
  while attached** (wheel-over-video then hits the engine window, which
  ignores it, while the bezel ring still resizes). Both are now covered: the
  attached tick re-Shows a hidden overlay, `Sync-Position` verifies resizes
  by read-back (one SetWindowPos retry), and `frame-debug.log` records every
  zoom step, video resize (+ MoveWindow result/readback), candidate, attach
  and detach — a recurrence is diagnosable from the log alone. The log rolls
  to `.old` at 512 KB. Harness gotcha for next time: the settling SW_HIDE
  ends a `ShowDialog` loop, so a fake test window must use
  `Application.Run`, or it exits the moment the frame hides it.

- **Fullscreen and the frame are mutually exclusive (2026-07-20 late
  evening, both user-reported).** `-fs` sizes the video window to the whole
  monitor, and the frame happily adopted it: a monitor-wide landscape
  "iPhone" with the portrait mirror floating inside. Now checking either UI
  switch clears the other (ahead of the suppress/self-test guards, so a
  legacy settings file with both on reconciles on restore), Start won't open
  the frame for a fullscreen session, and the frame switch is disabled while
  a fullscreen session runs (flipping it mid-session would adopt the
  fullscreen window). The frame itself also refuses: caption-less +
  covers-its-monitor is the fullscreen fingerprint
  (`Test-FullscreenVideoWindow`, verified against real windows), and such a
  window is blacklisted with an explanation instead of adopted — that guard
  is what covers a CLI `start-airplay.ps1 -Fullscreen` next to a standalone
  frame. Same evening, same report: **the PIN now rides every running
  state** — it used to be appended only to the idle "Discoverable" detail,
  so it vanished the moment the hero advanced to "iPhone connected" /
  "Mirroring" (exactly when a second phone joining via `-nohold` takeover
  still needs it). Sub-status MinHeight grew to 4 lines (worst case = stall
  instruction + PIN line); both behaviours are `-SelfTest`-asserted.

Resolution note: the UI's segmented control is **1440p / 1080p** (default
1440p; 720p dropped as unused, **4K removed entirely**). The "bigger
advertised size cannot make things worse" assumption was **wrong and is
disproven twice**: with `-s 3840x2160` the phone (iOS 26.5.2, USB-tether/
hotspot link) established the RTSP connection and **never opened the data
connection** — no video window, no error, exactly the iOS-27 symptom in *iOS
compatibility*. `-s 2560x1440` on the same link streamed immediately, both
times. 4K was briefly offered as an opt-in segment and got selected in normal
use within the hour, reproducing the stall — an option whose only observable
behaviour is "mirroring silently breaks" is a trap, not a feature, hence
removed. The UI self-test asserts it stays gone. (Root-caused later the same
day: 4K streams are always H.265 and `-h265` was missing — see the resolved
stall in *Status*. The segment stays removed on sharpness grounds anyway.)

Perceived sharpness note, learned the same session: raising `-s` raises the
ENCODED pixel count, but what anyone sees is bounded by the on-screen size of
the mirror (and, over a call, by what the share of that region carries). A
1440p stream shown 380px wide looks identical to 1080p. That is why
`frame-mirror.ps1` defaults to ~80% of the work-area height.

## Known pitfalls (in the order they actually bite)

1. **Windows Firewall / Public network profile.** mDNS inbound is blocked on the
   Public profile, so the PC never appears in the mirroring list. `setup.ps1`
   adds explicit rules for all profiles; if that isn't enough, run
   `setup.ps1 -SetNetworkPrivate`.
2. **AP / client isolation on the router.** Blocks Wi-Fi clients from reaching
   wired devices. Common on guest SSIDs, and a silent killer of this exact
   wired-PC / Wi-Fi-phone setup. `doctor.ps1 -PhoneIP <ip>` detects it.
3. **Phone on a different subnet or a guest SSID.** mDNS does not route across
   subnets. `doctor.ps1 -PhoneIP <ip>` compares subnets explicitly.
4. **Extra network adapters** (VPN, Hyper-V, Wi-Fi also on) can make Bonjour
   advertise the wrong IP. `doctor.ps1` warns when more than one is up.
5. **Bonjour missing or stopped.** On the 1.72 build this repo uses, Apple
   Bonjour *is* the mDNS responder (`uxplay.exe` imports `dnssd.dll`). Without it
   there is no advertisement at all, and nothing on screen says so. `setup.ps1`
   fails hard when it is absent; `doctor.ps1` check 2 reports it.
6. **Bonjour running but DEAF.** Proven live on the home PC 2026-07-20 (see
   *Status*): the Apple Devices app's helpers (`AMPDevicesAgent`,
   `AppleMobileDeviceLauncher` — installed *for the USB tether*) can own the
   network-facing UDP 5353 sockets, leaving `mDNSResponder` with loopback only.
   Registration succeeds, every service check passes, and the PC is invisible
   to every iPhone on the LAN — while **USB-tether sessions still work**,
   because that interface gets fresh sockets when the cable comes up. The
   symptom is precisely "Wi-Fi doesn't work, cable does". Trigger: the Bonjour
   service (re)starting while the Apple helpers are running (fresh install day;
   Apple software updates). Detection: `doctor.ps1` check 2, the UI
   ("Not discoverable"), and `start-airplay.ps1` all now check socket
   ownership via `Get-BonjourSocketState`. Repair (elevated, ONE line — the
   helpers re-grab a freed port within seconds, so it must be one shot):
   `Stop-Process -Name AMPDevicesAgent,AppleMobileDeviceLauncher -Force -ErrorAction SilentlyContinue; Restart-Service 'Bonjour Service'`

## iOS compatibility — read before debugging

How this works at all: UxPlay advertises a deliberately old `srcvers`, which
makes iOS fall back to the **"Legacy Protocol"** originally built for the
3rd-gen Apple TV. Every open-source AirPlay receiver depends on this fallback.
No open-source project implements modern AirPlay 2 mirroring.

| iOS | Status |
|---|---|
| 17 | Confirmed working (stated in UxPlay's README) |
| 18 | Working — no UxPlay-specific breakage reported |
| 26 | Working. A broad iOS 26 AirPlay regression hit *commercial* receivers and was fixed in 26.2 |
| **27** | **At risk — unresolved.** See below |

**iOS 27:** UxPlay issue [#535](https://github.com/FDH2/UxPlay/issues/535)
(opened 2026-07-15, still open): the phone negotiates mirroring and holds the
RTSP connection, but **never opens the data connection**. A new SETUP plist key
`combinedGetInfoWithControlSetup` signals a different data-connection
negotiation mode that no open-source receiver handles. Currently a **single
report** (iPhone16,2 / Arch / Wayland, no maintainer response), so it is
uncertain whether it's universal.

**If mirroring fails on a current iPhone, check its iOS version first** — this
may be an upstream protocol break, not a local misconfiguration. In that case
`doctor.ps1` will pass every check while mirroring still fails, and the symptom
is specifically: the PC *appears* in the mirroring list and the phone seems to
connect, but no video ever arrives.

## Licensing note

This repo is MIT and contains only PowerShell glue — no protocol code. But it
installs **UxPlay, which is GPL-3.0** and vendors `playfair` (a
reverse-engineered FairPlay implementation containing verbatim Apple response
blobs). That is fine for personal use; it matters if this is ever redistributed
as a bundle. Also note **Apple-DRM content (Apple TV+, Netflix) will never
mirror** to any open-source receiver.

## Ports used

| Port | Proto | Purpose |
|---|---|---|
| 7000, 7001, 7100 | TCP | AirPlay control / RTSP / mirroring |
| 6000–6009 | UDP | RTP video, audio, timing |
| 5353 | UDP | mDNS (Bonjour discovery) |

**The table above is misleading, and this cost real confusion.** Observed live on
2026-07-20: started with no `-p`, UxPlay 1.72 does **not** bind 7000. It lets the
OS pick an **ephemeral port** (seen: TCP 49408, different every run) and publishes
that port in the mDNS record. The iPhone reads the port from mDNS, so mirroring
works fine — but it means:

- The fixed 7000/7001/7100 and 6000–6009 rules **cover nothing in normal use**.
- The **program-scoped rule for `uxplay.exe`** (any port, all profiles) is the
  only thing actually permitting inbound traffic. It is **load-bearing, not a
  convenience**. If it is missing or points at a stale path after a version
  upgrade, the phone will *see* the PC and then fail to connect — which looks
  exactly like a protocol problem and isn't.
- Any check that only looks at port 7000 will report "receiver not running" while
  it is running perfectly. `doctor.ps1` now enumerates the engine's real
  listening ports by PID and verifies the program rule's path matches the engine.

Pass `-Port 7000` if you specifically need the fixed rules to be the ones doing
the work (e.g. a restrictive third-party firewall that ignores program rules).

## Conventions

- PowerShell scripts, no external runtime dependencies beyond UxPlay itself.
- Anything that changes system state lives in `setup.ps1`, announces itself
  before acting, and supports `-WhatIf`.
- `doctor.ps1` must stay strictly read-only.

## Status

### Verified (on a *different* Windows 11 PC, against UxPlay 1.72.1.3)

Confirmed by actually running it — don't re-test these:

- `doctor.ps1` runs correctly end to end and correctly caught a real port
  conflict (another AirPlay app holding TCP 7000).
- `start-airplay.ps1` locates the engine, sets `GST_PLUGIN_PATH`, and launches.
  **All tuned flags are accepted** by 1.72: `-vsync no`, `-vs d3d11videosink`,
  `-as wasapisink`, `-fps 60`, `-nc`, `-nohold`, `-p`.
- GStreamer selects **`d3d11h264dec`** — hardware H.264 decode is active.
- mDNS registration works:
  `register_dnssd: advertised AirPlay service with "Features" code = 0x527FFEE6`.
  Bit 27 is off, which is what makes clients skip legacy pairing and removes a
  ~5s connection delay.

Also verified: the full elevated `setup.ps1` firewall path runs and produces
four correct rules.

### Attempted and failed — for network reasons, not software

One real mirroring attempt was made (2026-07-20) and the PC **did not appear**
in the iPhone's list. Cause: the phone was on a **guest Wi-Fi SSID** while the
PC was wired. Guest networks apply client isolation and usually a separate
subnet/VLAN, either of which alone prevents discovery. Nothing on the PC can
bridge that.

This was *not* a software failure — the receiver was confirmed advertising
correctly at the time (its advertised name visible via `dns-sd -B _airplay._tcp` on the
PC itself). **Don't go looking for a bug in this scenario.** The home network
(PC on cable, phone on the same non-guest Wi-Fi, one router) is the supported
case and is what still needs testing.

### Verified on the work PC (2026-07-20, UxPlay 1.72.1-3)

Machine: Ethernet only (Realtek 2.5GbE, single wired /24) — **there is no Wi-Fi
adapter**, so this PC cannot host a hotspot or join the phone's SSID.

- Engine launches and reaches `Initialized server socket(s)`.
- **mDNS advertisement confirmed live**: a receiver started as `SMOKETEST` showed
  up in `dns-sd -B _airplay._tcp` within seconds. This is the first direct
  confirmation of advertisement on this machine.
- All four firewall rules already exist and are enabled.
- Every flag in `start-airplay.ps1` was checked against `uxplay -h` for 1.72 and
  is valid.
- `airplay-ui.ps1 -SelfTest` passes; all 19 named elements bind.

Two environment problems were found and fixed:

- **PigeonCast was running and holding TCP 7000**, with a stale `uxplay.exe` of
  its own from an earlier session. Both advertised over mDNS, so the iPhone would
  have shown extra entries. Killed. PigeonCast is **not** in any Run key or
  Startup folder, so it will not come back by itself.
- **ESET Security is installed.** It has its own firewall, independent of Windows
  Firewall. If discovery ever fails while `doctor.ps1` passes everything, this is
  the first suspect — `doctor.ps1` check 4b now names it.

Note the Ethernet adapter sits on the **Public** profile. The explicit rules
cover it, and mDNS demonstrably works, so this was left alone rather than
elevating to change it.

### Verified on the home PC (2026-07-20, UxPlay 1.72.1.3 via winget)

Machine: wired Ethernet (Intel I226-V, Private profile), plus
an Xbox Wireless Adapter and a Hyper-V vEthernet also up — doctor warns about
the extra adapters but mDNS was not tested against them yet.

- Fresh setup done this date: Apple Bonjour 3.1.0.1 (winget `Apple.Bonjour`) and
  Apple Devices (Store) installed; `setup.ps1 -UseWinget` ran elevated and its
  verify pass passed everything — engine found, d3d11 sink present, all four
  firewall rules created and active.
- `airplay-ui.ps1 -SelfTest` passes (31 elements bind); the UI launches cleanly
  via `AirPlayPC.cmd`.
- **First successful mirroring session ever (2026-07-20):** the iPhone found
  the PC in Screen Mirroring, connected, and video appeared on the PC.
  Note the video took a few seconds to appear after the phone showed "linked" —
  long enough that it was briefly reported as "connected but no screen". That
  lag looks normal; don't debug it unless it exceeds ~10s.

### Wi-Fi discovery root-caused and repaired (home PC, 2026-07-20 afternoon)

"Mirroring only works over the USB cable, never over Wi-Fi" had a PC-side
cause, found and fixed this date — now pitfall 6:

- Timeline from process creation times: `AMPDevicesAgent` (Apple Devices)
  started **12:33:37** — the iPhone's first USB connection; `mDNSResponder`
  (re)started **12:34:05** (Bonjour had been winget-installed minutes earlier)
  and lost the bind race. The Apple helper held the LAN interface's `:5353`; Bonjour
  held **only loopback**. Every LAN advertisement since was silence, while the
  tether interface kept working because it got fresh sockets on each cable
  plug. Cable-yes / Wi-Fi-no is the fingerprint of this state.
- The repair needs ONE elevated shot (`Stop-Process` both helpers `;`
  `Restart-Service 'Bonjour Service'`): after a first, split attempt,
  `AppleMobileDeviceLauncher` re-grabbed the freed port within seconds.
- Verified after repair: `mDNSResponder` owns 5353 on the LAN, vEthernet and
  tether interfaces; a `dns-sd` register+browse round trip passes; the browse
  also sees the other AirPlay devices on the LAN arriving —
  mDNS demonstrably flows both ways again.
- Detection is now permanent: `Get-BonjourSocketState` (shared), doctor
  check 2 (FAIL with the one-line fix), the UI ("Not discoverable" heads-up),
  and a `start-airplay.ps1` pre-launch warning.
- **End-to-end CONFIRMED the same afternoon (~14:17)**: with the phone on the
  home SSID (same /24 subnet as the PC), the PC appeared in
  Screen Mirroring — after a short delay, so give the list a few seconds —
  and a full mirroring session ran with no cable. First Wi-Fi mirroring
  session ever on this machine. The UI's new engine-output capture recorded
  the whole negotiation (`connection request from iPhone (iPhone16,1)` →
  `raop_rtp_mirror starting mirroring` → `Begin streaming to GStreamer video
  pipeline`); the RTSP connection again came in over home-LAN IPv6.

Also corrected this date: the home PC **does** have a Wi-Fi adapter (Intel Wi-Fi
6E AX211, not connected to any network) plus a Bluetooth PAN adapter — the
"no Wi-Fi radio" statements elsewhere in this file describe the *work* PC.

### Not verified

1. ~~An actual iPhone has never successfully connected.~~ **Resolved 2026-07-20**
   — see the home-PC section above; a real mirroring session completed.
2. **The v2.x install layout.** Both machines so far run **1.72.1-3** (winget).
   The scripts now *search* for the engine instead of hardcoding paths, so a
   layout change should no longer break them — but this is untested.
3. **~~`setup.ps1`'s GitHub install path.~~** Now exercised by CI on every
   push: real release lookup, download, SHA-256 verification and silent
   install on a clean runner (`ci.yml`, added 2026-07-20).

### Guest Wi-Fi: the USB tether path

The recurring blocker is a phone on a guest SSID and a wired PC. Client isolation
and a separate subnet each independently prevent discovery, and **no PC-side
change can bridge either** — especially here, where the PC has no Wi-Fi radio.

The working answer is **USB Personal Hotspot**: it puts the phone and PC on one
direct link and removes the router from the picture entirely. Requires the
**Apple Devices** app from the Microsoft Store for the USB network driver —
**verified installed on the work PC (1.1540.23042.0) as of 2026-07-20**, so
that prerequisite is now met. Once the tether link is Up, AirPlay runs over it
normally. `doctor.ps1` check 8 walks through this and reports whether the link
is live.

**The one remaining blocker on this PC is the cable.** The cable on hand is
charge-only and the iPhone is never enumerated — `Get-PnpDevice -Class Net`
shows no Apple network adapter, and no Wi-Fi radio exists to fall back on
(single Realtek 2.5GbE NIC). A data-capable Lightning/USB-C cable
is therefore the entire shopping list for a first real mirroring test. Note that
a charge-only cable fails *silently*: no Trust prompt, no device in Device
Manager, nothing that names the cable as the cause.

### RESOLVED (2026-07-20 evening): the "RTSP holds, no video" stall was a missing `-h265`

**Status: root-caused and fixed — `Build-UxPlayArgs` now always passes
`-h265`. The section is kept for the method and the disproved theories.**

Symptom: phone says "connected", engine holds exactly one established TCP
connection (RTSP, phone 172.20.10.1 → PC 172.20.10.6 on the engine's ephemeral
port), **no GStreamer video window is ever created**, no error anywhere.
All of today's affected sessions ran over the iPhone **USB-tether/hotspot
link** (172.20.10.x, Apple's fixed hotspot subnet); the PC's LAN IP stayed
its home-LAN address.

Today's timeline (all UxPlay 1.72.1-3, same phone, iOS 26.5.2):

| ~Time | Launcher | `-s` | Result |
|---|---|---|---|
| 13:0x | UI | 1920x1080 | **WORKED** (first-ever session; link for this one possibly home LAN, not tether) |
| 13:1x | UI | 3840x2160 | stall |
| 13:2x | CLI (`start-airplay.ps1`, logged) | 2560x1440 | **WORKED** immediately, same tether link |
| 13:36 | UI | 3840x2160 | stall |
| 13:38 | UI (pid 18044, stopped from UI) | 2560x1440 | stall |
| 13:39 | UI (pid 4676) | 2560x1440 | **stall — this breaks the "only 4K stalls" theory** |

So `-s 3840x2160` reliably stalls (2/2) and was removed from the UI — that part
stands. But the last two stalls were at 1440p, meaning at least one more factor
exists. Untested hypotheses, in order of plausibility:

1. **Stale phone-side session state after many rapid reconnects** to freshly
   restarted engines (the phone re-ran Screen Mirroring ~8 times within the
   hour against ~6 different engine instances). Test: toggle Screen Mirroring
   fully off, toggle the hotspot/cable, or reboot the phone, then one clean
   attempt.
2. **UI-launched vs CLI-launched engine.** The one certain tether success was
   CLI-launched (console visible, output logged). But the first 1080p success
   was UI-launched, so this is not clean either. Test: reproduce the stall
   with `.\start-airplay.ps1` (logging is on by default) and read the log to
   see the exact request where negotiation stops — **no stalled session's
   engine output has ever been captured**, because the UI cannot capture it
   (see *Logging*).
3. Something about the tether link degrading over time (it worked at 13:2x).

Debugging aids that now exist: UI session logs and CLI engine logs in
`%LOCALAPPDATA%\pcairplay\`; the frame's aspect-measure log in
`frame-debug.log` in the same directory. The framed mirror does nothing until
a GStreamer window exists, so it is not a suspect for the stall itself.

**Afternoon update (2026-07-20, instrumentation pass):**

- **The "same tether link" claim for the 13:2x CLI success is wrong.** Its
  engine log shows the RTSP client arriving over **home-LAN IPv6** (a ULA
  local ↔ global remote pair), not 172.20.10.x. Which link any session's RTSP
  actually used is unknown for every other row too — do not build
  launcher-vs-launcher or link theories on that column.
- All of the day's sessions ran while **Bonjour was deaf on the LAN**
  (12:34–~14:00, see *Status*), so discovery rode the tether interface's
  sockets. Whether that relates to the stall is unknown; it is fixed now, so a
  recurring stall would prove it unrelated.
- **No stalled session ever created a GStreamer window**: `frame-debug.log`
  does not exist, and the frame writes it only once attached — consistent with
  the netstat observation (RTSP established, video data connection never
  opened). The "no image in the mockup" report is the stall, not a frame bug.
- **The missing evidence now collects itself**: the UI tees engine output into
  `airplay-ui-*.log` (see *Logging*), the hero shows "Connected, no video"
  with the phone-side reset after 12 s of RTSP-without-video, and the frame's
  placeholder says the same. The CLI gained `-EngineDebug` (uxplay `-d 1`) for
  a verbose capture of exactly where negotiation stops. **Next stall: read the
  newest `airplay-ui-*` / `start-airplay-*` log.**

**Evening resolution:** the very first stalled session with engine output
captured showed the cause outright, twice in one log:

    raop_rtp_mirror starting mirroring
    *** ERROR: raop_rtp_mirror: received type 0x01 packet with no payload:
    this indicates non-h264 video but Airplay features bit 42
    (SupportsScreenMultiCodec) is not set
    use startup option "-h265" to set this bit and support h265 (4K) video
    Connection closed for socket 12460

The phone sometimes elects to send **H.265** — always at 4K (why
`-s 3840x2160` stalled 2/2) and sometimes at 1440p — and without `-h265` the
engine rejects the stream *after* the phone has shown its tickmark. Codec
choice varies per session, which is exactly the "same settings, different
outcome" pattern in the table above. Launcher, link and phone-state theories
are all dead. A session captured minutes after the fix confirmed the
counterpart: same settings, phone chose H.264, streamed fine without the
flag — the dice-roll, observed from both sides. The 4K UI segment stays
removed regardless, on the perceived-sharpness grounds above.

### Before a demo — do this in order

1. `.\doctor.ps1` — expect no blocking problems, and check 5 to report no
   competing receiver.
2. If the phone can join the PC's network: get its IP and run
   `.\doctor.ps1 -PhoneIP <ip>`. **Don't skip this** — it is what separates a
   software problem from a network-topology one.
3. If the phone is on a guest SSID: set up the USB tether above instead. Do it
   *before* the demo, not during — it needs a Store install and a Trust prompt.
4. Launch `AirPlayPC.cmd`, press Start, mirror from the phone.
5. Update this section with what actually happened.

## Gotchas found the hard way

- **Do not launch the engine from a PowerShell background job / anything without
  a console.** Under `Start-Job` it prints its banner and then silently stalls
  before initialising GStreamer — no error, no mDNS registration. Use
  `Start-Process` or a real console. This cost real debugging time; it looks
  exactly like a broken install. **A HIDDEN console is fine**: under
  `Start-Process -WindowStyle Hidden` the console exists (window never shown)
  and the engine reaches `Initialized server socket(s)` — verified live
  2026-07-20, and it is how the UI runs the engine with nothing in the taskbar.
- **The engine's `-pin[xxxx]` help notation lies.** `-pin1234` (concatenated,
  as the help reads) → `unknown option -pin1234`, engine exits. `-pin 1234`
  (separate token) works. Both verified live on 1.72.1-3. `Build-UxPlayArgs
  -PinCode` emits the working form; the UI displays the code it generated.
- **PowerShell converts `$null` to `""` when binding a .NET `string`
  parameter** — so `[Native]::FindWindow($null, $title)` searches for class
  name *empty string* and returns 0 for every window on the system, silently.
  Cost a live debugging round: the UI "closed" a frame that never got the
  message. Call a C#-side wrapper whose `null` is real
  (`PCAirPlayNative.FindWindowByTitle`), or pass `[NullString]::Value`. Any
  new P/Invoke with an optional string parameter needs this treatment.
- **A PowerShell function "returning" a `byte[]` unrolls it into the
  pipeline**, so the caller receives `object[]` — and `BinaryWriter.Write`
  then resolves the overload by COERCING `object[]` TO BOOLEAN, writing one
  `0x01` byte per call, silently. The icon generator shipped a 142-byte .ico
  whose directory claimed 57 KB this way. Return binary arrays with the unary
  comma (`, $bytes`) and cast at the write site (`[byte[]]`).
- **`New-NetFirewallRule -LocalPort` needs an array, not a comma-separated
  string.** `'7000,7001,7100'` fails with "The port is invalid"; ranges like
  `'6000-6009'` are fine. This already bit once: the TCP control rule silently
  never got created while the script still reported success, so setup looked
  clean while the receiver stayed unreachable. Fixed, but **don't reintroduce
  it, and don't trust a "created" message without verifying the rule exists.**
- **`dns-sd.exe` lives in `C:\Windows\System32`**, not in `C:\Program Files\Bonjour`.
- **`dns-sd -B` never exits** and has no working `-timeout` in this build — always
  run it as a job you stop explicitly, or it hangs the shell.
- **`Failed to load plugin 'libgstcurl.dll'` on stderr is benign** — it only
  affects HLS/YouTube streaming, not mirroring. Ignore it.
- **`0xFFFFFFFF` is `[int]` -1 in PowerShell**, so `[uint32]0xFFFFFFFF` and
  `[uint64]0xFFFFFFFF` both throw. In `doctor.ps1` this left the netmask `$null`,
  so the subnet comparison became `0 -eq 0` and **reported "Same subnet" for
  every address ever passed** — silently defeating the one check that
  distinguishes a guest SSID from a real fault. Use the decimal literal
  `4294967295`. Regression-test any change here with a same-subnet IP *and* a
  deliberately foreign one; a check that only ever passes looks identical to a
  check that works.
- **`DragMove()` dispatches timer ticks inside its modal move loop.** A
  DispatcherTimer tick that does slow work on the UI thread (CIM queries,
  `Get-NetTCPConnection`) lands mid-drag and freezes the window under the
  cursor for its duration — reported as "moving the window lags for a
  second". Pause polls around DragMove; better, keep heavy work off the UI
  thread entirely.
- **WPF triggers cannot target `GradientStop` by name.** `<Setter TargetName="g1"
  Property="Color">` where `g1` is a GradientStop throws a `KeyNotFoundException`
  wrapped in "Initialization of 'System.Windows.Setter' threw an exception" —
  gradient stops are not in the template's namescope. Swap the whole brush
  instead. Two `<Setter>`s for the same property in one trigger fail the same
  way. Both cost real time; `airplay-ui.ps1 -SelfTest` catches them in a second.
- **`StaticResource` inside a `ControlTemplate` trigger** is fragile in these
  inline templates — literal colours are used there deliberately.
- **Another AirPlay app is a discovery bug, not just a port clash.** PigeonCast
  and friends also advertise over mDNS, so the phone lists two devices and the
  wrong one gets tapped. `doctor.ps1` check 5 and the UI both look for this.
- If the iPhone connects but the window is black, suspect `GST_PLUGIN_PATH`.
- If everything passes but no video arrives, check the iOS version against the
  compatibility table above.

### The argv fork (fixed 2026-07-20 — don't recreate it)

`start-airplay.ps1` and `airplay-ui.ps1` built the argument vector and sanitised
the device name independently. Both drifted, and **neither side could see it**,
because each looked correct read on its own:

- The UI emitted `-s "$res@$fps"` — wiring the **framerate** into the **refresh
  rate** slot of `-s wxh@r`. Picking "30 fps" in the UI therefore also asked the
  phone for a 30 Hz *display mode*, which is a different request. This was the
  exact conflation commit `3e8f02c` had already split apart on the CLI side; the
  UI kept the bug for another four commits.
- The UI's name sanitiser had **no leading-`-` guard**, so a device name of
  `-Demo` reached the engine and produced `*** ERROR: invalid: "-n" had no
  argument` — the failure the CLI path rejects up front, and the one that sends
  people off looking for a broken install.
- The UI could not reach `-ShareSafe`, `-NoAudio`, `-SoftwareDecode`, `-Port` or
  `-LegacyPorts` at all.

Now both call `Build-UxPlayArgs` / `Resolve-UxPlayDeviceName`. `-SelfTest`
asserts the `@r` half stays 60 when 30 fps is chosen, that share-safe reaches
`glimagesink`, and that `-Demo` is rejected — it used to *print* the command line
and assert nothing about it, which is how the drift survived.

**Lesson worth keeping:** two entry points over one engine will drift unless the
argv is shared, and a self-test that prints without asserting is decoration.

## Known remaining work

Surveyed 2026-07-20. Roughly in payoff order:

1. **~~Nothing parses the engine's stdout.~~ Mostly done 2026-07-20:** the UI
   captures engine output (see *Logging*) and derives true connection state
   from the engine's TCP sockets plus log markers — `Resolve-SessionState`
   drives Discoverable / iPhone connected / Mirroring / the stall warning.
   ~~Remaining: the PIN is not surfaced in the UI itself~~ — closed the same
   evening by *generating* the PIN in the UI and passing `-pin nnnn`, so
   there is nothing to parse; the sub-status shows "PIN: nnnn". The CLI does
   no parsing (its console shows everything anyway).
2. **~~`doctor.ps1`'s port-holder scan still hardcodes `7000, 7001, 7100`.~~**
   Fixed 2026-07-20 — the list now derives from `Get-PCAirPlayPortRule`.
3. **Only `Fail` feeds doctor's exit code.** A disabled firewall rule, a missing
   d3d11 plugin, a Public profile, or a running third-party suite are all `Warn`
   and thus invisible to `exit 1`.
4. **~~No settings persistence.~~ Done 2026-07-20 evening** — the UI persists
   name/resolution/fps/toggles to `%LOCALAPPDATA%\pcairplay\ui-settings.json`
   (saved on close and on Start, restored in `Add_Loaded` — NOT earlier: a
   switch checked before its template applies misses its Checked storyboard
   and renders off-looking while on). The frame persists position and zoom
   the same way (`frame-settings.json`, clamped to the virtual screen on
   restore). Both round-trips are `-SelfTest`-asserted.
5. **~~No test suite or CI.~~ CI added 2026-07-20** — `ci.yml` runs the real
   `setup.ps1` (engine download + firewall + `-Uninstall`), both `-SelfTest`s,
   `-DryRun`, `-WhatIf` and an installer smoke-build on every push. Still
   true: `uxplay-common.ps1` and `doctor.ps1` have no direct unit coverage.
6. **~~No uninstall/teardown.~~ Done 2026-07-20** — `setup.ps1 -Uninstall`
   removes the four firewall rules (and only them — UxPlay/Bonjour keep their
   own uninstallers), and the installer's uninstaller runs it. Reverting
   `-SetNetworkPrivate` stays manual (the original profile is unknowable);
   the uninstall output prints the command.
7. **`Get-UxPlayVersion`'s fallback** returns the first uninstall entry's
   version when no `InstallLocation` matches — the stale-second-install case its
   own doc comment says it was written to prevent.
8. **~~The UI neither launches nor tracks the framed view.~~ Done 2026-07-20
   evening** — the "iPhone frame" switch (see *The framed mirror*): opens with
   the UI and on Start, closes on uncheck and always on UI close, resyncs to
   reality on a ~12 s poll. `Test-FramedMirrorRunning` /
   `Close-FramedMirrorWindow` are the shared checks in `uxplay-common.ps1`.

## Distribution (added 2026-07-20)

Public since 2026-07-20; the packaging decisions below are deliberate.

- **Two repos, two histories.** The public `gbulog/pcairplay` starts from a
  fresh root commit authored with the GitHub noreply address. The full dev
  history lives in the private `gbulog/pcairplay-dev` and must **never** be
  pushed to the public remote: its commits carry a private email, and old
  CLAUDE.md revisions carry machine/network details (`CLAUDE.local.md` has
  the specifics). Publishing = copy the tracked tree into the public clone
  and commit there; `publish.local.ps1` in the private checkout does it.
- **The installer is a NET-installer on purpose.** It ships only this repo's
  MIT scripts. Bundling was rejected twice over: Apple Bonjour may not be
  redistributed without Apple's bundling agreement, and shipping `uxplay.exe`
  would make this project the distributor of GPL-3.0 code plus playfair's
  FairPlay blobs. `setup.ps1` downloading from upstream at install time (with
  the SHA-256 check) keeps all of that upstream. Do not "optimise" this into
  an offline bundle. The 1.72.1-3 asset is additionally pinned by a hardcoded
  SHA-256 in `setup.ps1` (`cb45de36…7986`) — the API digest alone would move
  with a swapped asset, so it only backstops releases newer than the pin.
- **Releases:** tag `v<x.y.z>` on the public repo → `release.yml` builds
  `AirPlayPC-setup-<x.y.z>.exe` + `AirPlayPC-<x.y.z>.zip` + `SHA256SUMS.txt`
  and publishes the GitHub release. The exe is **unsigned**: SmartScreen shows
  "Windows protected your PC" until download reputation accrues — documented
  in the README. If that becomes a real adoption problem, Azure Trusted
  Signing (~$10/month) or an OV certificate is the fix. This stack
  (PowerShell + firewall edits + hidden consoles + a VBS launcher) is
  heuristic bait for AV engines: publish hashes with every release, and never
  wrap the scripts in PS2EXE — it is a notorious false-positive magnet.
- **winget** (not done): after a few releases, submit a manifest to
  `microsoft/winget-pkgs` pointing at the release exe. The installer's silent
  path already runs setup hidden, so a winget install ends fully configured.

## Logging (changed 2026-07-20)

Both launchers write to **`%LOCALAPPDATA%\pcairplay\`**, five files kept per
prefix, via `New-PCAirPlayLogPath` in the shared module. One directory on
purpose: someone asked to "send the log" should not have to know which launcher
produced the session.

**`start-airplay.ps1` now logs by default.** It used to be `-Log`, opt-in, on the
grounds that piping changes the launch (colour is lost, PowerShell reformats the
engine's stderr into a `NativeCommandError` block). That trade was wrong here:
mirroring fails silently and after the fact, so an opt-in log is never on for the
run that actually needed it. `-NoLog` opts out; `-Log` is still accepted and does
nothing. Verified 2026-07-20 by a real launch — the log reaches `Initialized
server socket(s)`, so piping still is not the no-console launch that stalls.

**The UI now captures engine output too (changed 2026-07-20 afternoon).** The
old blocker — `-RedirectStandard*` suppresses the console the engine needs and
nulls `.ExitCode` — is sidestepped rather than solved: the UI launches the
engine as a child of a **powershell.exe wrapper console** (fully HIDDEN since
the evening pass — a hidden console is still a real console, verified live by
the engine reaching `Initialized server socket(s)` under `-WindowStyle Hidden`;
PIN mode no longer needs it visible because the UI generates a fixed PIN and
displays it itself) which runs the same `2>&1 | Tee-Object` pipeline the CLI
uses and relays the engine's exit code via `exit $LASTEXITCODE`. `airplay-ui-*.log` is
therefore the session-record header, then `---`, then the engine's own words,
then a footer with engine PID / how it ended / exit code. The whole file is
UTF-16LE (Tee-Object's encoding; the header matches or the file reads as
"u x p l a y"). Verified live: the wrapped engine reaches `Initialized server
socket(s)` — the wrapper console is a real console, not the no-console stall.
Caveat: the engine's stdout is block-buffered through the pipe, so lines can
arrive late; they always flush by process exit. That is why the UI's live state
comes primarily from the engine's TCP sockets (below), with log markers as
confirmation.

### Firewall checks are shared now (2026-07-20)

`Get-FirewallRuleState` moved from `doctor.ps1` into `uxplay-common.ps1`, because
`setup.ps1`'s **verify pass** — the code whose entire job is to confirm the
result — was doing the bare `$r.Enabled -eq 'True'` on an unwrapped
`Get-NetFirewallRule` that this helper exists to replace. The loosest check in
the script was the one blessing it.

`Test-PCAirPlayPortOverlap` (also shared) replaces doctor's hardcoded
`'5353','7000','7001','7100'` block-rule list. That list omitted **UDP 7011 and
the whole 6000–6009 RTP range**, and compared with `-in` against single ports, so
a BLOCK rule written as a *range* — the way a human would most likely write one —
matched nothing. It now intersects intervals against `Get-PCAirPlayPortRule`, and
reports `Any` rather than skipping it. Regression-tested against nine cases
including three that must NOT match; per the netmask lesson above, test the
negatives or a broken check looks exactly like a working one.

## Not built

- Auto-start-on-login (discussed, not implemented). The **tray icon exists**
  since 2026-07-20: minimise sends the UI to the tray by toggling
  `ShowInTaskbar` — deliberately NOT `$window.Hide()`, which ends a
  `ShowDialog` loop and would silently exit the whole app.
- Session recording (`-vdmp`/`-admp`) — `.gitignore` already anticipates the
  dump files, but no script can produce them.

---
> Source: [gbulog/pcairplay](https://github.com/gbulog/pcairplay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
