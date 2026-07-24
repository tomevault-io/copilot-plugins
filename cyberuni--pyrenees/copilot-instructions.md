## updating-packages

> How to update packages

# Updating Packages in the Monorepo

This rule explains how to update dependencies in a pnpm-based monorepo, including workspace management, best practices, and commit guidelines.

## Step-by-step Instructions

1. **Check for outdated packages**:
   - In the root: `pnpm outdated`
   - In a specific package: `pnpm outdated --filter <package-name>`
2. **Update a dependency in a specific package**:
   - `pnpm up <package-name>[@version] --filter <workspace>`
   - Example: `pnpm up lodash --filter ./libraries/design`
3. **Update all dependencies in a specific package**:
   - `pnpm up --latest --filter <workspace>`
4. **Update a dependency across all workspaces**:
   - `pnpm up <package-name> -r`
5. **Update all dependencies in the monorepo**:
   - `pnpm up --latest -r`
6. **Update the lock file**:
   - After updating, run `pnpm install` at the root to ensure the lock file is up to date.
7. **Commit your changes**:
   - Commit the updated `package.json` and `pnpm-lock.yaml` files.
   - Follow the commit message guidelines in [commit-messages.mdc](mdc:commit-messages.mdc).
8. **Create a PR if required**:
   - If updating dependencies for security or maintenance, describe the reason and any breaking changes in the PR.

## Guidelines
- Always check for breaking changes in the updated packages.
- Prefer updating to the latest stable version unless a specific version is required.
- Coordinate with the team when updating shared dependencies.
- Reference [installing-packages.mdc](mdc:installing-packages.mdc) for related instructions.

## Example

```sh
# Check for outdated packages in the root
pnpm outdated

# Update a single package everywhere
pnpm up lodash -r

# Update all dependencies in a workspace
pnpm up --latest --filter ./apps/web

# Update everything in the monorepo
pnpm up --latest -r
```

When you use this rule file, let me know this rule file is being used in the chat by mentioning the rule filename.

---
> Source: [cyberuni/pyrenees](https://github.com/cyberuni/pyrenees) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
