## message-consolidator

> **[CRITICAL ROUTING RULE]**

# [Project Specific: Project GEM Addon]

## 0. Mandatory Operation Protocol (Serena MCP)
**[CRITICAL ROUTING RULE]**
Any request involving file inspection, code modification, or analysis MUST follow this exact sequence:

* **STEP 1: File Type Evaluation**
    * IF target file is source code (`.go`, `.ts`): Proceed to STEP 2.
    * IF target file is configuration/text (`.yaml`, `.json`, `.md`, `.env`): Use native reading tools. HALT sequence.
* **STEP 2: Forced Serena Execution**
    * You MUST invoke `serena` tools (`find_symbol`, `search_for_pattern`, `list_dir`) exclusively.
    * Using native `read_file` or broad `analyze` tools on source code is a critical failure.
* **STEP 3: Zero-Guessing Validation**
    * IF `serena` execution fails or returns empty: HALT operation immediately.
    * Output exactly: "[Serena Error] Requesting manual clarification for AST target." DO NOT fallback or guess code context.

## 1. Architecture & Migration
- **Strict Logic Delegation:** Go (1.25.6) = 100% Business Logic, Calculations, Aggregations. TypeScript (Vanilla + Vite) = UI Rendering, DOM, Events ONLY.
- **Migration Policy:** NO mutations on existing `.js`. Rewrite strictly in `.ts` following the delegation rule.
- **Database:** SQLite (Turso).

## 2. Design System (CSS)
- **Constraint:** BEM is mandatory. `px` and `hex` are explicitly banned. Use `rem` and `variables.css` tokens.
- **Validation Check:** You MUST run `node verify-css.cjs` and confirm success before marking a task as complete.

## 3. Infrastructure & Monitoring
- **Observability:** Account for 150MB memory overhead strictly reserved for WhaTap agents.
- **Type Safety:** Explicit integer conversion for ALL ID parameters is MANDATORY across all layers.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JaejinSong)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/JaejinSong)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
