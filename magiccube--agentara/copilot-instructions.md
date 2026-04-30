## agentara

> > All documents should be written in simple and concise English.


> All documents should be written in simple and concise English.

@docs/overview.md

## Notes

- Use `bun check` to type-check and lint your code after you finish writing.
- Always use `logger` or `createLogger` from `@/shared` for logging. Never use `console.log`/`console.error` directly.
- Always import from `@/shared` directly, not from sub-paths like `@/shared/messaging` or `@/shared/utils`. The barrel export covers everything.
  - For example, instead of `import { logger } from "@/shared/logging"`, use `import { logger } from "@/shared"`.
- Use `context7` when you're working with `bunqueue`.
- IMPORTANT: No more Korean, I'm Chinese!
- When adding a new page, always check `web/src/components/app-breadcrumb.tsx` and `web/src/components/app-sidebar.tsx` to add the route and nav entry.

---
> Source: [MagicCube/agentara](https://github.com/MagicCube/agentara) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
