## website-sakura-love

> The site supports 5 design styles (apple/linear/spotify/figma/notion) switchable at runtime. Switching must be smooth — no frame drops, no stutter.

# CLAUDE.md — website_Sakura_Love

## Style switching (theme/design) — do not regress

The site supports 5 design styles (apple/linear/spotify/figma/notion) switchable at runtime. Switching must be smooth — no frame drops, no stutter.

### Two-path architecture

```
applyStyleSmooth(style)
  ├─ prefers-reduced-motion → applyStyle() directly, skip animation
  ├─ document.startViewTransition (Chromium) → View Transitions API crossfade
  └─ fallback (Firefox/Safari) → CSS transition class toggling
```

### JS (script.js)

- `applyStyle(k)` — atomic class swap via `classList.replace(oldClass, newClass)`, toggles `theme-dark`, then calls `syncRootThemeTokens`
- `syncRootThemeTokens(safe)` — sets `backgroundColor`, `color`, `colorScheme` on `:root` (html element). **Do NOT set `--canvas` or `--text` here** — those are defined by `body.design-*` CSS classes, and inline settings on `:root` are either dead (overridden by body) or cause redundant style recalc. Removed 2026-05-25.
- `syncRootThemeTokens` is called AFTER the class swap, so the CSS body class provides the custom property values first, then the html root element gets its background/text color.
- `applyStyleSmooth(style, afterApply)` — picks the right animation path
  - View Transitions path: wraps `applyStyle()` + `afterApply()` inside `document.startViewTransition(callback)`
  - CSS fallback: adds `style-transitioning` class, then `requestAnimationFrame(() => { applyStyle(); afterApply(); endStyleTransition() })`
- `beginStyleTransition()` / `endStyleTransition()` — adds/removes `style-transitioning` on `<body>`, auto-cleans after 440ms

### CSS (styles.css)

- `body.style-transitioning *` — 320ms transitions on `color, background-color, border-color, box-shadow` only. **Do NOT add `transform` or `opacity`** to this list.
- **Do NOT add `will-change` to `body.style-transitioning` elements.** Past attempt to add `will-change: transform, opacity` to 18+ sections (header, hero, cards, panels, footer, etc.) caused heavy GPU layer allocation that was the primary source of stutter. Removed 2026-05-25.
- View transition animations: 400ms `cubic-bezier(0.4, 0, 0.2, 1)` fade-out/fade-in keyframes (not browser default 200ms crossfade — too abrupt).
- `.notransition` class — suppresses all transitions/animations, used as escape hatch.

### Why this works

- View Transitions API takes a screenshot before and after, animates between them — zero layout cost.
- CSS fallback only transitions 4 cheap properties (color, bg, border, shadow) — no layout triggers.
- `classList.replace()` is one DOM mutation instead of add+remove — the browser sees one style change, not two.
- Custom properties on `:root` change once, then all descendants transition in parallel via the `*` rule.

### Cold-start compositor warmup (2026-05-25)

After page load, the browser's GPU compositor is cold — no transition infrastructure, no composited layers cached. The first style switch forces the browser to set up transition tracking AND create GPU layers AND animate simultaneously, causing stutter. Subsequent switches are smooth because everything stays warm.

Fix: after initial `applyStyle()` on page load, do a throwaway `style-transitioning` toggle (add class → double-rAF → remove class). This forces the browser to pre-build transition pipelines and warm the compositor. The user never sees it because no properties actually change during the warmup.

```js
const _warmupCompositor = () => {
  if (!document.body.classList.contains("style-transitioning")) {
    document.body.classList.add("style-transitioning");
    requestAnimationFrame(() => requestAnimationFrame(() => {
      document.body.classList.remove("style-transitioning");
    }));
  }
};
setTimeout(_warmupCompositor, 600);
window.addEventListener("load", () => setTimeout(_warmupCompositor, 400), { once: true });
```

Both `setTimeout` (catches early-ready pages) and `load` event (catches resource-heavy pages) — only one will fire since the guard prevents double-warmup.

---
> Source: [SakuraLoveForever/website_Sakura_Love](https://github.com/SakuraLoveForever/website_Sakura_Love) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
