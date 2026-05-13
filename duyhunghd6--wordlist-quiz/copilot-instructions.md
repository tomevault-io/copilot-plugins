## wordlist-quiz

> - **Project Structure**: This project enforces a maximum depth of 2 levels for nested project directories.

# GEMINI Dev Guidelines

## Project Capabilities & Constraints

- **Project Structure**: This project enforces a maximum depth of 2 levels for nested project directories.
- **Task & Dependency Management**: We are using the `beads` CLI to manage tasks and dependencies.
- **Product Requirements**: The `README.md` file serves exactly as the Product Requirements Document (PRD).
- **Design System**: Elements and components must follow the Design System located at `docs/design`. This is the "document" for all designs to follow, reducing the gap between the PRD and the real product.
- **Documentation Ground-Truth**: All architectural, pedagogical, and system design documentation (other than PRD) is centralized exclusively in the `./docs` directory. AI Agents must reference this directory for project context and component logic.

### Documentation Directory Mapping (`./docs/`)
To ensure knowledge is centralized and updated correctly, ALL Agents must adhere to the following directory structure inside `docs/` for reading context and creating new files:

- **`docs/architecture/`**: Detailed architectural blueprints, component logic, game mechanics flow, and engineering decisions. Agents should read this to understand *how* the app works underneath and update it when creating new core components.
  - `game_journey_logic.md`: Details the component logic, filtering conditions, and rendering constraints for the pedagogical map (Game Journey).
  - `runner-game-engine.md`: Details the physics loop, entity states, and obstacle configuration for the Endless Runner game wrappers.
- **`docs/design/`**: The Wordlist Quiz Design System. Contains foundational UI tokens, CSS layouts, and HTML showcases. Agents MUST update this directory BEFORE modifying any UI components in `src/`.
  - `README.md`: The core rulebook and ID registry (124+ nodes) mapping the UI components to React.
- **`docs/requirements/`**: Source requirements, PRDs, and external datasets. Read-only context for data ingestion.
  - `study-materials/grade3/esl/learning-objectives.md`: Baseline curriculum mapping and objectives for Grade 3 English acquisition.
- **`docs/api/`**: API specifications, endpoint documentation, and payload schemas. (Currently empty/reserved for backend scaling).
- **`docs/guides/`**: Developer setup runbooks, contribution guidelines, and how-to manuals. (Currently empty/reserved).
- **`docs/research/`**: Exploratory research, prototyping notes, and pedagogical methodological studies. Use this to store outputs when analyzing abstract technological or pedagogical concepts.
  - `Building Grammar Skills for Vietnamese Kids.md`: Core cognitive framework for teaching English syntax to Vietnamese learners.
  - `Interactive ESL Tense App Design.md`: UX patterns mapping pedagogical theories to interactive mobile architectures.
  - `Phương pháp luận phần mềm học tập trẻ em.md`: Systemic methodology guiding retention through spaced repetition (SM-2 Lite) and active recall.
  - `ideation/tenses_game_ideation.md`: Concept brainstorming and scratchpad logic for tense-based gameplay loops.
- **Root Files (`docs/*.md`)**: High-level macro overlays.
  - `architecture.md`: The defining Wordlist Quiz Brownfield strategy and global integration boundaries.
  - `brownfield-architecture.md`: Deeper evaluation of the codebase layout, constraints, and transition from monolithic structs.

## Agile Dev Agentic Coding Skills

For this type of project, the Agentic SE workflow utilizes four core skills/personas:

1. **Business Analyst Persona (`agile-analyst`)**
2. **Architect Persona (`agile-architect`)**
3. **Engineer Persona (`agile-engineer`)**
4. **QA Persona (`agile-qa`)**

---

## Workflow Orchestration

### 1. Plan Mode Default

- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately - don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity

### 2. Subagent Strategy

- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One task per subagent for focused execution

### 3. Self-Improvement Loop

- After ANY correction from the user: update `tasks/lessons.md` with the pattern
- Write rules for yourself that prevent the same mistake
- Ruthlessly iterate on these lessons until mistake rate drops
- Review lessons at session start for relevant project

### 4. Verification Before Done

- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness

### 5. Demand Elegance (Balanced)

- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes - don't over-engineer
- Challenge your own work before presenting it

### 6. Autonomous Bug Fixing

- When given a bug report: just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests - then resolve them
- Zero context switching required from the user
- Go fix failing CI tests without being told how

### 7. Port Conflicts

- The local development server (Vite) MUST run on port **9997**.
- If `npm run dev` fails due to the port already being in use, you MUST autonomously kill the existing process on port 9997 (e.g., `lsof -ti:9997 | xargs kill -9`) before retrying. Do not ask the user to free the port.

### 8. Design System First (CRITICAL RULES)

- **ALL UI changes, including new game screens (Game Scenes) or components, MUST be implemented in the Design System (`docs/design`) FIRST.**
- Only after the Design System is updated should you apply those changes to the actual React application screens.

### 9. Centralized Tense Question Bank

- **ALL grammar/tense games MUST fetch and use the centralized TOON database (`public/db/tense_sentences_esl.toon`).**
- Do NOT mock grammar questions using regex scrambling in individual components.
- The TOON format is a pipe-delimited (`|`) tabular standard optimizing parsing speed and context windows over JSON.

## Task Management (Beads CLI Workflows)

### 0. When to use Beads CLI & Git (CRITICAL)

- **Hotfixes & Small Tasks (DEFAULT):** Just implement the fix directly. **DO NOT** create beads issues, and **DO NOT** commit or push to GitHub.
- **New Planned Tasks:** Only commit and create beads issues if it is a new task that needs to be planned, or if the user explicitly asks you to use the beads CLI.

All four personas (Analyst, Architect, Engineer, QA) must strictly follow this `beads` CLI task management workflow for planned work. Do NOT use `tasks/todo.md`.

### 1. Check & Claim (All Personas)

- NEVER start work blindly. Run `bd ready --json` to find unblocked work.
- Review task context: `bd show <id> --json`.
- Claim atomically: `bd update <id> --claim --json`.

### 2. Persona-Specific Execution

- **`agile-analyst`**: Create epics/tasks (`bd create ... -t feature -p 2`), execute research templates (`/gsafe:create-doc`, `/gsafe:facilitate-brainstorming-session`, etc).
- **`agile-architect`**: Break down epics into technical stories (`bd create ... -t story --deps parent:<epic_id>`), execute design workflows (`/gsafe:risk-profile`, etc).
- **`agile-engineer`**: Execute implementation sequentially (`/gsafe:execute-checklist`). Track discovered bugs immediately (`bd create "Found X" -t bug --deps discovered-from:<id>`).
- **`agile-qa`**: Review implementations (`/gsafe:qa-gate`), address feedback (`/gsafe:apply-qa-fixes`), or reject/fail items with clear reasons (`bd update <id> --fail "Reason"`).

### 3. Iterative Progress & Documentation

- **Explain Changes**: One `beads` item can be finished through multiple iterations. Update the issue description with what was tried in previous iterations and provide a high-level summary at each step.
- **Document Results**: Add a review section or comment to the `beads` issue upon completion.
- **Capture Lessons**: Update `tasks/lessons.md` after any self-correction or user correction.

### 4. Close & Sync (All Personas)

- Once validations pass or the document/session is complete, close the issue: `bd close <id> --reason "..." --json`.
- Always sync state: `bd sync && git pull --rebase && git push`.
- **Version Bump**: Every deploy/push to Vercel requires updating the app version in the `<title>` tag of `public/index.html` (for example, incrementing to v1.21).
- Run `bd ready --json` again to hand off state.

## Core Principles

- **Simplicity First**: Make every change as simple as possible. Impact minimal code.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Minimal Impact**: Changes should only touch what's necessary. Avoid introducing bugs.

---
> Source: [duyhunghd6/wordlist-quiz](https://github.com/duyhunghd6/wordlist-quiz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
