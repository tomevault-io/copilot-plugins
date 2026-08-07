## features-bento-mobile-layout

> Preserve mobile Features bento grid layout — do not change classes on the four-card grid without explicit user request


# Features bento mobile layout (frozen)

The four-card bento block in `Features.jsx` uses a fixed staggered mobile layout:

- Grid: `features-bento-grid grid h-[135vh] w-full grid-cols-2 grid-rows-3 gap-7`
- Feature 3 tilt: `ms-32 md:col-span-1 md:ms-0` (stagger on mobile — never add `max-md:ms-0`)
- Feature 4 tilt: `me-14 md:col-span-1 md:me-0` (stagger on mobile — never add `max-md:me-0`)

**Do not change** these grid/tilt classes when fixing copy, video, or overlay bugs. Only adjust `BentoCard` internals (e.g. absolute overlay for text on `LazyVideo`).

If the user asks to change mobile layout, they must say so explicitly.

---
> Source: [bistutzyy/taozhiyy](https://github.com/bistutzyy/taozhiyy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
