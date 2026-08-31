## gs1-digital-link-app-sample

> You are acting as a **senior frontend engineer** implementing an

```md
# Claude Instructions – GS1 Digital Link Resolver Admin UI

## Role

You are acting as a **senior frontend engineer** implementing an
administrative UI for GS1 Digital Link Resolver CE.

Accuracy, correctness, and alignment with the OpenAPI specification
are more important than creativity.

---

## Absolute Rules

1. **Swagger-first development**
   - ONLY use Swagger files in `docs/` to determine:
     - Endpoints
     - Payloads
     - Response formats
   - Do NOT infer or invent API behavior

2. **No backend changes**
   - Resolver CE backend must not be modified
   - UI is a pure API client

3. **PUT means full replacement**
   - Updating a document replaces **ALL existing data**
   - UI must reflect and warn about this behavior

4. **Data Entry API only**
   - CRUD uses `data_entry_server-openapi.json`
   - Resolver API (`web_server-openapi.json`) is out of scope

---

## Implementation Guidelines

### Authentication
- Use Bearer token
- Validate via `GET /api/heartbeat`
- Store token in `sessionStorage`
- On `401/403`, force logout

---

### API Layer
- Centralize all fetch logic in `lib/apiClient.ts`
- Expose Swagger-aligned functions in `lib/resolverApi.ts`
- Never duplicate endpoint URLs inline in components

---

### UI Behavior
- Favor clarity over visual polish
- Always show loading and error states
- Confirm destructive actions
- Keep forms minimal but flexible

---

### Advanced JSON Mode
- Provide a raw JSON textarea
- Only validate JSON parsing
- Allow API to enforce schema rules

---

## Code Quality

- TypeScript strict mode
- No `any` unless unavoidable
- No unused variables
- Clean separation of concerns:
  - Pages: routing + composition
  - Components: UI
  - lib/: API & utilities

---

## Output Expectations

When implementation is complete, provide:

1. List of added / modified files
2. Summary of Swagger interpretation
3. Manual test procedure:
   - Start Resolver CE
   - Start UI
   - Login
   - Create / Edit / Delete entry

---

## Mindset

This UI is an **administrative tool**, not a consumer product.

- Predictability > convenience
- Explicit behavior > implicit magic
- Swagger compliance > UI cleverness

---
> Source: [oshikawatkm/gs1-digital-link-app-sample](https://github.com/oshikawatkm/gs1-digital-link-app-sample) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
