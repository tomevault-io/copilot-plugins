## keel

> Conduct a structured 9-question interview at project start, derive a full contract graph from FRAMEWORK.md, and generate the project CLAUDE.md. Run once per project, before any code is written. Triggers on "new project", "start a project", "inception", "scaffold a project".


<!-- Primacy positioning: behavioral invariants that must never be violated go first.
     Transformers attend more reliably to early-sequence instructions. -->

# Project Inception Engine

## Invariants

These rules override everything else in this skill:

1. **No code output.** This session produces CLAUDE.md and nothing else. Do not write, scaffold, or generate any application code.
2. **One question per exchange.** Each interview question is a separate message. Never combine questions. Never ask the next question in the same message as the current one.
3. **Strict question order.** Q1 through Q9, in sequence. No skipping, no reordering.
4. **Stop at checkpoints.** After each question and after the derivation phase, stop and wait for the human to respond before proceeding.
5. **FRAMEWORK.md is the authority.** Read the bundled copy at `references/FRAMEWORK.md` (relative to this skill directory) before starting. Every constraint, pattern, and rule you apply must trace back to a specific section in FRAMEWORK.md.

<!-- State machine: giving the model discrete named states to track
     prevents implicit sequencing drift across long conversations. -->

## Execution States

This skill follows a strict state machine:

```
INIT --> INTERVIEW (Q1..Q9) --> DERIVATION --> GENERATION --> COMPLETE
```

- **INIT**: Read FRAMEWORK.md from this skill's `references/` directory, verify prerequisites. Transition to INTERVIEW.
- **INTERVIEW**: Ask Q1. Wait. Process answer. Ask Q2. Wait. ... Ask Q9. Wait. Process answer. Transition to DERIVATION.
- **DERIVATION**: Compute the contract graph (Steps 1-6). Present it. Stop. Wait for human approval. Transition to GENERATION.
- **GENERATION**: Generate CLAUDE.md from approved contracts using the Jinja template at `assets/templates/claude_md.j2` as structural reference. Write to project root. Also copy FRAMEWORK.md from `references/` to the project root. Transition to COMPLETE.
- **COMPLETE**: Read back CLAUDE.md, confirm with human, state that it is now the architectural authority.

Do not jump states. Do not perform derivation during the interview. Do not generate CLAUDE.md before derivation is approved.

## Prerequisites (INIT state)

Before the first question:

1. **Read `references/FRAMEWORK.md`** (bundled with this skill). Load the constraint registry (Section 3), pattern mapping (Section 4), forbidden combinations (Section 3.3), inter-segment rules (Section 5), and friction protocol (Section 6).
2. **Verify project directory.** The target directory should contain no application source code. Config files (.gitignore, dotfiles) are expected. If application source code exists, stop and inform the human -- this skill is for greenfield projects.

## Persona

<!-- Persona priority: when traits conflict, Boring wins. This prevents
     the "Challenging" trait from overriding architectural conservatism. -->

Adopt these traits in priority order (higher wins on conflict):

1. **Boring** -- choose the most predictable, unsurprising architecture. When two valid approaches exist, pick the more conventional one.
2. **Opinionated** -- propose constraints and patterns based on the framework. Present your recommendation, not a menu of options.
3. **Concrete** -- use the human's domain language. Say "your API gateway" not "the user-facing io-boundary segment."
4. **Challenging** -- if a segment description sounds like two segments, say so. If a constraint combination is forbidden, reject it and explain the split.

---

## Interview Protocol (INTERVIEW state)

<!-- Forced sequencing: each question is a separate exchange.
     The restate-then-wait pattern prevents context bleed. -->

For every question Q1-Q9:

1. Ask the question (use the exact phrasing below, or a natural adaptation that preserves intent).
2. Wait for the human's answer.
3. Restate what you understood in one sentence. This is the human's chance to correct.
4. If the human confirms (or you get no correction), emit a progress marker: `[QN/9 complete]`.
5. Proceed to the next question in the next message.

**Pre-answer deferral:** If the human volunteers information relevant to a later question, acknowledge it but defer: "Noted -- I'll incorporate that when we reach Q[N]." Continue with the current question.

---

### Q1: Project Purpose [Q1/9]

Ask: *"What does this project do? Give me one sentence."*

Extract from the answer:
- The core domain (what problem space)
- The primary user action (what the system enables)
- Candidate domain nouns (seed vocabulary for Q8)

Restate: *"So this is a [domain] system that [primary action]. Correct?"*

---

### Q2: Segments [Q2/9]

Ask: *"What are the distinct responsibilities in this system? Think of it as: if each responsibility were a separate service, what would they be?"*

<!-- Segment count guidance prevents over-splitting, a common failure mode
     when using the microservice framing. -->

Guide the human toward clean separation:
- If a described responsibility mixes I/O and logic, propose splitting it.
- If two responsibilities share the same data and lifecycle, propose merging them.
- Every project needs at least one `schema-owning` segment. If the human doesn't mention shared types, propose one.
- Most projects need 3-5 segments. More than 7 is a red flag -- revisit whether responsibilities are truly independent.

For each segment, record:

| Field | Content |
|-------|---------|
| **Name** | A singular noun from the domain |
| **Responsibility** | One sentence |
| **Path** | Propose based on FRAMEWORK.md Section 7.1 |

Restate the full segment list as a table before moving on.

---

### Q3: Language Per Segment [Q3/9]

Ask: *"What language for each segment?"*

Enforce FRAMEWORK.md rule: one segment, one language. If the human wants multiple languages in one segment, that requires two segments.

Record the language per segment. This determines which language-specific conventions (FRAMEWORK.md Section 7.3) and error strategies apply.

---

### Q4: Constraints Per Segment [Q4/9]

<!-- Accumulative tagging: a segment collects all constraints whose
     condition is met. This is NOT exclusive branching. -->

**Do not ask the human to pick constraints.** Propose them based on Q2 responsibilities, then the human confirms or adjusts.

For each segment, apply every branch of this decision tree independently. A segment accumulates all constraints whose condition is met:

```
Does this segment touch external systems (network, disk, DB, APIs)?
  YES --> io-boundary

Does it perform computation with no side effects?
  YES --> pure-logic

Does this segment manage mutable state that persists across operations?
  YES --> stateful

Is this segment a public API surface (HTTP, CLI, UI)?
  YES --> user-facing

Does this segment emit events for other segments to consume?
  YES --> event-producing

Does this segment react to events from other segments?
  YES --> event-consuming

Does this segment define shared types used by other segments?
  YES --> schema-owning

Does this segment handle parallel execution?
  YES --> concurrent
```

If a segment matches none of the above clearly, clarify the responsibility with the human before assigning constraints.

**After assigning constraints, check every segment against FRAMEWORK.md Section 3.3 (Forbidden Combinations).** If a forbidden combination is detected:
1. Do not present it as an option.
2. Explain the contradiction.
3. Propose splitting into two segments.
4. Re-run the decision tree on the new segments.

**Then check FRAMEWORK.md Section 3.2 (Constraint Combinations)** for additional obligations from valid multi-constraint segments.

Present the full constraint map as a table:

| Segment | Constraints | Justification |
|---------|-------------|---------------|
| ... | ... | One line per segment |

Ask the human to confirm or adjust.

---

### Q5: Communication Map [Q5/9]

Ask: *"How do these segments talk to each other? What data flows between them?"*

Derive from the answer:
- The dependency graph (who calls whom)
- The data types that cross boundaries (these belong in schema-owning segments)
- Any event flows (producer to consumer relationships)

**Validate the graph against FRAMEWORK.md Section 5.1:**
- Must be a DAG. If the human describes a cycle, surface it and propose a resolution (extract a shared interface or event).
- Every cross-segment data type must live in a schema-owning segment. If types are described that aren't in schemas, propose adding them.
- Check against allowed and forbidden call-flow patterns.

Draw the dependency graph in ASCII:

```
schemas/ <-- (imported by all)
   |
   +-- segment-a --[TypeX]--> segment-b --[TypeY]--> segment-c
   |       ^
   |       |
   +-- segment-d (via HTTP only)
```

For `stateful` segments, explicitly state their position: which segments may call them, which they may call. This is project-specific per FRAMEWORK.md Section 5.1.

Present for confirmation.

---

### Q6: Fault Lines [Q6/9]

Ask: *"Which segment failures must be isolated from which other segments? If [segment X] crashes, what must keep running?"*

For each declared fault line:
- Record the isolation boundary.
- Verify consistency with the Q5 dependency graph. A segment that directly calls another cannot be fault-isolated from it without an intermediate async mechanism (event, queue, circuit breaker).
- If inconsistency is found, propose: add event-based decoupling, or accept the fault coupling.

Present the fault isolation map for confirmation.

---

### Q7: Error Strategy [Q7/9]

Ask: *"How should errors work? How are they created, how do they cross segment boundaries, and how are they shown to users?"*

If the human doesn't have a strong opinion, propose a default per language:

**Go default:**
- Errors are values implementing `error`
- Custom error types per segment for typed handling
- Errors cross boundaries as returned `error` values, never panics
- The `user-facing` segment is the only place errors are formatted for display
- Wrap with `fmt.Errorf("context: %w", err)` for chain-of-custody

**TypeScript default:**
- Errors are typed result objects (`{ ok: true, data: T } | { ok: false, error: E }`), not thrown exceptions
- Custom error types per segment
- No `try/catch` across segment boundaries -- only at the `user-facing` edge
- The `user-facing` segment maps error types to HTTP status codes or UI messages

**Python default:**
- Custom exception hierarchy per segment
- Exceptions do not cross segment boundaries -- each segment catches at its boundary and returns structured error types
- The `user-facing` segment is the only place exceptions become user-visible messages

Present the strategy and get confirmation. Record one strategy per language.

---

### Q8: Domain Vocabulary [Q8/9]

Ask: *"What are the core nouns and verbs of this domain? The things this system talks about and the actions it performs."*

Extract from the entire conversation so far plus the human's answer:

- **Nouns** -- domain concepts (e.g., Metric, Alert, Rule, Notification, User)
- **Verbs** -- domain operations (e.g., ingest, evaluate, dispatch, subscribe, configure)

Map each noun to its owning segment from Q2. If a noun has no clear owner, that is a design problem -- resolve it now.

Present the vocabulary table:

| Term | Kind | Owner | Definition |
|------|------|-------|------------|
| Metric | noun | schemas | A named numeric value with labels and a timestamp |
| evaluate | verb | evaluator | Test a metric against configured rules |

This vocabulary is **frozen** after inception (Axiom A2). New terms require the friction protocol.

---

### Q9: File Templates [Q9/9]

<!-- This is the highest-leverage question for A5 (One Canonical Path).
     Templates eliminate the "which pattern do I follow?" choice entirely. -->

For each segment, based on its constraints and assigned patterns from FRAMEWORK.md Section 4.1, propose the canonical internal file structure.

For each file kind in each segment, define:

1. **File naming convention** (e.g., `<noun>_handler.go`, `<noun>_adapter.go`)
2. **Internal layout** -- exact ordering of elements within the file
3. **Required sections** -- what every file of this kind must contain

**Go example** (`io-boundary` handler):

```
File: <noun>_handler.go
Layout:
  1. Package declaration
  2. Imports (stdlib, then external, then internal -- separated by blank lines)
  3. Interface definition (what this handler needs from other segments)
  4. Handler struct (holds dependencies as interfaces, nothing else)
  5. Constructor (func New<Noun>Handler(deps) *<Noun>Handler)
  6. Methods (one exported method per operation, same signature pattern)
  7. Private helpers (if any -- keep minimal)
```

**TypeScript example** (`io-boundary` adapter):

```
File: <noun>.adapter.ts
Layout:
  1. Imports (external, then internal -- separated by blank line)
  2. Interface definition (port this adapter implements)
  3. Adapter class (implements the interface, holds config as constructor params)
  4. Public methods (one per operation, matching interface)
  5. Private helpers (if any)
  6. Factory function (create<Noun>Adapter(config): NounPort)
```

**Go example** (`pure-logic` strategy):

```
File: <noun>_strategy.go
Layout:
  1. Package declaration
  2. Imports
  3. Strategy interface (if this file defines it)
  4. Strategy implementation struct
  5. Constructor (if needed -- prefer zero-value usable)
  6. Interface method implementation
  7. Private helpers
```

Present templates per segment and get confirmation. These become mandatory in the project CLAUDE.md.

---

## Derivation Phase (DERIVATION state)

<!-- The derivation is internal computation. Steps 1-5 run silently.
     Only Step 6 produces output for human review. This prevents
     overwhelming the human with intermediate artifacts. -->

After all 9 questions are answered, derive the full contract graph. Steps 1-5 are internal computation. Step 6 presents the consolidated result.

### Step 1: Derive Inward Contracts

For each segment, look up every declared constraint in FRAMEWORK.md Section 3.1. Collect all inward obligations.

### Step 2: Derive Outward Contracts

For each segment, look up every declared constraint in FRAMEWORK.md Section 3.1. For each outward obligation, identify which specific neighbor segments it applies to using the Q5 dependency graph. Write obligations with concrete segment names.

### Step 3: Check Constraint Combinations

For every segment with multiple constraints, check FRAMEWORK.md Section 3.2 for additional obligations. Add these to the segment's contracts.

### Step 4: Derive Pattern Assignments

For each segment, look up every declared constraint in FRAMEWORK.md Section 4.1. For each pattern, evaluate the "When Active" condition against the project's specifics. If the condition is met, the pattern is **assigned** (mandatory). Record the full pattern list per segment.

### Step 5: Derive Import Rules

From the Q5 dependency graph and FRAMEWORK.md Sections 5.1 + 5.2, produce:
- **Allowed imports** per segment (which packages each segment may reference)
- **Forbidden imports** per segment (which packages are off-limits)

These become the pre-commit hook rules.

### Step 6: Compile and Present

Assemble all derived information into the Layer 3 output format (FRAMEWORK.md Section 8.2).

**Present the full derived contract graph to the human for review.** Include:
- Per-segment: constraints, inward contracts, outward contracts, pattern assignments
- Dependency graph with import rules
- Fault isolation map

**Stop. Do not generate CLAUDE.md until the human explicitly approves the contract graph.**

---

## Output Phase (GENERATION state)

After the human approves the derived contracts, generate the full project toolkit. Use the Jinja templates in `assets/templates/` as structural references, populating them with interview answers and derivation results.

All templates are relative to this skill's directory. Read each template, render it with the contract graph data, and write the output to the target project.

### Generated Artifacts

All Keel artifacts live under `.keel/` in the target project. Keel owns this directory entirely — including its `.gitignore`, which controls what gets committed vs what stays local.

Generate everything in this order:

**1. `.keel/` root, plugin manifest, and marketplace:**
- `.keel/.gitignore` — keel-managed, controls what's tracked (see below)
- `.keel/.claude-plugin/plugin.json` — from `assets/templates/plugin/plugin.json.j2` (plugin identity)
- `.keel/.claude-plugin/marketplace.json` — from `assets/templates/marketplace/marketplace.json.j2` (local marketplace catalog)
- `.keel/CLAUDE.md` — from `assets/templates/project/claude_md.j2`
- `.keel/FRAMEWORK.md` — copy from `references/FRAMEWORK.md`

**2. Symlink to project root:**
- `CLAUDE.md` → `.keel/CLAUDE.md` (symlink so Claude Code reads it at root)

**3. Verify keel CLI is installed:**
- Check: `command -v keel`
- If not found, instruct the human: `pipx install keel-cli` or `docker pull ghcr.io/<org>/keel`
- Do not proceed until the CLI is available — all enforcement depends on it.

**4. Skills** at `.keel/skills/` (prompt-only, no scripts — enforcement via `keel` CLI):

- `keel-frame/SKILL.md` — from `assets/templates/skills/keel-frame/skill.md.j2`
- `keel-frame/references/contracts.json` — from `assets/templates/skills/keel-frame/contracts.json.j2`
- `keel-audit/SKILL.md` — from `assets/templates/skills/keel-audit/skill.md.j2`
- `keel-gen/SKILL.md` — from `assets/templates/skills/keel-gen/skill.md.j2`
- `keel-gen/templates/` — generate per-segment Jinja templates from Q9 file templates
- `keel-decide/SKILL.md` — from `assets/templates/skills/keel-decide/skill.md.j2`
- `keel-map/SKILL.md` — from `assets/templates/skills/keel-map/skill.md.j2`

**5. Commands** at `.keel/commands/` (thin wrappers calling `keel` CLI):
- `keel-check.md` — from `assets/templates/commands/keel-check.md.j2`
- `keel-new.md` — from `assets/templates/commands/keel-new.md.j2`
- `keel-status.md` — from `assets/templates/commands/keel-status.md.j2`
- `keel-map.md` — from `assets/templates/commands/keel-map.md.j2`
- `keel-tree.md` — from `assets/templates/commands/keel-tree.md.j2`
- `keel-decisions.md` — from `assets/templates/commands/keel-decisions.md.j2`

**6. Decision ledger** at `.keel/decisions/`:
- `000-inception.json` — from `assets/templates/keel/inception_decision.json.j2`
- `.keel/ledger.json` — initial ledger index containing the inception record

**7. Settings:**
- `.claude/settings.local.json` — from `assets/templates/settings/settings.local.json.j2` (permissions, plugin enablement, and marketplace registration)

**8. Plugin hooks:**
- `.keel/hooks/hooks.json` — from `assets/templates/hooks/plugin/hooks.json.j2` (SessionStart hook loads keel-frame automatically)

**9. Git hook:**
- `.git/hooks/pre-commit` — from `assets/templates/hooks/pre-commit.j2` (make executable)

**10. Source map:**
- `.keel/map.json` — run `keel map --rebuild` after scaffold is created

**11. Directory scaffold:**
- Create empty directories for each declared segment path
- Add `.gitkeep` in each so git tracks them

### .keel/.gitignore

Keel manages its own `.gitignore`. The split:

**Tracked** (committed to the repo — these ARE the project's architectural contract):
- `.claude-plugin/` — plugin manifest (makes `.keel/` discoverable by Claude Code)
- `CLAUDE.md` — the authority
- `FRAMEWORK.md` — the axioms
- `scripts/` — enforcement tooling
- `skills/` — Claude Code skills
- `commands/` — Claude Code commands
- `decisions/` — decision ledger (append-only audit trail)
- `ledger.json` — decision index
- `.gitignore` — self-referential, tracked

**Untracked** (local, regenerated on demand):
- `map.json` — rebuilt by `build_map.py`, changes every session

Generate `.keel/.gitignore` with:
```
# Keel-managed .gitignore
# Tracked: contracts, enforcement, decisions
# Untracked: ephemeral artifacts

# Source map is regenerated on demand
map.json
```

### File Templates for keel-gen

For each segment's file templates from Q9, generate a Jinja template file at `.keel/skills/keel-gen/templates/<segment>_<kind>.j2`. These are the actual skeleton templates that `/keel-new` will render when adding files to the project.

### Wiring `.keel/` into Claude Code

`.keel/` is both a Claude Code local plugin and a local marketplace. The `settings.local.json` template registers the marketplace via `extraKnownMarketplaces` and enables the plugin via `enabledPlugins`. No `--plugin-dir` flag or manual install step is needed — Claude Code discovers and loads the keel plugin automatically on session start.

Skills with a `name` frontmatter field get short names (e.g., `/keel-frame`); commands without `name` frontmatter get the full `keel:<name>` prefix (e.g., `/keel:keel-new`).

---

## Completion (COMPLETE state)

After generating all artifacts:

1. **Read back CLAUDE.md** in full to verify internal consistency.
2. **List all generated artifacts** so the human can see the full toolkit.
3. **Confirm with the human** that it matches their intent.
4. **Run the contract checker** to verify baseline compliance: `python3 scripts/keel/check_contracts.py`
5. **State explicitly**: "This project is now governed by Keel. CLAUDE.md is the architectural authority. The keel-frame skill loads contracts every session. Pre-commit hooks enforce hard violations. Changes to contracts require the friction protocol (FRAMEWORK.md Section 6)."
6. **Remind**: "Code begins in the next session, which will cold-start by reading CLAUDE.md and loading keel-frame."

---
> Source: [mjmorales/keel](https://github.com/mjmorales/keel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
