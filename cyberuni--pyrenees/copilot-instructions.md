## project-structure

> This rule describes the standard project structure and file organization conventions for this monorepo. Follow these guidelines when adding new code, packages, or configuration files.

# Project Structure & File Organization Guidelines

This rule describes the standard project structure and file organization conventions for this monorepo. Follow these guidelines when adding new code, packages, or configuration files.

## Top-level Directory Overview

- `/apps/` — Folder that contains one or more application projects (e.g., web apps, storybook, etc.)
- `/libraries/` — Folder tha contains one or more shared libraries and modules (e.g., design system, types)
- `/tools/` — Project-specific tools, scripts, or configuration helpers
- `/react/` — React-specific projects
- `/.cursor/` — Cursor rules and configuration
- `/.github/` — GitHub workflows and automation
- `/.vscode/` — VSCode workspace settings and recommendations
- `/node_modules/` — Installed dependencies (auto-generated)
- `pnpm-workspace.yaml` — pnpm workspace configuration
- `package.json` — Root package manifest
- `pnpm-lock.yaml` — pnpm lock file
- `tsconfig.json` — Root TypeScript configuration
- Other dotfiles — Project-wide configuration (e.g., `.editorconfig`, `.gitignore`)

## Guidelines

1. **Place new apps in `/apps/`**
2. **Place shared library code in `/libraries/`**
4. **Place project-specific tools in `/tools/`**
5. **Use snake_case for directory and file names**
6. **File names can contain additional sub-context, such as tests, context, and TypeScript types. They are added to the file name seperating by `.`. For example: `data-grid.spec.tsx`, `data-grid.ctx.ts`, `data-grid.theme.ts`, `data-grid.types.ts`.**
7. **Place test and Storybook file next to the source file, not in different folders**
8. **Keep configuration files at the root unless project-specific**
9.  **Do not place code or packages in the root directory**
10. **Update this rule if the structure changes**

## Example Structure

```text
/ (repo root)
├── apps/
│   └── storybook/
├── libraries/
│   ├── design/
│   └── types/
├── tools/
│   └── tsconfig/
├── react/
│   └── pyrenees/
├── .cursor/
│   └── rules/
├── .github/
├── .vscode/
├── package.json
├── pnpm-workspace.yaml
└── ...
```

When you use this rule file, let me know this rule file is being used in the chat by mentioning the rule filename.

---
> Source: [cyberuni/pyrenees](https://github.com/cyberuni/pyrenees) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
