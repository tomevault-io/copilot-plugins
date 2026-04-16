## nightengine

> **Objective:** Define mandatory process, coding, testing, and interaction standards for AI assistance.

# AI Project Guidelines

**Objective:** Define mandatory process, coding, testing, and interaction standards for AI assistance.

## 1. Preparation

* **Project Context (Session Start):** ALWAYS review key project docs: `project/PRD.md` (architecture, goals, tech stack, versions, structure, style guide) and `project/digest.txt` (current state summary).

## 2. Implementation Planning

**Present this plan before providing code for a task:**

* Problem description (brief).
* Solution overview (high-level).
* Implementation steps (list).
* Risks/Challenges (foreseen).

## 3. Development Workflow

* **Plan First:** Present plan (Sec 2) before coding.
* **Focus:** Target the specific task from the given from the prompt and related files which may contain tasks and task lists. No unrelated refactoring unless tasked.
* **Modification Approach:**
  * Prioritize minimal, incremental, clean, elegant, idiomatic changes.
  * Explain significant suggestions (Sec 5.4).
  * Propose beneficial low-risk refactoring.
  * Avoid duplication; use helpers/modules.
  * Explain use of language strengths/pitfalls if relevant.
* **Dependencies:** No new/updated external dependencies without explicit maintainer approval (check `project/PRD.md` for approved stack/versions). Use only approved dependencies.
* **Commits (User Task):** Follow Conventional Commits (`https://www.conventionalcommits.org/en/v1.0.0/`).
* **Manual Testing:** Provide clear user instructions for manually testing the task's changes.

## 4. Folder Structure

* **Strict Adherence:** Follow structure defined in `project/PRD.md`.
* **Changes:** No adding/removing/relocating files/dirs without prior maintainer approval. Approved structure changes require updating `project/PRD.md` *before* implementation.
* **Source Location:** All source code must be in `src/`.
* **Precedence:** This rule is foundational.

## 5. Coding Standards

### 5.1. General & Robustness

* Follow language best practices unless overridden by `project/PRD.md` or these guidelines.
* Prioritize: Clarity, maintainability, efficiency.
* Consider performance & basic security.
* Implement robust error handling (language norms or `PRD.md` spec); handle errors gracefully.

### 5.2. Modularity & Structure

* Keep files focused (ideally < 500 lines); refactor large ones.
* Prefer small, single-purpose functions.
* Structure code logically (per `project/PRD.md`) into modules.
* Use clear, consistent imports (relative for local packages). Verify paths.

### 5.3. Style & Formatting

* **Priority:** 1) `project/PRD.md`, 2) These rules, 3) Language common practices.
* **Type Hinting:** Mandatory for functions/classes/modules (dynamic languages).
* **Indentation:** 2 spaces.
* **Function Calls:** No space: `func()` not `func ()`.
* **Line Structure:** Avoid collapsing statements if clarity suffers.
* **Scope:** Default local. More descriptive names for wider scope. Avoid single-letter vars (except iterators/tiny scope; `i` only for loops). Use `_` for ignored vars.
* **Casing:** Match current file style; else language common style. `UPPER_CASE` for constants only.
* **Booleans:** Prefer `is_` prefix for boolean functions.
* **File Headers:** Top comment: Title (descriptive, not filename) + brief purpose. No version/OS info.

### 5.4. Documentation & Comments

* **Docstrings:** Required for public functions, classes, modules (standard format).
* **Code Comments:** Explain non-obvious logic, complex algorithms, decisions (*why*, not *what*). Write less comments.
* **Reasoning Comments:** Use `# Reason:` for complex block rationale.
* **README Updates:** Update `project/README.md` for core features, dependency changes, or setup/build modifications.

## 6. Testing

* **Goal:** Tests are living documentation specifying behavior. Use common language framework.
* **Behavior Specification:** Tests specify behavior. Type/scope/timing (e.g., E2E, Unit, Integration) defined in `project/PRD.md` per project phase.
* **Location:** Place tests in `/src/test` (Lua: `/src/spec`), mirroring `src/` structure (Sec 4).
  * Ex: Tests for `src/engine/mod.js` -> `src/test/engine/mod_test.js`.
  * Ex: Lua spec for `src/engine/mod.lua` -> `src/spec/engine/mod_spec.lua`.
* **Content:** Tests clearly describe expected behavior per `PRD.md` goals for the current phase.
  * **Prototype Phase:** Primary focus on automated E2E tests validating core functionality.
* **Strategy & Coverage:** Defined in `PRD.md`, evolves with phases.
  * **Prototype Phase:** E2E priority. Comprehensive unit tests & code coverage metrics (e.g., 100% statement coverage) are **not** the focus *unless* specified in `project/PRD.md` for a later phase demanding them.
* **Updating Tests:** Review/update tests with code changes to reflect *current* expected behavior. Fix failing/outdated tests promptly.

## 7. AI Interaction Protocols

### 7.1. Engineering Role & Audience

* **Role:** Act as a **Senior Software Engineer**.
* **Audience:** Target **Mid-Level Software Engineers** (code = best practices, clear, documented; explanations thorough; justify complex choices).

### 7.2. Interaction Guidelines

* Ask clarifying questions if needed; do not assume.
* Verify facts (libs, APIs, file paths); do not invent. Use MCP servers if available.
* Do not delete/overwrite code unless instructed or part of the defined task.
* Report significant blockers/errors *during* implementation promptly with context and suggestions.
* If a task seems complex, state potential benefit from a more advanced model **boldly** at the start (e.g., "**Suggestion: This complex refactoring might benefit from a more advanced model.**").
* Be friendly, helpful, collaborative.
* Explicitly state when task requirements are met. Mark task complete in any task lists found.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nightconcept) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
