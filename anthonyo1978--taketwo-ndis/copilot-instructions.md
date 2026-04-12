## taketwo-ndis

> - `pnpm run dev` - Start development server

# Claude Code Configuration

## Available Scripts

- `pnpm run dev` - Start development server
- `pnpm run build` - Build for production
- `pnpm run start` - Start production server
- `pnpm run lint` - Run ESLint
- `pnpm run lint:fix` - Fix ESLint issues
- `pnpm run prettier` - Check code formatting
- `pnpm run prettier:fix` - Fix code formatting
- `pnpm run test` - Run unit tests
- `pnpm run test:watch` - Run tests in watch mode
- `pnpm run e2e:headless` - Run E2E tests
- `pnpm run storybook` - Start Storybook

## Development Notes

- Uses Next.js 15 with App Router
- Tailwind CSS v4 for styling
- TypeScript with strict mode
- Testing with Vitest + Playwright
- Component development with Storybook
- Absolute imports configured

## Project Awareness & Context

 - Always read PLANNING.md at the start of a new conversation to understand the project's architecture, goals, style, and constraints.
  - Check TASK.md before starting a new task. If the task isn’t listed, add it with a brief description and today's date.
- Use consistent naming conventions, file structure, and architecture patterns as described in PLANNING.md.

Code Structure & Modularity

Never create a file longer than 500 lines of code. If a file approaches this limit, refactor by splitting it into modules or helper files.
Organize code into clearly separated modules, grouped by feature or responsibility. 
Use clear, consistent imports (prefer relative imports within packages).
Use clear, consistent imports (prefer relative imports within packages).

🧪 Testing & Reliability

Always create unit tests for new features (functions, classes, routes, etc).
After updating any logic, check whether existing unit tests need to be updated. If so, do it.
Tests should be co-located with components (e.g., Button.tsx + Button.test.tsx).
Include at least:
1 test for expected use
1 edge case
1 failure case

✅ Task Completion

Mark completed tasks in TASK.md immediately after finishing them.
Add new sub-tasks or TODOs discovered during development to TASK.md under a “Discovered During Work” section.


📚 Documentation & Explainability
Update README.md when new features are added, dependencies change, or setup steps are modified.
Comment non-obvious code and ensure everything is understandable to a mid-level developer.
When writing complex logic, add an inline # Reason: comment explaining the why, not just the what.

🧠 AI Behavior Rules

Never assume missing context. Ask questions if uncertain.
Never hallucinate libraries or functions – only use known, verified npm packages.
Always confirm file paths and module names exist before referencing them in code or tests.
Never delete or overwrite existing code unless explicitly instructed to or if part of a task from TASK.md.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anthonyo1978)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/anthonyo1978)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
