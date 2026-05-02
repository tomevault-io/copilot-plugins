## lucid-memory

> Bun, Rust, TypeScript

# Project: lucid-memory

## Stack

Bun, Rust, TypeScript

## Navigation

See `.claude/PROJECT_MAP.md` for project structure & key file locations.

## Code Quality

Good code is self explaining. Comment only when absolutely needed.

## Conventions

- Named exports, no default exports

## Gotchas

**timeout isn't available on macOS. Use gtimeout**

## Complete

A TODO is COMPLETE only when **all 5 gates** pass:

| Gate | Requirement | How To Verify |
|------|-------------|---------------|
| **1. Implemented** | Code written, compiles, no errors | `bun test` passes |
| **2. Wired** | Connected to the systems that call it (wake cycles, hooks, handlers, prompts) | Grep for function calls from outside the module |
| **3. Tested** | Unit tests + integration tests covering happy path and edge cases | Test count documented, all pass |
| **4. Verified in Production** | Evidence of actual usage in the live database | SQL query showing non-zero rows, event counts, or call logs |
| **5. No Orphaned Outputs** | Every computed output is consumed by at least one downstream system | Grep for each output → confirm a reader exists |

---
> Source: [JasonDocton/lucid-memory](https://github.com/JasonDocton/lucid-memory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
