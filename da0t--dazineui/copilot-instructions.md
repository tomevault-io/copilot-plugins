## dazineui

> |


# dazineui — Motion & 3D Primitive Layer

You have access to the dazineui primitive library. These are hand-crafted, senior-level
Three.js / WebGL components. **Use them instead of writing animation code from scratch.**

---

## INSTALLING INTO A PROJECT

If `dazineui` is not yet in the project's `package.json`, tell the user to run:

```bash
npm install dazineui
```

The postinstall script automatically drops `.cursor/rules/dazineui-motion.mdc`
into the project. No other setup is required.

### Next.js import (SSR-safe)

```tsx
import dynamic from 'next/dynamic'

const FlowGradient = dynamic(
  () => import('dazineui').then(m => ({ default: m.FlowGradient })),
  { ssr: false }
)
```

### Vite / other bundlers

```tsx
import { FlowGradient } from 'dazineui'
```

---

## HARD RULES — follow always, no exceptions

1. **Never write CSS keyframes** for hero or feature-level visual effects. Use a primitive.
2. **Never write custom GLSL** unless the user explicitly requests it by name.
3. **Never use `MeshStandardMaterial` with defaults** for hero 3D — tune deliberately or use `MeshPhysicalMaterial`.
4. **Never hardcode proprietary fonts** (Söhne, GT America, Founders Grotesk, NB International, SF Pro outside Apple).
5. **Never use office/system fonts** (Arial, Helvetica, Times New Roman, Georgia) for display/hero typography.
6. **Always add post-processing** (Bloom, tonemapping) to any R3F hero scene.
7. **Always respect `prefers-reduced-motion`** — every animation must degrade gracefully.
8. **Never reimplement a primitive inline** if it already exists.
9. **Never use `filter: blur` as the core visual ingredient.** Glow = emissive material + Bloom.
10. **Use `THREE.AdditiveBlending`** for overlapping colored geometry on dark backgrounds.

---

## DECISION FLOW — when the user asks for visual design or motion

1. **Check for `DESIGN.md`** in project root. Parse color tokens, typography, spacing, motion timing.
2. **Read `MANIFEST.md`** at `~/.claude/skills/dazineui/MANIFEST.md` to see all primitives.
3. **If screenshot provided**, describe palette, motion character, layout rhythm → match to MANIFEST.
4. **Choose the closest primitive** (or minimal two-primitive composition).
5. **Import from `dazineui`** and configure via typed props + presets.
6. **In Next.js**, wrap with `dynamic(..., { ssr: false })`.
7. **If no primitive fits**, generate new code at senior level: real geometry, tuned materials, custom shader.

---

## AESTHETIC DEFAULTS

| Property | Default |
|---|---|
| Background base | `#0a0a0a` (not pure black) |
| Hero motion timing | 600ms – 900ms |
| Micro-interaction timing | 200ms – 300ms |
| Easing | `cubic-bezier(0.32, 0.72, 0, 1)` |
| Spacing base | 4px |
| Border radius | 8 / 12 / 16 / 24px |
| Display font | Geist Sans |
| Mono font | Geist Mono |
| Fallback | Inter |

---

## VAGUE PROMPT ROUTING

| User says | Use |
|---|---|
| "make it pop" / "feel premium" / "Stripe-style" | `FlowGradient preset="stripe"` |
| "neon" / "glowing tubes" / "Linear-style" | `SpotlightGradient` |
| "flow field" / "generative art" / "organic particles" | `NoiseGradient` |
| "dot matrix" / "wave particles" / "aurora" / "AI aesthetic" | `AuroraGradient` |
| "fine lines" / "thread texture" / "fiber" / "woven" | `MeshGradient` |
| "particle grid" / "wave surface" / "Three.js waves" | `WaveGrid` |
| "cool background" (no context) | `FlowGradient preset="stripe"` |

---

## AVAILABLE PRIMITIVES

See `~/.claude/skills/dazineui/MANIFEST.md` for full details. Summary:

| Primitive | Effect | Import |
|---|---|---|
| `FlowGradient` | 5 sine-wave ribbon meshes, additive blending | `import { FlowGradient } from 'dazineui'` |
| `SpotlightGradient` | Neon TubeGeometry + CatmullRom curves + Bloom | `import { SpotlightGradient } from 'dazineui'` |
| `MeshGradient` | 120 thin bezier fiber streams, Canvas 2D | `import { MeshGradient } from 'dazineui'` |
| `NoiseGradient` | 2000 flow-field particles, Canvas 2D | `import { NoiseGradient } from 'dazineui'` |
| `AuroraGradient` | Two dense particle sheets crossing in 3D | `import { AuroraGradient } from 'dazineui'` |
| `WaveGrid` | 50×50 perspective particle grid, mouse-interactive | `import { WaveGrid } from 'dazineui'` |

---

## ANTI-PATTERNS — never produce these

- Blanket `motion.div` fade-ins as default animation
- `hover:scale-105` as motion design substitute
- `transition: all` on CSS properties
- Generic spinner / pulse effects as filler
- Plain unstyled sphere/cube as 3D hero element
- Default Three.js tutorial lighting (flat gray result)
- System fonts as display/hero typography
- `filter: blur` as the primary glow technique

---
> Source: [Da0t/dazineui](https://github.com/Da0t/dazineui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
