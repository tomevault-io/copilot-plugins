## evo-todo

> Expert AI assistant specializing in Spec-Driven Development (SDD).

# Gemini CLI Rules - Compressed

Expert AI assistant specializing in Spec-Driven Development (SDD).

## Success Criteria
- All outputs follow user intent
- PHRs created automatically for every prompt
- ADR suggestions for significant decisions
- Small, testable changes with precise code references

## Core Guarantees
- Record every user input in PHR (verbatim, not truncated)
- PHR routing: `history/prompts/` → constitution/feature-name/general
- ADR suggestions: "📋 Architectural decision detected: <brief>. Document? Run `/sp.adr <title>`"
- Never auto-create ADRs; require user consent

## Development Guidelines

### 1. Authoritative Source Mandate
Use MCP tools and CLI commands for all information gathering. NEVER assume solutions from internal knowledge.

### 2. Execution Flow
Prefer CLI interactions over manual file creation. Treat MCP servers as first-class tools.

### 3. PHR Creation (After Every Request)

**When**: Implementation work, planning, debugging, spec/task/plan creation, multi-step workflows

**Process**:
1. Detect stage: constitution | spec | plan | tasks | red | green | refactor | explainer | misc | general
2. Generate 3-7 word title → slug
3. Resolve route: constitution → `history/prompts/constitution/`, feature → `history/prompts/<feature-name>/`, general → `history/prompts/general/`
4. Read template: `.specify/templates/phr-template.prompt.md`
5. Allocate ID (increment on collision)
6. Fill ALL placeholders: ID, TITLE, STAGE, DATE_ISO, SURFACE, MODEL, FEATURE, BRANCH, USER, COMMAND, LABELS, LINKS, FILES_YAML, TESTS_YAML, PROMPT_TEXT, RESPONSE_TEXT
7. Write file with agent tools
8. Validate: No placeholders, title/stage/dates match, PROMPT_TEXT complete
9. Report: ID, path, stage, title

**Skip PHR only for `/sp.phr` itself**

### 4. ADR Suggestions
When significant decisions made (during `/sp.plan`, `/sp.tasks`), test:
- **Impact**: Long-term consequences?
- **Alternatives**: Multiple viable options?
- **Scope**: Cross-cutting system design?

If ALL true: Suggest ADR, wait for consent.

### 5. Human as Tool Strategy
**Invoke user when**:
- Ambiguous requirements → 2-3 targeted questions
- Unforeseen dependencies → surface and ask prioritization
- Architectural uncertainty → present options
- Completion checkpoint → summarize and confirm

## Default Policies
- Clarify and plan first
- Never hardcode secrets; use `.env`
- Smallest viable diff; no unrelated edits
- Cite code with references (start:end:path)
- Keep reasoning private

## Execution Contract (Every Request)
1. Confirm surface and success criteria
2. List constraints, invariants, non-goals
3. Produce artifact with acceptance checks
4. Add follow-ups and risks (max 3)
5. Create PHR in appropriate subdirectory
6. Surface ADR suggestion if significant decisions

## Architect Guidelines

**Plan must address**:
1. Scope and Dependencies (In/Out/External)
2. Key Decisions (Options, Trade-offs, Rationale)
3. Interfaces (APIs, Versioning, Errors)
4. NFRs (Performance, Reliability, Security, Cost)
5. Data Management (Schema, Migration, Retention)
6. Operational Readiness (Observability, Deployment)
7. Risk Analysis (Top 3, blast radius, guardrails)
8. Evaluation (Definition of Done, validation)
9. ADR (Link for each significant decision)

## Project Structure
- `.specify/memory/constitution.md` — Principles
- `specs/<feature>/spec.md` — Requirements
- `specs/<feature>/plan.md` — Architecture
- `specs/<feature>/tasks.md` — Testable tasks
- `history/prompts/` — PHRs
- `history/adr/` — ADRs
- `.specify/` — Templates and scripts

## Code Standards
See `.specify/memory/constitution.md` for quality, testing, performance, security, and architecture principles.

## Recent Changes
- 001-chatbot-enhancements: Added [if applicable, e.g., PostgreSQL, CoreData, files or N/A]
- 002-chat-fixes: Fixed chat history disappearing and 422/401 errors by correcting state management and API request payloads.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Sheikh-Muhammad-Mujtaba)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Sheikh-Muhammad-Mujtaba)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
