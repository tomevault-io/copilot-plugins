## open-agent-engine

> Cross-platform multi-agent scaffolding, skill package manager, and transpiler engine

# Agent Directives: open-agent-engine

Cross-platform multi-agent scaffolding, skill package manager, and transpiler engine

## 1. Context Routing Index
- Active Sprint & Backlog: inspect [agent_docs/backlog.md](file:///agent_docs/backlog.md)
- Product & Scope: inspect [PRODUCT.md](file:///PRODUCT.md)
- Design & Visual Identity: inspect [DESIGN.md](file:///DESIGN.md)
- Architecture & Compilers: inspect [agent_docs/architecture.md](file:///agent_docs/architecture.md)
- Workflow & Educational Concepts: inspect [agent_docs/workflow_and_concepts.md](file:///agent_docs/workflow_and_concepts.md)
- Beginner's ELI5 Guide: inspect [agent_docs/beginners_guide.md](file:///agent_docs/beginners_guide.md)
- Subagents Directory: inspect [.agents/personas/](file:///c:/.agents/personas/)

## 2. Environment & Shell Commands
Execute in shell with strict sequential separators:
- Install: `pnpm install`
- Build: `pnpm build`
- Test: `pnpm test`
- Typecheck: `pnpm typecheck`
- Dev Mode: `pnpm dev`

## 3. Operational Invariants & Boundaries ("Rule + Why")

### A. ALWAYS Do
1. **Validate All Manifests Against Zod Schemas**
   - *Why*: Catch malformed user configuration early before transpilation to downstream IDEs.
2. **Preserve Progressive Disclosure**
   - *Why*: Keep root instruction files under 150 lines to prevent prompt cache invalidations.
3. **Verify All Tests Pass Before PR**
   - *Why*: Prevent broken regressions across adapter pipelines.
4. **Submit Changes via Branch PR with Approval**
   - *Why*: `main` branch is protected by GitHub Repository Rulesets to prevent unvetted code changes.

### B. NEVER Do
1. **Never Write Platform-Specific Logic in Core Schemas**
   - *Why*: Maintain 100% vendor-neutral canonical AST representations.
2. **Never Emit Malformed Frontmatter**
   - *Why*: Target IDE parsers fail unpredictably when encountering broken YAML headers.

---
> Source: [jedmamosto/open-agent-engine](https://github.com/jedmamosto/open-agent-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
