## just-silo

> You must run td usage --new-session at conversation start (or after /clear) to see current work.

# just-silo — AGENTS.md

## MANDATORY: Use td for Task Management

You must run td usage --new-session at conversation start (or after /clear) to see current work.
Use td usage -q for subsequent reads.

---

## Git Workflow (Solo Contributor)

**Direct push to `main` is allowed.** Branch protection is off.

**Branch when it earns it, push directly when it doesn't.** The decision is
size and reversibility, not policy:

- **Direct push to main:** small, self-contained changes (a single playbook,
  a doc edit, a fix). One logical change per commit. Run `just canon-check`
  first when touching `canon/`.
- **Feature branch + PR:** larger work, multi-step changes, anything that
  benefits from CI running on a branch before merge. The PR is a review
  trail and a CI gate, not a permission gate.

**Why no branch protection** (recorded so this doesn't regenerate as a
barnacle): branch protection's main value is the peer-review gate, and this
is a solo repo — there are no peers. CI gates (`quality`, `lint-markdown`)
still run on branches and PRs regardless. The friction-to-utility ratio
failed the Watt test for a repo whose work is mostly meta-text (playbooks,
briefs, decisions, AGENTS.md edits). Removed 2026-07-05.

**Server-side AI reviewers** (CodeRabbit, Qodo) run **advisory**, not gating.
They are strong on code (smells, missing tests, type errors, security) and
near-zero on meta-text (they can't judge whether a playbook embodies its
discipline). Read their findings on code PRs; discount them on prose PRs.
Do not block on them.

See: `playbooks/git-workflow-playbook.md`

---

## MANDATORY: Use tree-sitter for Code Audit

For code analysis and refactoring tasks, the agent shall perform a structural audit via Tree-sitter before executing any text-based (grep/regex) searches. This protocol requires: 1) Identification of the relevant grammar; 2) Execution of structural queries to map function definitions, dependencies, and type-hierarchies; 3) Conversion of this map into DOT notation for planning. Text-based search is permitted only for non-code assets (comments, documentation)

---

## MANDATORY: Canon Consistency on Startup

The `canon/` bundle at repo root holds curated, portable method playbooks (OKF-conformant). It is operational only when self-consistent — when `INDEX.jsonl` (source of truth) matches the files on disk, and every entry carries OKF frontmatter.

**Before reading, editing, or pulling from canon — run:**

```bash
just orient        # ends with a canon consistency check
```

`just orient` runs `just canon-check` as its last step. If the canon check fails, **do not proceed** with other work — fix the drift first (`canon/INDEX.jsonl` is the source of truth; regenerate with `just canon-index`). Do not build on an inconsistent foundation.

**Adding to canon:** one entry per commit, branch off main, update `INDEX.jsonl` then `just canon-index` then append to `canon/log.md`, `just canon-check` must pass before commit. See `canon/index.md` for the full protocol.

---

## Workflow

### Local Development

Direct. No agents, no multiplexers.

- **Editor:** Zed
- **Terminal:** Built-in or standalone
- **Run:** `bun run`, `just`, `bash`
- **Test:** `bun test`

### Remote Development

Monitoring only.

- Watch: `git log`, `td status`, `bun test`
- Act: Only when necessary

---

## The Silo Framework 

The **silo** framework for externalizing domain knowledge is still useful:

```
my-silo/
├── .silo              # Manifest
├── README.md          # Domain rules
├── schema.json        # Validation
├── queries.json       # Named jq filters
├── justfile           # Recipes
├── harvest.jsonl      # Raw data
└── process.sh        # Domain logic
```

**When to use silos:**
- Persisting domain knowledge across sessions
- Structured data pipelines
- Named query reuse

**When not to use silos:**
- One-off scripts → just write the script
- Simple tasks → direct terminal
- Real-time interaction → direct terminal

---

## Prerequisites

- `bun` — runtime
- `just` — task runner (`brew install just`)
- `jq` — JSON processing (`brew install jq`)
- `zed` — editor (optional)

---

## Quick Reference

```bash
# Run a script
bun run server.ts

# Test
bun test

# Watch mode
bun --watch run server.ts

# Task runner
just --list

# Watch tests
bun test --watch
```

---

## Experimental Tiers

Scripts evolve through tiers as they mature:

```
@scripts/lab/     → scripts/  → src/
  ↑                 ↑           ↑
  Tier 0            Tier 1      Tier 2
  (experimental)    (stable)   (production)
```

| Tier | Location | Rules |
|------|----------|-------|
| 0 | `@scripts/lab/` | Experimental. No review. Promote when stable. |
| 1 | `scripts/` | Stable. Reviewed. Production-ready. |
| 2 | `src/` | Full rigor. Types, tests, documentation. |

### Tier 0: @scripts/lab/

Experimental scripts with `@` prefix. Agents try ideas here:

```bash
scripts/lab/
├── @entropy-viz.sh    # Trying visualization
├── @jq-playground.sh  # Testing jq patterns
└── @experiment.sh     # Wild idea
```

**Rules:**
- No review required
- Promote to `scripts/` when stable
- Gamma-loop archives stale experiments

### Tier 1: scripts/

Stable, production scripts. Reviewed before promotion:

```bash
scripts/
├── silo-create.sh       # Stable
├── silo-ignite.sh      # Stable
└── silo-log.sh        # Stable
```

### Tier 2: src/

TypeScript/Production code. Full rigor:

```bash
src/
├── silo-api-server.ts  # Types, tests, docs
└── lib/
    └── silo-logger.ts  # Full quality

**TypeScript config tiers:**
```json
tsconfig.json          // Tier 2: strict, no implicit any
tsconfig.scripts.json // Tier 1: relaxed, allowJs
```

**Commands:**
```bash
just dev-typecheck   # Check all tiers
just dev-checkall   # Type + tests
```


---

## Agent Scratchpad

The **scratch/** directory is the agent's private workspace:

```bash
scratch/          # NOT COMMITTED
├── draft-*.md     # Thinking out loud
├── test-*.jsonl   # Playground data
└── exploration/   # Free-form
```

**Rules:**
- Never committed (in `.gitignore`)
- Agent's private workspace
- Gamma-loop archives stale scratch

---

## The Simplicity Rule

> **Extract to scripts. Keep justfiles thin.**

| Situation | Don't Do | Do |
|-----------|----------|----|
| Recipe > 80 chars | Inline jq | `scripts/process.sh` |
| Data transformation | `jq` in justfile | `scripts/transform.sh` |
| Multi-step logic | Long recipe with `\` | Call scripts |

See: `playbooks/justfile-design-playbook.md`

---

## See Also

- `playbooks/git-workflow-playbook.md` — Git workflow with two worktrees
- `playbooks/bestiary-human-tendencies.md` — Known failure modes
- `playbooks/justfile-design-playbook.md` — Recipe design rules
- `playbooks/gamma-loop-playbook.md` — Gamma-loop pattern
- `briefs/` — Project briefs and research
- `presentation/` — SSE presentation server

---

**Status:** Simplified. Local dev is direct. Remote is monitoring.

---
> Source: [pjsvis/just-silo](https://github.com/pjsvis/just-silo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
