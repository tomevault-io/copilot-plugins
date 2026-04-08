## kernul-alpha

> Scope: All AI-generated code and automated modifications

# AGENTS.md — KERNUL Agent Execution Rules

Status: Binding  
Scope: All AI-generated code and automated modifications  
Authority: Subordinate to REPO_RULES.md, ARCHITECTURE.md, CODING_STANDARD.md, and COMMENTING_GUIDE.md  

Purpose:  
Define deterministic behavior rules for AI agents working on the KERNUL codebase.  
Prevent architectural drift, unsafe changes, and uncontrolled modification of kernel state.

This document governs **all automated code generation**, including:

- AI coding assistants
- automated refactoring tools
- scripted code generators
- continuous integration automation
- test-generation systems

---

# 1. Core Principle

The agent modifies the repository.

The agent does not redesign the system.

The agent does not invent architecture.

The agent implements explicitly defined behavior.

If instructions are unclear:

The agent must stop.

---

# 2. Agent Laws

All agents operating on this repository must follow these laws.

## Law 1 — Explicit Authority

The agent may modify only files explicitly authorized by the task.

All other files are read-only.

Unauthorized modification is a failure condition.

---

## Law 2 — Deterministic Behavior

Agent output must be:

- reproducible
- predictable
- bounded
- reviewable

The agent must not:

- generate random behavior
- infer missing requirements
- speculate about architecture
- introduce undocumented changes

---

## Law 3 — No Architectural Drift

The agent must not:

- redesign subsystems
- introduce new architectural patterns
- modify subsystem boundaries
- create new abstractions without authorization
- change public interfaces silently

Architecture is defined in:

docs/ARCHITECTURE.md

The agent implements architecture.

The agent does not redefine it.

---

## Law 4 — Local Reasoning

The agent must reason only within:

- the provided task
- the authorized files
- the authoritative documentation

The agent must not:

- modify unrelated subsystems
- introduce cross-subsystem dependencies
- restructure directories
- move files

---

## Law 5 — Visible Cost

The agent must not introduce hidden behavior.

The agent must not:

- allocate memory implicitly
- acquire locks implicitly
- block execution implicitly
- transfer ownership implicitly
- change interrupt state silently

All cost must be visible in code.

---

## Law 6 — Safety Before Completion

If a task conflicts with repository rules:

The agent must stop.

Completion is not more important than correctness.

---

# 3. Required Workflow

Every agent task must follow this sequence.

Step 1 — Read governing documents:

- docs/ARCHITECTURE.md
- docs/CODING_STANDARD.md
- docs/COMMENTING_GUIDE.md
- docs/REPO_RULES.md
- MAINTAINERS.md

Step 2 — Validate scope.

Step 3 — Implement behavior.

Step 4 — Verify constraints.

Step 5 — Stop.

---

# 4. Mandatory Stop Conditions

The agent must stop immediately if:

- instructions are ambiguous
- required files are missing
- scope is undefined
- architectural conflict is detected
- locking behavior is unclear
- ownership rules are unclear
- concurrency rules are unclear
- required invariants are not specified

Stopping is correct behavior.

Continuing without clarity is failure.

---

# 5. Forbidden Actions

The agent must never:

Modify boot sequence logic without explicit instruction  
Modify scheduler behavior without explicit instruction  
Modify memory management logic without explicit instruction  
Modify interrupt handling without explicit instruction  
Modify syscall ABI without explicit instruction  
Modify concurrency primitives without explicit instruction  
Modify hardware interaction code without explicit instruction  
Change subsystem interfaces without documentation updates  
Remove safety checks  
Remove validation logic  
Suppress error handling  
Introduce hidden state  
Introduce global mutable state without ownership  
Change lock ordering  
Introduce lockless code without justification  
Introduce blocking behavior in interrupt context  
Introduce recursion in hot paths  
Introduce dynamic allocation in hot paths  
Generate unreachable code  
Silently ignore errors  

These are hard failures.

---

# 6. Allowed Actions

The agent may:

Implement explicitly defined behavior  
Add missing validation  
Add defensive checks  
Fix correctness bugs  
Improve readability without changing behavior  
Add comments that clarify invariants  
Add tests  
Refactor code when behavior remains identical  
Improve error handling  
Improve documentation  

All allowed changes must preserve system behavior.

---

# 7. Code Generation Rules

Generated code must be:

- simple
- explicit
- readable
- reviewable
- deterministic

Generated code must not:

- use clever tricks
- rely on undefined behavior
- hide control flow
- hide ownership
- hide locking
- hide allocation
- hide error handling

---

# 8. Concurrency Rules

When modifying shared state:

The agent must:

Document the protecting lock  
Document the caller context  
Document whether sleeping is allowed  
Document ownership rules  

The agent must not:

Change lock ordering  
Introduce new locks without justification  
Convert locked code to lockless code  
Add blocking operations in interrupt context  

---

# 9. Memory Rules

The agent must:

Validate allocation success  
Release resources on failure  
Preserve lifetime invariants  

The agent must not:

Leak memory  
Double-free memory  
Transfer ownership silently  
Allocate memory in interrupt context  
Allocate memory in hot paths without authorization  

---

# 10. Error Handling Rules

The agent must:

Return explicit status codes  
Handle failure paths  
Preserve cleanup logic  

The agent must not:

Ignore errors  
Suppress return codes  
Continue after failure  
Convert errors into silent behavior  

---

# 11. Commenting Rules

All generated code must follow:

docs/COMMENTING_GUIDE.md

The agent must document:

- invariants
- locking
- ownership
- ordering
- hardware constraints

The agent must not:

Write decorative comments  
Write redundant comments  
Write speculative comments  

---

# 12. Interface Modification Rules

Public interfaces include:

- exported functions
- public headers
- syscall interfaces
- ABI-visible structures
- device interfaces

When modifying a public interface:

The agent must:

Update documentation  
Preserve compatibility  
Provide migration guidance  

Silent interface changes are prohibited.

---

# 13. Test Requirements

The agent must add tests when:

- new behavior is introduced
- correctness logic changes
- concurrency logic changes
- memory handling changes
- interface behavior changes

Tests must be:

- deterministic
- reproducible
- isolated
- reviewable

---

# 14. Logging and Debugging

The agent may add:

- debug assertions
- tracepoints
- validation checks

The agent must not:

Remove safety diagnostics  
Add excessive logging in hot paths  
Change panic behavior without instruction  

---

# 15. Performance Discipline

The agent must not optimize code without evidence.

Performance changes require:

- measurable justification
- preserved correctness
- preserved readability

Premature optimization is prohibited.

---

# 16. Compatibility Discipline

When modifying user-visible behavior:

The agent must:

Preserve compatibility  
Document behavior change  
Add regression tests  

The agent must not:

Break ABI silently  
Change syscall semantics silently  
Change file descriptor behavior silently  
Change signal behavior silently  

---

# 17. Output Discipline

Agent output must be:

Deterministic  
Complete  
Minimal  
Reviewable  

The agent must not:

Generate partial implementations  
Leave incomplete code paths  
Introduce TODO placeholders in production code  

---

# 18. Recovery Model

If a failure occurs:

The agent must:

Stop  
Report the failure  
Preserve repository state  

The agent must not:

Attempt silent recovery  
Modify unrelated code  
Guess a solution  

---

# 19. Authority Hierarchy

Order of authority:

1 — REPO_RULES.md  
2 — ARCHITECTURE.md  
3 — CODING_STANDARD.md  
4 — COMMENTING_GUIDE.md  
5 — MAINTAINERS.md  
6 — AGENTS.md  

If conflict exists:

Higher authority wins.

---

# 20. Definition of Success

A successful agent execution is:

Correct  
Deterministic  
Reviewable  
Bounded  
Safe  

Not merely complete.

---

# Final Rule

When uncertain:

Stop.

Do not guess.

Do not improvise.

Do not invent.

Stop.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/irgordon)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/irgordon)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
