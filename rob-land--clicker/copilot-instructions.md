## clicker

> A native GNOME remote control application for Roku, Android TV/Google TV/NVIDIA Shield, and Apple TV streaming devices. App ID: `land.rob.clicker`.

# Clicker — CLAUDE.md

## What this project is

A native GNOME remote control application for Roku, Android TV/Google TV/NVIDIA Shield, and Apple TV streaming devices. App ID: `land.rob.clicker`.

## Code quality

A core goal is well-structured, readable code that follows idiomatic Python (PEP 8) and GNOME / libadwaita conventions; the cohort-shared [`STYLE_GUIDE.md`](STYLE_GUIDE.md) layers on top. When existing code doesn't meet that bar, refactor rather than perpetuate the pattern.

## Before making changes

Read [`STYLE_GUIDE.md`](STYLE_GUIDE.md) first when touching any of:

- Meson build files, the Flatpak manifest, or `requirements.txt`
- Anything under `data/ui/` or `data/icons/`
- New top-level Python files, or new modules under `src/<pkg>/`
- Imports — especially `import gi` / `gi.require_version`
- New launcher / `.in` substitution targets

The five-project unification (banter, clicker, finlit, jamjar, tonic)
established conventions that drift easily from intuition. The recurring
slip is reintroducing per-file `gi.require_version` blocks in new
modules; the launcher (`<project>.in`) is the single declaration site.

## Tech stack

- **Language**: Python 3.10+
- **UI toolkit**: GTK4 + libadwaita (PyGObject / GObject Introspection), with Blueprint (`.blp`) UI templates compiled to `.ui` at build time
- **Build system**: Meson + Ninja
- **Packaging**: Flatpak (manifest: `build-aux/flatpak/land.rob.clicker.json`)
- **Async**: `asyncio` background event loop (`src/async_loop.py`) for device communication
- **GNOME Platform**: 50

## Device support

| Device | Protocol | Python package |
|--------|----------|---------------|
| Roku | ECP over HTTP | None (built-in) |
| Android TV / Google TV / NVIDIA Shield | Android TV Remote v2 | `androidtvremote2` |
| Apple TV (untested) | pyatv | `pyatv` |

Discovery: Roku via SSDP multicast; Android TV / Apple TV via mDNS (`zeroconf`).

## Source layout

```
src/
├── main.py                 Entry point
├── application.py          Adw.Application subclass
├── window.py               Main window
├── async_loop.py           Shared asyncio background loop
├── debug.py                Debug logging (--debug flag)
├── const.py.in             Build-time paths (processed by meson → const.py)
├── clicker.in              Launcher script (processed by meson)
├── devices/
│   ├── base.py             DeviceProtocol, StreamingDevice, BaseClient
│   ├── roku.py             Roku ECP client
│   ├── android_tv.py       androidtvremote2 client
│   └── apple_tv.py         pyatv client
├── discovery/
│   ├── ssdp.py             Roku SSDP multicast discovery
│   └── mdns.py             mDNS discovery via python-zeroconf
└── pages/
    ├── discovery_page.py
    ├── remote_page.py
    ├── apps_page.py
    └── dialogs/
        ├── android_tv_pairing.py
        └── apple_tv_pairing.py
data/ui/                    GTK4 UI template files (no custom CSS — Adwaita classes only)
```

## Python dependencies (runtime)

Listed in `requirements.txt`: `androidtvremote2`, `pyatv`, `zeroconf`.

These must be bundled into the Flatpak via `python3-deps.json` (committed — CI builds straight from it). **Regenerate and commit** when requirements change.

## Flatpak build workflow

`python3-deps.json` is multi-arch: each tarball is replaced with one wheel
source per target arch, gated on `only-arches`. A single `flatpak-builder
--arch=…` run picks the matching wheels. Pure-Python wheels are emitted
once, arch-agnostic.

### One-shot build

```bash
./build-all.sh                  # builds clicker-x86_64.flatpak + clicker-aarch64.flatpak
./build-all.sh --arch x86_64    # single arch
./build-all.sh --install        # also installs the host-arch bundle (--user)
./build-all.sh --regen-deps     # regenerate python3-deps.json from requirements.txt first
```

The script handles `fix-flatpak-deps.py` (idempotent), warns if qemu binfmt
isn't registered for cross-arch builds, and writes bundles to the project
root.

### Manual workflow

```bash
# Optional: regenerate python3-deps.json after editing requirements.txt
~/.local/bin/flatpak_pip_generator \
    --runtime='org.gnome.Sdk//50' \
    --requirements-file=requirements.txt \
    --output build-aux/flatpak/python3-deps

# Patch tarballs → wheels for both arches (default)
python3 fix-flatpak-deps.py build-aux/flatpak/python3-deps.json

# Build per arch
flatpak-builder --arch=x86_64  --repo=repo --force-clean _flatpak_x86_64  build-aux/flatpak/land.rob.clicker.json
flatpak-builder --arch=aarch64 --repo=repo --force-clean _flatpak_aarch64 build-aux/flatpak/land.rob.clicker.json

# Bundle
flatpak build-bundle --arch=x86_64  repo clicker-x86_64.flatpak  land.rob.clicker
flatpak build-bundle --arch=aarch64 repo clicker-aarch64.flatpak land.rob.clicker
```

### Phone install (FuriOS / Phosh / postmarketOS)

```bash
scp clicker-aarch64.flatpak <user>@<phone-ip>:~/
ssh <user>@<phone-ip> 'flatpak install --user --noninteractive --bundle clicker-aarch64.flatpak'
```

## Key conventions

- UI is defined in Blueprint files under `data/ui/*.blp`, compiled to `.ui` at build time and bundled via GResource. Keep logic in Python, layout in `.blp`.
- `const.py.in` is processed by Meson at build time to produce `const.py` with real install paths.
- Debug output is gated behind `--debug` flag; see `src/debug.py`.
- GSettings schema: `land.rob.clicker.gschema.xml` — used for persistent last device.
- `fix-flatpak-deps.py` replaces PyPI source tarballs (e.g. `cryptography`) with pre-built wheels so no Rust toolchain is needed inside the Flatpak build sandbox. By default emits wheels for both x86_64 and aarch64 with `only-arches` filters; pass `--arch x86_64` (or `aarch64`) for a single-arch manifest. Idempotent — safe to re-run.

## Things to watch out for

- `python3-deps.json` is generated but committed (CI relies on it) — regenerate when requirements change.
- Apple TV support is included but untested.
- Translations directory (`po/`) exists but is empty — i18n not yet implemented.
- The app requires network socket permissions (`--share=network`) for all device communication.

---
> Source: [rob-land/clicker](https://github.com/rob-land/clicker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
