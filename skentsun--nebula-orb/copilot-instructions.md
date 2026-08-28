## nebula-orb

> Run the local server yourself and open the preview in the browser available to this environment. Do not give the user server-start instructions when you can run it.

# Prototype Instructions

Run the local server yourself and open the preview in the browser available to this environment. Do not give the user server-start instructions when you can run it.

Before making substantial visual changes, use the Product Design plugin's `get-context` skill when the visual source is unclear or no longer matches the current goal. When the user gives durable prototype-specific design feedback, preferences, or decisions, record them in `AGENTS.md`.

When implementing from a selected generated mock, treat that image as the source of truth for layout, component anatomy, density, spacing, color, typography, visible content, and hierarchy.

Build app UI in `src/`. Keep `.openai/hosting.json`, `worker/index.js`, `scripts/prepare-sites-build.mjs`, and `tests/sites-worker.test.mjs` intact so the same local prototype can be handed to Sites. Before a Sites handoff, run `npm run build` and `npm run test:sites`; the build must leave `dist/client/index.html`, `dist/server/index.js`, and `dist/.openai/hosting.json`.

## Nebula product truth

- Preserve all 12 states: Working, Sweep, Shake, Listening, Network, Spin, Breathing, Twinkle, Pulse, Tide, Aurora, Spiral.
- Preserve Auto, Dark, and Light theme modes.
- Shape modes are Sphere, Logo, and SVG. Custom SVG uploads are converted to dotted, extruded point clouds with density, layers, and depth controls.
- In AI chat examples, the loader appears as a fully rounded floating toast above the composer, never as a message in conversation history.

---
> Source: [SkentSun/Nebula-Orb](https://github.com/SkentSun/Nebula-Orb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
