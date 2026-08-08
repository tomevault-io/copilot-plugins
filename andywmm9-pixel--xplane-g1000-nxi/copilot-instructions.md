## macos-universal-release

> macOS GitHub releases ship both arch folders with mt_cairo (NanoVG is standalone-only)


# macOS release packaging

When cutting or triggering a GitHub Release (tag push, `workflow_dispatch`, or
asking the agent to publish/build a release):

- The **X-Plane plugin must use mt_cairo on macOS for both arches** — never ship
  a NanoVG-only `.xpl` in a macOS release. NanoVG is for the **standalone** app
  only.
- Ship both plugin folders X-Plane probes:
  - `xplane-avionics/mac_arm64/` — thin **arm64** `.xpl` + arm64 Homebrew Cairo
    dylibs (`@loader_path`, via `installer/macos/bundle_cairo_dylibs.sh`).
  - `xplane-avionics/mac_x64/` — thin **x86_64** `.xpl` + Intel Homebrew Cairo
    dylibs (Rosetta `/usr/local` brew on CI), same bundler.
- Standalone `avionics-standalone` stays a **universal** (`x86_64;arm64`)
  NanoVG binary.
- Do not build one fat `.xpl` linked to a single Homebrew prefix — arm64 and
  x86_64 Cairo bottles differ; use split builds + per-folder bundles.
- After staging, verify: thin arches per folder, `@loader_path/libcairo` on
  **both** `mac_arm64` and `mac_x64`, bundled `*.dylib` present, standalone
  still fat (CI "Verify macOS arch slices + Cairo bundle" step).
- Do not "fix" Intel/Rosetta by shipping NanoVG in `mac_x64`.

Local `tools/install-xplane-plugin.sh` builds may stay host-arch-only with
Brew Cairo; releases follow the split above with Cairo on both.

---
> Source: [andywmm9-pixel/xplane-g1000-nxi](https://github.com/andywmm9-pixel/xplane-g1000-nxi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
