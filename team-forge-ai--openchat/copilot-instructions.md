## always

> 1. Before writing any code, come up with an extremely good plan, review the plan, and then ask the user for permission to execute the plan.


## IMPORTANT: Development cycle

1. Before writing any code, come up with an extremely good plan, review the plan, and then ask the user for permission to execute the plan.
2. Then PAUSE and ask the user for permission to execute the plan.
3. Once the plan has completed, suggest areas of improvement and refactoring.

## Notes

- To check Typescript changes run: `pnpm typecheck` and `pnpm lint`
- To check Rust changes run: `cd src-tauri && cargo check`
- If there are any Typescript linting errors, run `pnpm fix` before attempting to fix the errors yourself.
- Never try to start a dev server or run `tauri dev` - prompt the user to do this.

REMEMBER you must always come up with a plan, and get sign off from the user, BEFORE writing any code.

---
> Source: [team-forge-ai/openchat](https://github.com/team-forge-ai/openchat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
