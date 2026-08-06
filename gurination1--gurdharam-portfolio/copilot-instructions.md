## gurdharam-portfolio

> - Vite + React + TypeScript portfolio app.

# Gurdharam Portfolio Agent Guide

## Project Shape

- Vite + React + TypeScript portfolio app.
- Main application code lives in `src/main.tsx`.
- Global styling and responsive layout live in `src/styles.css`.
- Static portfolio assets live in `public/assets/portfolio/`.
- `dist/` is build output. Do not edit generated files directly; run `npm run build` after source changes.

## Working Rules

- Prefer small, source-level edits that preserve the existing visual language: dark editorial portfolio, gold/blue/green accents, large display typography, glassy AI-native sections, and animated 3D/Spline hero.
- Use existing components in `src/components/ui/` before adding new abstractions.
- Keep responsive behavior explicit. Check desktop, tablet, and mobile breakpoints when changing layout.
- Do not add secrets, API tokens, or private credentials to source, docs, screenshots, or build output.
- Avoid unrelated refactors. This project is animation-heavy, so layout changes should be narrow and easy to verify.
- Use the installed `caveman` skill for concise routine updates when brevity helps, while preserving exact technical details and warnings.
- For code reviews, use the installed `code-review` skill with its `code-review-*` sub-skills. Map changed files, dependency impact, and visual/UI risk first; then report findings first with file and line references.
- Use `caveman-review` only for terse PR-style comments when that format is explicitly useful.

## Commands

- Install dependencies: `npm install`
- Develop locally: `npm run dev`
- Production build: `npm run build`
- Preview build: `npm run preview -- --port 4173`

## Verification

- Always run `npm run build` after UI or source changes.
- For visual changes, inspect at least:
  - Desktop: 1440px wide
  - Tablet: around 768px wide
  - Mobile: around 390px wide
- If browser automation is unavailable in the environment, state that clearly and verify the changed source plus build output.

---
> Source: [gurination1/Gurdharam-Portfolio](https://github.com/gurination1/Gurdharam-Portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
