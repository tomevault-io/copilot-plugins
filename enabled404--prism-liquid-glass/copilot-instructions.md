## prism-liquid-glass

> Luxury WebGL interactive showcase: three.js + GSAP core with synthesized Web Audio and a thin React/Next.js wrapper.

# PRISM WebGL Engine — Architecture Notes

Luxury WebGL interactive showcase: three.js + GSAP core with synthesized Web Audio and a thin React/Next.js wrapper.

## File Map

| File | Owns |
| --- | --- |
| `lib/carousel/config.js` | `PROJECTS` catalog + extended metadata, category definitions, `LENS_PRESETS`, `CONFIG`, `LENS`, `FOCUS`, `ENTRY`, `UI_ANIM`. Data only, no logic. |
| `lib/carousel/engine.js` | Canvas rendering loop: WebGL renderer, infinite row, touch gesture physics, GLSL refraction shader with `uLensFx` dissolve, theme interpolation, focus mode, entry animation. |
| `lib/carousel/audio.js` | Synthesized Web Audio API sound engine (scrolling ticks, focus pops, sweep transitions, UI clicks). Zero external audio file dependencies. |
| `lib/carousel/gui.js` | Shader Studio panel (lil-gui). Mutates config objects live for design tuning. |
| `Components/Navbar.jsx` | Top luxury header overlay: brand identity, category filter chips, auto-tour toggle, theme preset selector, audio mute toggle, shader studio toggle. |
| `Components/ProjectModal.jsx` | Extended project case study drawer: project category, year, tech stack pills, performance metrics, descriptions, and direct live links. |
| `Components/CarouselSection.jsx` | Main React overlay controller: mounts WebGL canvas, top navbar, floating bottom focus control bar, modal drawer, and keyboard navigation shortcuts. |
| `app/page.js` | Renders `<CarouselSection />`. |

## Architecture Invariants

- **The engine talks to React only via callbacks** (`onActiveChange`, `onFocusChange`, `onEntryDone`) and the returned handle (`closeFocus`, `replayEntry`, `refreshLayout`, `setTheme`, `jumpToIndex`, `destroy`). Don't import React into the engine.
- **GSAP animates plain numbers, never meshes.** Timelines tween values in `drop[]`, `pEntry[]`, `growArr[]`, `focusScale`, `focusState.lensFx`; `layout()` reads them each frame and derives final mesh transforms.
- **Master Lens Opacity (`uLensFx`)**: Ensures the liquid glass shader dissolves smoothly to 0 opacity during focus mode without black lens artifacts or background darkening.
- **One Easing System.** Scroll motion is a single lerp (`scroll += (target - scroll) * EASE`). The settle-snap redirects `target` only.
- **The FBO is sized in device pixels** (`W * dpr`), including in `onResize`.
- **Textures need mipmaps + anisotropy** (set in the load callback).
- **1 world unit = 1 px** (orthographic camera).

---

Developed by Saad Khalid • Inspired by WebGL liquid glass concepts

---
> Source: [enabled404/prism-liquid-glass](https://github.com/enabled404/prism-liquid-glass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
