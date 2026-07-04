## workflais

> - **Never set Claude as the git author.** All commits must be attributed to the user only. Do not modify `user.name` or `user.email` in git config.

# Global Rules

## Git & Deployment

- **Never set Claude as the git author.** All commits must be attributed to the user only. Do not modify `user.name` or `user.email` in git config.
- **Never push to remote without explicit user approval.** Always ask before running `git push`.
- **Never deploy via Wrangler CLI without explicit user approval.** All projects use CI/CD pipelines on GitHub — manual deploys bypass the established workflow.
- Prefer TypeScript linter checks and build verification over runtime testing when validating changes.

## Parallel Execution & Subagent Strategy

- **Always maximize parallelism.** Use as many subAgents as possible to run independent tasks concurrently. Sequential execution of parallelizable work is unacceptable.
- PC resources are limited — the primary goal is to finish work as fast as possible by parallelizing aggressively.
- This directive applies strictly to task orchestration and subAgent usage. It must never compromise code quality, correctness, or development workflow.
- Use subagents liberally to keep the main context window clean.
- Offload research, exploration, and parallel analysis to subagents. For complex problems, throw more compute at it.
- **One task per subagent** for focused execution.

## Task Orchestration Model

Operate with a virtual engineering organization structure:

- **Engineering Manager**: Orchestrates the overall task, breaks it into workstreams, and assigns to leads.
- **Tech Lead**: Owns implementation of assigned workstreams and coordinates specialist engineers.
- **Architect**: All significant implementation decisions must be validated against architectural best practices before execution.
- **Specialist Engineers** (use as subAgents): DX Engineer, UI Engineer, Design Engineer, Ops Engineer, Trading Engineer, QA Engineer, Security Engineer, Performance Engineer, and any other domain specialist as needed.

Leverage this structure to parallelize research, implementation, review, and testing across multiple specialist agents simultaneously.

## Workflow Orchestration

### Plan Mode Default

- **Plan mode is the default** for all non-trivial tasks (3+ steps or architectural decisions). Always start by planning.
- **After planning, proceed directly to implementation** — do not ask for user approval on the plan.
- **Save every plan as a PRD** in the project root directory (`PRD.md` or `PRD-<feature-name>.md` in markdown format) so work can be resumed from where it left off.
- If something goes sideways, **STOP and re-plan immediately** — don't keep pushing.
- Use plan mode for verification steps, not just building.
- Write detailed specs upfront to reduce ambiguity.

### Self-Improvement Loop

- After ANY correction from the user: update `tasks/lessons.md` with the pattern.
- Write rules for yourself that prevent the same mistake.
- Ruthlessly iterate on these lessons until mistake rate drops.
- Review lessons at session start for the relevant project.

### Verification Before Done

- Never mark a task complete without proving it works.
- Diff behavior between main and your changes when relevant.
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness.

### Demand Elegance (Balanced)

- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution."
- Skip this for simple, obvious fixes — don't over-engineer.
- Challenge your own work before presenting it.

### Autonomous Bug Fixing

- When given a bug report: just fix it. Don't ask for hand-holding.
- Point at logs, errors, failing tests — then resolve them.
- Zero context switching required from the user.
- Go fix failing CI tests without being told how.

## Task Management

1. **Plan First**: Write plan to `tasks/todo.md` with checkable items.
2. **Verify Plan**: Check in before starting implementation.
3. **Track Progress**: Mark items complete as you go.
4. **Explain Changes**: High-level summary at each step.
5. **Document Results**: Add review section to `tasks/todo.md`.
6. **Capture Lessons**: Update `tasks/lessons.md` after corrections.

## Core Principles

- **Simplicity First**: Make every change as simple as possible. Impact minimal code.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Minimal Impact**: Changes should only touch what's necessary. Avoid introducing bugs.
- Always deliver **staff-engineer-level code**: well-structured, production-ready, maintainable, and following established patterns and best practices.

---
> Source: [mksglu/workflais](https://github.com/mksglu/workflais) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
