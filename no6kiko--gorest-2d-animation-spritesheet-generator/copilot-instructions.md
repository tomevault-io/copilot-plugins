## gorest-2d-animation-spritesheet-generator

> Gorest is a no-UI 2D game and animation generative platform. The user describes the target scene, asset, spritesheet, or editor workflow in natural language; the agent creates assets, wires metadata, and verifies the browser-visible result.

# Gorest Agent Instructions

Gorest is a no-UI 2D game and animation generative platform. The user describes the target scene, asset, spritesheet, or editor workflow in natural language; the agent creates assets, wires metadata, and verifies the browser-visible result.

## Read First

- For normal asset/scene work, this file is the required entry point.
- Read `SPRITESHEET_GENERATION_POLICY.md` only when creating, editing, or debugging character/creature/mascot spritesheets.
- Read `docs/AGENT_WORKFLOW.md` only for larger end-to-end scene workflows or when the project structure is unclear.
- Read `README.md` only for product background, onboarding, or user-facing documentation changes.

## Core Rules

- Keep work aligned with the existing React structure and `public/generated/game_asset_library.json`.
- Store generated app assets under `public/generated/` unless the asset is a brand/logo asset.
- Add reusable visual assets and scenes to `public/generated/game_asset_library.json` with deterministic descriptive IDs.
- One-frame sprites are static images. Do not configure them as auto-playing animations unless asked.
- Multi-frame spritesheets need consistent frame size, sheet size, grid columns, fps, frames, and a reusable animation clip.
- App logo usage should stay black in the app. The colored Gorest logo is for README/community presentation.

## Character Art Rules

- Character, portrait, facial-expression, creature, and mascot spritesheets must use finely drawn image-generation/diffusion-style art as the visible source.
- Do not use SVG, Canvas, PIL, CSS, or procedural overlays as the final drawing for eyes, tears, mouths, hair, clothes, limbs, or expression changes.
- Code may only post-process drawn art: background removal, alpha cleanup, fixed-frame cropping, anchor/scale normalization, spritesheet packing, preview generation, and metadata wiring.
- Require very high clarity: sharp eyes, hair strands, tears, mouth shapes, accessories, fabric texture, and clean alpha edges.
- Prefer high-detail source art and downsample into the final frame size. Do not upscale blurry frames and call them final.
- For exact visible text, prefer code-assisted text rendering over pure image generation.

## Product Rules

- The browser is the result surface; Codex handles generation, JSON wiring, metadata, and code changes.
- Keep app controls restrained, readable, and practical. Avoid marketing-heavy UI copy.
- Maintain right-click and keyboard workflows on the 2D Canvas.
- Keep `Spritesheet Only` detail back navigation returning to the `Spritesheet Only` gallery.

## License Boundary

- Source code is MIT unless otherwise stated.
- Media under `public/` and `docs/` is example/demo material and is not CC0 or MIT unless a file explicitly says so.
- Do not add language that lets users treat Gorest IP, generated examples, or demo images as freely reusable assets.

## Verification

- Run `npm.cmd run lint` and `npm.cmd run build` after code or asset-library changes.
- Visually inspect newly generated image assets when possible.
- Do not commit or push unless the user explicitly asks.

---
> Source: [NO6KIKO/gorest-2d-animation-spritesheet-generator](https://github.com/NO6KIKO/gorest-2d-animation-spritesheet-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
