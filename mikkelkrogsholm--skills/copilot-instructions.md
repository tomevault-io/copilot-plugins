## skills

> This project creates Claude Code skills that wrap CLIs built with [bunli](https://github.com/user/bunli). Each skill is **self-contained** — the CLI is bundled inside the skill folder (`skills/<name>/cli/`) so the skill is portable and can be packaged as a `.skill` file via skill-creator's `package_skill.py`.

# Skills Factory

This project creates Claude Code skills that wrap CLIs built with [bunli](https://github.com/user/bunli). Each skill is **self-contained** — the CLI is bundled inside the skill folder (`skills/<name>/cli/`) so the skill is portable and can be packaged as a `.skill` file via skill-creator's `package_skill.py`.

## Development Method: Documentation-Driven + Test-Driven

All work is delegated to sub-agents. The main conversation acts as orchestrator — monitoring progress, tracking phases via the **task system** (`TaskCreate`/`TaskUpdate`), and never doing implementation work directly.

When the user points at an online source (website, API, feed, etc.), the orchestrator creates tasks for each phase, sets up dependencies, and delegates to sub-agents. Phases run sequentially unless explicitly noted as parallelizable.

---

## Phases

### Phase 1: Research

Research is split into two steps. The orchestrator decides after Step 1 whether Step 2 is needed.

**Step 1: Scout (single fast agent)**
- Opens the site, checks for sitemap.xml, robots.txt
- Gets a high-level overview: what sections exist, what the site offers
- Returns a structured list of areas to research
- Writes findings to `research/<name>/scout.md`

**Step 2: Deep research (parallel agents) — only if needed**
- One agent per area identified by the scout
- Each agent writes findings to `research/<name>/<area>.md`
- Each agent reads `.agents/skills/agent-browser/SKILL.md` first and starts its own browser session with a unique name (`--session <name>-<area>`)

**When to skip Step 2:** The source has a clean public REST/JSON API, the scout covered all areas in one pass, and the scout's findings are sufficient to draft the README. This is the common case.

**When to use Step 2:** The source has 4+ distinct feature areas, requires auth investigation, has complex GraphQL schemas, or the scout identified gaps it could not resolve.

**Done when:** `research/<name>/scout.md` exists and the orchestrator has read and confirmed it covers all necessary areas.

---

### Phase 2: Documentation (sub-agent)

Write the "contract" that the CLI must match:

- `skills/<name>/cli/README.md` — full CLI docs with every command, every flag, and **exact JSON output shapes** (test these by running the API manually if possible)
- `skills/<name>/SKILL.md` — the skill wrapper (see Skill Conventions below)
- `skills/<name>/cli/package.json` — dependencies
- `skills/<name>/cli/tsconfig.json` — TypeScript config

The README is the source of truth. Every JSON shape, flag name, and error format documented here must be exactly what the CLI produces.

**Done when:** `bun install` succeeds in the package directory and all documentation files exist.

---

### Phase 3: Tests (sub-agent) — MUST wait for Phase 2

Tests are written based on Phase 2's README — not the research directly. This prevents doc/implementation drift.

- Place tests in `tests/commands/<command>.test.ts`
- Create a shared `tests/helpers.ts` with `runCLI(args)` and `parseJSON<T>(result)` helpers
- Tests run against the live API (no mocking) with small `--per-page` values
- Assert on the exact JSON shapes documented in the README
- Cover: basic output shape, filters, pagination, sort, format flags, error cases, edge cases
- Avoid vacuous tests — ensure assertions actually exercise real data fields

**Done when:** Test files compile and `bun test` exits (tests will fail since CLI isn't built yet, but they should parse correctly).

---

### Phase 4: Build CLI (parallelized per command)

Building is split into two steps to maximize parallelization:

**Step 1: Scaffold (single agent)**
- Create `src/helpers.ts` with `BASE_URL`, `apiFetch`, shared formatters
- Create `src/cli.ts` skeleton that imports and registers all commands (commands can be stubs initially)
- Run `bun install`
- This step is fast and unblocks all parallel command agents

**Step 2: Build commands (parallel agents — one per command)**
- Each agent gets ONE command to build and ONE test file to satisfy
- Each agent reads: the README (for its command's contract), its test file, `helpers.ts`, and the bunli skill
- Each agent writes its command file in `src/commands/<name>.ts`
- Each agent runs only its test: `bun test tests/commands/<name>.test.ts`
- Each agent iterates until its tests pass
- All agents run simultaneously since commands are independent files

**Step 3: Integration check (orchestrator)**
- After all command agents complete, run `bun test` (full suite) to catch any conflicts
- Run `bun run typecheck` (tsc --noEmit)
- Fix any integration issues

**Done when:** Full `bun test` all green + `bun run typecheck` clean.

---

### Phase 5: Validation (sub-agent)

Verify that everything is internally consistent:

- Run each command and compare actual JSON output against README docs
- Check that SKILL.md command signatures match the real CLI
- Check that flag names, types, and defaults are consistent across commands
- Fix any drift found
- Register the skill in `skills-lock.json`

**Done when:** README, SKILL.md, tests, and implementation all agree. Skill is registered.

---

### Phase 6: Skill Testing (orchestrator + skill-creator)

Test the skill as Claude would use it:

- Write 2-3 realistic test prompts (the kind a real user would say)
- Run them through the skill-creator eval loop if available
- Review outputs qualitatively

**Done when:** User approves the skill outputs.

---

### Phase 7: Description Optimization (optional)

Use the skill-creator's `run_loop.py` to optimize the SKILL.md description for trigger accuracy:

- Generate 20 eval queries (10 should-trigger, 10 should-not-trigger)
- Run the optimization loop
- Apply the best description

**Done when:** Trigger score meets threshold or user is satisfied.

---

## Orchestrator Workflow

When starting a new skill, the orchestrator:

1. Creates a task per phase using `TaskCreate`
2. Sets up dependencies: Phase 3 blocked by Phase 2, Phase 4 blocked by Phase 3, Phase 5 blocked by Phase 4, etc.
3. Starts Phase 1 (research) and waits for it
4. On each phase completion, marks the task as `completed` and starts the next
5. Parallelizes where safe: Phase 1 deep-research agents run in parallel; Phase 6/7 can overlap

The task list is the single source of truth for progress. Every phase transition goes through `TaskUpdate`.

---

## Project Structure

```
skills/                                # Project root
├── CLAUDE.md                          # This file
├── skills-lock.json                   # Installed skill registry
├── .gitignore                         # Ignores research/
├── .agents/skills/                    # Shared agent skills (bunli, agent-browser)
├── .claude/skills/                    # Symlinks to .agents/skills/
├── research/                          # Research findings (git-ignored)
│   └── <name>/                        # One folder per data source
│       ├── scout.md                   # High-level overview from scout agent
│       └── <area>.md                  # Deep research per area (if needed)
└── skills/                            # Each skill is self-contained
    └── <name>/
        ├── SKILL.md                   # Skill definition + trigger description
        └── cli/                       # Bundled CLI (travels with the skill)
            ├── src/
            │   ├── cli.ts             # Entry point (createCLI + commands)
            │   ├── helpers.ts         # Shared: BASE_URL, formatPrice, apiFetch, etc.
            │   └── commands/          # One file per command
            ├── tests/
            │   ├── helpers.ts         # runCLI + parseJSON test utilities
            │   └── commands/          # One test file per command
            ├── package.json
            └── tsconfig.json
```

---

## CLI Conventions

### Framework
- Use `@bunli/core` for command definition (`defineCommand`, `option`, `createCLI`)
- Use `@bunli/utils` for colors/validation
- Always use `z.coerce.number()` and `z.coerce.boolean()` for numeric/boolean flags (CLI args are strings)
- Use `signal.aborted` checks in handlers for long-running operations

### Output
- Default output format: JSON (`--format json|table|plain`)
- Include a `--limit` flag for list commands (client-side cap on results)
- Include `--page` and `--per-page` for paginated list commands
- All errors go to stderr as `{ "error": "...", "code": "..." }` with exit code 1

### Shared Code
- Every CLI must have a `src/helpers.ts` with at minimum:
  - `BASE_URL` constant
  - `apiFetch<T>(path, params)` — builds URL, calls fetch, handles non-2xx errors
  - Any formatting helpers used by multiple commands (`formatPrice`, `getAddressStr`, etc.)
- Never duplicate constants or utility functions across command files

### API Resilience
- **Retry with backoff:** `apiFetch` should include retry logic with exponential backoff + jitter for 429/5xx responses. Tests hit the real API concurrently, which can trigger rate limits. Pattern: 6 retries, 500ms base delay, max 5s, random jitter.
- **Client-side filtering:** If the upstream API does not reliably enforce filter params (common with price/area/rooms filters), apply client-side filtering after the fetch. Comment: `// Client-side filter — API does not enforce reliably`
- **Sort value mapping:** When API sort field names differ from user-facing flag values, define a `sortMap` record and apply it. Document the mapping in a comment.
- **Response normalization:** Remap API field names to match the documented JSON output shape. Inject null defaults for fields that may be absent. Never emit raw API field names if they differ from the documented output.

### Dependencies
- After scaffolding `package.json`, always run `bun install` before running or testing the CLI
- Add a `typecheck` script to package.json: `"typecheck": "tsc --noEmit"`

---

## Test Conventions

- Tests live in `tests/commands/<command>.test.ts`
- Shared helper at `tests/helpers.ts` exports:
  - `runCLI(args: string[]): Promise<{ stdout, stderr, exitCode }>` — spawns `bun run src/cli.ts`
  - `parseJSON<T>(result): T` — asserts exit code 0 and parses stdout
- Tests hit the real API — no mocking
- Use small `--per-page` values (1-5) to keep tests fast
- Avoid vacuous tests: ensure assertions check fields that actually exist on the response
- Cover: basic shape, all filter types, pagination, sort, all format flags, error cases

---

## Skill Conventions

### Structure
Each skill is a self-contained folder with its CLI bundled inside:

```
skills/<name>/
├── SKILL.md              # Skill definition (what Claude sees)
├── cli/                  # Bundled CLI (travels with the skill)
│   ├── src/
│   ├── tests/
│   ├── package.json
│   └── tsconfig.json
└── references/           # Optional extended docs
```

- The CLI lives inside the skill so it can be packaged and distributed as one unit
- Set `allowed-tools: Bash(bun run skills/<name>/cli/src/cli.ts *)` to scope tool access
- Use `context: fork` for skills that do heavy data fetching
- Keep SKILL.md under 500 lines; put extended docs in `references/`

### Description (the most important part)
The description is how Claude decides whether to use the skill. Follow skill-creator best practices:

- **Be pushy:** Actively say "Make sure to use this skill whenever..." and include edge cases ("even if they just mention housing prices in a Danish city without naming boligsiden")
- **Explain the why:** Don't just list commands — explain why certain patterns work and when to combine commands
- **Include workflows:** Show natural sequences (e.g., "search → detail → timeline")
- **Cover both languages:** Include trigger phrases in both Danish and English

### Registration
After creating a skill, add it to `skills-lock.json` with `name`, `source: local`, and `path`.

---

## Agent Browser Usage

When exploring a new data source:

1. Read `.agents/skills/agent-browser/SKILL.md` first
2. `agent-browser open <url> --session <unique-name>` — each agent gets its own session
3. `agent-browser snapshot -i` — get interactive element refs
4. Inspect network requests, page structure, and data formats
5. Identify the best extraction method (scraping, API, RSS, etc.)
6. Write findings to `research/<name>/` before building the CLI

---

## Sub-Agent Configuration

- All sub-agents run on **Sonnet** (`model: "sonnet"`) — Opus is reserved for orchestration
- Use `mode: "bypassPermissions"` for autonomous execution
- Run research agents in the background when possible
- Every agent prompt must include: what files to read first, what to produce, and the done criteria

---

## Tech Stack

- **Runtime**: Bun
- **CLI framework**: bunli (`@bunli/core`, `@bunli/utils`, `@bunli/tui`)
- **Validation**: Zod
- **Language**: TypeScript

---
> Source: [mikkelkrogsholm/skills](https://github.com/mikkelkrogsholm/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
