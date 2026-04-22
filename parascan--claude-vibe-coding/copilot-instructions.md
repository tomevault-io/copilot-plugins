## claude-vibe-coding

> This file provides guidance for AI assistants (Claude Code and similar) working in this repository. It covers project structure, development workflows, conventions, and best practices.

# CLAUDE.md — Claude Vibe Coding

This file provides guidance for AI assistants (Claude Code and similar) working in this repository. It covers project structure, development workflows, conventions, and best practices.

---

## Repository Overview

**Claude-Vibe-Coding** is a repository dedicated to AI-assisted development workflows using Anthropic's Claude models and Claude Code CLI. It serves as a reference implementation and playground for "vibe coding" — a development style where a developer works collaboratively with an AI assistant to rapidly prototype, build, and iterate on software.

The primary application in this repository is a **Workout Generator** — a Progressive Web App (PWA) built with React and TypeScript that generates personalized strength and cardio workout sessions.

---

## Project Structure

```
Claude-Vibe-Coding/
├── CLAUDE.md                          # This file — AI assistant guidance
├── workout.html                       # Standalone single-file HTML build (212 KB)
├── .github/
│   └── workflows/
│       └── deploy.yml                 # GitHub Pages CI/CD pipeline
└── workout-app/                       # Main React + TypeScript application
    ├── index.html                     # HTML entry point
    ├── package.json                   # Dependencies & scripts
    ├── vite.config.ts                 # Vite config with PWA plugin
    ├── tsconfig.json                  # TypeScript base config
    ├── tsconfig.app.json              # App-specific TS config
    ├── tsconfig.node.json             # Node/build TS config
    ├── eslint.config.js               # ESLint configuration
    ├── public/
    │   ├── apple-touch-icon.png       # iOS app icon
    │   └── icons/
    │       ├── icon-192.png           # PWA icon (192px)
    │       └── icon-512.png           # PWA icon (512px)
    └── src/
        ├── App.tsx                    # Root component with top-level state
        ├── main.tsx                   # React entry point
        ├── App.css                    # Main styles
        ├── index.css                  # Global styles
        ├── components/
        │   ├── ModeSelect.tsx         # Strength vs Cardio mode picker
        │   ├── TimeInput.tsx          # Duration input component
        │   ├── WorkoutDisplay.tsx     # Strength workout plan display
        │   ├── WorkoutTimer.tsx       # Live strength timer with audio cues
        │   ├── ExerciseCard.tsx       # Individual exercise card with demo images
        │   ├── CardioDisplay.tsx      # Cardio session selection UI
        │   ├── CardioRoutePlanner.tsx # Map-based route toggle (loop / out-and-back)
        │   └── CardioTimer.tsx        # Live cardio interval timer with audio
        ├── data/
        │   ├── exercises.ts           # 40+ exercise definitions with demo images
        │   └── cardio.ts              # Cardio activity definitions (road & Peloton)
        └── lib/
            ├── generator.ts           # Strength workout generation algorithm
            └── cardioGenerator.ts     # Cardio session generation algorithm
```

---

## Technology Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| UI framework | React | 19.2.0 |
| Language | TypeScript | 5.9.3 |
| Build tool | Vite | 7.3.1 |
| Maps | Leaflet + react-leaflet | 1.9.4 / 5.0.0 |
| PWA | vite-plugin-pwa | 1.2.0 |
| Linting | ESLint | 9.39.1 |
| Deployment | GitHub Pages | — |

---

## Application Features

### Strength Mode

- Generates time-based dumbbell circuit workouts (10–60+ minutes)
- Multi-round structure with configurable work/rest/transition periods
- 40+ exercises targeting all major muscle groups
- Equipment: SelectTech 552 dumbbells with weight recommendations
- Demo images for each exercise (fetched from free exercise database)
- Estimated calorie burn (~8 cal/min)
- Live workout timer with slide animations and spoken audio cues

### Cardio Mode

- Road running and Peloton cycling session plans
- Interval generation based on total session duration
- Effort-based descriptions: easy / moderate / hard
- Workout types: HIIT, tempo, fartlek, steady state
- Live interval timer with spoken audio cues
- Route planner: map-based loop vs. out-and-back toggle with distance in miles

### PWA / Offline

- Installable on iOS and Android home screens
- Offline support via service worker caching
- Standalone single-file HTML build (`workout.html`) as a fallback

---

## Development Workflows

### Getting Started

1. Clone the repository and navigate to the app:
   ```bash
   git clone <repo-url>
   cd Claude-Vibe-Coding/workout-app
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Start the development server:
   ```bash
   npm run dev
   ```
   The app is served at `http://localhost:5173` with Hot Module Replacement.

### Available Scripts

Run all commands from inside `workout-app/`:

```bash
npm run dev       # Start Vite dev server (HMR enabled)
npm run build     # TypeScript type-check + Vite production build
npm run lint      # ESLint check across src/
npm run preview   # Preview production build locally
```

> **Note:** There is currently no automated test suite. The `npm test` script is not configured. When adding tests, use Vitest (already compatible with the Vite setup) and follow the test conventions below.

### Deployment

The app deploys automatically to GitHub Pages on every push to `main` that touches `workout-app/**`. The workflow is defined in `.github/workflows/deploy.yml`:

1. Checkout → Node 20 setup → `npm ci` → `npm run build`
2. Upload `workout-app/dist/` artifact
3. Deploy artifact to GitHub Pages

Manual deploys can also be triggered via `workflow_dispatch` in the GitHub Actions UI.

### Branch Strategy

- `main` — stable, production-ready code; deploys to GitHub Pages automatically
- `claude/<feature>-<session-id>` — AI-generated feature branches (created by Claude Code sessions)
- `feat/<description>` — human-authored feature branches
- `fix/<description>` — bug fix branches

**AI sessions must always work on a designated `claude/` branch and push to it — never directly to `main`.**

### Commit Conventions

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(optional scope): <short description>

Types: feat, fix, docs, style, refactor, test, chore, ci
```

Examples from this repo's history:
```
feat(route-planner): add loop and out-and-back route type toggle
fix(route-planner): display distances in miles instead of km
feat(cardio): add live interval timer with spoken audio cues
feat(timer): show exercise demo images during workout timer
docs: update CLAUDE.md with current codebase state
```

---

## Architecture Notes

### State Management

All top-level state lives in `App.tsx` using React `useState` hooks. State is passed down as props — no global state library is used. Keep this pattern for new features unless complexity clearly warrants otherwise.

### Workout Generation Logic

Business logic is isolated in `src/lib/`:

- `generator.ts` — Strength workout: binary search for optimal station count, priority rotation for balanced muscle group coverage, work/rest/transition time calculations.
- `cardioGenerator.ts` — Cardio session: interval strategy selection based on duration and activity type, effort-level sequencing.

Components consume data from `src/data/` and call generators to produce typed workout objects rendered by display/timer components.

### Adding a New Exercise

1. Add the definition to `src/data/exercises.ts` following the existing `Exercise` type shape.
2. Include a `demoUrl` if a free image is available.
3. Assign appropriate `muscleGroups` tags — the generator uses these for balanced rotation.

### Adding a New Cardio Activity

1. Add the activity definition to `src/data/cardio.ts`.
2. If the activity needs custom interval logic, extend `cardioGenerator.ts`.

---

## Code Conventions

### General

- Prefer **clarity over cleverness** — code should be readable at a glance.
- **Minimal abstraction** — only abstract when a pattern repeats three or more times.
- No premature optimization. Profile before you optimize.
- Avoid unnecessary comments; let self-documenting code speak for itself.
- All new business logic in `src/lib/` should have associated tests.

### Naming

| Context | Convention |
|---------|-----------|
| Variables / functions | `camelCase` |
| React components | `PascalCase` |
| TypeScript interfaces / types | `PascalCase` |
| Constants | `UPPER_SNAKE_CASE` |
| Files | `PascalCase.tsx` for components, `camelCase.ts` for lib/data |
| Git branches | `kebab-case-descriptions` |

### TypeScript

- Use explicit types for function parameters and return values in `src/lib/`.
- Prefer `interface` for object shapes that may be extended; `type` for unions and aliases.
- Avoid `any`. Use `unknown` with type narrowing if the type is genuinely unknown.

### Error Handling

- Validate at system boundaries (user input, external APIs, env vars).
- Do not add error handling for scenarios that cannot occur in practice.
- Surface errors with enough context to diagnose — avoid swallowing exceptions silently.

### Security

- Never commit secrets, API keys, or credentials. Use `.env` files (git-ignored).
- Sanitize all user input before using in queries, shell commands, or HTML output.
- Avoid `eval`, dynamic `require`, and similar patterns.
- When using the Claude API, never expose API keys client-side.

---

## Testing

### Philosophy

- Write tests for business logic (especially `src/lib/generator.ts` and `src/lib/cardioGenerator.ts`), not for trivial getters/setters.
- Tests should be fast, deterministic, and isolated.
- Prefer integration tests over mocking internal implementation details.

### Recommended Setup (Vitest)

Vitest is the recommended test runner — it integrates natively with Vite and requires minimal config. To add it:

```bash
npm install -D vitest @vitest/ui
```

Add to `package.json` scripts:
```json
"test": "vitest run",
"test:watch": "vitest",
"test:coverage": "vitest run --coverage"
```

### Test File Location

- Co-locate unit tests with source files: `src/lib/generator.test.ts` next to `src/lib/generator.ts`.
- Place integration/e2e tests in a top-level `tests/` directory.

---

## Claude Code Usage

### Session Workflow

1. Start Claude Code in the project root: `claude`
2. Describe your task clearly — include context, constraints, and expected outcomes.
3. Review all changes Claude proposes before approving file writes or shell commands.
4. Commit frequently with descriptive messages.
5. Push to your `claude/` branch when the task is complete.

### Key Commands

| Command | Purpose |
|---------|---------|
| `/help` | Show available commands |
| `/commit` | Create a git commit with an AI-generated message |
| `/review-pr` | Review an open pull request |
| `/fast` | Toggle fast mode |
| `Escape` | Interrupt the current operation |

### Permissions Model

Claude Code prompts for approval on:
- **File writes/edits** — review diffs carefully before approving
- **Shell commands** — especially destructive ones (`rm`, `git reset --hard`, etc.)
- **Network requests** — any external API calls

Trust the prompts. When in doubt, deny and ask Claude to explain the action.

---

## Claude API Integration

This project does not currently call the Claude API at runtime. If Claude API integration is added in the future, follow these guidelines.

### Setup

```bash
export ANTHROPIC_API_KEY=sk-ant-...
```

Or add to `.env` (git-ignored):
```
ANTHROPIC_API_KEY=sk-ant-...
```

### Model Selection

| Use Case | Recommended Model |
|----------|------------------|
| Complex reasoning, long context | `claude-opus-4-6` |
| Balanced performance | `claude-sonnet-4-6` |
| Fast, lightweight tasks | `claude-haiku-4-5-20251001` |

Default to `claude-sonnet-4-6` unless you have a specific reason to use another model.

### API Best Practices

- Always set a reasonable `max_tokens` limit.
- Use system prompts to establish context and constraints.
- Handle rate limit errors (429) with exponential backoff.
- Cache responses where appropriate to reduce cost.
- Log API calls in development for debugging, not in production.

### Example (TypeScript)

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic(); // reads ANTHROPIC_API_KEY from env

const message = await client.messages.create({
  model: "claude-sonnet-4-6",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Hello, Claude!" }],
});

console.log(message.content[0].text);
```

---

## AI Assistant Guidelines

When working as an AI assistant in this repository:

### Do

- Read existing code before modifying it.
- Make the minimal change needed to accomplish the task.
- Follow the conventions already established in the codebase.
- Run `npm run lint` and `npm run build` after making changes to catch type errors.
- Commit changes with descriptive messages following Conventional Commits.
- Push to the designated `claude/` branch.

### Do Not

- Push to `main` directly.
- Delete files or branches without explicit user confirmation.
- Add unrequested features, comments, or abstractions.
- Commit secrets or credentials.
- Use `--no-verify` to skip git hooks.
- Amend published commits (create new ones instead).
- Force push to shared branches.

### When Uncertain

- Ask before taking irreversible actions.
- Prefer the safer, more reversible option.
- Explain what you are about to do and why before doing it.

---

## Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `ANTHROPIC_API_KEY` | Claude API authentication key | Only if Claude API is integrated |

No runtime environment variables are required to run the workout app locally.

---

## Contributing

1. Create a branch from `main` following the branch naming conventions above.
2. Make changes inside `workout-app/` with clear, focused commits.
3. Run `npm run lint && npm run build` to verify no errors before pushing.
4. Open a pull request with a descriptive title and summary.
5. Address review feedback before merging.

---

## Useful Resources

- [Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)
- [Anthropic API Reference](https://docs.anthropic.com/en/api)
- [Anthropic TypeScript SDK](https://github.com/anthropics/anthropic-sdk-typescript)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Vite Documentation](https://vite.dev/)
- [vite-plugin-pwa](https://vite-pwa-org.netlify.app/)
- [Claude Code GitHub Issues](https://github.com/anthropics/claude-code/issues)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/parascan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
