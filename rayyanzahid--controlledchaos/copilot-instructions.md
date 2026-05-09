## controlledchaos

> **I. Hierarchy is the only job.** Every pixel answers: what first, what second, what never? Size is loudest, then weight, color, position, motion. The most powerful tool is omission.

# Controlled Chaos Design System

## The Three Axioms (These override everything else)

**I. Hierarchy is the only job.** Every pixel answers: what first, what second, what never? Size is loudest, then weight, color, position, motion. The most powerful tool is omission.

**II. Tension is beauty.** Near-symmetry over perfect. Every composition needs one discord — one break that proves the rest is intentional. Whitespace is counterforce, not absence. Grids exist to be *almost* followed.

**III. The medium is time.** Screens are not posters. Revelation over time creates narrative. Scroll, viewport, pointer, and elapsed time are design variables. If it could be printed unchanged, it's not using the screen.

**Decision engine — apply to every choice:**
1. Does this serve hierarchy? → Remove or subordinate if not.
2. Does this create intentional tension? → Add a discord if everything is too resolved.
3. Does this use the medium? → Make it respond to scroll/viewport/pointer/time.

See `AXIOMS.md` for the full derivation, corollaries, and the relationship between craft and taste.

## What This Is
A creative design system for building award-level UI/UX — inspired by Awwwards SOTD winners and Cargo graphic templates. NOT a corporate system. A system of controlled chaos.

## How to Use This System

When building any page or component:

1. **Choose a mode** (or mix them):
   - `swiss` — Neue Grafik editorial, grid-locked, systematic
   - `brutalist` — Maximum volume, raw, unapologetic
   - `immersive` — Dark atmospheric, scroll-driven, experiential
   - `kinetic` — Typography in motion, animated, alive
   - `gallery` — White cube minimal, work speaks for itself

2. **Set the forces** via `data-forces` attribute on the root element

3. **Choose a mood** via `data-mood` attribute:
   `void`, `void-inverse`, `dusk`, `signal`, `earth`, `frost`, `blaze`, `bruise`, `concrete`

4. **Use the typographic voices**:
   - `.voice-monument` — Architectural display (Space Grotesk)
   - `.voice-editorial` — Elegant serif (Playfair Display)
   - `.voice-swiss` — Systematic sans (Inter)
   - `.voice-brutalist` — Raw mono (Space Mono)
   - `.voice-expressive` — Versatile (DM Sans)

5. **Import the tokens**:
   ```html
   <link rel="stylesheet" href="tokens/layout.css">
   <link rel="stylesheet" href="tokens/typography.css">
   <link rel="stylesheet" href="tokens/color.css">
   <link rel="stylesheet" href="tokens/motion.css">
   <link rel="stylesheet" href="tokens/forces.css">
   ```

## Design Principles (Non-Negotiable)
- Type is the hero. If it works without color/images, you've won.
- Tension is beauty. Near-symmetry over perfect symmetry.
- Whitespace is pressurized, not empty.
- Motion has meaning or it doesn't exist.
- Break rules only after understanding why they exist.
- The screen is not paper — use viewport, scroll, pointer, time.

## File Structure
- `tokens/` — CSS custom properties (forces, typography, color, motion, layout)
- `modes/` — Complete aesthetic presets (reserved for future CSS-only mode files)
- `components/` — Reusable component patterns (reserved for future)
- `showcase/` — Live demos of each mode (index, swiss, brutalist, immersive, kinetic, gallery)

## Key CSS Variables
- `--type-massive` through `--type-micro` — Non-linear type scale
- `--color-bg`, `--color-fg`, `--color-accent-1`, `--color-discord` — Mood-driven colors
- `--force-structure` through `--force-shout` — Creative force dials (0 to 1)
- `--dur-instant` through `--dur-meditate` — Motion tempo
- `--ease-out-expo`, `--ease-spring`, `--ease-snap` — Timing curves

## When Creating New Pages
Always start from the five forces — where does this piece live on each spectrum?
Then pick the mode, mood, and voices that match. Compose from tokens, not from scratch.

---
> Source: [RayyanZahid/controlledchaos](https://github.com/RayyanZahid/controlledchaos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
