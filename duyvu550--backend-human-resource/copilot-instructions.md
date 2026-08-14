## backend-human-resource

> - **Principle**: Best code is code never written. Always find the shortest, simplest working solution.

# Claude Project Instructions

## Token Efficiency & Engineering Rules

### 1. Ponytail (Minimalism & YAGNI)
- **Principle**: Best code is code never written. Always find the shortest, simplest working solution.
- **Ladder**:
  1. Check if the task needs to exist at all (YAGNI).
  2. Reuse existing helpers, types, and utility patterns in this codebase.
  3. Prefer Standard Library & Native Platform features over custom dependencies.
  4. Minimize new code: one line over fifty, diff over rewrite.
- **Output Guidelines**:
  - Code-first responses. Keep explanations under 3 short lines (`[code] → skipped: [X], add when [Y]`).
  - Eliminate unrequested prose, essays, or speculative scaffolding to save context tokens.

### 2. Graphify (Architecture & Code Analysis)
- **Principle**: Inspect structural relationships before dumping full files into context.
- **Usage**:
  - For questions regarding codebase structure, file dependencies, or data flows, query existing graphify data or trace call graphs systematically.
  - Avoid bulk-reading unnecessary files to save context window tokens.

### 3. General Directives
- **Root Cause over Symptom**: Trace callers and fix shared bugs once at the source.
- **No Over-Engineering**: No single-use interfaces, unneeded factories, or speculative configurations.

---
> Source: [DuyVu550/Backend_human_resource](https://github.com/DuyVu550/Backend_human_resource) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
