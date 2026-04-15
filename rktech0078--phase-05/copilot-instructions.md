## phase-05

> These rules are derived from `QWEN.md` and MUST be followed for every interaction.

# AI Agent Global Rules

These rules are derived from `QWEN.md` and MUST be followed for every interaction.

## 1. Prompt History Records (PHR)
**Mandatory**: After completing any request (implementation, planning, debugging), you **MUST** create a PHR in `history/prompts/`.
- Use the template at `.specify/templates/phr-template.prompt.md`.
- Route to `history/prompts/general/` by default, or specific feature folders.
- Increment ID based on existing files.

## 2. Source of Truth
- **Never assume**: Always verify with tools (grep, view_file, etc.).
- **Files over Memory**: Read the file content before editing.

## 3. Architecture Decision Records (ADR)
- Suggest an ADR when significant architectural decisions are made.
- Do not auto-create; ask user for consent.

## 4. Environment & Secrets
- Never hardcode secrets.
- Use `.env.local` for local development.
- Instruct user to set production variables for deployment.

## 5. Human as Tool
- Ask clarifying questions if requirements are ambiguous.
- Do not guess; ask.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rktech0078) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
