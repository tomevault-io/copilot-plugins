## x-md

> - Pages: `index.html` (`src/main.ts`), `docs.html` (`src/docs.ts`)

# Agent notes

## Frontend structure

- Pages: `index.html` (`src/main.ts`), `docs.html` (`src/docs.ts`)
- Shared header/footer/mobile menu: `src/chrome.ts`
- Design tokens live in the Tailwind v4 `@theme` block in `src/style.css` (semantic colors: `ink`/`ink-2`/`ink-3`/`ink-4`, `surface`, `raised`, `raised-2`, `line`, `hair`, `accent`, `accent-deep`, `accent-soft`, `cyan`, plus dark `code`/`code-ink`/`code-dim` for code panes). Use these utilities instead of hard-coded hex values in markup.
- Visual language: light warm paper (`#f7f6f2`) with Satoshi (Fontshare), deep green accent (`#146c43`), pill buttons, 16px-radius cards, and dark near-black code panes for contrast. No pcstyle.dev styling.
- Landing copy frame (per Theo feedback): "Tweets are just markdown now", what-you-see vs what-your-agent-sees curl comparison in the bento under `#how`, skill install under `#agents`. No tech-stack/provider-chain talk on the landing page.
- Homepage motion uses GSAP + ScrollTrigger (`setupMotion` in `src/main.ts`): hero stagger, scrubbing word reveal (`[data-scrub-text]`), card rises (`[data-rise-card]`), and a desktop-only pinned split (`[data-pin-section]`/`[data-pin-target]`). All motion is gated behind `prefers-reduced-motion` via `gsap.matchMedia`.
- `/docs` is rewritten to `/docs.html` in `vercel.json` and registered as a Rollup input in `vite.config.ts`.

---
> Source: [pc-style/x-md](https://github.com/pc-style/x-md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-05 -->
