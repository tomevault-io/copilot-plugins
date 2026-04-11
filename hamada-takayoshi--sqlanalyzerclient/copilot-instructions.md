## sqlanalyzerclient

> This file defines mandatory architectural, implementation, and quality rules.

# AGENTS.md
# Project Governance Rules for Codex

This file defines mandatory architectural, implementation, and quality rules.
These rules apply to ALL phases.

The documents under /docs are the single source of truth.
If implementation conflicts with docs, docs win.

---

# 1. Core Philosophy

This project follows strict separation of concerns:

- Domain = pure data model (dialect-agnostic)
- SqlServer = SQL Server specific parsing & formatting
- App = WPF UI + orchestration only

No layer may violate these boundaries.

---

# 2. Hard Dependency Rules (Must Never Be Violated)

Allowed:
- App -> Domain
- App -> SqlServer
- SqlServer -> Domain

Forbidden:
- Domain -> anything
- Domain -> SqlServer
- App -> parser libraries
- App -> networking libraries

Domain must not reference:
- ScriptDom
- Any SQL parser library
- Any WPF type
- Any UI framework

---

# 3. Offline-Only Runtime Policy (Critical)

At runtime, the application must:

- Not perform HTTP/HTTPS requests
- Not use HttpClient
- Not use WebView
- Not load remote scripts
- Not connect to any database
- Not send telemetry

All processing must be local.

Violation of this rule is not allowed.

---

# 4. Domain Model Integrity Rules

- Domain types must be immutable (prefer record types).
- Domain must remain JSON-serializable.
- No parser AST types may leak into Domain.
- TableRelation must reference tables by ID only.
- JoinType must always be an enum value.

Domain is the central contract.
It must remain stable.

---

# 5. Implementation Phases (Must Follow Order)

Follow docs/13_task_plan.md strictly.

Do not:
- Skip phases
- Merge multiple phases into one
- Implement parser before boundary extraction

Each phase must:
- Compile successfully
- Build without errors
- Be verifiable before moving forward

---

# 6. UI (MVVM) Rules

- Views must not contain parsing logic.
- ViewModels must not reference parser libraries.
- ViewModels coordinate services only.
- Async operations must not block UI thread.
- Cancellation must be supported where defined.

---

# 7. SQL Parsing Rules

When implementing SQL Server parsing:

- Use ScriptDom only inside SqlAnalyzer.SqlServer.
- Immediately transform parser output into Domain model.
- Do not expose ScriptDom types outside SqlServer project.
- JOIN and APPLY must map to JoinType enum.
- DDL must generate DDL_NOT_SUPPORTED diagnostic.

---

# 8. Logical Name Extraction Rules

Logical names must be extracted from:

- Block comments: /* Logical Name */
- Line comments: -- Logical Name

Whitespace variations must be tolerated.

Extraction logic must not break SQL parsing.

---

# 9. Boundary Extraction Rules

Single statement rule:

- First ';' OR first 'GO' (case-insensitive)
- Ignore trailing statements
- Emit MULTI_STATEMENT_TRUNCATED if extra content exists

Boundary extraction must be implemented BEFORE full parsing.

---

# 10. Diagram Rules

Diagram generation:

- Must depend only on Domain model
- Must not require network access
- Must not modify Domain model

Mermaid text and PNG are read-only transformations of Domain.

---

# 11. Code Quality Standards

- Enable nullable reference types.
- Use async/await properly.
- Async methods must end with Async.
- Avoid static mutable state.
- Avoid reflection unless strictly required.
- Avoid dynamic typing.

Build must not contain warnings where avoidable.

---

# 12. Naming Conventions

- PascalCase for types and enums.
- CamelCase for local variables.
- Enum values must be explicit.
- No abbreviations unless standard (Id, SQL).

---

# 13. Extensibility Strategy

Future dialects (e.g., Oracle) must:

- Implement analyzer/formatter in a new project.
- Output the same Domain model.
- Not require changes to Domain.

App must route by dialect selection.

---

# 14. Documentation Synchronization

If implementation deviates from docs:

- Update docs first
- Then update implementation

Docs are authoritative.

---

# 15. Build Discipline

After meaningful changes:

- Run dotnet restore
- Run dotnet build
- Fix all build errors before proceeding

Do not accumulate broken builds.

---

# 16. Safety Constraints

Do not:

- Introduce external services
- Add unnecessary NuGet packages
- Over-engineer architecture
- Refactor outside the current phase scope

Stay aligned with the current phase only.

---

# 17. Completion Criteria

A phase is complete only if:

- The app builds
- The app runs
- The defined Definition of Done is satisfied
- No architectural rule is violated

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hamada-takayoshi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hamada-takayoshi)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
