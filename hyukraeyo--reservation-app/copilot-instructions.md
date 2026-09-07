## reservation-app

> - **Styling**: strictly use **SCSS Modules** (`.module.scss`). **DO NOT use Tailwind CSS**.


# Agent Rules

## Technology Preferences
- **Styling**: strictly use **SCSS Modules** (`.module.scss`). **DO NOT use Tailwind CSS**.
- **Framework**: Next.js 15 (App Router).
- **Language**: TypeScript (Strict Mode).

## Design Philosophy
- **Mobile-First & Responsive**:
  - All designs must primarily target mobile devices (320px - 480px width) as the default state.
  - Use media queries (`@media (min-width: ...)` ) to adapt for tablets and desktops.
  - Verify UI elements do not overflow or break on small screens.
- **Premium Aesthetics**:
  - Use high-quality fonts, smooth transitions, and modern spacing.
  - Avoid "default HTML" looks.

## Coding Standards
- **Imports**: Use absolute imports (`@/`) where possible.
- **Components**:
  - Prefer Server Components for data fetching.
  - Use `'use client'` only when hook usage or interactivity is required.
- **PWA**:
  - Always consider offline states and touch interactions.
  - Ensure touch targets are at least 44x44px.

---
> Source: [hyukraeyo/reservation-app](https://github.com/hyukraeyo/reservation-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
