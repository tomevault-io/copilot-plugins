## webflash

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

WebFlash is a static, browser-based firmware installer for Sense360 ESP32 hubs. The site is a single page that drives ESP Web Tools via Web Serial; there is no application server and no bundler. It is published to GitHub Pages from the repository root.

The codebase has two halves that meet at `manifest.json` — a **publishing pipeline** (Python + GitHub Actions) and a **wizard frontend** (vanilla ES modules). The human-facing explanation of the two-halves model, the `manifest.json` boundary, the desktop-only constraint, the ESP Web Tools standard, the cross-repo contract with `sense360store/esphome-public`, and the deploy gate lives in **[`docs/architecture.md`](docs/architecture.md)** — keep it as the canonical narrative and update it (not this file) when the architecture changes. This file keeps only the AI-actionable constraints; the per-slice change records (`docs/conventions-history.md`) are archived under `DOCS-DISPOSITION-001` — see [`docs/archive-index.md`](docs/archive-index.md).

WebFlash is **downstream** of `sense360store/esphome-public`, which publishes **unsigned** `.bin` artifacts + checksums (WebFlash is the production signing / manifest authority). The cross-repo boundary is exactly three stable surfaces — release **tags**, **config-string** values, and **artifact names**; upstream's internal board/bundle/alias/shim YAML layering is invisible to WebFlash, and no WebFlash file references any upstream `packages/` or `products/` path. Do not couple WebFlash to upstream internals — keep `firmware/sources.json`, `manifest.json`, and `scripts/data/` pinned to config strings and artifact names only. See [`docs/architecture.md`](docs/architecture.md) → *Cross-repo contract* for the full narrative; the `WEBFLASH-ARCH-SYNC-001` no-drift re-audit record (`docs/product-import-readiness.md`) is archived — see [`docs/archive-index.md`](docs/archive-index.md).

### Platform and standards

- **Follows the ESP Web Tools / esptool.js standard** for flashing ESP32 devices. The install view renders the upstream `<esp-web-install-button>` component (loaded from unpkg) and consumes the standard ESP Web Tools manifest schema (`name`, `version`, `builds[].chipFamily`, `builds[].parts[].path`/`offset`, `improv`, etc.). Do not invent custom flash flows — the upstream component drives connect/erase/write/verify.
- **Laptop / desktop only.** Web Serial is not available on iOS, Android Chrome, or any mobile browser, so WebFlash explicitly targets desktop Chromium-based browsers (Chrome, Edge, Opera) on Windows / macOS / Linux. Firefox and Safari are unsupported. Capability detection lives in `scripts/capabilities.js` (reached through the `engine.capabilities` facade) and the install view (`scripts/install.js`) surfaces the unsupported-browser banner. Do not add mobile-first layout assumptions or features that imply mobile is a supported runtime — the install path will not work there.

## Cross-repository operating model

Before starting any cross-repository work, Claude Code must read the SOT operating model: <https://github.com/sense360store/SOT/blob/main/CLAUDE-OPERATING-MODEL.md>. In brief:

- **SOT owns programme-level truth**: accepted cross-repository decisions, programme IDs, cross-repository status, and owner actions.
- **WebFlash owns distribution**: browser flashing, firmware distribution, manifests, binary metadata, checksums, signatures, install gates, release channels, installer copy, and distribution execution records.
- WebFlash must not claim firmware behaviour that is not proven by `sense360store/esphome-public`.
- Distribution completion never independently redefines a programme as verified or complete.
- When WebFlash evidence materially changes programme state, the SOT update is made in a separate PR, never bundled into the WebFlash change.
- This repository-local `CLAUDE.md` and [`docs/standing-invariants.md`](docs/standing-invariants.md) remain authoritative for repository-internal distribution and installer rules.

## Sense360 hardware reference (canonical SKUs)

This table is a **synchronized local selection/display mirror** of the canonical hardware catalog owned by `sense360store/esphome-public` (`config/hardware-catalog.json` and `docs/product-taxonomy.md`) — physical board identity is owned upstream, never here; when the two disagree, upstream wins and this table is the thing to fix. The **Friendly name** column is the canonical user-facing label — use it verbatim in wizard markup, manifest descriptions, and module metadata. There is no Model/Variant axis: each SKU is its own product, and "Base / Pro" or model/variant terminology must be dropped when touching this code. The **Old name** column lists deprecated internal/historical names and exists only to help recognise legacy references; do not use these in new code.

| Group | Type | Friendly name | SKU | Rev | Old name | What it does |
|---|---|---|---|---|---|---|
| Ceiling | Hub | Sense360 Core | S360-100 | R4 | `360Core_Ceiling_V3_R` | Main board. Has the ESP32-S3 and connectors for all other modules. |
| Ceiling | Sensor | Sense360 RoomIQ | S360-200 | R4 | Presence + Comfort (two boards) | Merged board. On board: EKMC1601111 PIR, LTR-303ALS (light), SHT4x (temp and humidity), BMP581 (pressure). Connector-attached options (capability, not inclusion): LD2450 radar (J2 connector), SEN0609/C4001 radar (J3 connector). |
| Ceiling | Sensor | Sense360 AirIQ | S360-210 | R4 | `AirlQ Ceiling` (typo in old name) | Air quality board. CO2 (SCD41), VOC/NOx indices (SGP41), gas (MiCS-4514 with STM8, uncalibrated). External connector for PM (SPS30, optional, not included). Formaldehyde sensor fitment unresolved; not exposed as a supported customer function. |
| Ceiling | Sensor | Sense360 VentIQ | S360-211 | R4 | Bathroom Pro | Smaller air quality board for bathrooms. SGP41 (VOC/NOx indices) is the only on-board sensor. External connectors for optional IR surface-temp and SPS30 (not included). |
| Ceiling | Indicator | Sense360 LED | S360-300 | R4 | LED Ring | Ring of WS2812B LEDs. |
| Inline | Driver | Sense360 Relay | S360-310 | R4 | `S360-Relay-C`, `Sense360 Fan Relay` | On / off relay for bathroom fans. |
| Inline | Driver | Sense360 PWM | S360-311 | R4 | `12vFan_PWM_PulseCounter`, `Sense360 Fan PWM` | 12V PWM fan driver, up to 4 fans with tach feedback. |
| Inline | Driver | Sense360 DAC | S360-312 | R4 | `Fan_GP8403`, `Sense360 Fan DAC` | 0 to 10V analog fan driver, for example Cloudlift S12. |
| Inline | Driver | Sense360 TRIAC | S360-320 | R4 | `TRIAC_Board` | Phase dimmer for mains fan or lamp. |
| Power | PSU | Sense360 240v PSU | S360-400 | R4 | PWR Module, `Sense360 Mains PSU` | 240V mains to 5V using HLK-5M05. |
| Power | PSU | Sense360 PoE PSU | S360-410 | R4 | PoE Module | PoE to 5V. |

Notes:

- The current wizard exposes Ceiling mount only; a "Wall" branch lingers in legacy aliases but is not a supported product.
- The sensor/driver/indicator SKUs are *separately selectable* via `scripts/data/module-requirements.js`. Nothing is bundled — each SKU is its own product, and the user picks every module they have. The Core (S360-100) is the one exception: it is implicit because every flashable device is a Core. The 240v PSU (S360-400) and PoE PSU (S360-410) are surfaced through the `power` selection (`pwr` / `poe`) rather than as their own module entries. When introducing a new *selectable* SKU, add it to `module-requirements.js` and update the wizard SKU labels in `state.js` (`MODULE_LABELS`, `MODULE_VARIANT_LABELS`, `MODULE_SEGMENT_FORMATTERS`) — do not regress to model/variant nomenclature, and do not describe any SKU as "bundled".
- `scripts/data/module-requirements.js` is authoritative only for WebFlash's **local selection and compatibility behaviour** — never for physical board identity (owned by `sense360store/esphome-public`) or commercial bundle contents (owned by `sense360store/SOT`).

## Room presets vs commercial bundles (WEBFLASH-TAXONOMY-RECONCILE-001)

The customer-facing Step 1 experience is **room/use-case led**: the first customer decision is the room ("Choose your room"), never a module composition, a Base/Pro tier, or a raw config string. Board names stay visible as technical contents beneath the room choice, and the config string stays visible as the secondary technical identifier. The manual module builder remains available as the advanced route, never the primary path.

A `scripts/data/kits.json` entry is an **installer firmware preset** (`presentation: "firmware-preset"`), not a commercial listing:

- A preset selects a known hardware composition and resolves to a real `manifest.json` config. Stable/served firmware **never** implies the corresponding commercial bundle is available or buyable.
- Commercial bundle names, status, visibility and buyability come **only from SOT** (`sense360store/SOT` `bundles.yaml`). WebFlash mirrors the slice it needs into `scripts/data/sot-commercial-mirror.json` — a synchronized evidence snapshot with provenance (regenerated by `scripts/refresh-sot-mirror.py --sot-path <local SOT checkout>`), never commercial authority, never hand-authored. As mirrored today NO bundle is available or buyable, so customer copy must never show "buy now", "on sale", prices, shop links, or any equivalent.
- Each preset carries `room_label` + `recommended_rooms` (identical hardware = one preset with several recommended rooms, never duplicate cards) and `commercial_bundle_id` (the join key to the SOT mirror). Drift guards: `__tests__/webflash-taxonomy-reconcile.test.js`, `__tests__/wf2-room-preset-copy.test.js`, `__tests__/kit-served-consistency.test.js`.
- Commercial status is **not** an install gate — the mirror only constrains copy. Firmware existence is never permission to expose a new room card (the served stable `Ceiling-POE-AirIQ-RoomIQ` Kitchen candidate stays withheld from the picker), and `recommended` on a preset is an installer firmware recommendation, never a commercial claim.
- The internal filename `kits.json` and `kit`-named code identifiers remain for compatibility; rendered customer copy says room preset / firmware preset / setup.

## Commands

```bash
# Tests (Jest with experimental ESM VM modules — required because the codebase is pure ESM)
npm test
npm test -- url-config                  # filter by name
npm test -- __tests__/manifest-health.test.js
npm test -- --watch

# Naming-policy validator (also runs in CI before manifest generation)
npm run validate:naming-policy

# Advisory product-import readiness classifier (contract doc archived; see docs/archive-index.md)
npm run validate:product-import-readiness

# Regenerate manifest.json + firmware-*.json after adding/removing firmware
python3 scripts/gen-manifests.py --summary
python3 scripts/gen-manifests.py --summary --dry-run    # preview without writing

# Deployed-headers check (CSP / CORS / cache rules)
npm run check:headers -- https://sense360store.github.io/WebFlash/

# Local dev server (no build step — open in Chrome/Edge/Opera; Web Serial is required)
python3 -m http.server 5000
```

There is no lint or typecheck step. CI runs `npm test -- --ci` with `continue-on-error: true` (the suite is being cleaned up — do not skip hooks/flags to bypass). The Python publishing scripts have no test suite.

## Architecture

### Wizard frontend (entry: `index.html` → `scripts/bootstrap.js` → `scripts/shell.js`)

The WebFlash 2.0 migration is complete (PR 13) and there is a **single view**. `index.html` loads `scripts/bootstrap.js`, which mounts the 2.0 view (`scripts/shell.js` → `scripts/app.js`'s `mountWebFlash2`) inside the production shell. The view modules (`scripts/app.js`, `scripts/identify.js`, `scripts/connect.js`, `scripts/install.js`, `scripts/data.js`) are a render layer only: they reach the **engine** through the `scripts/engine.js` facade and never own a gating decision. The architecture decision is recorded in [`docs/adr/0001-webflash-2-view-over-engine.md`](docs/adr/0001-webflash-2-view-over-engine.md).

`scripts/state.js` (~8000 lines) is the **central state module** and the source of truth. It owns:

- The wizard configuration object (`mounting`, `power`, `bathroom`, plus module keys `voice`, `led`, `airiq`, `fan`, `ventiq`).
- Step gating via `getMaxReachableStep()` — step 2 unlocks once `mounting` is set, step 3 once `power` is set, etc.
- Manifest loading, parsing of `config_string` values like `"Ceiling-POE-VentIQ-RoomIQ"` back into wizard state (`parseConfigStringState`), and matching builds to the current selection.
- The install-time preflight engine (`evaluatePreflightPolicy`) and connection-quality metrics fed by `navigator.serial` connect/disconnect events and ESP Web Tools `state-changed` events.
- All install/download gating: install only fires when no preflight `Fail` exists, the pre-flash checkbox is checked, and every `Warning` and required acknowledgement (release channel, advanced/manual-warning) is satisfied.

It exports a small surface (`getState`, `setState`, `replaceState`, `getStep`, `setStep`, `getMaxReachableStep`, `getTotalSteps`) and a `__testHooks` bundle used exclusively by Jest.

Other notable engine pieces:

- `scripts/data/module-requirements.js` — hardware compatibility matrix (SKUs, headers, conflicts, `recommended`/`ceilingOnly`/`requiresBathroom` flags). **Constraint enforcement reads from this file**; keep it consistent with the canonical SKU table above and with the option tables in [`docs/hardware-options.md`](docs/hardware-options.md).
- `scripts/utils/url-config.js` — bidirectional parser for sharable config URLs. Maintains legacy aliases (e.g. `pwr` → `ac`, `BathroomAirIQ*` → `VentIQ`, fan `pwm` ↔ `base`) so old links still resolve. The wizard URL key `voice` historically maps to `core` in the URL alias set.
- `scripts/utils/release-channels.js` — release-channel policy (per-channel `defaultSelectable`, `requiresAcknowledgement`, `hiddenByDefault`). Preview builds are never auto-selected and gate install on a channel acknowledgement.
- `scripts/utils/module-availability.js` — presentation-only availability classifier for module variants (states such as `available-stable`, `available-preview`, `no-firmware`, `advanced-manual-warning`). Static overrides take precedence over manifest-derived states; the classifier is **not** the install gate.
- `scripts/utils/flash-history.js` — flash attempts logged to `localStorage` for diagnostics; entries strip deprecated keys.
- `sw.js` — service worker. Strategy is network-first for `*.bin` and `manifest.json`, stale-while-revalidate for everything else. **When you add new top-level scripts, add them to `STATIC_ASSETS` or `SCRIPT_MODULES` in `sw.js`** or they will not be available offline.

### Publishing pipeline

`scripts/gen-manifests.py` is the only way `manifest.json` and `firmware-*.json` should change — these files are **generated, not hand-edited**. It scans `firmware/`, parses each filename via the canonical pattern (see below), produces a single `manifest.json` with full per-build metadata (including hashes and a `config_string` like `Ceiling-POE-VentIQ-RoomIQ`), and writes one `firmware-<index>.json` per build (the standard ESP Web Tools per-product manifest format). Per-build indices are **not stable** — runtime resolves builds via `manifest.json` + `config_string`; never hardcode a `firmware-N.json` index. The generated manifest must match the actual `.bin` files on disk — the `__tests__/manifest-health.test.js` guard fails CI before deploy if any `manifest.json` or `firmware-*.json` build references a missing `.bin`, if a `firmware/configurations/*.bin` lacks its `.meta.json` sidecar, if the per-build manifests drift out of sync with `manifest.json`, if a blocked token (`FanTRIAC` globally, plus any `block_tokens` declared for a matching source in `firmware/sources.json`) reappears in a `config_string`, or if a `REQUIRED_CONFIGS` entry is missing from `manifest.json`.

New firmware that is meant to ship enters the tree through the cross-repo importer, not through hand-copying a `.bin` into `firmware/configurations/`. Declare the source in [`firmware/sources.json`](firmware/sources.json) and run [`scripts/import-firmware-sources.py`](scripts/import-firmware-sources.py) (or dispatch `.github/workflows/firmware-import.yml`) — the importer fetches the upstream `.bin` from `sense360store/esphome-public`, verifies its SHA256 against the upstream `checksums-sha256.txt` (and the pinned `expected_sha256` when present), enforces the per-source `block_tokens` allowlist, and writes the `<asset>.meta.json` sidecar. The Rescue firmware (built in-tree under `firmware/rescue/`) is the only sanctioned exception. The full importer contract record (`docs/firmware-import.md`) and the import readiness matrix (`docs/webflash-import-readiness-matrix.md`, the record of whether a candidate config may be imported at all) are archived — see [`docs/archive-index.md`](docs/archive-index.md).

`scripts/gen-manifests.py` still carries a legacy `model` / `variant` code path for binaries placed outside `firmware/configurations/`. **Treat it as deprecated** — do not introduce new firmware down that branch and do not extend Model/Variant metadata in new code. New SKUs and configurations belong in `firmware/configurations/` with the canonical `Sense360-...-vX.Y.Z-<channel>.bin` filename, identified by SKU/config-string only.

`scripts/validate-naming-policy.js` enforces:

- Canonical filename shape `Sense360-...-vX.Y.Z-(stable|preview|beta).(bin|md)`.
- Disallowed token migrations (mirroring `DISALLOWED_TOKEN_MIGRATIONS` in the validator, which is the source of truth): `AirIQProv` / `AirIQPro` / `AirIQBase` → `AirIQ`, `BathroomAirIQ` (and its `Base`/`Pro` suffixes) → `VentIQ`, `FanAnalog` → `FanDAC`. Fan variants are preserved as variant-specific tokens (`FanRelay`, `FanPWM`, `FanDAC`, `FanTRIAC`) so each driver SKU lands on a different firmware binary; the legacy generic `Fan` token must not be used in new firmware.
- Channel placement: only `*-stable.md` is allowed under `firmware/configurations/`. Preview/beta/dev release notes belong in `firmware/previews/`.

`.github/workflows/firmware-publish.yml` runs unit tests, the naming-policy validator, the manifest generator, and a `REQUIRED_CONFIGS` allowlist that fails the build if any of the expected `config_string` values are missing from `manifest.json`. The live array in the workflow file is the source of truth; it holds exactly the configs WebFlash must be able to ship (currently `Ceiling-POE-VentIQ-RoomIQ` and `Rescue`), each backed by a real signed `.bin` on disk. `REQUIRED_CONFIGS` is **production-only**: preview builds never enter it. Do not add a `config_string` until its source entry exists in `firmware/sources.json`, the `.bin` + `.meta.json` sidecar have been imported, and the regenerated manifest contains the build.

Two invariants travel with the allowlist and must not be regressed:

- **FanTRIAC stays import-blocked.** Every `firmware/sources.json` entry carries `FanTRIAC` in `block_tokens`, the importer refuses to ingest a FanTRIAC-bearing asset, and the manifest-health guard fails CI if a `FanTRIAC` token ever appears in a generated `config_string`. FanTRIAC remains a *permitted filename token* for the naming-policy validator and a real SKU in the hardware table; the block is enforced at import + manifest-health time. Selecting TRIAC in the wizard is possible only behind the advanced/manual-warning acknowledgement, which is an in-installer warning gate, **not** a compliance certification.
- **LED ships only through deliberate LED-bearing source entries.** Non-LED sources keep `LED` in `block_tokens`; an LED-bearing build enters the manifest only when a source entry is added for it on purpose (currently preview-channel only).

The product-lifecycle guard `__tests__/product-catalog-alignment.test.js` (vendored fixture at `__tests__/fixtures/esphome-product-catalog.json`; set `PRODUCT_CATALOG_PATH` to validate a fresh upstream catalog) cross-checks `firmware/sources.json`, `manifest.json`, every `firmware-*.json`, `REQUIRED_CONFIGS`, and `scripts/data/kits.json` against the upstream catalog. `production` status is required for `REQUIRED_CONFIGS`; `preview` status or an upstream `webflash_import_eligibility.eligible: true` flag is import / manifest / kit eligible but never `REQUIRED_CONFIGS` eligible; every other status fails; `Rescue` is exempt by name. The advisory classifier `scripts/validate-product-import-readiness.js` (contract doc `docs/product-import-readiness.md` archived — see [`docs/archive-index.md`](docs/archive-index.md); behaviour pinned by `__tests__/product-import-readiness.test.js`) reports the same rules per config. The full per-slice history of how these rules landed (`docs/conventions-history.md`) is archived — see [`docs/archive-index.md`](docs/archive-index.md).

### Frontend ↔ pipeline contract

The wizard's selection is reduced to a `config_string` (e.g. `Ceiling-POE-VentIQ-RoomIQ`) and matched against `build.config_string` in `manifest.json`. `parseConfigStringState` in `state.js` and the canonical token formatters in `MODULE_SEGMENT_FORMATTERS` define how segments encode/decode (`AirIQ` → `airiq=airiq`, `VentIQ` → `ventiq=airiq`, `FanRelay` → `fan=relay`, `FanPWM` → `fan=pwm`, `FanDAC` → `fan=analog`, `FanTRIAC` → `fan=triac`, etc.). When you add a new module token, update both:

1. The wizard's segment formatter and `parseConfigStringState` in `scripts/state.js`.
2. `CANONICAL_MODULE_TOKENS` / token-handling logic in `scripts/gen-manifests.py`.

Otherwise the frontend will fail to find a build that the manifest claims exists.

## Conventions and gotchas

- **Desktop / Web Serial only.** The install path depends on `navigator.serial`, which is only implemented on desktop Chromium browsers. Do not add code paths that assume mobile or non-Chromium browsers can flash; gate any new install-time UI behind the existing capability detection in `scripts/capabilities.js`.
- **Pure ESM.** Tests require `NODE_OPTIONS=--experimental-vm-modules` (already set by `npm test`). Do not introduce CommonJS modules under `scripts/` or in tests; new tests should use `import { ... } from '@jest/globals'`. Jest config (`jest.config.cjs`) sets `transform: {}` — no transpilation.
- **No external runtime dependencies in the wizard.** The only third-party script loaded by `index.html` is `esp-web-tools` from unpkg, which is allowed by the `Content-Security-Policy` in `_headers`. If you need new origins (scripts, fonts, connect-src), update the CSP there.
- **`_headers` is GitHub-Pages-style.** It controls CORS, CSP, and cache rules. Firmware binaries are served with `Cache-Control: max-age=31536000`, so versioned filenames are critical — never overwrite a published `.bin` in place.
- **Disabled options live in the matrix, not in markup.** The canonical SKU table above documents the products; runtime gating comes from `module-requirements.js` (e.g. `ceilingOnly`, `requiresBathroom`) and the visibility logic in `getVisibleModuleGroupKeys` in `state.js`. AirIQ ↔ VentIQ is mutually exclusive and driven by the Bathroom toggle on Ceiling mounts.
- **No Model/Variant axis.** The product taxonomy is flat (one SKU per product). When extending the wizard, do not add Base/Pro variants or model/variant fields; add a new SKU entry to `module-requirements.js` and a new module key to `MODULE_KEYS` in `state.js` instead.
- **Sensitive-value redaction.** `Copy diagnostics` and flash history both pass through redaction (`SENSITIVE_KEY_PATTERN` in `state.js`, `stripDeprecatedConfigurationFields` in `flash-history.js`). When adding new fields to diagnostics or history, audit whether they should be redacted before they ship.
- **Service worker cache versioning.** The cache name lives in `CACHE_NAME` in `sw.js` (currently `webflash-v20`); bumping it is how forced refreshes are landed. The `activate` handler deletes any cache that starts with `webflash-` but is not the current name.
- **Generated files are committed.** `manifest.json`, every `firmware-*.json`, and every `firmware/configurations/*.bin` are tracked in git. Regenerate with `gen-manifests.py` and commit the diff together with the firmware change in the same commit.
- **Branch policy.** All AI-assisted development on this repo runs on a dedicated `claude/...` branch (see workflow instructions). Never push to `main` directly.
- **Standing invariants.** [`docs/standing-invariants.md`](docs/standing-invariants.md) carries the standing blockers / invariants that gate every WebFlash PR (2.0 sole view, engine owns every gate, stable Bathroom PoE default, preview gating, TRIAC blocked, FanDAC address acknowledgement, and the do-not-change guardrails for docs-only PRs). Never regress them. The former `UPCOMING_PR.md` working queue tracker was retired — see [`docs/archive-index.md`](docs/archive-index.md); PR queue state now lives in GitHub, not in a tracked file.
- **Wizard UX roadmap.** `docs/wizard-ux-roadmap.md` (archived — see [`docs/archive-index.md`](docs/archive-index.md)) holds the UX audit, severity-classified findings, and PR sequencing; UX-shaped changes must round-trip its do-not-change guardrails. The LED-preview operator hardware proof container (`docs/led-preview-webflash-proof.md`, archived — see [`docs/archive-index.md`](docs/archive-index.md)) recorded no operator evidence before archiving, so the proof is still pending — never claim hardware, bench, or compliance verification that has not been recorded by an operator.
- **No internal IDs in customer-facing copy.** Internal engineering / task / release / tracking identifiers, internal doc filenames, and internal script paths must never appear in customer-visible wizard copy; they are allowed only in code comments, developer/support-only data fields, and machine-readable diagnostic codes. Every unavailable customer option must give one plain-language reason plus one suggested next step. Guard tests: the rendered-prose assertions in `__tests__/module-availability.test.js` (IDs allowed in `reasonCode`, never in `detail`).
- **Fan-token guardrail.** Fan firmware uses variant-specific tokens only (`FanRelay`, `FanPWM`, `FanDAC`, `FanTRIAC`); the generic `Fan` token is banned. FanTRIAC stays import-blocked (see *Publishing pipeline*), TRIAC selection requires the advanced/manual-warning acknowledgement, and no fan build becomes a kit / default / recommended surface without an explicit product decision.
- **Preview stays gated.** Preview-channel builds are never auto-selected (`defaultSelectable: false`) and install gates on the channel acknowledgement in `scripts/utils/release-channels.js`. Adding new builds or UI must strengthen gates, never weaken them.
- **Import readiness before import.** Any candidate firmware import must round-trip the archived `docs/webflash-import-readiness-matrix.md` (see [`docs/archive-index.md`](docs/archive-index.md)) and the eligibility rules above. Release artifact existence does not mean WebFlash import; import does not mean `REQUIRED_CONFIGS`; import does not mean kit / default exposure; an advanced / manual-warning import is not compliance certification.
- **History lives in the archive.** The changelog-style per-slice records that used to fill this section (WF-WIZARD-AVAIL-001 through WF-FAN-BUNDLE-IMPORT-READINESS-001, the WF-UX series, the preview import records) were archived verbatim in `docs/conventions-history.md`, itself now archived under `DOCS-DISPOSITION-001` — recoverable via [`docs/archive-index.md`](docs/archive-index.md). They are historical records — do not "fix" them.

## WebFlash 2.0 migration (history) and standing conventions

The `wf2-*` migration is **complete** (PR 0 through PR 13; PR 13 = #492). The 2.0 view over the unchanged 1.0 engine is the only view: the `?ui` flag, `scripts/ui-version.js`, and the entire 1.0 render layer were removed, and the former `webflash-2/` tree was folded into the repo root. No firmware, `manifest.json`, `firmware/sources.json`, `REQUIRED_CONFIGS`, kit, or release-channel/install-gate logic changed. Plan and history: [`docs/adr/0001-webflash-2-view-over-engine.md`](docs/adr/0001-webflash-2-view-over-engine.md) and the archived records (`docs/webflash-2-migration.md`, `docs/webflash-2-migration-delivery.md`, `docs/conventions-history.md`) via [`docs/archive-index.md`](docs/archive-index.md).

The following conventions remain in force for all future work, alongside the standing blockers / invariants in [`docs/standing-invariants.md`](docs/standing-invariants.md):

Repo conventions:

- Vanilla ES modules. No build step. No React or Babel runtime. No new third-party runtime dependency beyond ESP Web Tools from the documented unpkg origin already allowed by the CSP.
- Desktop Chromium only. Keep the mobile and unsupported fallback intact.

Trust model is non-negotiable:

- Never weaken a blocking gate: provenance, channel acknowledgement, manifest freshness, service-worker update.
- Never claim cryptographic signature verification the engine has not actually performed. Real Ed25519 verification is implemented and ENFORCED at the install gate: `verifyFirmwareIntegrity` (state.js) verifies the downloaded bytes against the pinned trust list (`scripts/utils/firmware-trusted-keys.js`) alongside the SHA-256 check, and `evaluateInstallGate` refuses install unless both verdicts pass. Never weaken this gate; UI copy may claim verification only from the engine's runtime verdict (`docs/firmware-provenance.md` → "Signature verification — enforced install gate").
- Never expose any kit, module, or channel that 1.0 does not already expose. The release gates are the source of truth for what installs.

Implementation discipline:

- When you replace a simulation with a real binding, delete the simulation in the same PR. No dead simulated code paths.
- Implement only the current PR's scope. Do not start the next PR's work.

Self-verify before opening the PR, and paste the output into the PR body:

- `npm test`
- `python3 scripts/gen-manifests.py --strict-validate --dry-run`
- `npm run check:headers -- https://sense360store.github.io/WebFlash/`
- If the PR touches provenance, the manifest, or the install gate, fill the Reviewer checklist from the `docs/firmware-provenance.md` "Reviewer checklist" section into the PR body.

PR body: a summary, the key changes, the verify output, and an explicit statement of what stays gated. Plain prose. Do not use hyphens as sentence breakers. No emojis.

---
> Source: [sense360store/WebFlash](https://github.com/sense360store/WebFlash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
