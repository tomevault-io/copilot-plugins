## create-opencode-plugin

> - **Typecheck**: `npm run lint` (runs `tsc --noEmit`)

# AGENTS.md

<instructions>
## One-Shot Verification
- **Typecheck**: `npm run lint` (runs `tsc --noEmit`)
- **Build**: `npm run build` (runs lint + `bun run build.ts`)
- **Safety**: You MUST NOT run long-running or blocking processes (dev servers, watch modes).

## Project Structure & Routing
- **Entry Point**: `create-opencode-plugin/src/index.ts`
- **Build Config**: `create-opencode-plugin/build.ts` (uses Bun for bundling)
- **Plugin Config**: `opencode.json` in the root directory.
- **Documentation**: 
  - `README.md`: Basic setup and usage.
  - `CODING-TS.MD`: TypeScript-specific conventions.
  - `TS59.MD`: TypeScript 5.9 feature usage and constraints.
  - `SKILL-BUNDLING-SPEC.md`: Technical details for skill bundling.

## Rules & Constraints
- **Runtime**: This project uses **Bun** for bundling and execution.
- **Dependencies**: Use `npm install` for dependency management.
- **Type Safety**: All contributions MUST pass `npm run lint` (TS 5.9 strictness).
- **Paths**: You SHOULD use relative paths within the `create-opencode-plugin/` subdirectory.
- **Read-Only**: `create-opencode-plugin/dist/` is generated and SHOULD NOT be modified.
</instructions>

<workflow>
## Task Routing
- **Adding Features**: Modify files in `create-opencode-plugin/src/`, then run `npm run build`.
- **Refactoring**: Update logic, verify types with `npm run lint`.
- **Skill Updates**: Refer to `SKILL-BUNDLING-SPEC.md` when modifying bundling logic.
</workflow>

---
> Source: [IgorWarzocha/create-opencode-plugin](https://github.com/IgorWarzocha/create-opencode-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
