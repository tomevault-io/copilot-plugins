## scribble

> Collaborative drawing app that allows multiple users to work on a project simultaneously with real-time sync. AI agents assist development; AI is not an end-user feature.

# Scribble

Collaborative drawing app that allows multiple users to work on a project simultaneously with real-time sync. AI agents assist development; AI is not an end-user feature.

## Gotchas

- AI is used by the development team (agents) for building the app. The app itself does not contain AI features for end users.
- There are two separate dependency lists: `server/requirements.txt` (Python) and `client/package.json` (JavaScript).
- The Electron shell loads the same React app as the web version — no separate codebase.
- The frontend must be built (`cd client && npm run build`) before presenting a feature for approval. Tailwind CSS requires PostCSS compilation — raw `@tailwind` directives in the built CSS will cause a white screen.
- `postcss.config.js` and `autoprefixer` must be present for Tailwind CSS to compile correctly.

## Project Boundaries

This is a sub-project of the parent workspace. The project root is
`Scribble/`, not the parent directory.

## Conventions

- Mermaid in docs/diagrams/
- `requirements.txt` contains the high-level product requirements
- Session handoff via `docs/NEXT_SESSION.md` (written at end of every session, read at start)

## Technology Stack

- Implementation language: **Python, Javascript** (3.11)
- Build system: **setuptools**
- UI framework: **React Flask Electron**
- Testing framework: **pytest** (backend), **Jest + React Testing Library + jest-canvas-mock** (frontend)
- Database: **SQLite** (development) via **SQLAlchemy ORM** (migratable to PostgreSQL)
- Memory checking: **No memory checker configured**
- GUI testing: **None**
- Code style: **PEP 8**

## Development Team

The project uses a structured team of LLM agents:

| Agent | Role | Can write code? |
|---|---|---|
| `project-manager` (primary) | Orchestrates process, delegates tasks, gates on human approval at feature completion | No |
| `business-analyst` (subagent) | Converts requirements into tech specs (`docs/specs/`) | No |
| `ui-ux-designer` (subagent) | Designs CLI and React Flask Electron interfaces (`docs/design/`) | No |
| `software-developer` (subagent) | Writes Python, Javascript implementation | Yes |
| `code-reviewer` (subagent) | Reviews implementation and test code for style, efficiency, and architecture alignment | No |
| `security-expert` (subagent) | Reviews code for web application security vulnerabilities | No |
| `qa-engineer` (subagent) | Writes and runs tests (`tests/`) | Test code only |
| `documentation-specialist` (subagent) | Maintains AGENTS.md, README, and project-level docs | Docs only |
Agent definitions live in `.opencode/agents/`.

## Development Process (Iterative Model)

Build ONE feature at a time. Never start the next feature until the current
one is approved by the human (client).

For each feature, the Project Manager enforces this sequence automatically,
proceeding through all phases without stopping. Human approval is only
required at the final APPROVE phase:

```
1. PLAN     → Business Analyst writes tech spec in docs/specs/
2. DESIGN   → UI/UX Designer writes design doc in docs/design/
3. DEVELOP  → Software Developer implements the feature in Python, Javascript
3b. REVIEW  → Code Reviewer reviews the implementation code
              ↻ If issues found, loop back to step 3 (developer fixes)
              → Only proceed when Code Reviewer is satisfied
4. TEST     → QA Engineer writes tests and runs them
              ↻ If ANY test fails, loop back to step 3
              → Only proceed when ALL tests pass
4b. REVIEW  → Code Reviewer reviews the test code
              ↻ If issues found, loop back to step 4 (QA fixes tests)
              → Only proceed when Code Reviewer is satisfied
4c. SECURITY → Security Expert reviews the implementation for vulnerabilities
               ↻ If issues found, loop back to step 3 (developer fixes)
               → Only proceed when Security Expert is satisfied
4d. BUILD    → Build the frontend to verify it compiles correctly
               Run: cd client && npm run build
               ↻ If build fails, loop back to step 3 (developer fixes)
               → Verify the built CSS contains compiled Tailwind classes (not raw @tailwind directives)
               → Only proceed when build succeeds
5. APPROVE  → Present completed feature to human for approval
               → Human decides: approved (next feature) or revise
               ↻ If revision needed, return to step 3 (developer fixes)
```

The PM proceeds through phases 1–4d without pausing for human input.
The human is only consulted at step 5 when the feature is fully built and tested.

### Code Review Standards

The Code Reviewer enforces:
- **PEP 8** — Python naming, import order, type hints, no mutable defaults,
  `with` statements, exception specificity, f-strings
- **ESLint** — JavaScript `const`/`let`, arrow functions, React hooks rules,
  no `console.log` in production code
- **Code efficiency** — Python list comprehensions and generators where
  readable, React memoization where needed, SQLAlchemy N+1 avoidance,
  batched WebSocket messages
- **Architecture alignment** — module boundaries respected, thread
  safety maintained, Mermaid in docs/diagrams/
- **Readability** — code must be legible to a junior Python, Javascript developer

The review produces a report with issues categorized as:
- **Critical** — must fix (bugs, thread safety, core style violations)
- **Warning** — should fix (style violations, inefficiencies)
- **Suggestion** — nice to have (preferences)

## Rules

- **Architecture diagrams must stay in sync.** Every time the architecture
  changes (new module, new class relationship, new component), update the
  Mermaid in docs/diagrams/.
- **Code must be human-legible.** A junior Python, Javascript developer should understand
  it. Prefer clarity over cleverness.
- **Feature progress** tracked in `docs/progress.md` with status markers:
  Planned → In Design → In Development → In Testing → Awaiting Approval →
  Approved.
- **The human is the client.** They have final say on requirements and
  feature completion.
- **Session handoff is mandatory.** At the end of every session, the PM must:
  - Update `docs/progress.md` with the current phase and status
  - Write `docs/NEXT_SESSION.md` with: current feature, completed work,
    what needs to happen next, and any context the next PM needs to resume
- **At session startup**, the PM must read `docs/NEXT_SESSION.md` and
  `docs/progress.md` before doing anything else. If NEXT_SESSION.md
  exists and has instructions, follow them precisely. Do not redo completed
  work.
- **If an agent lacks write tools** (e.g., business-analyst, ui-ux-designer),
  delegate the file-writing to the `software-developer` agent as a write
  proxy. Provide the full content to write.

---
> Source: [Ablacon-Studios/Scribble](https://github.com/Ablacon-Studios/Scribble) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
