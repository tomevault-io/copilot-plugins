## esg-hub-prototype

> File structure, UI components, data mapping, and development workflow guidelines

# File naming and structure conventions
1. All mock data is in data/ folder. We will use json-server and api will be written in api.ts.
2. All source code is in src/ and we use app router of Next.
3. Interface, datatypes in src/types in component names.

# Use tools
1. Use context7 mcp to check for latest documentation before generating code.
2. Use the exact library in the techstack.

# UI components
1. Use shadcn/ui if the component is available. 
2. All UI components will be in components/.
3. All common components like button, typography is in components/ui.
4. Component for one page (route) will be in src/components/{route}.tsx
5. You need to add 'use client' in the head of every UI component file.

# Data type mapping
1. Do not use any type anywhere.
2. Define types in correct file in src/types.
3. Read schema.sql to understand the data types and match it with the types in src/types.
4. When create new api route, you may need to create new types to map the data from schema to the types in src/types.

# Problems and lint
1. Use `pnpm lint` to check for problems.
2. Use `pnpm lint:fix` to fix problems.
3. Use `pnpm check` to check for types.
4. Use `pnpm build` to check for build errors.
5. Fix all problems after completing a big task.

# Plan and schedule
1. Keep track and update docs/todo.md for tasks.
2. After completing a task, update docs/memory.md with the task details. Read the file when you need to remember what you have done.
3. Leave todo comments in the code if needed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/datamonsterr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
