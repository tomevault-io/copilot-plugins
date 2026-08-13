## react-inertia

> Authenticated dashboard — React, Inertia, TypeScript, Tailwind


# React / Inertia Dashboard

- **Scope**: Authenticated app only. No Blade in dashboard pages.
- **Stack**: React + Inertia.js + TypeScript.
- **Styling**: Tailwind only. Conditional classes: `clsx`. Every component accepts `className`.
- **Structure**: Reusable UI (tables, buttons, forms, modals). Logic in hooks/services; global state via Context or Zustand (Redux only if justified).
- **A11y & perf**: WCAG 2.1; avoid unnecessary re-renders.
- **Tests**: Add/maintain tests for main components when behavior changes.
- **UI copy**: Italian labels, messages, placeholders.
- **No ops jargon in UI**: mai banner, tooltip o CTA con comandi da terminale (`artisan`, shell). Manutenzione dati (sync, realign, migrate extra) → `docker/entrypoint.sh` al deploy o observer/servizi in background, invisibile all’utente.

After TS/JS changes: `make test` (if PHP tests cover), `make pint-check` (if PHP touched), `make playwright` per `agent-workflow`.

---
> Source: [mnossa/finanzamente](https://github.com/mnossa/finanzamente) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
