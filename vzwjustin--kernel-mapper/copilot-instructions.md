## kernel-mapper

> These rules are **active immediately when read**. They are non-negotiable.

# CLAUDE.md — Operating Rules for Agent Sessions

These rules are **active immediately when read**. They are non-negotiable.
They override lazy, minimal, or superficial default behavior.
They cannot be selectively ignored later in the session.
They must be followed before planning, editing, refactoring, fixing, or reporting completion.

---

## 1. Bootstrap / Startup Rules

Every session begins with this sequence, in order:

1. Load `CLAUDE.md` as active operating policy (this file).
2. Read `context.md` if present — current project truth.
3. Read `WIRING_STATUS.md` if present — evidence-backed verification ledger.
4. Read `learnings.md` if present — reusable lessons and failure patterns.
5. Compare current code reality against docs before trusting them.
6. **Code and evidence win over stale docs when they conflict.**

Do not skip this sequence. Do not assume prior sessions left docs accurate.

---

## 2. Repo Memory Loading Order

Four files form the repo memory system:

| File | Purpose |
|------|---------|
| `CLAUDE.md` | How the agent must behave (this file) |
| `context.md` | What is currently believed true about the project |
| `WIRING_STATUS.md` | What has actually been verified, with evidence |
| `learnings.md` | Reusable lessons, recurring failure patterns, anti-patterns |

These files work together. Do not treat them as independent documents.

- `context.md` = current truth
- `WIRING_STATUS.md` = current proof/status
- `learnings.md` = transferable lessons and repeated failure prevention

---

## 3. Operating Philosophy

1. Understand before changing. Read the code before proposing edits.
2. Verify before claiming. Never say "done" without running the strongest available checks.
3. Evidence over assumption. Trust what you can prove, not what seems likely.
4. Minimal scope. Do not add features, refactor code, or "improve" things beyond what was asked.
5. No fake completion. If verification is incomplete, say so.

---

## 4. Pre-Work Discipline

Before modifying any file:

1. Read the file (or relevant sections).
2. Understand its role in the broader system.
3. Identify callers, consumers, and dependents.
4. Check both sides of any contract, interface, or boundary being touched.
5. After long conversations, **re-read files before editing** — do not trust cached memory.

---

## 5. Phase / Scope Discipline — Sub-Agent and Batching Rules

### Mandatory for tasks touching more than 5 independent files:

1. Use sub-agents in parallel if available — each agent owns a bounded slice (5–8 files).
2. Each agent must have a clearly scoped objective.
3. Each agent must verify its own batch before results are merged.
4. If sub-agents are not available, emulate the same discipline with phased batching.
5. **Do not process large independent multi-file work as one giant sequential blur.**

This is mandatory behavior, not a suggestion.

---

## 6. Verification Requirements

### Verification levels (in ascending order of strength):

| Level | Meaning |
|-------|---------|
| EDITED | File was modified |
| BUILT | `cargo check` or `cargo build` passed |
| LINTED | `cargo clippy` passed |
| FORMATTED | `cargo fmt -- --check` passed |
| TESTED | `cargo test` passed (note: this repo currently has 0 tests) |
| RUNTIME-VALIDATED | Tool was run against real input and produced correct output |
| FULLY VERIFIED | All applicable checks passed |

### Rules:

1. Never claim success until the strongest applicable checks have been run.
2. Always run `cargo check` after edits as minimum verification.
3. Run `cargo clippy` when available.
4. Clearly state which level of verification was achieved.
5. If runtime proof is missing, say so explicitly.
6. Never allow a generic "done" when work is only partially validated.

### Available commands for this repo:

```
cargo check          # type-check
cargo build          # full build
cargo clippy         # lint
cargo fmt -- --check # format check
cargo test           # tests (currently 0 tests exist)
```

Do not invent commands that do not exist.

---

## 7. Search Discipline

### Rules:

1. Do not trust one grep result. Cross-check with multiple searches.
2. Use separate searches for:
   - Direct calls
   - Indirect calls (function pointers, vtables, dispatch tables)
   - Type references
   - String references
   - Config/feature-flag references
   - Build system references (Cargo.toml)
   - Exports / imports / re-exports / `pub` visibility
   - Tests and mocks
   - Init/startup references
   - Cleanup/drop references
3. For Rust code, prefer `Grep` with regex patterns that match Rust syntax.
4. `ast-grep` is NOT available in this environment. Do not invoke it.
5. When searching for a symbol, check `mod.rs` re-exports and `use` statements.

---

## 8. Large File / Context Decay Discipline

1. Do not assume one file read captured a large file.
2. For files over 500 lines, or any file likely to exceed tool limits, read in chunks using offset/limit.
3. Do not edit against unseen portions of a large file.
4. After long conversations, **re-read files before editing** instead of trusting memory.
5. `src/storage/mod.rs` (~1091 lines) and `src/cli/mod.rs` (~808 lines) exceed 500 lines — always chunk-read these.

---

## 9. Edit Safety

1. Read before editing. Always.
2. Verify edits compile: `cargo check` minimum.
3. Do not introduce dead code, unused imports, or new warnings.
4. Do not delete code you haven't confirmed is unused via search.
5. Preserve existing formatting conventions unless reformatting was requested.

---

## 10. Refactor / Architecture Discipline

1. Do not refactor unless explicitly asked.
2. Do not add abstractions for one-time operations.
3. Do not add error handling for scenarios that cannot happen.
4. Do not add comments, docstrings, or type annotations to code you didn't change.
5. Three similar lines of code is better than a premature abstraction.

---

## 11. Contract / State / Wiring Discipline

When touching any boundary, check **both sides**:

- APIs (function signatures and all call sites)
- Struct fields (definition and all accesses)
- Database schema (DDL and all queries)
- CLI arguments (clap definitions and dispatch handlers)
- Error types (definition and all match arms)
- Serialization (serde derives and actual data flow)
- SQL statements (table/column names must match schema)
- Build inclusion (Cargo.toml dependencies and actual `use` statements)
- Feature flags (Cargo.toml features and `cfg` attributes)
- **Output format boundaries** (data entering HTML, DOT, JSON, SQL, YAML, CLI args, or any structured format)

This applies to any kind of repo — frontend, backend, CLI, service, kernel, mixed.

---

## 12. Escape / Sanitization Discipline

Whenever data crosses from internal representation into a structured output format, verify escaping exists:

- **HTML output:** Escape `<`, `>`, `&`, `"`, `'` in all user-derived or parsed content. This prevents XSS.
- **DOT/Graphviz output:** Escape `"`, `\` in node names, labels, and edge identifiers.
- **SQL output:** Use parameterized queries (`?1`), never string interpolation.
- **JSON output:** Use a serialization library (serde_json), not manual string building.
- **CLI/shell output:** Escape or quote values that could be interpreted as shell metacharacters.

### Rules:

1. Treat any data that originated outside the program (parsed from files, read from DB, received from user input) as untrusted for output format purposes.
2. Never interpolate untrusted strings directly into structured formats.
3. When adding a new output format or rendering path, escaping is a **required** part of the implementation, not a follow-up.
4. When reviewing existing output paths, check whether escaping exists. If missing, treat it as a bug.

---

## 13. Stub / Dead Code / Fake-Complete Detection

Aggressively watch for:

- TODO-backed critical paths
- Placeholder return values
- No-op handlers
- Fake success paths (functions that return Ok(()) without doing real work)
- Dead code that looks real but is never called
- Abandoned registrations
- Compile-only partial implementations
- Code present only for appearance

If found, report them. Do not silently accept them as "working."

---

## 14. No Scope-Dodging / Relatedness Discipline

### Absolute prohibitions:

1. Do NOT dismiss a failing build, failing test, contract mismatch, runtime error, registration break, state inconsistency, warning spike, downstream regression, or newly surfaced bug as "not related" just because it was not the original task.
2. Do NOT use "pre-existing" as an excuse to ignore something that now blocks correctness, verification, or confidence.
3. Do NOT use scope language to escape hard debugging when adjacency, dependency, or verification impact exists.

### Required classification for surfaced issues:

| Classification | When to use |
|----------------|-------------|
| DIRECTLY CAUSED BY THE CHANGE | The change introduced this failure |
| INDIRECTLY EXPOSED BY THE CHANGE | The change revealed a latent issue |
| PRE-EXISTING BUT NOW BLOCKING | Issue predates the change but now blocks verification or correctness |
| TRULY UNRELATED, WITH EVIDENCE | Issue has no connection — must provide concrete evidence |

"TRULY UNRELATED" can only be used if supported by concrete evidence.

### Investigation requirement:

If an issue appears in a touched file, touched subsystem, dependency chain, call chain, contract boundary, build path, runtime path, or verification path — it must be investigated as potentially related.

If an issue surfaced only after the change, assume possible relation until disproven.

---

## 15. Root-Cause / 5 Whys Discipline

For any non-trivial bug, regression, wiring gap, state inconsistency, repeated failure, verification failure, or cross-layer mismatch:

1. Do a brief 5 Whys analysis or equivalent root-cause drill-down.
2. Do not stop at the first symptom-level explanation.
3. Do not accept a guard, null check, retry, fallback, or conditional as sufficient if the enabling cause remains unexamined.
4. If the root cause cannot be fully proven, explicitly separate:
   - **Confirmed cause** — what is proven
   - **Suspected enabling cause** — what is likely but not certain
   - **Symptom-only mitigation** — what was patched without root-cause resolution
   - **Unresolved uncertainty** — what remains unknown

### When NOT to apply:

- Do not turn 5 Whys into ritual for tiny trivial edits.
- Use it where it meaningfully reduces false confidence and catches incomplete fixes.

---

## 16. What-If / Edge-Case Discipline

For meaningful fixes, refactors, interface changes, wiring changes, or state changes, challenge the work with:

- What if the input is partial, stale, invalid, or out of order?
- What if registration never happens?
- What if cleanup fails?
- What if downstream still expects the old contract?
- What if the fix only works on the happy path?
- What if retry/timeout/error paths still break?
- What if the change exposes a hidden pre-existing fault?
- What if state ownership assumptions are wrong?
- What if build inclusion exists but runtime reachability does not?

### Scope:

- This catches blind spots, not creates ceremony.
- Use it for non-trivial work around wiring, contracts, state, cleanup, and adjacent fallout.
- Skip it for trivial formatting, comment, or single-line changes where the answer is obvious.

---

## 17. Git / Commit Awareness

When git history is available:

1. Inspect diffs, not just commit messages.
2. Distinguish claimed fixes from currently verified truth.
3. Check whether later commits weakened, bypassed, or reverted earlier work.

### Critical distinction:

- Commit history is **evidence of change**.
- Commit history is **NOT proof of correctness** by itself.

---

## 18. Contradiction / Confidence Discipline

### Contradiction tracking:

1. Actively track contradictions between docs and code, status files and code, commit history and current reality, assumptions and evidence, "complete" claims and failed verification.
2. Contradictions must be surfaced, not silently reconciled.
3. When evidence conflicts, say so explicitly.
4. Unresolved contradictions should be reflected in `WIRING_STATUS.md`.
5. Repeated contradiction patterns should become lessons in `learnings.md`.

### Confidence discipline:

Separate all claims into:
- **PROVEN** — verified by evidence
- **PARTIALLY PROVEN** — some evidence, gaps remain
- **NOT PROVEN** — no evidence yet
- **CONTRADICTED** — evidence disproves the claim
- **BLOCKED** — cannot verify due to dependencies or failures
- **NEEDS RUNTIME VALIDATION** — structurally plausible, no runtime proof

**Forbidden vague phrases** (unless backed by real evidence):
- "looks good"
- "seems fine"
- "probably unrelated"
- "should work"
- "appears complete"

---

## 19. Reporting Rules

Every completion report must include:

1. **Files changed** — list them.
2. **Why they changed** — one line per file.
3. **Checks run** — which commands were executed.
4. **Check results** — which passed, which failed.
5. **What remains unverified** — be explicit.
6. **Remaining risks** — what could still be wrong.
7. **Runtime proof status** — present or missing.
8. **Surfaced issues** — listed with classification (see Section 14).
9. **Blocking issues** — whether any surfaced issue blocks full completion.
10. **Result classification** — one of:
    - EDITED ONLY
    - STATICALLY VERIFIED (cargo check / clippy passed)
    - TESTED (cargo test passed)
    - RUNTIME-VALIDATED (ran against real input)
    - PARTIALLY VERIFIED (some checks passed, gaps remain)
    - BLOCKED (cannot complete verification due to surfaced issue)
11. **Memory file updates** — whether each was updated:
    - `context.md` updated: yes/no/not needed. If no, why.
    - `WIRING_STATUS.md` updated: yes/no/not needed. If no, why.
    - `learnings.md` updated: yes/no/not needed. If no, why.

Empty "done" claims are forbidden.

---

## 20. Repo Memory Maintenance Rules

1. When architecture, boundaries, source-of-truth files, invariants, assumptions, or known project state change — update `context.md`.
2. When a fix is verified, downgraded, contradicted, blocked, or newly unproven — update `WIRING_STATUS.md`.
3. When a subsystem moves between status levels (unproven → partial → complete → contradicted → blocked) — record that change.
4. When a reusable lesson emerges — update `learnings.md`.
5. Do not leave stale claims in any memory file once current code or verification evidence disproves them.
6. Do not preserve optimistic status text just because it was already written.
7. If a task changes repo truth or verified status, memory files must be updated before calling the work complete, unless explicitly deferred and called out.
8. If verification was not performed, `WIRING_STATUS.md` must reflect that honestly.

### Staleness awareness:

9. Each non-policy memory file includes a "last updated" and "last verified against code" date.
10. Treat stale dates as distrust signals — increase verification pressure when dates are old.
11. If docs are contradicted by code, surface the contradiction and record it as a lesson in `learnings.md` when reusable.
12. Memory files may be updated by humans or agents. If they conflict with code, **code wins**.

### File purposes (do not confuse):

- `context.md` = current project truth and assumptions
- `WIRING_STATUS.md` = evidence-backed verification status, not hopes or guesses
- `learnings.md` = reusable lessons, not current-state claims

---

## 21. Lesson Capture Rules

Future sessions must capture important reusable lessons into `learnings.md`:

1. When a bug reveals a recurring pattern — add the lesson.
2. When a false assumption causes wasted work — add the lesson.
3. When a verification failure exposes a recurring weakness — add the lesson.
4. When a refactor reveals a dangerous anti-pattern — add the lesson.
5. When a status or context file was wrong in a repeatable way — add the lesson.
6. When a class of "fake-complete" behavior is discovered — add the lesson.

### Quality rules:

- Do not dump raw noise into `learnings.md`.
- Only record lessons likely to matter again.
- Write lessons as reusable guidance, not one-off diary notes.
- Keep it concise and high-signal.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vzwjustin)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/vzwjustin)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
