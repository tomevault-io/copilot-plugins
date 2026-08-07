## liquid-glass-carousel

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# Project notes for AI assistants

WebGL portfolio carousel: three.js + GSAP core with a thin React/Next.js wrapper. Human docs: `README.md` (overview) and `HOW-IT-WORKS.md` (architecture walkthrough) — read the latter before changing engine behavior.

## File map

| File | Owns |
| --- | --- |
| `lib/carousel/config.js` | `PROJECTS` image list + all tunables (`CONFIG`, `LENS`, `FOCUS`, `ENTRY`, `UI_ANIM`). Data only, no logic. |
| `lib/carousel/engine.js` | Everything on the canvas: renderer, infinite row, scroll, lens shader (inline GLSL), focus mode, entry animation, render loop. Framework-free — no React imports, ever. |
| `lib/carousel/gui.js` | lil-gui dev panel (hidden by default). Mutates the config objects live. |
| `Components/CarouselSection.jsx` | React overlay only: heading, counter, "View" cursor label, Close button, viewport gate (<1025px shows a black screen and never boots WebGL). |
| `app/page.js` | Just renders `<CarouselSection />`. |

Routing common requests: add/change images → `config.js` `PROJECTS`. Scroll feel / snap / panel size → `config.js` `CONFIG`. Lens look → `config.js` `LENS` (uniforms mirror it 1:1). Entry/focus choreography → `ENTRY` / `FOCUS`. Overlay text or breakpoint → `CarouselSection.jsx`. Motion/render behavior → `engine.js`.

## Architecture invariants — do not break these

- **The engine talks to React only via callbacks** (`onActiveChange`, `onFocusChange`, `onEntryDone`) and the returned handle (`closeFocus`, `replayEntry`, `refreshLayout`, `destroy`). Don't import React into the engine or reach into engine internals from the component.
- **GSAP animates plain numbers, never meshes.** Timelines tween values in `drop[]`, `pEntry[]`, `growArr[]`, `focusScale`, `focusState.lensFx`; `layout()` reads them each frame and derives final mesh transforms. Keep new animations in this pattern or motion will fight itself.
- **One easing system.** Scroll motion is a single lerp (`scroll += (target - scroll) * EASE`). The settle-snap redirects `target` only — it must never move `scroll` directly or introduce a second tween on it. (History: layered easing/snap systems here caused jumpy scrolling and were rebuilt twice.)
- **The FBO is sized in device pixels** (`W * dpr`), including in `onResize`. Sizing it in CSS pixels makes everything blurry on retina.
- **Textures need mipmaps + anisotropy** (set in the load callback). Panels render ~80px tall during entry; plain linear filtering looks mushy.
- **1 world unit = 1 px** (orthographic camera). All layout math assumes this.
- **Input gating:** wheel/click are ignored while `focusState.active || entryActive || entrySettled`. `focusState.active` stays true until the close timeline *finishes*, even though `onFocusChange(false)` fires at close *start* (so the overlay UI syncs with the returning cards). Preserve that ordering.
- **Panel indices can be any integer** — `centerForIndex`/`nearestIndex` use an unbounded index (`source = idx mod N`, loop = `floor(idx / N)`) so the row can target the nearest copy of a panel across the infinite wrap. Don't clamp them to `0..N-1`.

## Conventions

- Comment style: short, lowercase, practical (`// match page bg so FBO gaps blend`). Section markers are `// ---- name ----`. No JSDoc blocks, no banner rulers — the owner wants the code to read hand-written.
- Keep `config.js` values in sync with `gui.js` slider ranges when adding tunables, and mirror any new `LENS` key as a shader uniform.
- JS only (no TypeScript). Path alias `@/*` → repo root.

## Verify changes

```bash
npm run build     # compiles (Turbopack) — catches wiring mistakes
npx eslint Components lib
npm run dev       # feel-check scroll/entry/focus by hand; these are not unit-testable
```

---
> Source: [Yousuf-developer/liquid-glass-carousel](https://github.com/Yousuf-developer/liquid-glass-carousel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
