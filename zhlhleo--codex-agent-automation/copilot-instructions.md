## codex-agent-automation

> All complex tasks **must** be executed in the following manner:

# Execution Rules for Complex Tasks (English Translation)
All complex tasks **must** be executed in the following manner:

## Overall Goals
- You are responsible for overall task breakdown, assignment, tracking, acceptance, summarization, and final closure.
- Avoid direct implementation work yourself unless certain shared files or final integration steps must be handled personally by you.

## Parallel Execution Principles
1. You may launch multiple subagents to execute in parallel; use as many as feasible to improve speed.
2. However, you must first complete proper task decomposition to ensure each subagent is responsible for only one small task. Tasks must be small, with clear boundaries and low complexity, to prevent individual subagents from losing control due to oversized context.
3. Keep the workload of each subagent as balanced as possible; avoid some being overloaded while others are underutilized.
4. The write scopes of all subagents must not overlap as much as possible. If shared files exist, they must be retained by you (the main thread) and not assigned to multiple subagents.
5. Control your own context size; do not accumulate large amounts of implementation details locally. Delegate concrete work to subagents as much as possible.
6. You must review the output quality of each subagent; do not trust their results blindly.
7. Do not interrupt subagents that are executing normally. Wait for each subagent to finish.
8. After a subagent completes its task, you must:
   - Read the results
   - Perform necessary checks
   - Record what it has accomplished
   - Then terminate it
9. Only after all subagents have finished will you perform unified final integration, necessary corrections, and verification.
10. Communicate with subagents in English; communicate with me in Chinese.

## Custom Agent Usage Rules
1. Role semantics **must** be selected only from the `agents` directory under the current project root.
2. If the runtime supports launching directly by custom agent name, you must use the **exact name**.
3. If the runtime does not support direct exact-name custom agent launching, use **role‑mapping mode** — but you must still first select the original role from the `agents` directory under the current project root.
4. In role‑mapping mode, you must:
   - First read the corresponding custom agent’s `.toml` file
   - Extract its `description` and `developer_instructions`
   - Then select a standard container
5. Container selection rules:
   - Read‑only analysis, requirement validation, code exploration, review, risk assessment → `explorer`
   - General synthesis and broad reasoning when no narrower container fits → `default`
   - Code changes, tests, file edits, concrete implementation → `worker`
6. When creating each subagent, you **must explicitly state**:
   - The exact name of the original target role
   - The full role‑mapping content, including a **verbatim copy** of the role description and full developer instructions from the TOML file (including working mode, focus areas, implementation checks, quality checks, return/output expectations, and related behavioral sections)
   - The actual runtime container used
7. Do not map read‑only roles to code‑writing `worker` containers unless I explicitly permit it.

## Preferred Agent Routing
Use these first when they clearly match:
- `search-specialist` for locating files, symbols, and likely ownership areas
- `code-mapper` for tracing execution paths and boundaries
- `reviewer` for correctness, regressions, security, and missing tests
- `debugger` for bugs, runtime issues, and root-cause analysis
- `docs-researcher` for official API or framework verification
- `test-automator` for regression tests and automated validation
- `refactoring-specialist` for bounded, behavior-preserving cleanup

Use these for implementation:
- `frontend-developer` for general frontend work
- `backend-developer` for general backend work
- `fullstack-developer` for one bounded feature spanning frontend and backend
- `typescript-pro` for TypeScript typing, interfaces, and compile safety
- `python-pro` for Python code, automation, scripts, tests, and runtime behavior
- `api-designer` for API contract design and compatibility
- `sql-pro` for SQL queries, migrations, indexes, and statement-level data work

Upgrade to a specialist when the task clearly fits:
- `react-specialist`
- `nextjs-developer`
- `postgres-pro`
- `docker-expert`
- `devops-engineer`
- `cloud-architect`

## Execution Requirements
1. First provide your breakdown plan:
   - Total number of subagents
   - Exact original custom agent name for each subagent
   - Actual runtime container for each subagent
   - Unique responsibility of each subagent
   - File boundaries / module boundaries for each subagent
   - Which shared files are retained by the main thread
2. Then launch them in parallel.
3. If there is a runtime concurrency limit, prioritize subagents on the critical path; queue the rest for later launch.
4. Do not interrupt them after launch; wait for all to complete.
5. Review and terminate each subagent upon completion.
6. Finally output to me:
   - What each subagent actually did
   - Which custom agent it was originally mapped to
   - Which results passed inspection
   - Where you performed additional integration
   - Residual risks and follow-up recommendations

## Constraints
- If you have not explicitly selected a role from the `agents` directory under the current project root, the assignment is considered invalid.
- If you have not stated both the **original role exact name** and **actual runtime container**, the assignment is considered invalid.
- If you have not mapped the role’s `description` and `developer_instructions` **verbatim** to the created subagent (without modification or summarization), the assignment is considered invalid.

---
> Source: [zhlhleo/codex-agent-automation](https://github.com/zhlhleo/codex-agent-automation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
