## research-paper-tracker

> All development MUST strictly follow the specifications in:

# Cursor Rules for Research Paper Reading Tracker

## Mandatory Documentation Adherence

All development MUST strictly follow the specifications in:
- `docs/BRD.md` (Business Requirements Document)
- `docs/TAD.md` (Technical Architecture Document)
- `docs/UI_UX_SPECS.md` (UI/UX Specifications)
- `docs/API_CONTRACTS.md` (API Contracts)

## Core Development Principles

### 1. No Feature Invention
- Implement ONLY what is explicitly defined in the documentation
- Do not add features, fields, or functionality beyond the documented requirements
- If a requirement is unclear, STOP and ask for clarification

### 2. No Enum or String Duplication
- All enums MUST be defined once in `packages/shared/enums/`
- Both frontend and backend MUST import from the shared package
- Never duplicate enum values or constants across packages

### 3. Controller Responsibilities
- Controllers are for request orchestration ONLY
- Controllers MUST NOT contain business logic
- All business logic belongs in service layers

### 4. Standardized API Responses
All API responses MUST follow this exact structure:
```json
{
  "code": "STRING_CODE",
  "data": {},
  "message": "Human readable message"
}
```

### 5. TypeScript Only
- All code must be written in TypeScript
- No JavaScript files allowed
- Maintain strict type safety

### 6. Clarification Protocol
- If requirements are ambiguous or missing, STOP
- Do not make assumptions
- Ask for explicit clarification before proceeding

## Architecture Enforcement

- Respect the monorepo structure defined in TAD.md
- Maintain clear separation between frontend, backend, and shared packages
- Follow the layered architecture pattern (routes → controllers → services → data layer)

## Code Quality Standards

- No hidden state or side effects
- Deterministic behavior required
- Clean, modular, and reviewable code
- Proper error handling following API_CONTRACTS.md


DEPENDENCY RULES (MANDATORY):

1. Always use the latest stable versions of libraries available at the time of writing.
2. Do NOT use legacy or deprecated versions unless explicitly instructed.
3. Prefer:
   - Express 5.x latest
   - TypeScript latest stable
   - Node.js 22+ compatibility
4. If a version is unspecified, choose the latest stable release.
5. Do NOT pin unnecessary patch versions unless required.
6. Workspace packages must be referenced using "workspace:*".
7. If dependency versions appear outdated, update them proactively.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BHUPESH003) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
