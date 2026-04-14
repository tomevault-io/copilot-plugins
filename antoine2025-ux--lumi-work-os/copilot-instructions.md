## lumi-work-os

> Authoritative AI Coding Constraints

# LOOPWELL .cursorrules
Authoritative AI Coding Constraints

This file defines non-negotiable rules for AI-assisted development in Loopwell.
If a rule exists here, it must be followed.
If a recurring issue appears, add a rule here.

AI must read this file before writing or modifying code.

---

## 1. GOLDEN RULE: COMPOUNDING ENGINEERING

- If you (the AI) make a mistake that requires correction, you must explicitly suggest whether a new rule should be added to this file to prevent recurrence.
- Repeated issues must become rules.
- This file is not documentation. It is enforcement.

---

## 2. PLANNING BEFORE EXECUTION (MANDATORY)

- For any task beyond a trivial change:
  - Start by proposing a clear plan.
  - List exact file paths to be modified.
  - Explain why the approach fits Loopwell's architecture.
  - Call out risks and edge cases.
- Do not write code until the plan is explicitly approved.

---

## 3. LOOPWELL ARCHITECTURE (NON-NEGOTIABLE)

Assume and respect the following stack at all times:

- Next.js 15 (App Router)
- TypeScript (strict)
- Prisma + PostgreSQL
- NextAuth / Auth.js
- Tailwind CSS with shadcn-style components
- Clean structure under:
  - `src/app/`
  - `src/components/`
  - `src/lib/`

Rules:
- Do not invent new architectural patterns.
- Do not move files without justification.
- Do not introduce new layers unless explicitly requested.

---

## 4. TYPE SAFETY & CODE QUALITY

- TypeScript strictness is mandatory.
- `any` is forbidden unless explicitly justified and approved.
- All code must include:
  - Proper null checks
  - Safe fallbacks
  - Defensive error handling
- Never silence TypeScript errors by weakening types.

---

## 5. NO DUPLICATION, NO CLUTTER

- Do not create duplicate components, hooks, utilities, or API routes.
- Reuse existing patterns and abstractions.
- If similar logic exists, extend or refactor it.
- Remove unused imports, dead code, and temporary scaffolding created during the task.

---

## 6. LOOPBRAIN COMPATIBILITY (CRITICAL)

Loopwell is built for contextual AI.

Rules:
- Every feature must expose structured, machine-readable data.
- Avoid ambiguous or loosely structured data models.
- Preserve clear relationships between:
  - People
  - Roles
  - Tasks
  - Projects
  - Docs
  - Activity
- Do not hide important context in unstructured blobs unless explicitly requested.

### Evidence Values

Evidence values exposed to Loopbrain must be shallow, JSON-serializable primitives or flat objects. No nested or opaque blobs.

### Blocker Priority

Blocker priority must be defined in exactly one canonical module and reused everywhere. See `src/lib/loopbrain/contract/blockerPriority.v0.ts`.

### Org Semantic Snapshot (S1+)

OrgSemanticSnapshotV0 is a machine contract, not a UI model.
- No UI component may depend on derived or reformatted fields from this snapshot.
- UI may only display snapshot data, never reinterpret it.
- Do not add or change snapshot fields for UI convenience.

---

## 7. VERIFICATION IS REQUIRED

After implementing changes, you must:

- Explain how the change is verified.
- Use the terminal when relevant.
- Run at least one of:
  - Typecheck
  - Lint
  - Tests
  - Build

Rules:
- Straightforward compile or lint errors may be fixed automatically.
- If fixing an error requires changing behavior, types, or architecture:
  - Pause
  - Explain the proposed fix
  - Wait for confirmation

---

## 8. LONG-RUNNING OR HIGH-RISK TASKS

For tasks that:
- Touch many files
- Affect core flows
- Risk architectural drift

You must:
- Pause after major steps
- Wait for confirmation before continuing
- Never assume correctness without verification

---

## 9. UI & DESIGN CONSISTENCY

- Use existing UI components from `src/components/ui`.
- Follow established layout and spacing patterns.
- Do not introduce new design tokens or styling conventions without approval.
- Avoid visual cleverness that reduces clarity.

---

## 10. DATABASE & PRISMA RULES

For any schema or data-layer changes:

- Validate Prisma schema changes.
- Ensure relations are explicit and correct.
- Generate safe migrations.
- Clearly explain:
  - Why the change is needed
  - Impact on existing data
- Never allow silent schema drift.

---

## 11. DEFAULT BEHAVIOR WHEN UNSURE

If requirements are unclear:
- Ask clarifying questions.
- Propose alternatives.
- Recommend the simplest, safest option.
- Do not guess.

---

## 12. DATABASE MIGRATIONS (CRITICAL)

- **Never use `prisma db push`** for schema changes in development. It modifies the database directly without creating migration files, causing staging/production drift.
- **Always use `prisma migrate dev --name <description>`** to create schema changes. This generates a migration file that the deployment pipeline (`prisma migrate deploy`) can apply to all environments.
- `prisma db push` is only acceptable for:
  - One-time emergency fixes against a specific database (with explicit justification)
  - CI test environments that are ephemeral
- After any schema change, verify the migration file exists in `prisma/migrations/` before committing.

---

## 13. FINAL RULE

Never "just do the thing."

Always:
- Plan
- Explain
- Execute
- Verify

AI speed comes from verification, not generation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/antoine2025-ux) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
