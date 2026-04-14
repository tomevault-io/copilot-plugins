## ai-ml-portfolio-starter

> **Policy:** For user requests involving code examples, setup/configuration steps, or library/API docs, route via a documentation-focused MCP tool.

# Documentation-First Routing (Always Apply)

    **Policy:** For user requests involving code examples, setup/configuration steps, or library/API docs, route via a documentation-focused MCP tool.

- Preferred tools: `docfork`, `sourcebot`, `git-mcp` (set `owner/repo`), generic RAG
- Fallbacks:
- Detection cues: "example", "install", "configure", "setup", "API", "SDK", "how to", "usage", "params", "options".
- Guardrails: Read-only unless explicitly elevated; no secrets exposure; respect repo permissions.

    **Example Invocation (conceptual):**
- User: "Show me how to configure OAuth for Service X."
- Agent: Selects `docfork` → extracts relevant guides → returns step-by-step with citations.

**Agentic Workflow & Guardrails (Playbook Summary)**

- **Spec-as-code:** Treat specs, AC, and ADRs as inputs that program the agent. Ambiguity is the main failure source.
- **Test-first AC:** Provide failing tests or explicit Acceptance Criteria spanning functional, performance, security, maintainability, and integration.
- **Definition of Done:** Enforce a checklist (formatting, tests added/passing, coverage non-regression, docs updated, no high/critical vulns, staged deploy, human review).
- **ADRs as guardrails:** Record context/decision/consequences; reference ADRs in prompts to prevent architectural drift.

- **Workflow:** Idea → Spec/AC/ADRs → Scaffold → Tight loop (write failing test → implement → emit diff → review/apply → run tests) → Docs → Release.
- **Personas & outputs:** Planner, Implementer, Tester, Reviewer. Require structured outputs (JSON schemas) and function/tool calling for predictable actions.
- **Context strategy:** Use repo maps for quick navigation and RAG for deep, file-level context. Retrieve only what’s relevant.

- **Safety rails:** Patch/diff only (no direct writes), filesystem allowlists, CI gates (format/lint/type-check/tests), read-only by default, no secrets in repo.
- **Templates & automation:** Prefer the Python/TypeScript templates; use `scaffold.py`, `repo_map.py`, `indexer.py`, CI PR validator, and `guardrail.py`.
- **Monorepo bias:** Favor monorepo for unified context; store canonical prompts/schemas in `.prompts/`.

- **Evaluation:** Maintain a lightweight harness (task spec → apply patch → run tests). Track KPIs: Task Success Rate, Diff Size/Churn, Turns/Tool Calls, Token Cost, Human Interventions. Add failed cases to regression and triage: classify → inspect trace → root cause → remediate → regress.

**Docs-First Routing (Reminder)**

- **Trigger words:** example, install, configure, setup, API, SDK, how to, usage, params, options.
- **Tools:** Prefer documentation tools; read-only by default; respect repo permissions; never expose secrets. Cite sources when feasible.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AcidicSoil)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AcidicSoil)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
