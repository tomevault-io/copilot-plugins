## frontend-standards

> Frontend conventions for Agate UI, Stylebook UI, and shared TS/React UI


# Frontend Standards

- **User-visible copy:** write labels, errors, tooltips, empty states, and dialogs for a **non-technical end user**. **Avoid technical or code-related language** (API routes, field names, stack traces, env vars, DB terms, etc.) unless the screen is explicitly developer-only. See `docs/development/frontend/conventions.md` → **User-facing copy** and `AGENTS.md` → **Engineering posture**.
- Keep components readable and explicit; extract smaller helpers or components when files become hard to scan.
- Prefer typed props and API response shapes over `any`.
- Reuse shared API helpers, UI primitives, and existing patterns before adding a new abstraction.
- Keep browser storage keys and custom event names consistent with existing prefixes and docs.
- Do not hand-edit generated node registry output unless the generation flow itself is changing.

---
> Source: [localangle/backfield](https://github.com/localangle/backfield) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
