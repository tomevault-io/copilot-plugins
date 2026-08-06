## agent-desktop-automation-mcp-server

> Agent Desktop Automation MCP Server is a TypeScript MCP server for policy-first desktop automation. The initial milestone is a safe foundation: expose server capabilities, classify requested automation actions, define licensed desktop interaction sessions, and keep real desktop mutation behind documented tool contracts, bounded session scope, audit logging, and user-granted task licenses.

# Agent Desktop Automation MCP Server Instructions

## Repository Context

Agent Desktop Automation MCP Server is a TypeScript MCP server for policy-first desktop automation. The initial milestone is a safe foundation: expose server capabilities, classify requested automation actions, define licensed desktop interaction sessions, and keep real desktop mutation behind documented tool contracts, bounded session scope, audit logging, and user-granted task licenses.

Do not add hidden autonomous desktop control, credential access, broad shell execution, destructive file operations, system configuration changes, unbounded screenshots, background polling, or raw mouse/keyboard control unless the requirements, safety model, feature design, and tests are updated first.

## Source Documents

Before feature work, read the relevant docs in this order:

1. `docs/product/requirements.md` - product scope, MVP behavior, acceptance criteria.
2. `docs/product/roadmap.md` - phased product boundary.
3. `docs/architecture/safety_model.md` - desktop automation safety posture and blocked actions.
4. `docs/architecture/licensed_desktop_interaction_sessions.md` - bounded task-license agency model.
5. `docs/process/development_workflow.md` - required development loop.
6. `docs/testing/test_strategy.md` - testing approach.
7. `docs/planning/mvp_implementation_plan.md` - implementation status, sequence, and acceptance gates.
8. `docs/planning/licensed_desktop_interaction_feature_design.md` - detailed session tool, provider, audit, and feature-slice design.

Use `README.md` for setup and commands.

## Required Workflow

Follow this loop for implementation tasks:

1. Feature planning.
2. Test planning.
3. Code implementation.
4. Test run.
5. Post-implementation documentation update.
6. Summary.

Do not start feature implementation before the feature and test plan are clear. If the user asks only for planning, review, or brainstorming, do not edit code.

## Scope Boundaries

- Keep MCP transport and protocol wiring in `src/server.ts` and `src/index.ts`.
- Keep reusable safety and policy logic in `src/policy/`.
- Keep session runtime and audit logic in `src/session/` once ADMCP-007 begins.
- Keep provider seams and provider implementations in `src/providers/` once ADMCP-009 begins.
- Keep MCP tool implementations small and test policy behavior outside the protocol layer when possible.
- Do not add a real desktop execution tool until its target, permissions, session-license behavior, observation requirements, failure modes, and audit output are documented.
- Do not reduce the project to a screenshot utility; the target loop is observe -> infer -> act -> observe delta -> repair -> continue.
- Do not expose raw desktop control outside a confirmed licensed session.
- Shell commands, credential access, and system changes are blocked in the initial safety model.
- Prefer narrow, explicit tools over broad remote-control primitives.

## Test Expectations

- For source changes, run `npm run typecheck`, `npm run test`, and `npm run build`.
- For documentation-only changes, no build is required; say that explicitly.
- Add unit tests for policy, tool input validation, and non-protocol logic.
- Add protocol or transport smoke tests when MCP wiring changes.
- Report tests run, skipped tests, failures, and residual risk.

## Documentation Rules

- Keep docs grouped by purpose under `docs/`.
- Product decisions belong in `docs/product/`.
- How-we-work instructions belong in `docs/process/`.
- Build sequencing belongs in `docs/planning/`.
- Test approach and test cases belong in `docs/testing/`.
- Technical design and safety notes belong in `docs/architecture/`.
- Prefer updating an existing doc over creating a near-duplicate file.

## Git And Reporting

- Check `git status --short` before edits and commits.
- Do not revert or overwrite user changes unless explicitly asked.
- Keep commits small and conventional.
- Final summaries should list changed files, behavior delivered, tests run, and next recommended step.

---
> Source: [TasirWimp/Agent_Desktop_Automation_MCP_Server](https://github.com/TasirWimp/Agent_Desktop_Automation_MCP_Server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
