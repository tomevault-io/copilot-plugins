## wince-dc

> Self-contained build that takes the Sega "Dragon" Windows CE 2.12 SDK and adds a real networking

# CLAUDE.md — wince-dc (Windows CE on the Sega Dreamcast)

Self-contained build that takes the Sega "Dragon" Windows CE 2.12 SDK and adds a real networking
stack (Ethernet + modem + dead-game-server revival), a windowed desktop shell with apps (browser,
media player, file explorer, network diagnostics), and SPI transports (W5500, SD card) — then
bakes a bootable Dreamcast disc with the vendored CE image tools + a real SH-4 PE compiler.
Everything builds from this repo via **CMake**; nothing external is needed.

> The user is a low-level Dreamcast/OS-porting expert — give expert-level, register-specific
> answers, no beginner framing. Sibling projects on their machine (ReactOS `sh4pe-toolchain`,
> `DreamShell`, `img4dc`) are SEPARATE — not part of this repo.

## Current state (resume here)
- ✅ **Self-contained CMake build.** The SH-4 compiler (`vendor/sh-toolchain`) and the whole
  CE 2.12 SDK (`vendor/wcesdk`: headers, libs, image tools, OS modules, patched kernels +
  `.map`/`.pdb`) are vendored. `CMakeLists.txt` (`project(NONE)`) drives the vendored
  `cl.exe`/`shasm.exe`/`link.exe` to build our modules, then `makeimg` → `wrap-image.ps1` →
  `make-gdi.ps1` for the bootable disc. Builds `retail` (silent) or `debug` (SCIF-console)
  images. See `toolchain/README.md`.
- ✅ **Networking — full TCP/IP over the STOCK CE stack.** The SDK's `mppp.dll` (dial-up PPP)
  is replaced by a universal link shim (`net/netif/`) so stock `microstk.exe` + `winsock.dll`
  run over Ethernet — no lwIP. **BBA path verified END-TO-END: DHCP → DNS → TCP → HTTP**, and a
  retail game (4x4 Evolution) dials + reaches its master server on real hardware. DNS chains
  DHCP option-6 → DC system-flash ISP config (`flashrom.c`) → public resolver. **Modem dial-up
  works** too: with no ethernet the shim delegates to the vendored original PPP driver
  (`mpppdial.dll`) and `dcwnet` RasDials to bring it up. **Revival mode** redirects dead game
  master servers to revival hosts — a revival DNS as PRIMARY (default `dns.flyca.st` = DCNet's
  `178.156.255.64`) + hardcoded-IP DNAT for the no-DNS games (Alien Front Online / Internet Game
  Pack); config in `HKLM\Comm\Netif`. Link-ABI + MTU/byte-order/DNS-registry gotchas are in the
  `net/netif/` sources.
- ✅ **DCWin desktop shell + apps.** `shell/` is a windowed desktop + PVR2/Direct3D compositor
  (move/resize/min/max, clipping, analog-stick pointer; the compositor quad buffers grow/shrink/
  free dynamically). Client apps run in their own processes: `dcwcalc`/`dcwclock`/`dcwexp`/
  `dcwtask`/`dcwmem`, `dcwnet` (Network Diagnostics: dial + DNS/TCP/HTTP test + download-to-RAM
  benchmark with progress/speed), `dcwlog` (in-app System Log viewer), `dcwplay` (music player).
  `dcshell` is the default autorun; `dcwboot.exe` (a D3D boot logo + live checklist that then
  launches `dcshell`) is an optional alternate. SDK-correct DirectInput (DC controller by Maple
  HID usage).
- ✅ **Music player** (`shell/dcwplay.c`) streams MP3 (minimp3) / WAV from disc into a 1-second
  AICA DirectSound ring. DC gotchas baked in: 32-byte buffer + Lock alignment (else
  `DSERR_NOT32BYTEALIGNED`), decoded-PCM carry (no clicks), and always-44100 output via a linear
  resampler (the AICA is pitch-exact only at its native 44100).
- ✅ **SD card over SPI** (`drivers/sdcard/`): a WDM FAT FSD mounts a FAT16 card on the SCIF SPI
  bus as `\External Storage` (FatFs); Explorer browses + launches off it.
- 🔄 **Internet Explorer** (`shell/iexplore.cpp` + `iehost`/`ieproto`) hosts the stock Trident
  WebBrowser control, baked behind **`-DHTML=on`** (the IMGHTML CoreOS_HTML split). Renders via a
  dcgfx page-layer; fetches http over a winsock `IInternetProtocol` (CE WinInet is dead on this
  image). Boots + `CoCreate` OK on Flycast; UNTESTED end-to-end on hardware.
- 🔄 **W5500/MACRAW backend over SPI** (`drivers/dcspi/` SCI hardware-SPI + SCIF bit-bang;
  `net/netif/w5500.c`). Detected over SPI (VERSIONR) and SMALL TCP transfers work (DNS, an HTTP
  308, the 4x4 master), but a SUSTAINED receive (a ~1 MB download) STALLS: the inbound side
  freezes (the netif `rx=` counter stops) ~6–7 KB in. A torn-read guard (re-read the MACRAW
  length header before discarding the ring) + RX-backlog/desync `SysLog` are in to localize it
  on the bit-bang SCIF bus; user suspects the Flycast W5500 emulation for the recent stalls.

## Setup on a fresh PC
1. `git clone <this repo>` — **fully self-contained.** Both the SH-4 compiler
   (`vendor/sh-toolchain`) and the CE 2.12 SDK (`vendor/wcesdk`: headers, libs, image tools,
   OS modules, patched kernels + their `.map`/`.pdb`) are vendored. No external SDK.
2. Install **CMake ≥ 3.20 + Ninja** (the VS-bundled pair works — see `toolchain/README.md`).
   Nothing else to configure; the build derives all paths from the repo.

## Build / commands (CMake — see `toolchain/README.md`)
```sh
cmake -G Ninja -DCMAKE_MAKE_PROGRAM=<ninja> -S . -B build
cmake --build build                  # all SH-4 modules -> build/modules/ (dcspi/mppp/dcshell/dcw*)
cmake --build build --target image   # makeimg -> NK.bin -> wrap -> build/0winceos.bin
cmake --build build --target gdi      # sparse/truncated multi-track GDI (fast; emulator)
cmake --build build --target gdi-full # full contiguously-padded GD-ROM (~1.1 GB; GDEMU/burning)
```
Configure-time options (kernel, DLL set, autorun, HTML, and disc extra-data are independent):
- `-DKERNEL=retail|debug` — `retail` (default) bakes the silent `nknodbg.exe`; `debug` bakes the
  patched (no-KD) SCIF-console `nkscifkd.exe`. This only changes boot logging.
- `-DDLLS=retail|debug` — `retail` (default) ships the stock OS DLLs (what real games run on);
  `debug` overlays the checked DLLs from `vendor/wcesdk/image-debug`. The checked DLLs assert and
  break some titles (e.g. DDHAL under DirectDraw), so a SCIF console over a *working* userland is
  `-DKERNEL=debug` alone (DLLs stay retail).
- `-DAUTORUN=<exe>` — program baked into `HKLM\init` `Autorun` (default `dcshell.exe`; pass
  `-DAUTORUN=dcwboot.exe` for the boot-screen-first path). Use forward slashes for a path, e.g.
  `-DAUTORUN=/CD-ROM/DC.EXE` to autostart a disc binary.
- `-DHTML=on|off` — `off` (default) is the lean image; `on` bakes the Trident/IE stack (`IMGHTML`:
  the CoreOS_HTML coredll/gwes/ole32 + IE registry) and stages `iexplore.exe` on the disc. Adds
  the browser; bigger ROM. The IE renderer DLLs stay in ROM (a `\CD-ROM` variant failed — wininet
  won't `LoadLibrary` from disc).
- `-DEXTRADATA=<dir>` — folder whose contents go into the disc's `\CD-ROM` (relative paths
  resolve against the repo root). Our `0winceos.bin` always overrides any `0WINCEOS.BIN` there.

The image step seeds the read-only `vendor/wcesdk/image` tree into `build/image`, applies the
optional checked-DLL overlay + our freshly-built modules + the Autorun edit
(`cmake/prep-image.cmake`), then runs `makeimg` (which re-merges the `.bib`s, so the `IMG*` env
flags pick the kernel). The makeimg env lives in `IMG_ENV` in `CMakeLists.txt`.

## Layout
- `CMakeLists.txt` + `cmake/prep-image.cmake` — the entire build (modules + image + gdi).
- `.clang-format` — house code style (Microsoft, tabs, Allman; see Conventions).
- `toolchain/` — `wrap-image.ps1`, `make-gdi.ps1`, `make-gdi-real.ps1`, `make-disc.ps1`,
  `unwrap-image.ps1`, `bootstrap.ps1`, `README.md` (the build doc).
- `net/` — networking. `netif/` = the universal microstk link shim (drop-in `mppp.dll`
  replacement; BBA verified + W5500 backend). `lwip-port/` + vendored lwIP = the alternative
  bring-your-own-stack route (built, unused). CMake `mppp` target.
- `drivers/` — `bba/` (RTL8139 driver — RETIRED into the shim's `bba_hw.c`, kept as HW ref);
  `dcspi/` (reusable SPI transport: SCI hardware-SPI + SCIF bit-bang; CMake `dcspi` target);
  `sdcard/` (WDM FAT FSD over the SCIF SPI bus → `\External Storage`; ChaN FatFs in `fatfs/` is
  vendored third-party; CMake `sdcard` target).
- `shell/` — the DCWin desktop shell (`dcshell`) + PVR2/Direct3D compositor (`dcgfx`) + DirectInput
  (`dcinput`) + client lib (`dcwlib`) + client apps (`dcwcalc`/`dcwclock`/`dcwexp`/`dcwtask`/
  `dcwmem`/`dcwnet`/`dcwlog`/`dcwplay`, `dcwboot` boot screen) + the IE browser
  (`iexplore`/`iehost`/`ieproto`, `-DHTML=on`). CMake `dcshell`/`dcwboot`/`iexplore`/`dcw*` targets.
- `vendor/sh-toolchain/` — SH-4 compiler + CE headers. `vendor/wcesdk/` — the vendored CE 2.12
  SDK: `inc`, `lib/retail`, `tools`, `image` (retail OS modules + config + patched kernels
  `nknodbg.exe`/`nkscifkd.exe` + `.map`/`.pdb`), `image-debug` (checked DLL overlay).
- `reference/MANIFEST.md` — build artifacts + SHA-256 (binaries gitignored).

## Next actions
1. **W5500 sustained receive.** Small TCP works; a large download stalls — the inbound `rx=`
   counter freezes ~6–7 KB in (the W5500 stops delivering frames; `tx` only carries the
   post-timeout FIN). The torn-read guard + RX-backlog/desync `SysLog` were added to tell buffer
   overflow vs a SCIF-read desync apart — read those lines in the System Log app (or the SCIF
   console with `-DKERNEL=debug`). Likely a Flycast W5500-emulation quirk; confirm on real
   hardware, and consider moving the chip to the faster SCI hardware-SPI bus.
2. **IE on hardware.** `iexplore` (`-DHTML=on`) boots + CoCreates on Flycast but is untested
   end-to-end on real hardware — verify the dcgfx page-layer render + the winsock `dcw://` fetch.
3. **Modem on hardware.** PPP delegation works in Flycast; verify a real dial-up modem +
   the revival DNS over PPP on hardware (a `628` PPP-negotiation failure was seen in Flycast).

## Conventions
- **Code style: Microsoft apps-Hungarian + tabs + Allman braces, enforced by `.clang-format`** —
  write new code to match (clang-format isn't on PATH; it's the pip wheel,
  `python -m pip install clang-format`). Hungarian prefixes: n/c, dw/w/b, p/pv/pb/pn, psz/lpsz, h,
  a+elem (arrays), g_/s_/m_ scope; PascalCase functions. **Shared struct members are NOT renamed**
  (DcShared/DcWindow/DcCmd in `dcwin.h`, the microstk ifnet/NDIS ABI, COM interface members) —
  they're ABI/protocol contracts across the shell + every app. `net/lwip-port`,
  `drivers/sdcard/fatfs`, and `shell/mp3impl.cpp`/minimp3 are vendored third-party (left as-is).
- **Git commits: NO `Co-Authored-By` trailer** (user preference); commit straight to `master`.
- Don't commit copyrighted binaries beyond what's already vendored; `reference/*.bin` stays
  ignored, `build/` and `build-html/` are ignored.
- Our SH-4 modules are free (retail) builds; the debug image mixes the patched SCIF kernel +
  checked stock DLLs + our free modules (compatible at the syscall/PSL ABI).

---
> Source: [maximqaxd/wince-dc](https://github.com/maximqaxd/wince-dc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-30 -->
