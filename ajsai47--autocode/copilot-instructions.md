## autocode

> Your repo's engineering department. Features, bugs, coverage, refactoring — from a unified work queue. Pure skill files — no build step, no dependencies.

# AutoCode

Your repo's engineering department. Features, bugs, coverage, refactoring — from a unified work queue. Pure skill files — no build step, no dependencies.

## Project Structure

- `commands/` — Claude Code slash commands (user-invokable skills)
- `agents/` — Agent role definitions (spawned by the orchestrator, not user-invokable)
- `schemas/` — JSON schema for `autocode.manifest.json`
- `examples/` — Example manifests for different project types
- `docs/` — Documentation

## Key Concepts

- **Manifest**: `autocode.manifest.json` is generated per-repo by `/autocode-bootstrap`. It's the immutable contract agents run against.
- **Work Queue**: Multiple sources feed into a unified, prioritized work queue: focus overrides, multi-PR plans, GitHub Issues, coverage gaps, backlog tasks, PR review feedback, proactive discovery, and tech debt signals.
- **Work Types**: Each work item has a type (`coverage`, `feature`, `bugfix`, `refactor`, `docs`, `dependency`, `review_response`) that determines pipeline routing — which agents are spawned and skipped.
- **Agents are constrained**: Scout reads only, Architect writes specs only, Planner decomposes tasks (read-only), Builder writes source only (+ `ci_fix` for CI failures), Tester writes tests only, Reviewer writes nothing, Discoverer finds work (read-only). All agents receive patterns from the pattern database.
- **Multi-PR Planning**: The Planner agent decomposes large tasks into dependency graphs of atomic PRs. Plans are stored in `.autocode/plans/` and steps are ingested as high-priority work items.
- **Daemon Mode**: Run AutoCode on a schedule via GitHub Actions with budget controls. `/autocode-daemon setup` generates the workflow.
- **Proactive Discovery**: The Discoverer agent finds untested commits, complexity hotspots, vulnerable dependencies, and stale TODOs. Runs once per session when enabled.
- **Persistent Brain**: Knowledge graph (`knowledge.json`) caches codebase analysis. Pattern database (`patterns.json`) replaces unstructured lessons with weighted, scored patterns. Human feedback loop extracts patterns from PR reviews.
- **CI-Aware Shipping**: On CI failure after merge, reads logs → categorizes failure → attempts fix before reverting. Tracks CI patterns in `ci_patterns.json`.
- **Memory**: Per-repo memory in `.autocode/memory/` prevents retry loops and accumulates structured knowledge.
- **Progressive difficulty**: Levels 1-6, auto-advances after 3 consecutive successes. Higher levels unlock more work types (L1-2: coverage only, L3+: bugs, L5+: features).

## Commands

| Command | Description |
|---------|-------------|
| `/autocode-bootstrap` | Analyze repo and generate manifest |
| `/autocode` | Run the autonomous code factory |
| `/autocode-status` | View current factory status and metrics |
| `/autocode-stop` | Gracefully stop the factory |
| `/autocode-report` | Generate a shareable summary |
| `/autocode-focus` | Manage the priority work queue |
| `/autocode-next` | Preview the next cycle (dry run) |
| `/autocode-plan` | Decompose large tasks into multi-PR plans |
| `/autocode-daemon` | Manage daemon mode (setup, status, pause, resume) |
| `/autocode-discover` | Run proactive codebase discovery |

## Development

When editing skill files:
- Commands in `commands/` are user-invokable via `/command-name`
- Agents in `agents/` are spawned by the orchestrator — they're NOT slash commands
- All files are markdown — no code to build or test
- Test changes by running `./install.sh` and using the commands in Claude Code

---
> Source: [ajsai47/autocode](https://github.com/ajsai47/autocode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
