## haivemind

> > These rules apply to **all** coding agents (Copilot, Claude, Cursor, autonomous) working in this repo.

# Universal Coding Agent Rules — hAIvemind

> These rules apply to **all** coding agents (Copilot, Claude, Cursor, autonomous) working in this repo.
> hAIvemind-specific overrides are in `.github/copilot-instructions.md`.
> Path-specific rules are in `.github/instructions/*.instructions.md`.

---

## 0. Meta-Rule: Continuous Improvement

Rules follow the cycle: **Research → Synthesize → Write → Validate → Repeat**.
- Distill findings into agent-agnostic, imperative directives.
- Remove rules that don't demonstrably improve output.
- Rules are living documents — never consider them finished.

---

## Core Rules (Always Active)

### 1. Explore Before You Act

The #1 failure mode is making changes without understanding the codebase.

- **Read before writing.** Before modifying any file, read surrounding code, tests, and docs.
- **Search broadly first.** Use grep, semantic search, or file listings to understand project structure.
- **Identify existing patterns.** Look at how similar problems are already solved. Follow those patterns.
- **Check for existing tests.** They document expected behavior better than comments.
- **Map dependencies.** Understand what depends on the code you're changing and what it depends on.

### 2. Plan Before You Implement

Jumping straight to code leads to rework.

- **Break complex tasks into steps.** List the discrete changes needed before writing code.
- **Identify the smallest viable change.** Minimum edit that moves toward the goal.
- **Anticipate edge cases.** Empty inputs, error states, concurrency, large data.
- **State your plan explicitly.** Document what you intend to do before doing it.
- **Update the plan as you learn.** Plans are hypotheses — revise when implementation reveals new info.

### 3. Verify Your Own Work

The single highest-leverage behavior. Verify before the user has to.

- **Run tests after every change.** If tests exist, run them. If they don't, write them.
- **Check for errors.** After editing, check for compile/lint/type errors.
- **Verify actual output.** Don't assume — confirm by running the code or inspecting results.
- **Test edge cases.** Empty input, null values, boundary conditions.
- **Never declare "done" without verification.** State what remains unverified if you can't.
- **Create verification artifacts.** Write a test or verification script. Produce observable proof.

### 4. Make Small, Incremental Changes

Large sweeping changes are hard to verify, debug, and roll back.

- **One logical change per step.** Don't mix refactoring with feature work.
- **Commit frequently.** Each commit = a working state. Use conventional messages: `feat:`, `fix:`, `refactor:`, `docs:`, `test:`.
- **Verify after each step.** Run tests, check errors, confirm behavior.
- **If something breaks, stop and fix it** before continuing.

### 5. Manage Context Aggressively

Context is the single most important resource an agent has.

- **Stay focused on one task.** Finish the current task before starting a new one.
- **Reference specific files and functions** — not vague descriptions.
- **Scope your searches.** Search specific directories, not the entire codebase.
- **Seed context with existing code.** Reference relevant modules, types, or patterns.
- **Start simple, then iterate to complex.** Each step builds on verified context.
- **Handle context overflow.** Externalize state to files. Summarize rather than drop.

### 6. Communicate Clearly

Miscommunication is the #2 failure mode after not exploring first.

- **Ask when requirements are ambiguous.** Don't guess — clarify. Propose a default to keep momentum.
- **Explain what you're doing and why.** Especially for non-obvious decisions.
- **Report what you've tried** when something fails. Include error messages and file paths.
- **Be specific about uncertainty.** "I'm not sure if X or Y" beats silently picking one.
- **Confirm meaningful changes before making them.** Especially destructive operations.

### 7. Write Quality Code

Clean code is easier for both humans and agents to maintain.

- **Follow the project's existing conventions.** Match indentation, naming, file organization.
- **Write meaningful error messages.** Include context: what happened, what was expected, what to do.
- **Prefer explicit over clever.** Readable > concise. Avoid abstractions that obscure intent.
- **Apply DRY sensibly.** Two is fine; three is a pattern.
- **Document WHY, not WHAT.** Code shows what; comments explain non-obvious reasoning.
- **Handle errors.** Don't swallow exceptions. Propagate or handle with clear messages.

### 8. Use Structured Workflows

Ad-hoc workflows lead to forgotten steps and inconsistent quality.

- **New features:** Spec → Plan → Implement → Test → Document → Commit.
- **Debugging:** Reproduce → Isolate → Fix → Verify fix → Add regression test.
- **Refactoring:** Identify target → Ensure test coverage → Refactor → Verify → Commit.
- **Code review:** Read diff → Correctness → Edge cases → Style → Actionable feedback.
- **Track progress.** Use todo lists or task trackers for multi-step work.

### 9. Delegate and Parallelize

Complex tasks often contain independent subtasks.

- **Batch independent operations.** Read multiple files in parallel, not sequentially.
- **Use subagents for investigation.** Deep research shouldn't pollute the main context.
- **Fan out, then synthesize.** Multiple queries → collect results → summarize.
- **Don't over-parallelize.** Respect dependency ordering.

### 10. Avoid Common Anti-Patterns

| Anti-Pattern | Fix |
|---|---|
| Kitchen-sink sessions | One clear scope per session |
| Over-correction | Apply feedback to the specific target, not the entire codebase |
| Infinite exploration | Set a limit; ask the user after reasonable search |
| Trust-then-verify gap | Always verify; especially after complex edits |
| Ignoring failures | If a test fails or build breaks, address immediately |
| Premature completion | Check full requirements against what's been delivered |
| Fixing a derailed agent | Revert to last known-good state; restart from plan |

---

## Advanced Rules

### 11. Red/Green TDD

- Write tests before implementation. Confirm they fail first (red).
- Implement until tests pass (green). The minimum code needed.
- Start every session with "First run the tests" — orients the agent.

### 12. Build and Reuse Knowledge Assets

- Hoard working examples — proof-of-concepts, snippets, validated solutions.
- Recombine known solutions: "Use pattern from X to build Y."
- Document tricks where agents can find them (READMEs, docs, example repos).
- Code is cheap; knowing what's possible is the real asset.

### 13. Combat Cognitive Debt

- Request code walkthroughs after an agent builds something complex.
- Review every diff. If you can't explain what a module does, it doesn't go in.
- Maintain a mental model of every part of the system.
- Use agents to teach you the underlying concepts.

### 14. Design Environments for Agents

- **Minimize context pollution.** Test harnesses print summaries, not thousands of bytes.
- **Maintain progress documents.** READMEs and status files updated frequently.
- **Make tasks independently solvable.** Multiple agents can make progress without stepping on each other.
- **Design test output for agents.** Pre-compute summaries. `ERROR` on the same line as the reason.
- **Eliminate flaky tests.** A test that fails intermittently fails catastrophically at agent scale.
- **Externalize working state.** Write progress and decisions to files; rewrite rather than append.

### 15. Iterate, Don't One-Shot

- Expect 2-5 refinement rounds on non-trivial tasks.
- Give specific feedback: "crashes on empty input" not "that's wrong."
- Ask for options: "Give me three approaches with tradeoffs."
- Build on partial success. 80% good? Keep that, iterate on the 20%.

### 16. Provide Precise Specifications

- Dictate function signatures — names, parameter types, return types.
- Provide input/output examples to eliminate ambiguity.
- Name the patterns you want: "Use strategy pattern" or "implement as middleware."
- Reference existing code as a template: "Follow the pattern in `handleUserCreate`."
- Constrain the solution space: "Use only standard library."

### 17. Know When to Take Over

- **Recognize the loop.** Same wrong answer after 2-3 attempts = agent is stuck.
- **Read the docs yourself** when the agent hallucinates API details.
- **Take over for domain-specific logic.** Business rules, security-critical code.
- **Split the work.** Write the tricky core; let the agent handle boilerplate and tests.
- **Account for training cut-off.** Provide docs for recent libraries/APIs.

### 18. Guard Long-Term Quality

- **Watch for verbose/redundant tests.** More tests ≠ better tests.
- **Check for lack of reuse.** Search for existing implementations before accepting new code.
- **Trim overly complex code.** Remove unnecessary abstractions, parameters, excess.
- **Beware brute-force fixes.** Increasing limits doesn't address root cause.
- **Review with three horizons:** (1) Works now? (2) Friction this iteration? (3) Maintainable in 6 months?

### 19. Build Context Incrementally

- Start with minimal rules. Begin with 3-5 based on your most common issues.
- Add rules when you notice repeated failures. Every rule earns its place.
- Prune rules that no longer help. Periodically review and remove dead weight.
- Context has a cost. 50 lines the agent follows > 500 lines it ignores.

### 20. Treat Agent Failures as System Signals

- Ask "what's missing?" not "why is it broken?" (Missing docs? Unclear tests? No linter?)
- Feed fixes back into the repository — update AGENTS.md, add linter rules, improve docs.
- Keep a "go-wrong" log. Track bad outputs; review weekly for patterns.
- Constrain architecturally, not just textually. Linters and pre-commit hooks > rules.

### 21. Security-Conscious Agent Use

- Treat rules files and MCP servers as supply chain components.
- Apply least privilege — restrict file system access, network, tool availability.
- Guard against approval fatigue — slow down for security-critical paths.
- Monitor agent-initiated changes: file modifications, dependency additions, network calls.
- Include agent workflows in threat modeling.
- Sandbox when possible — containers, VMs, restricted environments.

### 22. Calibrate Review Depth to Risk

Assess before reviewing: **(1) Probability** of error × **(2) Impact** if wrong × **(3) Detectability** by tests/types.

- Low probability + low impact + high detectability → light review.
- High probability + high impact + low detectability → deep review (as if you wrote it).
- "Imagine you're on call." Would you deploy this tonight and sleep soundly?

### 23. Write for Autonomous Agents

- Write issues/tasks as if for a newcomer — include explicit context.
- Include examples of desired patterns. Link to similar implementations.
- Make tasks atomic and self-contained.
- Define done criteria in the task.
- Provide custom instructions at the right scope: repo-level, org-level, task-level.

### 24. Engineer Multi-Agent Systems Deliberately

**Critical for hAIvemind** — this project IS a multi-agent system.

- **Use typed schemas at every agent boundary.** Define exact data shapes. Schema violations = contract failures.
- **Constrain allowed actions.** Define action schemas. Reject anything outside the schema.
- **Design for failure first.** Retry logic, fallback paths, circuit breakers before features.
- **Validate at boundaries, not after.** Check inputs before passing to the next agent.
- **Log intermediate state.** What each agent received, decided, and produced.
- **Expect retries and partial failures.** Design for idempotency.
- **Treat agents like distributed systems.** Contract testing, observability, graceful degradation.

### 25. Craft Agent-Computer Interfaces with Care

**Critical for hAIvemind** — the platform designs how agents interact with tools.

- Invest in tool descriptions like API docs — usage examples, edge cases, input formats.
- Error-proof tools (poka-yoke): require absolute paths, enforce unique matches.
- Surface constraints explicitly in tool descriptions.
- When tools fail, explain why specifically. Specific failure messages reduce retry loops by 40%.
- Test how agents actually use your tools — tool design is empirical.
- Prefer code as the action format over JSON for agent actions.

### 26. Make Your Codebase Agent-Legible

- **File structure is an interface.** `./billing/invoices/compute.ts` > `./utils/helpers.ts`
- **Prefer many small, well-scoped files.** Short enough to load in full stays fully in context.
- **Use semantic type and variable names.** Self-documenting types shrink the search space.
- **Make the repo the system of record.** Design docs as versioned markdown in the repo.
- **Use AGENTS.md as a map, not a manual.** Short entry point linking to deeper docs.

---

## Persona Guide

When starting a task, identify the primary persona. All share Core rules (1-10). Layer persona-specific rules:

| Persona | When | Critical Rules | Watch For |
|---------|------|----------------|-----------|
| **Coder** | Implementing, fixing, refactoring | 11 (TDD), 18 (Quality), 26 (Legibility) | Generating code without reading patterns; declaring done without tests |
| **Researcher** | Exploring, investigating, gathering info | 0 (Meta), 12 (Knowledge), 15 (Iterate) | Infinite exploration; not writing findings to files |
| **Reviewer** | Code review, auditing, evaluating | 22 (Review Depth), 13 (Cognitive Debt), 18 (Quality) | Rubber-stamping; reviewing when fatigued |
| **Planner** | System design, specs, task breakdown | 16 (Precise Specs), 23 (Autonomous Tasks), 24 (Multi-Agent) | Vague tasks; over-planning without implementing |
| **Operator** | CI/CD, deployment, infra, monitoring | 21 (Security), 14 (Environments), 20 (Signals) | Blanket production access; ignoring flaky tests |
| **Debugger** | Bug fixing, root cause analysis | 8 (Workflows), 20 (Signals), 11 (TDD) | Fixing without reproducing; brute-force fixes |

### Persona Combinations

- **Feature Development:** Planner → Coder → Reviewer (sequential)
- **Bug Investigation:** Researcher + Debugger (overlapping)
- **System Design:** Planner + Researcher (overlapping)
- **Self-Dev Session:** Planner → Coder → Reviewer → Operator (full cycle)

---

## hAIvemind-Specific Overrides

These override or extend the universal rules for this specific project:

| Override | Details |
|----------|---------|
| **Self-building** | Every agent session IS hAIvemind developing itself. See `.github/copilot-instructions.md`. |
| **Roadmap-driven** | All work comes from `docs/roadmap.md`. No freelancing. |
| **Protocol-first** | New message types → `shared/protocol.js` before use. |
| **Module boundaries** | Routes = HTTP, Services = logic, WS = real-time. |
| **ESM only** | No `require()`. 2-space indent. Single quotes. |
| **Push protocol** | Always use the tag-cleanup push command from copilot-instructions. |
| **Test-first** | New features get tests. No exceptions. Run before committing. |
| **Stats are concrete** | "1915 tests" not "comprehensive". "52 services" not "many". |
| **README < 200 lines** | Cut something if you need to add something. |

→ Full hAIvemind rules: `.github/copilot-instructions.md`
→ Architecture: `docs/architecture.md`
→ Roadmap: `docs/roadmap.md`
→ Definition of Done: `docs/definition-of-done.md`

---

## Quick Reference

| # | Rule | One-Liner |
|---|------|-----------|
| 0 | Meta-Rule | Research → synthesize → write → validate → repeat |
| 1 | Explore First | Read the codebase before changing it |
| 2 | Plan First | Break tasks into steps before implementing |
| 3 | Verify Work | Test, lint, and confirm after every change |
| 4 | Small Steps | One logical change at a time |
| 5 | Manage Context | Stay focused, be specific, seed with existing code |
| 6 | Communicate | Ask when unclear, explain when acting |
| 7 | Quality Code | Follow conventions, handle errors, be explicit |
| 8 | Structured Workflows | Follow the right workflow for the task type |
| 9 | Delegate | Parallelize independent work, use subagents |
| 10 | Avoid Anti-Patterns | No kitchen sinks, no over-correction, no skipping failures |
| 11 | Red/Green TDD | Write tests first, confirm they fail, then implement |
| 12 | Knowledge Assets | Hoard working examples; recombine them |
| 13 | Cognitive Debt | Understand your code; no black boxes |
| 14 | Agent Environments | Design tests, logs, progress files for agents |
| 15 | Iterate | Expect multi-turn refinement; build on partial success |
| 16 | Precise Specs | Dictate signatures, examples, constraints |
| 17 | Take Over | Recognize when the agent is stuck; split the work |
| 18 | Long-Term Quality | Watch for verbose tests, missed reuse, brute fixes |
| 19 | Context Incrementally | Start minimal; add rules from real failures |
| 20 | Failures = Signals | Fix the system, not just the prompt |
| 21 | Security | Least privilege, sandbox, monitor changes |
| 22 | Review Depth | Probability × impact × detectability |
| 23 | Autonomous Tasks | Atomic, self-contained, verifiable, newcomer-readable |
| 24 | Multi-Agent Systems | Typed schemas, failure-first, log everything |
| 25 | Agent Interfaces | Error-proof tools, specific failures, empirical design |
| 26 | Agent-Legible Code | Semantic names, small files, repo = system of record |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MrUnreal)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MrUnreal)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
