## smart-draw

> These guidelines apply to the entire `smart-diagram` repository.

## Scope

These guidelines apply to the entire `smart-diagram` repository.

---

## General Principles

- Prefer small, focused changes that match existing patterns.
- When in doubt, follow existing code in the same folder as the primary reference.

---

## Languages, Frameworks & Architecture

- Framework: Next.js App Router (see `app/layout.js`, `app/page.js`, `app/draw/page.js`).
- Runtime: React (function components, client/server components via `"use client"`).
- Styling: Tailwind CSS utility classes plus a few CSS files (for example `components/LandingPage.css`).
- UI: Custom components under `components/` and `components/ui/` (for example `Button`, `Textarea`, `Card`, `ScrollArea`).
- LLM & streaming:
  - Frontend builds complete `messages` arrays and `config` objects.
  - Backend proxy is `app/api/llm/stream/route.js` using SSE (`text/event-stream`).
- Diagram engines:
  - Draw.io: hooks and logic in `hooks/useDrawioEngine.js`, prompts in `lib/prompts/drawio.js`.
  - Excalidraw: hooks and logic in `hooks/useExcalidrawEngine.js`, prompts in `lib/prompts/excalidraw.js`.
  - Shared config and history logic in `lib/config.js`, `lib/config-manager.js`, `lib/history-manager.js`, `lib/indexeddb.js`.

---

## Code Style

- Use ES modules everywhere (`import` / `export`), no CommonJS.
- Use single quotes in JavaScript/JSX where practical, except when matching existing double-quoted imports in a file.
- Prefer descriptive variable and function names; avoid single-letter names except for trivial indices.
- Keep React components as pure as possible; side effects belong in `useEffect` or dedicated hooks.
- Co-locate related logic:
  - React components in `components/`.
  - Hooks in `hooks/`.
  - Reusable non-React utilities in `lib/`.
  - API entrypoints under `app/api/**/route.js`.
- For new files, mirror the structure and style of nearby files in the same directory (indentation, quote style, helper patterns).
- Do not add license or copyright headers.

---

## React & Components

- Use function components with hooks; do not introduce class components.
- Mark client components explicitly with `"use client";` at the top when they use hooks, browser APIs, or context.
- Keep props objects simple and explicit; prefer named props over spreading arbitrary objects.
- For new UI, prefer reusing existing primitives: `Button`, `Textarea`, `Card`, `ScrollArea`, etc.
- Preserve Tailwind class naming patterns already used in the component or folder.

---

## Hooks & Business Logic

- Put engine-specific logic into the corresponding hook (`useDrawioEngine`, `useExcalidrawEngine`) instead of UI components where possible.
- When working with Draw.io XML:
  - Use `fixUnclosed` from `lib/fixUnclosed.js` for repairing malformed XML.
  - Keep post-processing behaviour consistent with `postProcessDrawioCode` in `hooks/useDrawioEngine.js`.
- When optimizing Excalidraw arrows or geometry, follow the style and helper patterns in `lib/optimizeArrows.js`.
- New utility functions should go into an appropriate file under `lib/` or a new file within `lib/` rather than being inlined inside components if reused.

---

## API Routes & LLM Integration

- Keep API routes small and focused, with clear parameter validation and error handling:
  - Validate request payloads and return `NextResponse.json` with appropriate status codes for errors.
  - Log unexpected errors with `console.error` including context where helpful.
- For streaming responses:
  - Use `ReadableStream` with `text/event-stream` as in `app/api/llm/stream/route.js`.
  - Use a consistent SSE format: `data: <json-stringified-payload>\\n\\n` and terminate streams with `data: [DONE]\\n\\n`.
- Do not hard-code API keys; always rely on config objects and environment variables.

---

## Configuration & Persistence

- Centralize user-configurable settings in `lib/config.js` and `lib/config-manager.js`.
- Use `lib/indexeddb.js` and `lib/history-manager.js` for history and local persistence; do not access IndexedDB directly from random components.
- When adding new persisted entities, follow the existing object store naming and versioning patterns.

---

## Documentation

- When you add user-facing features or change workflow significantly, update:
  - `README.md` / `README_EN.md` for high-level usage.
  - `docs/prd.md` if the change affects core product behaviour or requirements.
- Keep documentation bilingual only if it already is for that section; do not mix languages arbitrarily.

---

## Tests & Validation

- **Do NOT add, modify, or run automated tests as part of routine code changes for this project.**
- After you finish code changes, you **do not need to consider tests or test-related tasks**:
  - Do not create new test files.
  - Do not update existing test suites.
  - Do not run `npm test`, `pnpm test`, or similar commands.
- It is acceptable (and preferred) to rely on manual validation via the development server (e.g. `pnpm dev` / `npm run dev`) when necessary.

---

## Tooling

- ESLint is configured via `eslint.config.mjs`; follow existing patterns to avoid large rule changes.
- Only introduce new dependencies when strictly necessary and after checking that a similar utility does not already exist in `lib/` or `components/ui/`.
- For formatting, follow the style of the surrounding file; do not introduce a new formatter configuration.

---
> Source: [liujuntao123/smart-draw](https://github.com/liujuntao123/smart-draw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
