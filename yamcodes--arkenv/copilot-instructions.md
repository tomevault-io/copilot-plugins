## arkenv

> pnpm package manager guidelines


# pnpm Guidelines

This monorepo uses **pnpm** as the package manager.

## Package Manager

- Always use `pnpm` for all package management operations
- Never use `npm` or `yarn` commands
- The project uses `pnpm@10.20.0` (specified in `packageManager` field)

The only exception to this rule are the examples in the `examples/` folder ,which are supposed to be standalone and run on their own, regardless of monorepo setup.

## Common Commands

```bash
# Install dependencies
pnpm install

# Add a dependency to a workspace package
pnpm add <package> --filter <workspace-name>

# Add a dev dependency
pnpm add -D <package> --filter <workspace-name>

# Run a script in a specific package
pnpm --filter <workspace-name> <script>

# Run a script across all packages
pnpm run <script>
```

## Workspace Structure

This is a pnpm workspace monorepo. Packages are defined in:
- `packages/` - Published npm packages
- `apps/` - Applications (www, playgrounds)
- `tooling/` - Development tools (not published)
- `examples/` - Example projects

## Workspace Protocol

When referencing workspace packages, use the `workspace:*` protocol:

```json
{
  "dependencies": {
    "arkenv": "workspace:*"
  }
}
```

## Only Built Dependencies

Certain dependencies are configured to use `onlyBuiltDependencies` in `pnpm.onlyBuiltDependencies`. These are typically native dependencies that need special handling:

- `@biomejs/biome`
- `@sentry/cli`
- `@swc/core`
- `@tailwindcss/oxide`
- `@vercel/speed-insights`
- `esbuild`
- `sharp`

## Lock File

- Always commit `pnpm-lock.yaml`
- Never manually edit the lock file
- Run `pnpm install` to update the lock file when dependencies change

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yamcodes) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
