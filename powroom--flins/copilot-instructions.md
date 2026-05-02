## flins

> Use **Bun** exclusively.

Use **Bun** exclusively.
Do **NOT** use npm, pnpm, or yarn under any circumstances.

## Plan Mode

When operating in Plan Mode:

- Produce concise, execution-focused plans.
- Clarity is required; verbosity is not.
- Do not include explanations, prose, or filler.
- Output only concrete steps.
- Every plan **MUST** end with a section titled **“Unresolved Questions”**.
- Include only questions that materially block execution.
- If there are no blockers, explicitly write:

  **Unresolved Questions: None.**

Failure to follow this structure is not allowed.

## Code Style & Generation Rules

All generated code must follow these rules:

- Code must be self-explanatory.
- Do not use comments.
- Intent must be expressed through naming, structure, and composition only.

Strict consistency is required across:

- File structure
- Module boundaries
- Naming conventions
- Architectural patterns

Before writing any code:

- Explore the existing project.
- Understand the directory layout, conventions, abstractions, and dependency patterns.

When adding new code:

- Place it only in logically correct locations.
- Do not introduce new patterns unless strictly necessary.

## Mandatory Skill Enforcement (Zero Tolerance)

All rules below are **NON-NEGOTIABLE**.

If any applicable skill is not used, the response is **INVALID**.

There are:

- No excuses
- No interpretation
- No flexibility

Rules:

- If a condition applies → use the skill.
- If unsure → assume it applies and use the skill.
- If multiple conditions apply → use all relevant skills.
- Do not proceed with an answer until all required skills are applied.

## Skill Requirements

**Planning & Scope**

- **Skill(avoid-feature-creep)**
  Required for feature planning, scope discussion, MVPs, roadmaps, and backlogs.

**CLI**

- **Skill(commander-guidelines)**
  Required for any CLI with commands, flags, options, arguments, or subcommands.
- **Skill(clack-guidelines)**
  Required for interactive CLI behavior: prompts, confirmations, selections, spinners, progress, autocomplete.

**Marketing & SEO**

- **Skill(copy-editing)**
  Required for editing, reviewing, or improving existing marketing copy.
- **Skill(copywriting)**
  Required for writing new marketing copy from scratch.
- **Skill(programmatic-seo)**
  Required for building SEO pages at scale using templates and data.
- **Skill(seo-audit)**
  Required for auditing, reviewing, or diagnosing SEO issues.

**Convex**

- **Skill(Convex Functions)**
  Required for any query, mutation, action, or HTTP action.
- **Skill(Convex Schema Validator)**
  Required for defining or modifying schemas and tables.
- **Skill(Convex Migrations)**
  Required for schema evolution and data backfills.
- **Skill(Convex Security Check)**
  Required for quick or high-level security reviews.
- **Skill(Convex Security Audit)**
  Required for deep or production-level security reviews.
- **Skill(Convex HTTP Actions)**
  Required for APIs, webhooks, and HTTP endpoints.
- **Skill(Convex File Storage)**
  Required for file upload, download, storage, serving, or deletion.
- **Skill(Convex Realtime)**
  Required for subscriptions, realtime updates, pagination, or optimistic UI.
- **Skill(Convex Cron Jobs)**
  Required for scheduled or background jobs.
- **Skill(Convex Component Authoring)**
  Required for reusable or isolated Convex components.
- **Skill(Convex Best Practices)**
  Required for production-ready architecture or long-term system design.
- **Skill(Convex Agents)**
  Required for persistent agents or long-running AI workflows.

## Final Warning

If a response matches a condition and does **not** apply the required skill:

**STOP. DO NOT ANSWER. FIX THE RESPONSE.**

This is enforcement, not guidance.

---
> Source: [powroom/flins](https://github.com/powroom/flins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
