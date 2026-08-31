## akb-crm

> - Stack: Vite + React + TypeScript, TanStack Start, Drizzle, Base UI, and Vite build tooling.

# Copilot Instructions

## Project context

- Stack: Vite + React + TypeScript, TanStack Start, Drizzle, Base UI, and Vite build tooling.
- Avoid introducing new major dependencies unless requested.
- Keep changes minimal and scoped; preserve existing patterns and conventions.

## Style and formatting

- Follow existing TypeScript/React patterns; prefer function components and hooks.
- Favor explicit types at module boundaries; keep internal inference when clear.
- Use Prettier defaults; no custom formatting. Use ASCII unless the file already uses otherwise.
- Keep comments concise and only where they add clarity for non-obvious code.

## React specifics

- Keep components small and focused; lift shared logic to utilities when reused.
- Prefer controlled inputs and explicit state; avoid unnecessary re-renders.
- Handle loading/error/empty states for UI that hits async data.

## Data and API

- Keep Drizzle schema changes backwards compatible when possible.
- Validate inputs on the server and client; avoid trusting route params blindly.

## Testing and safety

- Add or update tests when behavior changes; avoid breaking existing tests.
- Be cautious with migrations or schema changes; call them out clearly.

## Review mindset

- When asked for a review, lead with findings (bugs, risks, regressions, missing tests) before summaries.
- Provide next steps or verification guidance after code changes.

---
> Source: [h4wl/akb-crm](https://github.com/h4wl/akb-crm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
