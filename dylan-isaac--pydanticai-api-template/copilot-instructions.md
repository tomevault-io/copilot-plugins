## pydanticai-api-template

> Documentation-first guidance for source files (auto-attached)

# Documentation-First Guidance

📝 **Rule Triggered:** Code modification/addition in `src/`.

**Core Principle:** Keep documentation (`README.md`, `docs/**/*.md`) as the source of truth. Ensure every feature/change is documented with purpose, examples, and assumptions.

**Action Required:**
- Based on the changes made, determine which documentation files need updates:
  - API endpoint changes (routes, request/response models)? -> Update @docs/API.md
  - Pydantic model changes (new models, fields, validation)? -> Update @docs/MODELS.md
  - Architectural changes (new components, interactions, libraries)? -> Update @docs/ARCHITECTURE.md
  - Core concepts or overall structure changes? -> Consider updates to @docs/OVERVIEW.md or @README.md
  - CLI command changes? -> Update @README.md
- Ensure documentation is clear, concise, and reflects the current state of the code.

**Reference:** See @docs/CURSOR_RULES.md for how rules integrate with docs.

**Decision Point:** Inform the user which documentation files likely need updating based on the code changes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dylan-isaac) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
