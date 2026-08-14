## dv0vd

> - Explicit user instructions for the current task take priority over this file.

# Execution Policy
## Rule Priority
- Explicit user instructions for the current task take priority over this file.
- Explicit user approval overrides only the restriction for the specifically approved command or action.
- All unrelated restrictions and project rules remain in effect.
## Failures and Retries
- Do not get stuck in loops.
- If an action, command, or tool fails, retry the same approach at most once.
- If the issue can be bypassed safely, use an alternative approach and continue.
- Treat environment errors such as `EACCES`, `ENOENT`, network failures, missing tools, and permission errors as terminal for that approach unless the user explicitly requested debugging.
- Do not diagnose or attempt to repair environment errors unless explicitly requested.
- If the task cannot be completed without the failed step, stop, report the exact reason, and request permission only for the specific action required to continue.
## User Approval
- After receiving explicit approval, execute the approved command or action immediately.
- Do not request confirmation again or reconsider whether the approved action is permitted.
- Do not extend approval to other commands or actions.
- If the approved action still cannot be completed, stop and report the exact reason without further retries.
## Code Execution
- Prefer static analysis of files.
- Do not run the following commands without explicit user permission:
  - `php`
  - `artisan`
  - `composer`
  - `node`
  - `npm`
  - `pnpm`
  - `yarn`
  - `bun`
  - `git`
## Shell
- Use only simple, one-off commands.
- Set a timeout of no more than 10 seconds.
- Do not use:
  - loops (`for`, `while`, `until`);
  - multi-line shell scripts;
  - complex pipelines;
  - bulk file processing.
## Long Operations
- Before large-scale searches, bulk modifications, or other potentially long-running operations:
  1. present a concise plan;
  2. identify the expected scope;
  3. request explicit user approval.
## Hanging Tools
- If a tool hangs or exceeds a reasonable response time, cancel the attempt.
- Do not wait indefinitely or repeat the same hanging operation.
- Briefly report the issue and continue with an alternative approach when possible.
# Development Principles
- Make minimal changes.
- Follow existing project patterns.
- Reuse existing code before writing new code.
- Do not introduce new abstractions unless necessary.
- Do not refactor outside the task scope.
- Do not add comments unless necessary.
- Keep code testable.
- Show only relevant changed fragments.
- Answer at senior level without explaining basic concepts.
- If multiple solutions exist, provide a short list without unnecessary detail.
- Sort entities alphabetically only when it does not affect meaning, contracts, dependencies, or execution order.
## Git Commit Messages
- Follow the Conventional Commits specification.
- Include a scope when reasonably identifiable, for example: `feat(auth): add token refresh`.
- Use a single short sentence.
- Do not include detailed descriptions or bullet lists.
# Ignored Files and Directories
Do not analyze or modify the following unless the task explicitly requires them:
- `node_modules/`
- `vendor/`
- `dist/`
- `build/`
- `.nuxt/`
- `.output/`
- `coverage/`
- `storage/logs/`
- `bootstrap/cache/`
- `public/build/`
- `public/hot`
- `*.min.js`
- `*.min.css`
- `package-lock.json`
- `pnpm-lock.yaml`
- `yarn.lock`

---
> Source: [dv0vd/dv0vd](https://github.com/dv0vd/dv0vd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
