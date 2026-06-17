## portfolio-1

> Use this skill when editing the Hunter Ho Portfolio_1 site, especially UI, loading, image, and performance work.

# Portfolio_1 Update Skill

Use this skill when editing the Hunter Ho Portfolio_1 site, especially UI, loading, image, and performance work.

## Read First

1. Read `AGENTS.md`.
2. Check CX through `/Users/exchange/docs/cx/index.md`.
3. Inspect current files with `rg` before assuming prior state.

## Standard Workflow

1. Identify the exact section or behavior the user is commenting on.
2. Add or update a focused regression in `tests/portfolio-content.test.js`.
3. Run `pnpm test` and confirm the new test fails for the intended reason.
4. Make the smallest implementation change in `index.html`, `css/style.css`, or `js/main.js`.
5. Run:
   - `pnpm test`
   - `node --check js/main.js`
   - `git diff --check`
6. Verify rendered behavior in the in-app browser on desktop and mobile when layout or loading changes.
7. Save a compact CX note if the change creates reusable project knowledge.

## Loading And Media Checklist

- Head may contain the inline theme bootstrap only; no external script tags in the head.
- Use `display=swap` for Google Fonts so text is visible first.
- Preload only hero-critical imagery.
- Hero source image must use `loading="eager"`, `decoding="async"`, and `fetchpriority="high"`.
- Below-fold images and YouTube iframes must use `loading="lazy"`.
- Optional plugins load through `loadScriptOnce()` in `js/main.js`.
- If new generated images are added, prefer WebP and keep source dimensions explicit.

## Section-Specific Notes

- Hero: preserve phrase-stable headline rotation, special word effects, Option C 3D scene, hero blur placeholder, and the delayed Hunter popup.
- Startup Lab: keep it focused as portfolio proof, not a full startup website. Use one video, sound note, and seven compact startup icons.
- Contact/footer: keep real contact text as HTML over `images/founder-banner.jpeg`.
- Projects/games/3D models: use real demo links and lazy thumbnail images.

---
> Source: [HunterHo07/Portfolio_1](https://github.com/HunterHo07/Portfolio_1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
