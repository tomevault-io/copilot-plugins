## 01-project-overview

> Project overview, stack, and structure for Workflow Editor


# Workflow Editor – Project Overview

## Purpose

GUI for editing and visualizing **GitHub Actions workflows** (YAML). Users can view workflows as a flow graph (jobs and dependencies) and edit them via the UI. The app parses workflow YAML into a typed model and can serialize back to YAML.

## Stack

- **Package manager**: PNPM only. Use `pnpm install`, `pnpm add`, etc. No npm/yarn.
- **Target**: VSCode Extension (runs in VSCode, Cursor, Windsurf, and other VSCode-based IDEs).
- **Framework**: React 18 with TypeScript (webview UI).
- **Extension Host**: Node.js with VSCode Extension API.
- **Build**: TypeScript compiler for extension, Webpack for webview bundle.
- **Flow editor**: **React Flow** (`@xyflow/react` v12). Use for job graph (nodes = jobs, edges = `needs`).
- **Styling**: Tailwind CSS. No CSS-in-JS; use utility classes and `tailwind.config.js`.
- **Workflow format**: YAML. Use the `yaml` package for parse/stringify. Types live in `src/types/workflow.ts` (aligned with [GitHub workflow syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)).
- **Testing**: Vitest. Unit tests next to or under `src/**/*.test.ts` and `*.test.tsx`.
- **Linting**: ESLint with TypeScript and React (hooks, refresh). Run with `pnpm lint`.
- **Packaging**: `@vscode/vsce` for creating `.vsix` packages.

## Directory Structure

- `src/` – Extension source.
  - `extension/` – Extension host code (Node.js, VSCode API).
    - `extension.ts` – Extension entry point and activation.
    - `webview.ts` – Webview provider and message handling.
  - `webview/` – React app running in webview.
    - `App.tsx`, `main.tsx` – React app entry and root component.
    - `components/` – React components.
    - `lib/` – Core logic: `parseWorkflow`, `serializeWorkflow`, and tests.
  - `types/` – Shared TypeScript types (e.g. `workflow.ts`).
- `out/` – Compiled extension code (TypeScript output).
- `media/` – Webpack bundle output (webview JavaScript and CSS).
- Config at root: `webpack.config.js`, `vitest.config.ts`, `tailwind.config.js`, `tsconfig.json`, `tsconfig.webview.json`, `eslint.config.js`.

## Conventions

- Use path alias `@/` for `src/` (e.g. `import type { Workflow } from '@/types/workflow'`).
- Workflow model: `Workflow` → `jobs` (record of `WorkflowJob`) → each job has `steps` (`WorkflowStep[]`). Parse returns `{ workflow, errors }`; always handle `errors`.
- Keep UI and workflow logic separate: parsing/serialization in `lib/`, React Flow and UI in components.

---
> Source: [timoa/workflow-editor](https://github.com/timoa/workflow-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
