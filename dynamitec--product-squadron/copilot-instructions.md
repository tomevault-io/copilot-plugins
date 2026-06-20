## product-squadron

> Autonomous development squadron for full-stack products. Integrates hierarchical command (Nelson), token-optimized slang (Caveman), and tree-search dashboard (Evo).


# Product Squadron

High-performance autonomous agent framework for full-product delivery and optimization.

## Core Frameworks

### 1. Reliable Command (Nelson)
- **Hierarchy**: Defined in `references/squadron/hierarchy.md` (Admiral, Captain, Crew).
- **Worktrees**: `scripts/mission/worktree_orchestrator.cjs` for parallel agent ships.
- **Audit**: `scripts/mission/log.cjs` and **Web Dashboard** in `scripts/mission/web_dashboard.cjs`.

### 2. Terse Communication & Token Advisor (Caveman)
- **Protocol**: Mandated in `references/communication/terse-protocol.md`.
- **Advisor**: `scripts/utils/token_advisor.cjs` for user token-efficiency coaching.

### 3. Tree Search Optimization (Evo)
- **Commands**: `scripts/mission/branch.cjs` and `scripts/mission/prune.cjs`.
- **Method**: Parallel experimentation in `references/optimization/tree-search.md`.

## Expert Roles & Guardrails

- **Product Suite**: `references/roles/product-suite.md` (Architect, Designer, Full-stack, Mobile).
- **Quality & Ops**: `references/roles/quality-ops.md` (QA, Security, SRE, SEO).

## Mission Workflow

1. **Sailing Orders**: Define mission objective, tech stack, and constraints.
2. **Dashboard Init**: Initialize mission via `scripts/mission/init.cjs`.
3. **Branching Worktrees**: Launch expert "Ships" in parallel git worktrees.
4. **Monitoring**: Generate and view the **Evo-like Web Dashboard** via `scripts/mission/web_dashboard.cjs`.
5. **Finalization**: Compare ship outputs, prune regressions, and select the optimal solution.
6. **Turnover Brief**: Generate final report with log history and benchmark data.

---
> Source: [DynamiteC/product-squadron](https://github.com/DynamiteC/product-squadron) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
