## pulseboard

> This project uses a **multi-agent workflow**:

# Repository Guidelines

## Workflow & Agent Responsibilities

This project uses a **multi-agent workflow**:

- **Claude**: Handles planning, bug fixes, task tracking, and code review. Creates and maintains **TODO.md** with detailed task breakdowns.
- **Other Agents (e.g., ChatGPT Codex)**: Implement features and tasks listed in **TODO.md**. Update task status and commit changes following the guidelines below.

**Task Workflow**:
1. Claude breaks down feature requests into tasks in **TODO.md**
2. Implementation agent picks up tasks from **TODO.md**
3. Agent updates **WORKING.md** to claim the task and prevent conflicts
4. Agent implements the task following these guidelines
5. Agent updates task status in **TODO.md** and **CHANGELOG.md**
6. Agent commits changes with conventional commit messages (see format below)
7. Agent clears their entry from **WORKING.md**
8. Claude reviews changes and provides feedback

Always check **TODO.md** for current tasks and **WORKING.md** for active work before starting.

## Project Structure & Module Organization
Code lives in `app/` (Next.js routes and server actions) and `components/` (shared UI). Feature helpers sit in `lib/`, while static data sources are under `data/`. Public assets and icons are in `public/`. Configuration lives in files at the repo root (`next.config.ts`, `eslint.config.mjs`, `tailwind` and `postcss` configs). Support scripts, including vector-store helpers, are in `scripts/`; they expect to be run from the project root.

## Build, Test, and Development Commands
Install dependencies with `bun install` (preferred) or `npm install`. Start local development via `bun run dev` to launch Next.js with Turbopack. Build production assets using `bun run build`, then verify with `bun run start`. Lint the codebase using `bun run lint`. To work with the embedded ChromaDB service, use `bun run chroma:start` (or `stop`, `restart`, `status`, `logs`).

## Coding Style & Naming Conventions
This project uses TypeScript, React Server Components, and Tailwind. Follow ESLint recommendations (`bun run lint`) and prefer 2-space indentation. Name React components and files with PascalCase (e.g., `TeamCard.tsx`), hooks with `use` prefixes, and utility modules in `lib/` with camelCase filenames. Co-locate component-specific styles or helper functions next to the component. Keep imports absolute from the project root when clarity improves readability.

## Testing Guidelines
Automated tests are not yet in place; add new tests alongside features. Use Playwright or Jest with React Testing Library depending on scope, naming files `*.spec.ts(x)` next to the code under test. When adding features, cover edge cases for async server actions and serializing data read from `data/` sources. Ensure Chroma-dependent workflows are mocked in tests.

## Commit & Pull Request Guidelines

### Multi-Agent Commit Format

**Format**: `[agent-name] type: description`

**Examples**:
- `[claude] docs: update workflow guidelines`
- `[codex] feat: implement member search`
- `[claude] fix: correct member profile display bug`
- `[codex] chore: update dependencies`

**Types**: Use Conventional Commit prefixes:
- `feat:` - New feature
- `fix:` - Bug fix
- `docs:` - Documentation only
- `chore:` - Maintenance, dependencies, config
- `refactor:` - Code restructuring without behavior change
- `test:` - Adding or updating tests
- `style:` - Formatting, whitespace

**Guidelines**:
- Keep subject line under 72 characters
- Always pull before committing: `git pull`
- Expand critical context in the commit body
- Reference issue/task numbers when applicable

### Pull Requests

Pull requests should include:
- Concise summary of changes
- Screenshots or recordings for UI changes
- Testing evidence (`bun run lint`, relevant specs)
- References to tracked issues or TODO.md tasks
- Flag any required environment variables or Chroma setup changes

## File Ownership & Conflict Prevention

To prevent conflicts between agents, follow these ownership guidelines:

### Claude Owns (Primary)
- `*.md` files (documentation, changelogs, guidelines)
- `TODO.md` - Creates and organizes tasks
- `WORKING.md` - Monitors but doesn't claim work
- Bug fixes in any file

### Implementation Agents Own (Primary)
- `*.ts`, `*.tsx`, `*.js`, `*.jsx` - All code files
- `*.css`, `*.json` - Styling and configuration
- New feature development
- Large-scale refactoring

### Shared (Requires Coordination)
- `data/*.json` - High conflict risk, coordinate via **WORKING.md**
- `package.json` - Coordinate dependency changes
- Config files (`next.config.ts`, etc.) - Coordinate changes

### Conflict Prevention Protocol

1. **Before editing any file**:
   - Check **WORKING.md** for active work on that file
   - If file is in use, wait or coordinate with the other agent
   - Add your work to **WORKING.md** before starting

2. **Always pull before starting**: `git pull`

3. **For high-risk files** (data/*.json, package.json):
   - Announce in **WORKING.md**
   - Make a backup: `cp file.json file.json.backup`
   - Commit immediately after changes
   - Notify other agents in commit message

4. **If you encounter a merge conflict**:
   - Document in **WORKING.md** with "⚠️ CONFLICT"
   - Resolve carefully, preserving both agents' changes when possible
   - Test thoroughly after resolution
   - Document resolution in commit message

## Environment & Services
ChromaDB runs locally via the scripts in `scripts/`. Ensure Docker Desktop is available before invoking `bun run chroma:start`. Environment variables belong in `.env.local` (not checked in); mirror any changes in the PR description so other contributors can reproduce your setup.

---
> Source: [alexleekt/pulseboard](https://github.com/alexleekt/pulseboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
