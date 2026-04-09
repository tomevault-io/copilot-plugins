## zypher-app-template

> See README.md for project overview and getting started.

# CLAUDE.md

See README.md for project overview and getting started.

## Tooling

| What            | Backend (`api/`)                                   | Frontend (`web/`)               |
| --------------- | -------------------------------------------------- | ------------------------------- |
| Runtime         | Deno                                               | Node (via Vite)                 |
| Package manager | `deno add jsr:@scope/package` / `deno add npm:pkg` | `cd web && pnpm add pkg`        |
| Docs lookup     | `deno doc jsr:@zypher/agent`                       | —                               |
| Linter          | `deno lint`                                        | `cd web && pnpm lint` (ESLint)  |
| Formatter       | `deno fmt`                                         | `cd web && pnpm format` (Biome) |
| Type check      | `deno check main.ts`                               | `cd web && pnpm typecheck`      |
| Dev server      | `deno task dev`                                    | `cd web && pnpm dev`            |

## Restrictions

- Do NOT use npm/pnpm for backend deps — use `deno add` only.
- Do NOT use npm/yarn/deno for frontend deps — use `pnpm` only.
- Do NOT use Prettier — Biome is the formatter.
- Do NOT import from `@zypher/ui` JSR package. Use `@/lib/zypher-ui` instead
  (locally copied source).
- Do NOT replace the Zypher system prompt entirely — extend via
  `customInstructions` in `getSystemPrompt()`, or agent skills and programmatic
  tool calling will break.
- Do NOT manually edit `web/pnpm-lock.yaml`.

## Vendored Code

These directories are ignored by ESLint and can be customized but are not
project-authored code:

- `web/src/components/ui/` — shadcn/ui components
- `web/src/components/ai-elements/` — chat UI components

`web/src/lib/zypher-ui/` is also vendored (copied from `@zypher/ui` with inlined
types) but is linted. Edit `types.ts` if upstream types change.

## Adding shadcn Components

```sh
cd web && pnpm dlx shadcn@latest add <component-name>
```

Browse available components: https://ui.shadcn.com/docs/components Config:
`web/components.json` (style: new-york, icons: lucide).

## Common Changes

**New tool:** Create in `api/tools/`, register in `tools` array in
`api/agent.ts`. Run `deno doc jsr:@zypher/agent` for the Tool interface.

**New MCP server:** Add to `mcpServers` array in `api/agent.ts`. Types:
`"command"` (local process) or `"remote"` (HTTP endpoint).

**System prompt:** Uncomment `systemPromptLoader` in `api/agent.ts`. Always call
`getSystemPrompt()` as the base and pass custom instructions via
`customInstructions`.

**Frontend path alias:** `@/` maps to `web/src/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/corespeed-io)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/corespeed-io)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
