## hyprland-rpms

> Session handoff notes for continuing work on this repo.

# AGENTS.md

Session handoff notes for continuing work on this repo.

## Project intent

This repository is a Fedora RPM packaging monorepo for the Hypr ecosystem and
adjacent desktop utilities, intended for COPR SCM builds.

Goals:

- Build/maintain Hypr-related packages for Fedora 43, Fedora 44, and rawhide
- Keep a clean packaging repo (fresh start), not a fork of `solopasha/hyprlandRPM`
- Use COPR SCM package entries per package subdirectory
- Add automation later, after manual/local builds are stable

## Naming decisions (updated)

- Repo name: `hyprland-rpms` (chosen because it stays discoverable while still signaling an RPM packaging repo)
- Alternative considered: `hyprwm-rpms`
- COPR project name can still be shorter (`hyprland`) for discoverability

Why not just `hyprland` for the repo:

- This repo will include multiple packages (`hyprland`, `xdg-desktop-portal-hyprland`, `hyprlock`, `hypridle`, etc.), so a broader name is clearer.

## Reference used

Reference repo (for spec patterns only, not layout/process):

- `https://github.com/solopasha/hyprlandRPM`

What we intentionally avoided from the reference:

- Flat top-level directory sprawl for many unrelated packages
- Shared COPR SRPM logic with package-specific behavior
- Opaque update scripts/policies

## Scaffold created (2026-02-24)

Path:

- `~/Code/hyprland-rpms`

Key files/directories:

- `README.md` (repo overview and naming rationale)
- `.copr/Makefile` (generic `make_srpm` support for COPR SCM)
- `docs/`
  - `docs/copr-setup.md`
  - `docs/packaging-policy.md`
  - `docs/release-process.md`
- `scripts/`
  - `scripts/check-specs.sh`
  - `scripts/new-package.sh`
  - `scripts/bump-version.sh`
  - `scripts/copr-onboard-checklist.sh`
- `.github/workflows/`
  - `spec-lint.yml`
  - `repoclosure.yml`
- `templates/package/` (new-package template)
- `packages/hyprland/` (starter spec + metadata)
- `packages/xdg-desktop-portal-hyprland/` (starter spec + metadata)

## Current status

- Monorepo scaffold is complete and lintable.
- `make list` works.
- `make check-specs` passes (`rpmspec` parse + `rpmlint`).
- `packages/uwsm/` has been added (ported from `solopasha/hyprlandRPM` spec baseline, adapted to this monorepo style).
- `scripts/mock-matrix-build.sh` has been added for local Fedora/arch matrix runs (`chain` or `rebuild` mode), including optional `--addrepo`, `--skip-srpm`, and `--continue-on-failure`.
  - `--addrepo` placeholder expansion now normalizes rawhide to the COPR `fedora-rawhide-$basearch` path instead of the numeric mock `releasever`, so generic COPR URLs work across Fedora 43/44/rawhide.
- Latest COPR aarch64 rollout status (2026-03-02):
  - `hyprwayland-scanner` aarch64 bootstrap build succeeded as `10181653` on `fedora-43-aarch64`, `fedora-44-aarch64`, and `fedora-rawhide-aarch64`.
  - Remaining package chain was queued in order with `--after-build-id` for aarch64-only chroots:
    - `hyprutils` `10181665` -> `awww` `10181694`
  - Final status snapshot (2026-03-02):
    - `10181665`..`10181694`: completed; all intended aarch64 rollout packages succeeded.
    - `hyprland-plugins` was intentionally excluded from the aarch64 `0.54.x` path (legacy package is pinned to Hyprland `0.53.3` ABI family).
    - an intermediate failed `hyprland-plugins` aarch64 test build (`10181692`) was removed from COPR to avoid user confusion.
  - Current arch policy:
    - `aarch64` users track Hyprland `0.54.x` forward (no legacy `hyprland-plugins` publish path).
    - legacy `hyprland-plugins` support is kept for users staying on `hyprland 0.53.3` (x86_64 repo path).
  - Historical local emulated-aarch64 Fedora 44 issue (`qemu` + `g++/cc1plus` SIGSEGV at `aquamarine`) remains a known local-matrix limitation, but COPR native aarch64 builds are now being used for rollout.

Important:

- `packages/hyprland/hyprland.spec` is now published in COPR at `0.54.1` (build `10188633`, Fedora 43/44/rawhide builds passing) with the packaged local dependency stack (`hyprwayland-scanner`, `hyprutils`, `hyprlang`, `hyprcursor`, `hyprgraphics`, `aquamarine`, `hyprwire`, `hyprland-protocols`, `glaze`).
- Local follow-up (2026-03-10): `packages/hyprland/hyprland.spec` was bumped to `0.54.2`; SRPM generation and clean standalone `mock --rebuild` validation passed on Fedora 43/44/rawhide x86_64 against the `mineiro/hyprland` COPR dependency repo, and this step was later superseded by the successful `0.54.3` publish described below.
- Local follow-up (2026-03-28): `packages/hyprland/hyprland.spec` has been bumped again to `0.54.3`, `packages/glaze/glaze.spec` to `7.2.2`, and `packages/wl-vapi-gen/wl-vapi-gen.spec` to `1.1.0`; fresh SRPM generation passes for all three, the Fedora 43/44/rawhide x86_64 local `mock --chain` path (`glaze -> hyprland`) passes against the `mineiro/hyprland` COPR dependency repo, Fedora 43/44/rawhide x86_64 standalone `mock --rebuild` passes for `wl-vapi-gen`, `wl-vapi-gen`'s spec license was corrected to `MIT` to match the upstream `1.1.0` tag contents, and the resulting COPR builds were published successfully as `glaze` `10270247`, `wl-vapi-gen` `10270248`, and `hyprland` `10270249` across Fedora 43/44/rawhide on both x86_64 and aarch64.
- `packages/xdg-desktop-portal-hyprland/xdg-desktop-portal-hyprland.spec` was previously re-verified against the Hyprland `0.53.3` stack; full re-validation against `0.54.1` is pending.
- COPR onboarding is complete in `mineiro/hyprland` (after correcting an initial typoed project name `hyperland`):
  - SCM package entries created for the validated stack plus `uwsm`
  - all target x86_64 builds succeeded on Fedora 43/44/rawhide
- `repoclosure` now passes for Fedora 43/44/rawhide after adding `uwsm` (required by `hyprland-uwsm` subpackage runtime dependency).
- Clean standalone `mock --rebuild` (non-chain) has been re-validated for `hyprland` and `xdg-desktop-portal-hyprland` on Fedora 43/44/rawhide using the `mineiro/hyprland` COPR repos (`mock --addrepo ...`).
- Container smoke-test automation is implemented and validated:
  - `scripts/copr-smoke-tests.sh` (Podman wrapper + in-container checks)
  - `.github/workflows/copr-smoke-tests.yml` (Fedora 43/44/rawhide matrix using `COPR_OWNER`/`COPR_PROJECT`) now runs successfully in GitHub Actions
- Local KVM smoke-test harness is implemented and locally validated on libvirt/KVM:
  - `scripts/kvm-smoke-headless.sh` (Fedora cloud image auto-download/cache + checksum verification, cloud-init + SSH checks, log collection)
  - headless mode passes on Fedora 43/44/rawhide
  - graphical mode passes locally via GDM auto-login into the default Hyprland session (`hyprland.desktop`), with screenshot/log capture and a tty fallback path available for debugging
  - graphical runs now support best-effort virgl/SPICE GL acceleration with automatic fallback to non-accelerated graphics when host EGL/virgl is unavailable
- `packages/hyprlock/` has been added as the next ecosystem package starter (`0.9.2`), adapted from the `solopasha/hyprlandRPM` baseline; local SRPM and clean `mock --rebuild` pass on Fedora 43/44/rawhide using the `mineiro/hyprland` COPR repo for dependencies.
- `packages/hypridle/` has been added as the next ecosystem package starter (`0.1.7`), adapted from the `solopasha/hyprlandRPM` baseline; local SRPM and clean `mock --rebuild` pass on Fedora 43/44/rawhide using the `mineiro/hyprland` COPR repo for dependencies.
- `hypridle` has been onboarded to COPR (`mineiro/hyprland`) and COPR builds are passing on Fedora 43/44/rawhide.
- `packages/hyprtoolkit/` has been added at the latest upstream release (`0.5.3`), locally validated on Fedora 43/44/rawhide, and onboarded to COPR (`mineiro/hyprland`) with successful Fedora 43/44/rawhide builds. It currently carries a small compatibility patch set for the validated `hyprgraphics 0.4.0` stack (in-memory image constructor fallback) plus portability fixes (`NAME_MAX` / `<limits.h>`).
- `packages/hyprpaper/` has been updated back to the latest upstream release (`0.8.3`) and successfully built in COPR (`mineiro/hyprland`) after publishing `hyprtoolkit`; local validation currently includes Fedora 43/44/rawhide `mock --chain` (`hyprtoolkit` -> `hyprpaper`) with clean standalone `mock --rebuild` queued for a later validation pass.
- `packages/hyprpicker/` has been added at the latest upstream release (`0.4.6`), locally validated on Fedora 43/44/rawhide, and onboarded to COPR (`mineiro/hyprland`) with successful Fedora 43/44/rawhide builds. It currently carries a small upstream header-fix patch (`#include <mutex>`) needed on newer Fedora toolchains (Fedora 44/rawhide / GCC 15).
- `packages/hyprsunset/` has been added at the latest upstream release (`0.3.3`), locally validated on Fedora 43/44/rawhide, and onboarded to COPR (`mineiro/hyprland`) with successful Fedora 43/44/rawhide builds.
- `packages/hyprpolkitagent/` has been added at the latest upstream release (`0.1.3`), locally validated on Fedora 43/44/rawhide, and onboarded to COPR (`mineiro/hyprland`) with successful Fedora 43/44/rawhide builds. It has a runtime dependency on `hyprland-qt-support`, so the Qt support trio is now the next packaging focus.
- `packages/hyprland-qt-support/` has been added (`0.1.0`), locally validated via SRPM + clean `mock --rebuild` on Fedora 43/44/rawhide, and onboarded to COPR (`mineiro/hyprland`) with successful Fedora 43/44/rawhide builds; it carries a small upstream CMake patch to fix project version initialization order.
- `packages/hyprqt6engine/` has been added (`0.1.0`), locally validated via SRPM + clean `mock --rebuild` on Fedora 43/44/rawhide, and onboarded to COPR (`mineiro/hyprland`) with successful Fedora 43/44/rawhide builds; it carries a small upstream Qt private-module detection patch for newer Qt6.
- `packages/hyprland-guiutils/` has been added at the latest upstream release (`0.2.1`), locally validated via SRPM + clean `mock --rebuild` on Fedora 43/44/rawhide, and onboarded to COPR (`mineiro/hyprland`) with successful Fedora 43/44/rawhide builds. This is the upstream successor to `hyprland-qtutils`, so `hyprland-qtutils` packaging is being deferred in favor of `hyprland-guiutils`.
- `packages/hyprsysteminfo/` has been added at the latest upstream release (`0.1.3`), locally validated via SRPM + clean `mock --rebuild` on Fedora 43/44/rawhide, and onboarded to COPR (`mineiro/hyprland`) with successful Fedora 43/44/rawhide builds; it carries a small Qt6 Wayland private-module detection patch for newer Qt6.
- `packages/hyprlauncher/` has been added at the latest upstream release (`0.1.5`), locally validated via SRPM + clean `mock --rebuild` on Fedora 43/44/rawhide, and onboarded to COPR (`mineiro/hyprland`) with successful Fedora 43/44/rawhide builds; it carries a small portability patch adding missing POSIX includes (`NAME_MAX`, `read`) required on newer Fedora toolchains.
- `packages/hyprshot/` has been added at the latest upstream release (`1.3.0`, `Gustash/Hyprshot`), locally validated via SRPM + clean `mock --rebuild` on Fedora 43/44/rawhide, and onboarded to COPR (`mineiro/hyprland`) with successful Fedora 43/44/rawhide builds; this is a noarch shell-script package.
- `packages/hyprpwcenter/` has been added at the latest upstream release (`0.1.2`), locally validated via SRPM + clean `mock --rebuild` on Fedora 43/44/rawhide, and onboarded to COPR (`mineiro/hyprland`) with successful Fedora 43/44/rawhide builds.
- `packages/hyprdim/` has been added at the latest upstream release (`3.0.1`, `donovanglover/hyprdim`), locally validated via SRPM + clean `mock --rebuild` on Fedora 43/44/rawhide, and onboarded to COPR (`mineiro/hyprland`) with successful Fedora 43/44/rawhide builds. It uses vendored Rust crates (`Source1`) generated during SRPM creation so builds work offline in mock/COPR.
- `.copr/Makefile` now conditionally supports vendored-source generation for both Rust packages (`%cargo_prep` + `Source1`) and Go packages (`BuildRequires: golang` + `Source1`), including robust `%{_sourcedir}` / `spectool -R` handling for COPR SCM `make_srpm`, so offline-capable vendored SRPMs (for example `hyprdim`, `hyprls`, and `cliphist`) build reliably in COPR.
- `packages/hyprshutdown/` has been added at the latest upstream release (`0.1.0`), locally validated via SRPM + clean `mock --rebuild` on Fedora 43/44/rawhide and onboarded to COPR (`mineiro/hyprland`) with successful Fedora 43/44/rawhide builds.
- `packages/hyprland-plugins/` has been added as a split-plugin bundle package targeting the latest compatible upstream tag for the pinned plugin stack (`v0.53.0`), locally validated via SRPM + clean `mock --rebuild` on Fedora 43/44/rawhide, and onboarded to COPR (`mineiro/hyprland`) with successful Fedora 43/44/rawhide builds. It intentionally version-locks plugin subpackages to `hyprland 0.53.3`; Hyprland `0.54.x` is shipped separately without forcing plugin ABI compatibility yet.
- `packages/waybar/` has been added at the latest upstream release (`0.15.0`, `Alexays/Waybar`) as a stable `waybar` package (not `waybar-git`) to provide a newer build than Fedora's default repo version when needed; local SRPM and clean `mock --rebuild` pass on Fedora 43/44/rawhide, and COPR builds are now passing in `mineiro/hyprland`.
- `packages/awww/` has been added as a wallpaper daemon package for Wayland (Codeberg successor/rename of `swww`), locally validated via SRPM + clean `mock --rebuild` on Fedora 43/44/rawhide and onboarded to COPR (`mineiro/hyprland`) with successful Fedora 43/44/rawhide builds. It remains pinned as of `2026-03-11` to the upstream `main` snapshot (`2c86d41d`, `Version: 0.11.2^git20260212`) because the latest tagged release (`v0.11.2`) still builds the legacy `swww`/`swww-daemon` binaries.
- `packages/swayosd/` has been updated to the latest upstream release (`0.3.1`, `ErikReider/SwayOSD`), locally validated via SRPM + clean `mock --rebuild` on Fedora 43/44/rawhide x86_64, and onboarded to COPR (`mineiro/hyprland`) with successful manual build `10230976` across Fedora 43/44/rawhide on both x86_64 and aarch64. It uses vendored Rust crates (`Source1`) generated during SRPM creation so builds work offline in mock/COPR.
- `packages/hyprls/` has been added at the latest upstream release (`0.13.0`, `hyprland-community/hyprls`), locally validated via SRPM + clean `mock --rebuild` on Fedora 43/44/rawhide x86_64. It uses vendored Go modules (`Source1`) so offline mock builds work, and currently carries a small `go.mod` toolchain compatibility adjustment for Fedora 43's Go 1.25.x.
- `packages/tree-sitter-hyprlang/` has been added at the latest upstream release (`3.1.0`, `tree-sitter-grammars/tree-sitter-hyprlang`), locally validated via SRPM + clean `mock --rebuild` on Fedora 43/44/rawhide x86_64. It follows Fedora tree-sitter packaging macros with source package name `tree-sitter-hyprlang`, producing `libtree-sitter-hyprlang` and `libtree-sitter-hyprlang-devel`.
- `packages/app2unit/` has been added at the latest upstream release (`1.4.0`, `Vladimir-csp/app2unit`), locally validated via SRPM + clean `mock --rebuild` on Fedora 43/44/rawhide x86_64, and onboarded to COPR (`mineiro/hyprland`) with successful first manual build `10204233` across Fedora 43/44/rawhide on both x86_64 and aarch64. It is a noarch POSIX shell launcher package with a generated `scdoc` man page, runtime `Requires: systemd` / `xdg-utils`, and `Recommends: xdg-terminal-exec >= 0.13.0`.
- `packages/swappy/` has been added at the latest upstream release (`1.8.0`, `jtheoof/swappy`), based on Fedora's existing `swappy` packaging and locally validated via SRPM + clean `mock --rebuild` on Fedora 43/44/rawhide x86_64. It updates Fedora's stale `1.5.1` baseline while keeping the spec close enough for a future Fedora dist-git bump, and it is now onboarded to COPR (`mineiro/hyprland`) with successful first manual build `10204851` across Fedora 43/44/rawhide on both x86_64 and aarch64.
- `packages/cliphist/` has been added at the latest upstream release (`0.7.0`, `sentriz/cliphist`), based closely on Fedora's existing `cliphist` packaging and locally validated via SRPM + clean `mock --rebuild` on Fedora 43/44/rawhide x86_64. It is a Go-based Wayland clipboard history manager with runtime `Requires: wl-clipboard` / `xdg-utils`, uses a vendored Go `Source1` tarball so builds stay offline in mock/COPR, and is now onboarded to COPR (`mineiro/hyprland`) with successful first manual build `10204957` across Fedora 43/44/rawhide on both x86_64 and aarch64.
- `packages/departure/` has been added at the latest upstream release (`0.1.0`, `mpalatsi/departure`), locally validated via SRPM + clean `mock --rebuild` on Fedora 43/44/rawhide x86_64, and onboarded to COPR (`mineiro/hyprland`) with successful first manual build `10235517` across Fedora 43/44/rawhide on both x86_64 and aarch64. It is a GTK4/gtk4-layer-shell Rust Wayland logout app and uses a vendored Rust `Source1` tarball so builds stay offline in mock/COPR.
- `packages/satty/` has been added at the latest upstream release (`0.20.1`, `Satty-org/Satty`), imported from the working `~/Code/satty-rpms` package; local SRPM generation and clean Fedora 43/44/rawhide x86_64 `mock --rebuild` all pass, it uses a vendored Rust `Source1` tarball so builds stay offline in mock/COPR, and the container/KVM smoke-test package sets now include `satty`. Migration from the standalone `mineiro/satty` COPR into `mineiro/hyprland` is the next step.
- `packages/material-symbols-fonts/` has been added as a new Google icon-font package at upstream snapshot `20260327` (`Version: 0^git20260327`) from the actively updated `google/material-design-icons` `master` branch; local SRPM generation and clean Fedora 43/44/rawhide x86_64 `mock --rebuild` pass, it uses Fedora font packaging macros (`%fontpkg`) instead of manual font staging, repacks a minimal local `Source0` from the upstream raw files so the SRPM stays small, and it intentionally packages the current `Material Symbols` variable TTF set rather than trying to fake-update the legacy frozen `material-icons-fonts` `4.0.0` line.
- Repo-wide `rpmlint` filtering now suppresses the known false-positive `invalid-url Source1` warning for locally generated vendored `*-vendor.tar.gz` / `*.tar.xz` archives, so `make check-specs` only reports remaining actionable warnings.
- `packages/python-materialyoucolor/` has been added at the latest upstream release (`3.0.2`, `T-Dynamos/materialyoucolor-python`), locally validated via SRPM + clean `mock --rebuild` on Fedora 43/44/rawhide x86_64 plus installed-package smoke tests, and onboarded to COPR (`mineiro/hyprland`) with successful first manual build `10205354` across Fedora 43/44/rawhide x86_64. It packages the upstream pybind11-backed quantization extension so the native Material You quantizer remains available.
- `packages/caelestia-cli/` has been added at the latest upstream release (`1.0.6`, `caelestia-dots/cli`), locally validated via SRPM + Fedora 43/44/rawhide `mock --chain` with `python-materialyoucolor` plus installed-package smoke tests, and onboarded to COPR (`mineiro/hyprland`) with successful first manual build `10205434` across Fedora 43/44/rawhide x86_64. It is a noarch Python CLI package that installs the `caelestia` command, fish completion, and Fedora/COPR runtime recommendations for the broader Caelestia stack.
- `packages/dart-sass/` has been updated to the latest upstream release (`1.98.0`, `sass/dart-sass`). It packages the official standalone Linux x64 release, provides `/usr/bin/sass`, and intentionally `Conflicts: rubygem-sass`. The `1.98.0` bump has been re-validated locally via fresh SRPM generation plus a clean Fedora 43 x86_64 `mock --rebuild`; prior validation for the package family also includes Fedora 44/rawhide x86_64 rebuilds and installed-package compile smoke tests.
- On `2026-03-21`, the AGS/Astal stack was imported from the working `~/Code/ags-rpms` repo into this monorepo so users can install it from the same COPR as the Hyprland packages. New package directories now exist for `gnim`, `astal-io`, `astal3`, `astal4`, `astal-hyprland`, and `aylurs-gtk-shell`; `make check-specs` passes, local SRPM generation passes, and `mock --chain` passes on Fedora 43/44/rawhide x86_64 for the full stack. The Astal libraries were then bumped the same day to the newer upstream `Aylur/astal` snapshot commit `41b5029` (`snapshot_date 20260319`, upstream `main` as of `2026-03-19`), and the same Fedora 43/44/rawhide x86_64 chain remains green. `aylurs-gtk-shell` keeps a small downstream patch set for packaged `gnim`, safer GIR defaults, Fedora `npx`, and `gtk4-layer-shell` soname handling.
- Later on `2026-03-21`, `astal-mpris`, `astal-network`, and `astal-wireplumber` were added as split optional Astal integrations instead of a bundled `astal-libs` package. Local SRPM generation plus standalone Fedora 43/44/rawhide x86_64 `mock --rebuild` all pass for the three new packages. The same day, the shared Go vendoring path in `.copr/Makefile` was aligned with `scripts/prepare-go-vendor.sh`, which fixed the `aylurs-gtk-shell` COPR source-generation mismatch and led to successful retry build `10249186`.
- Later on `2026-03-21`, `astal-apps`, `astal-auth`, `astal-battery`, `astal-bluetooth`, `astal-greet`, `astal-notifd`, and `astal-power-profiles` were added as more split optional Astal integrations. `make check-specs` plus local SRPM generation and standalone Fedora 43/44/rawhide x86_64 `mock --rebuild` all pass for the whole batch.
- Later on `2026-03-21`, the remaining Astal follow-ups were completed by packaging the helper dependencies `appmenu-glib-translator`, `wl-vapi-gen`, and `libcava`, then adding `astal-wl`, `astal-tray`, `astal-river`, and `astal-cava`. `make check-specs` plus local SRPM generation and Fedora 43/44/rawhide x86_64 `mock --chain` all pass for that helper-and-follow-up stack. `libcava` keeps a packaging-side Meson asset staging fix so upstream's relative `.incbin` paths work in out-of-tree mock/COPR builds.
- `templates/package/Makefile` now stages package-local patches into the package `SOURCEDIR` root for local SRPM generation and honors optional `GO_VENDOR_SUBDIR` through `scripts/prepare-go-vendor.sh`, so future packages can generate vendored Go `Source1` archives locally without package-specific shell glue.
- Multi-arch rollout rule learned on `2026-03-11`: when expanding packages to
  new chroots, publish COPR-packaged dependencies first and wait for repo
  metadata to catch up before triggering dependent packages. This applies even
  to `BuildArch: noarch` packages; an initial `caelestia-cli` aarch64 attempt
  failed until `python-materialyoucolor` finished publishing on the aarch64
  chroots.

- TODOs remain for:
  - continue tightening graphical VM assertions/log diagnostics (PipeWire/portal/user-service readiness, etc.) without making the harness flaky
  - re-validate/tune newly added dependency floors and any Fedora version-specific conditionals as packages evolve
  - final packaging polish/review for bundled components (currently `xdg-desktop-portal-hyprland`, `hyprlock`, and `hypridle` carry bundled `sdbus-cpp` declarations)
  - revisit bundling declarations if upstream release contents or build paths change
  - package `@ts-for-gir/cli` (or split an AGS type-generation helper) so `ags init` / `ags types` stop fetching it through `npx` at runtime

## COPR strategy (agreed)

Use one COPR project with multiple SCM package entries.

Per package entry:

- Build source type: `SCM`
- Clone URL: this monorepo URL
- Committish: `main`
- Subdirectory: `packages/<pkgname>`
- Spec file: `<pkgname>.spec`
- Build SRPM with: `make_srpm`

Active project:

- `mineiro/hyprland` (current canonical COPR project; an earlier typoed project `mineiro/hyperland` was used during initial onboarding/testing)

Auto-rebuild/webhooks:

- Enable only after first manual builds succeed
- COPR can auto-rebuild from webhook events, but it does NOT auto-bump spec versions
- Upstream version bump automation should be handled later via GitHub Actions or Packit

## Suggested build/update order for Hypr stack

Start with foundational libraries before `hyprland`:

1. `hyprwayland-scanner`
2. `hyprutils`
3. `hyprlang`
4. `hyprcursor`
5. `hyprgraphics`
6. `aquamarine`
7. `hyprwire`
8. `hyprland-protocols`
9. `glaze` (pin to compatible `7.x` for Hyprland `0.54.x`)
10. `hyprland`
11. `xdg-desktop-portal-hyprland`

## Suggested build/update order for AGS / Astal stack

1. `gnim`
2. `astal-io`
3. `astal3`
4. `astal4`
5. `astal-hyprland`
6. `aylurs-gtk-shell`
7. `astal-apps`
8. `astal-auth`
9. `astal-battery`
10. `astal-bluetooth`
11. `astal-greet`
12. `astal-mpris`
13. `astal-network`
14. `astal-notifd`
15. `astal-power-profiles`
16. `astal-wireplumber`
17. `appmenu-glib-translator`
18. `wl-vapi-gen`
19. `libcava`
20. `astal-wl`
21. `astal-tray`
22. `astal-river`
23. `astal-cava`

Notes:

- Keep `astal-io`, `astal3`, `astal4`, and `astal-hyprland` on the same pinned
  `Aylur/astal` snapshot commit.
- Keep optional Astal integrations split instead of collapsing them into one
  bundle package, so AGS users do not pull `libnm` or `wireplumber` stacks
  unless they actually need those namespaces.
- Keep `appmenu-glib-translator`, `wl-vapi-gen`, and `libcava` packaged
  separately so `astal-tray`, `astal-river`, and `astal-cava` stay unbundled.
- `@ts-for-gir/cli` remains the main follow-up for a fully offline AGS
  bootstrap/type-generation workflow.

Additional runtime/support package now included in COPR:

12. `uwsm` (required so `hyprland-uwsm` is repo-installable and repoclosure passes)

This order is documented in `docs/packaging-policy.md`.

## Package matrix (tracking)

Use this table as the single place to track packaging/build progress across Fedora releases.

Status legend:

- `NS` = not started
- `SC` = scaffolded (directory/spec starter exists)
- `WIP` = actively being fixed
- `SRPM` = SRPM builds locally
- `M43` / `M44` / `MRH` = mock rebuild passes for Fedora 43 / 44 / rawhide
- `COPR` = COPR build passing
- `DONE` = published/stable in COPR for target chroots

Build result legend (per Fedora columns):

- `-` = not attempted
- `ok` = passing
- `fail` = failing
- `n/a` = not targeted

Note:

- `COPR builds` currently reflects the latest published x86_64 status for most packages.
- The aarch64 rollout chain `10181665`..`10181694` completed for the active `0.54.x` package set.
- `hyprland-plugins` remains a legacy `0.53.3` compatibility package and is intentionally excluded from aarch64 rollout builds.

| Package                       | Role                               | Priority | Spec status | F43 mock | F44 mock | Rawhide mock | COPR pkg entry | COPR builds | Notes                                                                                                                                                                                                                                                                                                                                                                                                      |
| ----------------------------- | ---------------------------------- | -------: | ----------- | -------- | -------- | ------------ | -------------- | ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `hyprwayland-scanner`         | core toolchain                     |        1 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | local SRPM builds (`0.4.5`) and mock rebuilds pass on Fedora 43/44/rawhide; COPR SCM entry/builds passing in `mineiro/hyprland`                                                                                                                                                                                                                                                                            |
| `hyprutils`                   | core library                       |        2 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | local SRPM builds (`0.11.0`); Fedora 43/44/rawhide builds revalidated via mock chain while testing `hyprland`/`hyprwire`; COPR builds passing                                                                                                                                                                                                                                                              |
| `hyprlang`                    | core library                       |        3 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | local SRPM builds (`0.6.8`) and mock rebuilds pass on Fedora 43/44/rawhide; rebuilt in COPR against `hyprutils` `0.11.0` ABI                                                                                                                                                                                                                                                                               |
| `hyprcursor`                  | core library                       |        4 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | local SRPM builds (`0.1.13`) and mock rebuilds pass on Fedora 43/44/rawhide; COPR builds passing                                                                                                                                                                                                                                                                                                           |
| `hyprgraphics`                | core library                       |        5 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | Fedora 43/44/rawhide mock chain (`hyprutils -> hyprlang -> hyprgraphics`) passes; clean F43 mock against distro `hyprutils` had previously failed due older dependency version                                                                                                                                                                                                                             |
| `aquamarine`                  | core library                       |        6 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | Fedora 43/44/rawhide mock chain (`hyprwayland-scanner -> hyprutils -> aquamarine`) passes; clean F43 mock had previously failed on unavailable BuildRequires in distro repos                                                                                                                                                                                                                               |
| `hyprwire`                    | core library/tooling               |        7 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | local SRPM builds (`0.3.0`) and Fedora 43/44/rawhide mock chain rebuilds pass; COPR build verified requiring `libhyprutils.so.10`                                                                                                                                                                                                                                                                          |
| `hyprland-protocols`          | protocol definitions               |        8 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | local SRPM builds (`0.7.0`) and Fedora 43/44/rawhide mock chain rebuilds pass; COPR builds passing                                                                                                                                                                                                                                                                                                         |
| `glaze`                       | compatibility dependency           |        9 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | local SRPM builds (`7.2.2`); pinned to the upstream `7.x` series for Hyprland `0.54.x` compatibility; Fedora 43/44/rawhide x86_64 local `mock --chain` passes as the first stage of the `glaze -> hyprland` validation path, and COPR build `10270247` published `7.2.2` successfully across Fedora 43/44/rawhide on both x86_64 and aarch64                                                                                                                 |
| `hyprland`                    | compositor                         |       10 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | local SRPM builds (`0.54.3`) and Fedora 43/44/rawhide x86_64 local `mock --chain` passes with `glaze 7.2.2` via the `mineiro/hyprland` COPR dependency repo; COPR build `10270249` published `0.54.3` successfully across Fedora 43/44/rawhide on both x86_64 and aarch64; package output includes `hyprpm`/`start-hyprland`/`hyprland-uwsm`                                                                                                           |
| `xdg-desktop-portal-hyprland` | portal backend                     |       11 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | local SRPM builds (`1.3.11`); full mock chain pass on Fedora 43/44/rawhide; clean standalone `mock --rebuild` also revalidated on Fedora 43/44/rawhide via `mineiro/hyprland`; includes `pkgconfig(libspa-0.2)` and bundled `sdbus-cpp` declaration                                                                                                                                                        |
| `uwsm`                        | session manager/runtime dependency |       12 | `COPR`      | `-`      | `-`      | `-`          | `yes`          | `ok`        | added to satisfy `hyprland-uwsm` runtime dependency; COPR builds pass on Fedora 43/44/rawhide; repoclosure passes after publishing                                                                                                                                                                                                                                                                         |
| `hyprlock`                    | ecosystem app                      |       13 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | starter spec added (`0.9.2`), includes documented temporary bundled `sdbus-cpp`; local SRPM and clean `mock --rebuild` pass on Fedora 43/44/rawhide via `mineiro/hyprland` COPR repo deps; COPR builds passing in `mineiro/hyprland`                                                                                                                                                                       |
| `hypridle`                    | ecosystem app                      |       14 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | starter spec added (`0.1.7`), includes documented temporary bundled `sdbus-cpp`; local SRPM and clean `mock --rebuild` pass on Fedora 43/44/rawhide via `mineiro/hyprland` COPR repo deps; COPR builds passing in `mineiro/hyprland`                                                                                                                                                                       |
| `hyprtoolkit`                 | ecosystem library/toolkit          |       15 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | latest upstream `0.5.3`; local SRPM + clean `mock --rebuild` pass on Fedora 43/44/rawhide via `mineiro/hyprland` COPR repo deps; COPR builds passing; carries temporary compatibility patch for current `hyprgraphics 0.4.0` stack and portability fixes (`NAME_MAX`, missing includes)                                                                                                                    |
| `hyprpaper`                   | ecosystem app                      |       16 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | latest upstream `0.8.3`; local SRPM + `mock --chain` (local `hyprtoolkit` + `hyprpaper`) pass on Fedora 43/44/rawhide via `mineiro/hyprland` COPR repo deps; COPR builds passing after `hyprtoolkit` publish                                                                                                                                                                                               |
| `hyprpicker`                  | ecosystem app                      |       17 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | latest upstream `0.4.6`; local SRPM + clean `mock --rebuild` pass on Fedora 43/44/rawhide via `mineiro/hyprland` COPR repo deps; COPR builds passing; carries temporary upstream header-fix patch (`#include <mutex>`) for Fedora 44/rawhide/GCC 15 builds                                                                                                                                                 |
| `hyprsunset`                  | ecosystem app                      |       18 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | latest upstream `0.3.3`; local SRPM + clean `mock --rebuild` pass on Fedora 43/44/rawhide via `mineiro/hyprland` COPR repo deps; COPR builds passing                                                                                                                                                                                                                                                       |
| `hyprpolkitagent`             | ecosystem app                      |       19 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | latest upstream `0.1.3`; local SRPM + clean `mock --rebuild` pass on Fedora 43/44/rawhide via `mineiro/hyprland` COPR repo deps; COPR builds passing; runtime `Requires: hyprland-qt-support`                                                                                                                                                                                                              |
| `hyprland-qt-support`         | Qt support stack                   |       20 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | version `0.1.0`; local SRPM + clean `mock --rebuild` pass on Fedora 43/44/rawhide via `mineiro/hyprland` COPR repo deps; COPR builds passing; carries temporary CMake patch for project/version initialization order                                                                                                                                                                                       |
| `hyprqt6engine`               | Qt support stack                   |       21 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | version `0.1.0`; local SRPM + clean `mock --rebuild` pass on Fedora 43/44/rawhide via `mineiro/hyprland` COPR repo deps; COPR builds passing; carries temporary Qt private-module detection patch for newer Qt6                                                                                                                                                                                            |
| `hyprland-guiutils`           | ecosystem app (GUI utils)          |       22 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | latest upstream `0.2.1`; local SRPM + clean `mock --rebuild` pass on Fedora 43/44/rawhide via `mineiro/hyprland` COPR repo deps; COPR builds passing; upstream successor to `hyprland-qtutils`                                                                                                                                                                                                             |
| `hyprsysteminfo`              | ecosystem app                      |       23 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | latest upstream `0.1.3`; local SRPM + clean `mock --rebuild` pass on Fedora 43/44/rawhide via `mineiro/hyprland` COPR repo deps; COPR builds passing; carries temporary Qt private-module detection patch for newer Qt6                                                                                                                                                                                    |
| `hyprlauncher`                | ecosystem app                      |       24 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | latest upstream `0.1.5`; local SRPM + clean `mock --rebuild` pass on Fedora 43/44/rawhide via `mineiro/hyprland` COPR repo deps; COPR builds passing; carries temporary portability patch for missing POSIX includes (`NAME_MAX`, `read`) on newer Fedora toolchains                                                                                                                                       |
| `hyprshot`                    | ecosystem app                      |       25 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | latest upstream `1.3.0` from `Gustash/Hyprshot`; local SRPM + clean `mock --rebuild` pass on Fedora 43/44/rawhide via `mineiro/hyprland` COPR repo deps; COPR builds passing; noarch shell-script package                                                                                                                                                                                                  |
| `hyprpwcenter`                | ecosystem app                      |       26 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | latest upstream `0.1.2`; local SRPM + clean `mock --rebuild` pass on Fedora 43/44/rawhide via `mineiro/hyprland` COPR repo deps; COPR builds passing                                                                                                                                                                                                                                                       |
| `hyprdim`                     | ecosystem app (Rust)               |       27 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | latest upstream `3.0.1` from `donovanglover/hyprdim`; local SRPM + clean `mock --rebuild` pass on Fedora 43/44/rawhide; COPR builds passing; uses vendored Rust `Source1` tarball generated during SRPM creation for offline mock/COPR builds                                                                                                                                                              |
| `hyprshutdown`                | ecosystem app                      |       28 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | latest upstream `0.1.0`; local SRPM + clean `mock --rebuild` pass on Fedora 43/44/rawhide via `mineiro/hyprland` COPR repo deps; COPR builds passing; depends on `hyprtoolkit`, `hyprutils`, `glaze`, `pixman`, and `libdrm`                                                                                                                                                                               |
| `hyprland-plugins`            | ecosystem plugins                  |       29 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | official Hyprland plugins bundle; latest compatible upstream tag for pinned plugin stack is `v0.53.0` (repo tags are version-family aligned); packaged as split plugin subpackages with strict runtime `Requires: hyprland = 0.53.3`; intentionally kept as a legacy compatibility path and excluded from the aarch64 `0.54.x` rollout                                                                     |
| `waybar`                      | desktop bar (Wayland/Hyprland)     |       30 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | latest upstream stable `0.15.0` from `Alexays/Waybar`; packaged as stable `waybar` (not `waybar-git`) to provide a newer build than Fedora default repos; local SRPM + clean `mock --rebuild` pass on Fedora 43/44/rawhide; COPR builds passing                                                                                                                                                            |
| `awww`                        | wallpaper daemon (Wayland)         |       31 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | Codeberg successor/rename of `swww`; re-checked on `2026-03-11`, and the latest tagged release `v0.11.2` still pre-dates the rename and builds `swww`, so this package remains pinned to upstream `main` snapshot commit `2c86d41d` (`Version: 0.11.2^git20260212`) to provide renamed `awww`/`awww-daemon` binaries; local SRPM + clean `mock --rebuild` pass on Fedora 43/44/rawhide with vendored Rust `Source1` tarball; COPR builds passing |
| `swayosd`                     | OSD service (Wayland)              |       32 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | latest upstream `0.3.1` from `ErikReider/SwayOSD`; local SRPM + clean `mock --rebuild` pass on Fedora 43/44/rawhide x86_64; uses vendored Rust `Source1` tarball generated during SRPM creation for offline mock/COPR builds; onboarded to COPR with successful manual build `10230976` across Fedora 43/44/rawhide on both x86_64 and aarch64                                                                                                        |
| `hyprls`                      | language server (Hyprland config)  |       33 | `MRH`       | `ok`     | `ok`     | `ok`         | `no`           | `-`         | latest upstream `0.13.0` from `hyprland-community/hyprls`; local SRPM + clean `mock --rebuild` pass on Fedora 43/44/rawhide x86_64; uses vendored Go `Source1` tarball for offline mock builds and a temporary `go.mod` compatibility adjustment for Fedora 43 Go 1.25.x                                                                                                                                   |
| `tree-sitter-hyprlang`        | tree-sitter grammar (Hyprlang)     |       34 | `MRH`       | `ok`     | `ok`     | `ok`         | `no`           | `-`         | latest upstream `3.1.0` from `tree-sitter-grammars/tree-sitter-hyprlang`; local SRPM + clean `mock --rebuild` pass on Fedora 43/44/rawhide x86_64; packaged with Fedora `BuildSystem: tree_sitter`, producing `libtree-sitter-hyprlang` and `libtree-sitter-hyprlang-devel`                                                                                                                                |
| `app2unit`                    | desktop-entry launcher helper      |       35 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | latest upstream `1.4.0` from `Vladimir-csp/app2unit`; local SRPM + clean `mock --rebuild` pass on Fedora 43/44/rawhide x86_64; onboarded to COPR with successful first manual build `10204233` across Fedora 43/44/rawhide on both x86_64 and aarch64; noarch POSIX shell package with generated `scdoc` man page and helper `app2unit-open*` / `app2unit-term*` links                                  |
| `swappy`                      | snapshot editor (Wayland)          |       36 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | latest upstream `1.8.0` from `jtheoof/swappy`; based on Fedora's existing `swappy` packaging and locally validated via SRPM + clean `mock --rebuild` on Fedora 43/44/rawhide x86_64; updates Fedora's official `1.5.1` baseline while keeping the spec close to a future Fedora dist-git bump; onboarded to COPR with successful first manual build `10204851` across Fedora 43/44/rawhide on both x86_64 and aarch64 |
| `cliphist`                    | clipboard history manager          |       37 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | latest upstream `0.7.0` from `sentriz/cliphist`; based closely on Fedora's existing `cliphist` packaging and locally validated via SRPM + clean `mock --rebuild` on Fedora 43/44/rawhide x86_64; Go-based Wayland clipboard history manager with runtime `Requires: wl-clipboard` / `xdg-utils`; uses vendored Go `Source1` tarball for offline mock/COPR builds; onboarded to COPR with successful first manual build `10204957` across Fedora 43/44/rawhide on both x86_64 and aarch64 |
| `departure`                   | logout app (Wayland)               |       38 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | latest upstream `0.1.0` from `mpalatsi/departure`; GTK4/gtk4-layer-shell Rust logout app; local SRPM + clean `mock --rebuild` pass on Fedora 43/44/rawhide x86_64; uses vendored Rust `Source1` tarball for offline mock/COPR builds; onboarded to COPR with successful first manual build `10235517` across Fedora 43/44/rawhide on both x86_64 and aarch64 |
| `gnim`                        | AGS JS runtime library             |       39 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | upstream release `1.9.0` from `Aylur/gnim`, packaged from the published npm tarball and installed under `/usr/share/ags/js/node_modules/gnim`; local SRPM + Fedora 43/44/rawhide x86_64 `mock --chain` pass with the imported Astal/AGS stack; COPR build `10249144` succeeded                                                                                                                             |
| `astal-io`                    | AGS / Astal core library           |       40 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | pinned `Aylur/astal` snapshot commit `41b5029` (`snapshot_date 20260319`); local SRPM + Fedora 43/44/rawhide x86_64 `mock --chain` pass; ships `AstalIO-0.1.gir` with the typelib for AGS type generation; COPR build `10249145` succeeded                                                                                                                      |
| `astal3`                      | Astal GTK3 widget library          |       41 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | pinned `Aylur/astal` snapshot commit `41b5029` (`snapshot_date 20260319`); local SRPM + Fedora 43/44/rawhide x86_64 `mock --chain` pass; build/runtime stack depends on `astal-io`; COPR build `10249146` succeeded                                                                                                                                            |
| `astal4`                      | Astal GTK4 widget library          |       42 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | pinned `Aylur/astal` snapshot commit `41b5029` (`snapshot_date 20260319`); local SRPM + Fedora 43/44/rawhide x86_64 `mock --chain` pass; build/runtime stack depends on `astal-io`; COPR build `10249147` succeeded                                                                                                                                            |
| `astal-hyprland`              | Astal Hyprland IPC binding         |       43 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | pinned `Aylur/astal` snapshot commit `41b5029` (`snapshot_date 20260319`); optional Hyprland-aware binding for AGS widgets; local SRPM + Fedora 43/44/rawhide x86_64 `mock --chain` pass; COPR build `10249148` succeeded                                                                                                                                    |
| `astal-apps`                  | Astal applications binding         |       44 | `MRH`       | `ok`     | `ok`     | `ok`         | `no`           | `-`         | pinned `Aylur/astal` snapshot commit `41b5029` (`snapshot_date 20260319`); optional applications integration library plus `astal-apps` CLI; local SRPM + standalone Fedora 43/44/rawhide x86_64 `mock --rebuild` pass                                                                                                      |
| `astal-auth`                  | Astal authentication binding       |       45 | `MRH`       | `ok`     | `ok`     | `ok`         | `no`           | `-`         | pinned `Aylur/astal` snapshot commit `41b5029` (`snapshot_date 20260319`); optional authentication/PAM integration library plus `astal-auth` CLI and PAM service; local SRPM + standalone Fedora 43/44/rawhide x86_64 `mock --rebuild` pass                                                                                 |
| `astal-battery`               | Astal battery binding              |       46 | `MRH`       | `ok`     | `ok`     | `ok`         | `no`           | `-`         | pinned `Aylur/astal` snapshot commit `41b5029` (`snapshot_date 20260319`); optional UPower battery integration library plus `astal-battery` CLI; local SRPM + standalone Fedora 43/44/rawhide x86_64 `mock --rebuild` pass                                                                                              |
| `astal-bluetooth`             | Astal Bluetooth binding            |       47 | `MRH`       | `ok`     | `ok`     | `ok`         | `no`           | `-`         | pinned `Aylur/astal` snapshot commit `41b5029` (`snapshot_date 20260319`); optional BlueZ integration library; local SRPM + standalone Fedora 43/44/rawhide x86_64 `mock --rebuild` pass                                                                                            |
| `astal-greet`                 | Astal greetd binding               |       48 | `MRH`       | `ok`     | `ok`     | `ok`         | `no`           | `-`         | pinned `Aylur/astal` snapshot commit `41b5029` (`snapshot_date 20260319`); optional greetd integration library plus `astal-greet` CLI; local SRPM + standalone Fedora 43/44/rawhide x86_64 `mock --rebuild` pass                                                                   |
| `astal-mpris`                 | Astal MPRIS binding                |       49 | `MRH`       | `ok`     | `ok`     | `ok`         | `no`           | `-`         | pinned `Aylur/astal` snapshot commit `41b5029` (`snapshot_date 20260319`); optional media-player integration library plus `astal-mpris` CLI; local SRPM + standalone Fedora 43/44/rawhide x86_64 `mock --rebuild` pass                                                                                                       |
| `astal-network`               | Astal NetworkManager binding       |       50 | `MRH`       | `ok`     | `ok`     | `ok`         | `no`           | `-`         | pinned `Aylur/astal` snapshot commit `41b5029` (`snapshot_date 20260319`); optional `libnm` integration for wifi and wired widgets; local SRPM + standalone Fedora 43/44/rawhide x86_64 `mock --rebuild` pass                                                                                                               |
| `astal-notifd`                | Astal notification-daemon binding  |       51 | `MRH`       | `ok`     | `ok`     | `ok`         | `no`           | `-`         | pinned `Aylur/astal` snapshot commit `41b5029` (`snapshot_date 20260319`); optional notification-daemon integration library plus `astal-notifd` CLI and GSettings schema; local SRPM + standalone Fedora 43/44/rawhide x86_64 `mock --rebuild` pass                               |
| `astal-power-profiles`        | Astal power-profiles binding       |       52 | `MRH`       | `ok`     | `ok`     | `ok`         | `no`           | `-`         | pinned `Aylur/astal` snapshot commit `41b5029` (`snapshot_date 20260319`); optional power-profiles integration library plus `astal-power-profiles` CLI; local SRPM + standalone Fedora 43/44/rawhide x86_64 `mock --rebuild` pass                                                   |
| `astal-wireplumber`           | Astal WirePlumber binding          |       53 | `MRH`       | `ok`     | `ok`     | `ok`         | `no`           | `-`         | pinned `Aylur/astal` snapshot commit `41b5029` (`snapshot_date 20260319`); optional PipeWire/WirePlumber integration using the `AstalWp` namespace; local SRPM + standalone Fedora 43/44/rawhide x86_64 `mock --rebuild` pass                                                                                                |
| `aylurs-gtk-shell`            | AGS scaffolding CLI                |       54 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | upstream release `v3.1.1`; local SRPM + Fedora 43/44/rawhide x86_64 `mock --chain` pass; RPM name avoids Fedora's unrelated `ags`, requires packaged `gnim` / `astal3` / `astal4` plus `/usr/bin/sass`, and carries a small downstream patch set for packaged `gnim`, GIR defaults, Fedora `npx`, and `gtk4-layer-shell` soname handling; COPR retry build `10249186` succeeded after Go vendor flow fixes |
| `appmenu-glib-translator`     | AppMenu DBusMenu translator        |       55 | `MRH`       | `ok`     | `ok`     | `ok`         | `no`           | `-`         | upstream tag `25.04` from `vala-panel-appmenu`; installs the upstream `AppmenuGLibTranslator-24.02` GIR/typelib namespace and local SRPM + Fedora 43/44/rawhide x86_64 `mock --chain` pass; helper dependency for `astal-tray`                                                                                               |
| `wl-vapi-gen`                 | Wayland VAPI generator             |       56 | `COPR`      | `ok`     | `ok`     | `ok`         | `yes`          | `ok`        | upstream tag `1.1.0`; noarch helper that generates Vala VAPI bindings from Wayland protocol XML and is used by `astal-wl` / `astal-river`; local SRPM + Fedora 43/44/rawhide x86_64 standalone `mock --rebuild` pass, the spec license now matches upstream's `MIT` license, and COPR build `10270248` published successfully across Fedora 43/44/rawhide on both x86_64 and aarch64                                                            |
| `libcava`                     | CAVA shared library                |       57 | `MRH`       | `ok`     | `ok`     | `ok`         | `no`           | `-`         | upstream tag `0.10.7`; builds the shared library only with optional backends disabled; packaging stages upstream embedded config/shader assets into the Meson build dir so out-of-tree builds succeed; local SRPM + Fedora 43/44/rawhide x86_64 `mock --chain` pass                                                        |
| `astal-wl`                    | Astal generic Wayland helper       |       58 | `MRH`       | `ok`     | `ok`     | `ok`         | `no`           | `-`         | pinned `Aylur/astal` snapshot commit `41b5029` (`snapshot_date 20260319`); generic Wayland helper library for Astal-based apps, built with `wl-vapi-gen`; local SRPM + Fedora 43/44/rawhide x86_64 `mock --chain` pass                                                             |
| `astal-tray`                  | Astal status notifier binding      |       59 | `MRH`       | `ok`     | `ok`     | `ok`         | `no`           | `-`         | pinned `Aylur/astal` snapshot commit `41b5029` (`snapshot_date 20260319`); optional tray library plus `astal-tray` CLI, backed by packaged `appmenu-glib-translator`; local SRPM + Fedora 43/44/rawhide x86_64 `mock --chain` pass                                              |
| `astal-river`                 | Astal River compositor binding     |       60 | `MRH`       | `ok`     | `ok`     | `ok`         | `no`           | `-`         | pinned `Aylur/astal` snapshot commit `41b5029` (`snapshot_date 20260319`); optional River WM binding built on `astal-wl` and `wl-vapi-gen`; local SRPM + Fedora 43/44/rawhide x86_64 `mock --chain` pass                                                                        |
| `astal-cava`                  | Astal CAVA binding                 |       61 | `MRH`       | `ok`     | `ok`     | `ok`         | `no`           | `-`         | pinned `Aylur/astal` snapshot commit `41b5029` (`snapshot_date 20260319`); optional CAVA visualization binding using the packaged system `libcava`; local SRPM + Fedora 43/44/rawhide x86_64 `mock --chain` pass                                                                 |
| `satty`                       | screenshot annotator (Wayland)     |       62 | `MRH`       | `ok`     | `ok`     | `ok`         | `no`           | `-`         | latest upstream `0.20.1` from `Satty-org/Satty`; imported from the standalone `~/Code/satty-rpms` packaging repo; local SRPM + clean `mock --rebuild` pass on Fedora 43/44/rawhide x86_64; uses vendored Rust `Source1` tarball generation for offline mock/COPR builds; smoke-test package sets now include it while migration into `mineiro/hyprland` is pending |
| `material-symbols-fonts`      | icon fonts (Google Material)       |       63 | `MRH`       | `ok`     | `ok`     | `ok`         | `no`           | `-`         | upstream snapshot `20260327` (`Version: 0^git20260327`) from the actively updated `google/material-design-icons` `master` branch; packages the current Material Symbols variable TTF families (`Outlined`, `Rounded`, `Sharp`) with Fedora font macros and a minimized repacked `Source0`, while the legacy `material-icons-fonts` package remains on the frozen classic `4.0.0` line |

Recommended usage:

1. Update `Spec status` whenever a package moves stages (`NS` -> `SC` -> `WIP` -> `SRPM` ...)
2. Record `F43/F44/Rawhide mock` results immediately after each `mock --rebuild`
3. Flip `COPR pkg entry` to `yes` only after the SCM package entry is created
4. Mark `COPR builds` as `ok/fail` per latest build result and keep failure reasons in `Notes`

## Validation strategy (updated)

Use a staged validation approach instead of a single "smoke test":

1. `CI container smoke` (fast, blocking)
   - Validate COPR repo metadata, dependency resolution, package installability, file placement, and basic CLI presence in clean Fedora 43/44/rawhide containers.
   - Primary tooling: `scripts/copr-smoke-tests.sh` and `.github/workflows/copr-smoke-tests.yml`.
2. `Local KVM headless` (slower, realistic system integration)
   - Validate a full Fedora VM with `systemd`/`logind`, package installation, user-unit file presence, and log collection (`journalctl`) for failures.
   - Primary tooling: `scripts/kvm-smoke-headless.sh` (implemented harness with Fedora cloud image auto-download/cache support).
3. `Local KVM graphical` (manual first, later automated)
   - Validate actual Hyprland session startup/runtime behavior (preferably libvirt/KVM first, then optionally real hardware/self-hosted systems for deeper confidence).
   - Current default path uses `gdm` auto-login into the default Hyprland session (`hyprland.desktop`) to exercise a distro-like user experience; tty-based graphical smoke remains available as a fallback/debug path.
   - Treat this as a separate stage from packaging/install smoke tests.

## Suggested next steps (carry-over)

1. Keep `hyprland-plugins` excluded from the aarch64 `0.54.x` rollout path unless/until upstream publishes a compatible plugin family.
2. Keep the CI container smoke workflow green and tune assertions conservatively when package outputs evolve.
3. Continue hardening the local KVM graphical smoke stage (service diagnostics, optional acceleration controls, clearer failure artifacts) while keeping it reliable on non-virgl hosts.
4. Re-run `repoclosure` after the aarch64 rollout completes and verify the repo closes cleanly across Fedora 43/44/rawhide for both x86_64 and aarch64.
5. Review bundling/unbundling options for `xdg-desktop-portal-hyprland`, `hyprlock`, and `hypridle` (`sdbus-cpp`) and document any policy changes in spec comments/docs.
6. Re-run `repoclosure` and clean standalone `mock --rebuild` for the latest `hyprpaper` (`0.8.3`) after batching a few more ecosystem packages (if desired).
7. Decide whether/when to broaden user-facing install recommendations from `hyprpaper` to `awww`, and update smoke tests/docs accordingly.
8. Decide when to enable COPR webhooks/auto-rebuilds, then add upstream version bump automation only after the manual workflow (including smoke tests) is stable.
9. Onboard `hyprls` and `tree-sitter-hyprlang` to COPR (`mineiro/hyprland`) and re-run `repoclosure` after publish.
10. Decide whether any user-facing meta package or install recommendations should pull in the new optional Astal libraries for a smoother desktop-shell experience.
11. Package `@ts-for-gir/cli` (or split an AGS type-generation helper package) so `ags init` / `ags types` no longer depend on runtime `npx` network fetches.

## Working conventions for future edits

- Keep one package per `packages/<name>/`
- Prefer stable specs first; add `-git` variants later if needed
- Keep package-specific patches in `patches/`
- Use `%autorelease` + `%autochangelog` unless there is a strong reason not to
- Validate with `make check-specs` before pushing
- Test SRPM + `mock` builds before enabling COPR webhooks

## Commands used in this repo

List packages:

```bash
make list
```

Check specs:

```bash
make check-specs
```

Build SRPM for one package:

```bash
make srpm PACKAGE=xdg-desktop-portal-hyprland
```

Run the Fedora/arch mock matrix (default: 43/44/rawhide x x86_64/aarch64):

```bash
./scripts/mock-matrix-build.sh --all-packages
```

Run a focused matrix for the core chain only:

```bash
./scripts/mock-matrix-build.sh hyprwayland-scanner hyprutils hyprlang hyprcursor hyprgraphics aquamarine hyprwire hyprland-protocols glaze hyprland xdg-desktop-portal-hyprland uwsm
```

Scaffold a new package:

```bash
./scripts/new-package.sh hyprlock https://github.com/hyprwm/hyprlock.git https://github.com/hyprwm/hyprlock/releases
```

Run container smoke tests against COPR (Fedora 43/44/rawhide):

```bash
./scripts/copr-smoke-tests.sh mineiro hyprland
```

Run local KVM smoke test (Fedora cloud image auto-download/cache supported):

```bash
./scripts/kvm-smoke-headless.sh --release 44
```

Run local KVM graphical smoke test (GDM + default Hyprland session):

```bash
./scripts/kvm-smoke-headless.sh --smoke-mode graphical --release 43 --keep-vm
```

Skip the virgl/SPICE GL acceleration probe on hosts where it is known to fail:

```bash
./scripts/kvm-smoke-headless.sh --smoke-mode graphical --graphics-accel off --release 43
```

## Notes for continuation

When resuming, start by reading:

1. `README.md`
2. `docs/packaging-policy.md`
3. `packages/hyprland/hyprland.spec`
4. This `AGENTS.md`

Primary near-term task:

- Keep the container smoke-test path (`scripts/copr-smoke-tests.sh` + CI workflow) as the fast baseline, continue hardening the local libvirt/KVM graphical smoke stage (GDM/default-session path), and then shift focus to packaging-policy cleanup plus the next ecosystem packages.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mineiro)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mineiro)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
