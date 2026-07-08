## colossus-rs

> > **Read this file first. Then read `COLOSSUS-CC-STANDARDS.md` immediately after.**

# CLAUDE.md — colossus-rs

> **Read this file first. Then read `COLOSSUS-CC-STANDARDS.md` immediately after.**
> COLOSSUS-CC-STANDARDS.md contains the pre-coding analysis template,
> STOP gate rules, forensic mode, completion report format, and all
> generic coding standards. Both files must be read before any task begins.

---

## What this repo is

**colossus-rs** — Shared Rust library workspace for Colossus applications.
Library crates only. No binary, no HTTP server, no frontend, no containers.
Consumed as a git dependency by colossus-legal and colossus-ai.

### Workspace crates

| Crate | Description |
|-------|-------------|
| colossus-auth | Authentik + Axum authentication integration |
| colossus-extract | Extraction types, traits, LLM/embedding providers, schema loader |
| colossus-rag | RAG pipeline (retriever, expander, synthesizer, decomposer) |
| colossus-pdf | PDF text extraction |
| colossus-graph | Neo4j query functions (domain-agnostic) |
| colossus-pipeline | Async job pipeline framework (domain-agnostic) |

Current phase, active task, and progress are in the session transition
document — not here. Read the transition doc for that context.
Branch for this repo is always `main`.

---

## Colossus-rs specific rules

These rules apply permanently to this repo regardless of phase.
They are IN ADDITION to everything in COLOSSUS-CC-STANDARDS.md.

### Reusability is non-negotiable
Before writing any code for colossus-pipeline or colossus-extract, ask:
"Could colossus-ai use this with zero code changes?"
If the answer is NO — the design is wrong. Stop and flag it.
Domain-specific config goes in YAML/env vars, never in Rust code.

### No application domain knowledge in this repo
These crates must have zero knowledge of:
- Legal documents, cases, or entities
- Any specific LLM provider by name in framework code
- colossus-legal's AppContext, AppState, or any of its types
- Any specific database schema from colossus-legal

### LlmProvider pattern
All LLM calls in the framework go through the LlmProvider trait.
Never call rig::providers::anthropic directly in framework code.
Provider selection happens via env vars, not compile-time types.

### Enum SQL binding — never hardcode status strings
All SQL status and control values bind via enum as_str() methods.
Never write 'ready', 'running', 'failed' etc. as SQL string literals.
JobStatus::as_str() and JobControl::as_str() are the single source of truth.
A typo in a literal is a silent runtime failure.
A typo in an enum variant is a compile error.

### Branch is always main
colossus-rs always works on main. No feature branches.
New crates are added directly to main.

### No rand crate for jitter
Retry jitter uses nanosecond-based deterministic seed.
Do not add the rand crate.

### Worker step contracts
Steps must never call tokio::spawn internally.
Steps are called by the Worker which handles all concurrency.

---

## Commands for this repo

```bash
# Branch verification (must be main)
git branch --show-current
git status

# Build and test a single crate
cargo build -p colossus-pipeline
cargo test -p colossus-pipeline -- --test-threads=1

# Build entire workspace (run after every crate change)
cargo build --workspace
cargo test --workspace -- --test-threads=1

# Lint
cargo clippy -p colossus-pipeline -- -D warnings

# Module size check (code lines, excluding comments)
find colossus-pipeline/src -name "*.rs" -exec sh -c \
  'lines=$(grep -v "^[[:space:]]*//\|^[[:space:]]*\*\|^[[:space:]]*$" "$1" | wc -l); \
   if [ $lines -gt 300 ]; then echo "OVER 300: $lines $1"; fi' _ {} \;
```

---

## Pre-Commit Agent Enforcement (MANDATORY)

Before committing ANY code changes, invoke ALL agents in `.claude/agents/`:

1. `Task(rules-enforcer)` — must PASS
2. `Task(library-api-reviewer)` — must PASS
3. `Task(test-auditor)` — must PASS
4. `Task(domain-boundary-checker)` — must PASS

If ANY agent returns FAIL, fix every violation before committing.
Do not skip agents. Do not override agent findings.
Do not commit with known violations "to fix later."

---

## What NOT to do in this repo

❌ Add colossus-legal specific types or domain knowledge to any crate
❌ Reference colossus-legal, colossus-ansible, or colossus-homelab paths
❌ Hardcode SQL status strings — use JobStatus::as_str() and JobControl::as_str()
❌ Add the rand crate for jitter — use nanosecond-based seed
❌ Call tokio::spawn inside Step implementations
❌ Create feature branches — work on main
❌ Bump version numbers — Roman does that
❌ Make a crate dependent on another colossus-rs crate unless truly necessary

---

*This file contains permanent rules only.*
*Current phase, active task, and progress: read the session transition document.*
*Full coding standards, STOP gate, pre-coding template: read COLOSSUS-CC-STANDARDS.md.*

---
> Source: [rhrywnak/colossus-rs](https://github.com/rhrywnak/colossus-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
