## astrasetu

> Work only inside this repository unless the user explicitly changes scope.

# AstraSetu Agent Instructions

Work only inside this repository unless the user explicitly changes scope.

## Product Standard

- Preserve AstraSetu as a premium, immersive, science-first product.
- Do not turn pages into generic card grids, template dashboards, or placeholder routes.
- Keep the visual system intentional: deep-space luxury, scientific precision, Indian scientific curiosity, calm mission-control intelligence.
- Maintain route distinction: Command Deck, Atlas, Missions, Lab, Observatory, Notebook, Quests, Settings, and About should not feel like renamed copies.

## Data Integrity

- Never invent live astronomy, mission, or sky data.
- Label data surfaces consistently: verified static data, educational simulation, demo data, seeded event data, local calculation, or future live integration.
- Keep demo and seeded content transparent in UI and docs.

## Engineering

- Use TypeScript and existing architecture boundaries.
- Keep provider interfaces clean so future database, auth, and live APIs can replace static providers.
- Do not add dependencies without clear product value.
- Preserve local-first behavior; no secrets or environment variables are required for v1.

## Accessibility and QA

- Keep semantic HTML, visible focus states, keyboard navigation, reduced-motion support, accessible labels, and sufficient contrast.
- Test important interactions before completion.
- Run `npm run typecheck`, `npm run lint`, `npm run test`, and `npm run build` before claiming completion.

## Visual QA

- Compare implementation against the stored concepts in `public/concepts/` when making major UI changes.
- Do not ship clipped text, broken mobile layouts, inert controls, fake charts, or dead buttons.

---
> Source: [SatyajitBeura2468/AstraSetu](https://github.com/SatyajitBeura2468/AstraSetu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
