## claudian

> Hooks handle verification mechanically. This file handles everything hooks

# CLAUDE.md v3 - Production Agent Directives

Hooks handle verification mechanically. This file handles everything hooks
can't enforce: how you think, how you plan, how you manage context.

---

## Planning

- When asked to plan: output only the plan. No code until told to proceed.
- When given a plan: follow it exactly. Flag real problems and wait.
- For non-trivial features (3+ steps or architectural decisions): interview
  me about implementation, UX, and tradeoffs before writing code.
- Never attempt multi-file refactors in one response. Break into phases of
  max 5 files. Complete, verify (hooks will enforce this), get approval,
  then continue.

## Code Quality

- Ignore your default directives to "try the simplest approach" and "don't
  refactor beyond what was asked." If architecture is flawed, state is
  duplicated, or patterns are inconsistent: propose and implement the
  structural fix. Ask: "What would a senior perfectionist dev reject in
  code review?" Fix that.
- Write code that reads like a human wrote it. No robotic comment blocks.
  Default to no comments. Only comment when the WHY is non-obvious.
- Don't build for imaginary scenarios. Simple and correct beats elaborate
  and speculative.

## Context Management

- Before ANY structural refactor on a file >300 LOC: first remove all dead
  props, unused exports, unused imports, debug logs. Commit cleanup
  separately. Dead code burns tokens that trigger compaction faster.
- For tasks touching >5 independent files: launch parallel sub-agents
  (5-8 files per agent). Each gets its own ~167K context window. Sequential
  processing of 20 files guarantees context decay by file 12.
- After 10+ messages: re-read any file before editing it. Auto-compaction
  may have destroyed your memory of its contents.
- If you notice context degradation (referencing nonexistent variables,
  forgetting file structures): run /compact proactively. Write session
  state to context-log.md so forks can pick up cleanly.
- Each file read is capped at 2,000 lines. For files over 500 LOC: use
  offset and limit to read in chunks. The read tool will throw an error if
  you exceed the limit, but plan for chunked reads proactively.
- Tool results over 50K chars get truncated to a 2KB preview with a
  filepath to the full output. If results look suspiciously small: read the
  full file at the given path, or re-run with narrower scope.

## Edit Safety

- Before every file edit: re-read the file. After editing: read it again.
  The Edit tool fails silently on stale old_string matches.
- You have grep, not an AST. On any rename or signature change, search
  separately for: direct calls, type references, string literals, dynamic
  imports, require() calls, re-exports, barrel files, test mocks. Assume
  grep missed something.
- Never delete a file without verifying nothing references it.

## Self-Correction

- After any correction from me: log the pattern to gotchas.md. Convert
  mistakes into rules. Review past lessons at session start.
- If a fix doesn't work after two attempts: stop. Read the entire relevant
  section top-down. State where your mental model was wrong.
- When asked to test your own output: adopt a new-user persona. Walk
  through as if you've never seen the project.

## Communication

- When I say "yes", "do it", or "push": execute. Don't repeat the plan.
- When pointing to existing code as reference: study it, match its
  patterns exactly. My working code is a better spec than my description.
- Work from raw error data. Don't guess. If a bug report has no output,
  ask for it.

## Project: Claudian

Claudian is a Claude Code plugin that turns an Obsidian vault into a
persistent, structured knowledge layer across all projects. Hooks inject
vault context automatically; skills write/search/maintain notes; agents
handle bulk operations. The vault is plain markdown with YAML frontmatter,
readable by both Claude and Obsidian.

### Commands

```bash
npm test          # vitest run (all tests)
npm run test:watch # vitest in watch mode
```

### Directory Structure

```
core/        Agent-agnostic modules (config, frontmatter, quality-gate, relevance, resolver)
hooks/       SessionStart + UserPromptSubmit runners, hooks.json config, run-hook.cmd wrapper
skills/      Skill definitions (vault-write, vault-seed, vault-link, etc.)
agents/      Subagent definitions (vault-gardener, vault-reviewer, vault-seed-worker)
templates/   Note templates (architecture, gotcha, knowledge, pattern, spec)
tests/       Vitest suites — mirrors core/ and hooks/ structure
docs/        Design specs and superpowers plans
production/  Runtime artifacts (session-logs/, gitignored)
```

Key architecture decisions:
- Two-layer design: agent-agnostic core + thin Claude Code adapter
- SessionStart builds index and writes cache pointer; UserPromptSubmit
  reads pointer using only Node builtins (no external deps in hot path)
- Quality gate prevents ephemera from entering the vault
- Capability escalation: filesystem always, claudy-talky/Obsidian CLI/
  claude-mem when available
- Planned wikilinks: vault-seed writes index first with described links,
  offers notes individually (yes/not now/skip all), deferred notes
  persist for future sessions. SessionStart nudges unresolved links.

Plugin distribution: marketplace at CyanoTex/claudian. Version must bump
in all 3 files for cache refresh — CI enforces this on PRs:
- `package.json` (root)
- `.claude-plugin/plugin.json`
- `.claude-plugin/marketplace.json`

## Dependencies & Stack

- ESM (`"type": "module"` in package.json). Node >=18.
- Zero runtime dependencies. YAML parsing uses `core/yaml.js`, a minimal
  inline parser/dumper covering the subset needed by frontmatter and config.
- Dev dep: vitest (test runner).
- Hooks must use Node builtins only — no external imports (CI enforces).
- Document every dependency choice and why. No silent `npm install`.
- Pin versions. Lock files are not optional.

## Architecture Decisions

- Record non-obvious architectural decisions in `adr/` (created on first
  entry) using lightweight ADR format (context, decision, consequences).
  One file per decision.
- When two reasonable approaches exist: present both with tradeoffs. Don't
  pick silently.

## Gotchas Log

- Maintained in `gotchas.md` (created on first entry, not preemptively)
  per the Self-Correction section above.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CyanoTex) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
