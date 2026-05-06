## appshot

> Guidance for AI agents working in this repository. Keep changes focused, predictable, and aligned with the compositor’s rules.

# AGENTS.md

Guidance for AI agents working in this repository. Keep changes focused, predictable, and aligned with the compositor’s rules.

## Project Snapshot
- Appshot is a CLI that composes App Store screenshots using sharp.
- Core layout lives in `src/core/compose.ts` (caption/device placement), frame selection in `src/core/frames-loader.ts` + `src/core/devices.ts`.
- Templates (human‑friendly presets) are in `src/templates/registry.ts` and get merged via `applyTemplateToConfig`.

## Layout Invariants (v2)
These are the source‑of‑truth rules used by the v2 compositor:

- Fixed zones: Header/footer layouts split the usable area into caption + device regions using device strategy ratios.
- Gap buffer: A fixed `regionGap` separates caption and device regions (per device strategy).
- Screenshot‑only: Device uses the full usable area (edge padding only).
- No overlay: Overlay captions are removed in v2.
- Text sizing: Font size is derived from output resolution and clamped by device strategy min/max.

## Frames
- `frames/Frames.json` defines frames. The registry stores:
  - `frameWidth`/`frameHeight`: full device image bounds
  - `screenRect`: the inset area where the screenshot is placed
  - `deviceType`: `iphone` | `ipad` | `mac` | `watch`
- Auto‑selection picks the closest `screenRect` aspect ratio for a given screenshot. A 10% mismatch warning is logged.

## Where to Change What
- Device/caption math: `src/core/compose.ts`
- Template defaults and device overrides: `src/templates/registry.ts`
- Frame selection/metadata: `src/core/frames-loader.ts` and `src/core/devices.ts`
- Build messaging / user notices: `src/commands/build.ts`

## Debugging Placement
- Run `appshot build --dry-run --verbose` to print chosen frame, frame/device dimensions, and key layout positions (`deviceTop`, `deviceBottom`, `captionTop`).
- For v2, verify the layout block shows the computed caption/device regions and the effective layout (header/footer/screenshot‑only).

## Style Guidance for Agents
- Don’t change API shape unless requested. Keep new behavior behind small, well‑named switches if you must.
- Prefer single‑point fixes in `compose.ts` over template scatter.
- Update README/CHANGELOG when layout behavior changes.
- Keep tests deterministic; avoid time/locale dependencies.

## Useful Commands
```bash
npm run build           # compile TypeScript
npm run dev -- build    # run CLI in dev mode: appshot build
appshot build --dry-run --verbose  # print layout decisions
```


## Fastlane Export Notes
- `src/commands/export.ts` powers the Fastlane-compatible export flow. Device ordering, language mapping, and dry-run behaviour are covered by integration tests in `tests/integration/export-command.test.ts`.
- Language code normalization lives in `src/services/fastlane-language-mapper.ts`; defaults and overrides are locked down by `tests/fastlane-language-mapper.test.ts`.
- File moves are handled by `src/services/screenshot-organizer.ts`; symlink vs copy, flattening, device prefixes, and iPad Pro renaming are verified in `tests/screenshot-organizer.test.ts`.
- Validation rules (requested-device checks, output safety) are centralized in `src/services/export-validator.ts` with coverage in `tests/export-validator.test.ts`.
- Fastlane config scaffolding (`Deliverfile`, `Fastfile`, README, `.gitignore`) comes from `src/services/fastlane-config-generator.ts` and is snapshot-tested in `tests/fastlane-config-generator.test.ts`.

---
> Source: [chrisvanbuskirk/appshot](https://github.com/chrisvanbuskirk/appshot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
