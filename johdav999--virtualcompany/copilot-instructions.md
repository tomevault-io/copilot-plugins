## virtualcompany

> When changing backend, data, workflow, agent, integration, approval, or AI orchestration code:

## Architecture



When changing backend, data, workflow, agent, integration, approval, or AI orchestration code:



- MUST follow `/docs/architecture-rules.md`

- Use `/docs/architecture-overview.md` only as background context

- Existing repository implementation wins if it conflicts with older planning documents



## Design / UI

When changing frontend UI, layout, styling, dashboards, agent cards, approval views, finance views, or user-facing text:

- MUST follow `/docs/design.md`
- Reuse existing design tokens, components, spacing, typography, colors, and interaction patterns defined there
- Do not introduce a new visual style unless explicitly requested
- Existing implemented design system wins if it conflicts with older planning documents
- Keep user-facing language plain English; avoid internal workflow names, enum names, or technical identifiers

## Multi-prompt implementation persistence

When implementing a sequence of prompts or a multi-part implementation plan:

- Do not stop at intermediate checkpoints.
- Do not stop after only reporting build, test, or analysis status.
- Continue through the ordered prompts until the full requested implementation sequence is complete, genuinely blocked, or the user explicitly asks to pause or stop.
- If a build or test fails, diagnose and fix the failure when it is in scope for the current implementation sequence, then continue.
- Only stop when further progress requires missing external credentials, unavailable services, an unsafe/destructive action without approval, or a user decision that cannot reasonably be inferred.

<!-- design-addin-instructions:start -->
# Design Add-in Workspace Instructions

- When a prompt asks for architecture-sensitive implementation, include and follow `architecture-inst.md`.
- When a prompt asks for UI, UX, styling, layout, components, navigation, or frontend implementation, include and follow `ui-instructions.md` if it exists.
- Always include and follow `production-implementation.md` in every prompt.
- Treat these generated instruction files as project context. If they conflict with explicit user instructions, follow the user and note the conflict.
<!-- design-addin-instructions:end -->

---
> Source: [johdav999/VirtualCompany](https://github.com/johdav999/VirtualCompany) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
