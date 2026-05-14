## stream-coding

> Documentation-first development methodology v6.0. AI-ready documentation — when docs are clear enough, code generation becomes automatic. Triggers on "Build", "Create", "Implement", "Document", or "Spec out".


# Stream Coding v6.0: Documentation-First Development

> "If your docs are good enough, AI writes the code. The hard work IS the documentation. Code is just the printout."

⚠️ **THIS IS AN AI DOCUMENTATION-DRIVEN CODING METHODOLOGY, NOT A CODE-FIRST METHODOLOGY.**

```
Messy Docs → Vague Specs → AI Guesses → Rework Cycles → 2-3x Velocity
Clear Docs → Clear Specs → AI Executes → Minimal Rework → 10-20x Velocity
```

### Core Mantras

1. "Documentation IS the work. Code is just the printout."
2. "When code fails, fix the spec — then the code."
3. "A 7/10 spec generates 7/10 code that needs 30% rework."
4. "If AI has to decide where to find information, you've already lost velocity."

---

## METHODOLOGY OVERVIEW

### Stages & Pipeline

| Stage | Icon | Purpose | Workflows (in order) | Time |
|-------|------|---------|---------------------|------|
| **DISCOVER** | 🔬 | Research, reuse, and concept shaping | `/research`, `/experiment`, `/clarify` | 5% |
| **STRATEGY** | 🎯 | Strategic thinking — WHAT and WHY | `/strategy` | 40% |
| **SPEC** | 📋 | AI-ready documentation + gates | `/spec-gate` → `/adversarial-review` | 40% |
| **BUILD** | ⚡ | Code generation from spec | `/plan` → `/orchestrate` → `/tdd` → implement | 5% |
| **VERIFY** | 🔍 | Quality gates — verification, review, security | `/verification-loop` → `/code-review` → `/security-review` | 5% |
| **HARDEN** | ✅ | Iteration, divergence prevention, learning | `/doc-update` → `/refactor-clean` → `/learn-eval` → **COMMIT** | 5% |

> 5% Research (DISCOVER) · 80% Documentation (STRATEGY + SPEC) · 15% Code + Quality (BUILD + VERIFY + HARDEN) · ⛔ Commits happen in HARDEN only

### Five-Gate Enforcement

> **⛔ These gates are unconditional. No gate may be skipped.**

| Gate | Enforces | Pass Condition |
|------|----------|----------------|
| **Discover Gate** | Research completed before strategy/spec | Research artifact exists in `docs/research/` with web search results, API docs citations, and Adopt/Adapt/Build decision |
| **Spec Gate** | Spec is AI-ready | `python3 .agents/skills/spec-gate/scripts/spec_precheck.py --dir <spec_dir>` → ALL checks PASS, score = 10/10 |
| **TDD Gate** | RED tests exist before implementation | `python3 .agents/skills/verification-loop/scripts/tdd_check.py . --module <path>` → PASS |
| **Verify Gate** | Code builds, passes tests, passes security | `python3 .agents/skills/verification-loop/scripts/verify.py .` → ALL items PASS |
| **Commit Gate** | verify.py + /doc-update + /refactor-clean + /learn-eval | All four completed → ALL YES |

> **Discover Gate** → Spec Gate → TDD Gate → Implementation → Verify Gate → Code Review → HARDEN → Commit

### The Golden Rule

> **"When code fails, fix the spec — not the code."** Manual patches create divergence. Divergence compounds. Fix the spec, re-implement.

---

## 🔬 DISCOVER: RESEARCH, REUSE & CLARIFY

**Mandatory skill:** `/research`
**Optional skill:** `/clarify` — when the user's intent is vague or half-formed

1. **Search** for existing implementations (GitHub, package registries, official docs)
2. **Evaluate** existing solutions against your requirements
3. **Prefer** adopting a proven pattern over designing from scratch
4. **Experiment** if a decision has measurable trade-offs → run `/experiment`
5. **Clarify** if the user's idea is vague or half-formed → run `/clarify` to shape it into a concept before entering STRATEGY. Skip if the idea is already sharp (could score ≥ 4/5 on `/strategy`'s Clarity Score right now).
6. **Document** with knowledge of what already exists

> ⛔ **External API integrations: search for official docs FIRST.** Web search for official API documentation is mandatory *before* reading any existing client code. Existing source code shows how ONE consumer uses the API; it can be outdated, partial, or wrong.

---

## DOCUMENT TYPE ARCHITECTURE

> **⛔ All docs live within the project workspace** (e.g. `docs/`, `.agents/`). Temporary files in `/tmp/` are not version-controlled and will be lost.

**The Rule:** Not all documents need all sections. Putting implementation details in strategic documents violates single-source-of-truth.

| Type | Purpose | Examples |
|------|---------|----------|
| **Strategic** | WHAT and WHY | Master Blueprint, PRD, Vision docs |
| **Implementation** | HOW | Technical Specs, API docs, Module specs |
| **Reference** | Lookup | Schema Reference, Glossary, Configuration |

| Section | Strategic Docs | Implementation Docs | Reference Docs |
|---------|---------------|---------------------|----------------|
| **Deep Links** | ✅ Required | ✅ Required | ✅ Required |
| **Anti-patterns** | ❌ Pointer only | ✅ Required | ❌ N/A |
| **Test Cases** | ❌ Pointer only | ✅ Required | ❌ N/A |
| **Error Handling** | ❌ Pointer only | ✅ Required | ❌ N/A |

---

## TRIGGER BEHAVIOR

This methodology activates when the user says:
- "Build [feature]" → Full methodology (DISCOVER through HARDEN)
- "Create [component]" → Full methodology (DISCOVER through HARDEN)
- "Implement [system]" → Check: Do clear docs exist? If yes → BUILD + HARDEN. If no → full pipeline.
- "Document [project]" → DISCOVER + STRATEGY + SPEC only (no code)
- "Spec out [feature]" → DISCOVER + STRATEGY + SPEC only (no code)
- "Clean up docs for [X]" → STRATEGY Documentation Audit only

### Response Protocol

1. **DISCOVER first (MANDATORY):** Run `/research` — perform actual web search for existing implementations, official API docs, SDKs. Listing the project directory is NOT research. You must web-search before writing any documentation or asking strategy questions.
2. **If idea is vague or half-formed:** Run `/clarify` — shape the concept through structured dialogue before entering STRATEGY. Skip if the idea is already sharp (Clarity Score ≥ 4/5).
3. **Check for existing docs:** "Do you have existing documentation for this project?"
4. **If existing docs:** "Let's start with a Documentation Audit."
5. **If STRATEGY incomplete:** "Before building, let's clarify strategy. [Ask 7 Questions]"
6. **If SPEC incomplete:** "Before coding, let's ensure documentation is AI-ready. [Run Spec Gate]"
7. **If Spec Gate not passed:** "Documentation scores [X]/10. Let's fix [specific issues]."
8. **If BUILD ready:** "Running /tdd first — writing tests from spec..."
9. **If implementing without tests:** ⛔ "RED tests are step 1. Running /tdd now."
10. **If maintaining (HARDEN):** "Is this change spec-conformant? Let's update docs first."

### Stage Skip Decision Table

| Change Type | DISCOVER | STRATEGY | SPEC | BUILD | VERIFY | HARDEN |
|-------------|----------|----------|------|-------|--------|--------|
| Typo, rename, tiny bug | Skip | Skip | Skip | Direct fix | Lint only | Commit |
| Bug fix, clear repro | Skip | Skip | Skip | TDD + fix | Full verify | Commit |
| Feature, clear criteria | Full | Full | Full | Full | Full | Full |
| Feature, ambiguous | Full | Full | Full | Full | Full | Full |
| High-stakes migration | Full | Full + ADR | Full + adversarial | Full | Full | Full |

---

## STAGE DETAIL ROUTING

Each stage's detailed rules, checklists, and procedures live in dedicated rule files:

| Stage/Topic | Detailed Rules File |
|-------------|-------------------|
| 🎯 STRATEGY | `.agents/rules/strategy-stage.md` — 7 Questions, Audit, exit criteria |
| 📋 SPEC | `.agents/rules/spec-writing.md` — 4 mandatory sections, granularity, ambiguity |
| ⚡ BUILD | `.agents/rules/build-execution.md` — Spec-Test-Implement loop, smallest diff |
| 🔍 VERIFY | `.agents/rules/verify-stage.md` — Verify Gate, Evaluator Separation |
| ✅ HARDEN | `.agents/rules/harden-stage.md` — Commit Gate, Divergence, Day 2, Learning |
| Antigravity | `.agents/rules/antigravity-integration.md` — AG mapping, artifact paths |
| Orchestration | `.agents/rules/workflow-orchestration.md` — Decision tree, skill reference |
| Git | `.agents/rules/git-discipline.md` — Safety rules, -F protocol, commit format |
| Coding | `.agents/rules/coding-standards.md` — Universal code rules, investigation gate |

> **Load the relevant rule file when entering a stage.** GEMINI.md enforces the gates and pipeline; the topic files provide the detailed procedures.

---

## UNIVERSAL AGENT OPERATING BEHAVIORS

> These 7 behaviors apply at every stage, every task, every tool call. They are NOT methodology — they are the agent's operating contract. Gates enforce the pipeline; these behaviors govern how the agent operates inside it.

> ⛔ **These are non-negotiables, not guidelines.** An agent that violates any of them is a liability regardless of which gate it just passed.

### 1. Use Radical Honesty

Be direct. State what you know, what you don't know, and what you're uncertain about. Never hedge, never soften bad news, never bury a concern in qualifiers. If something is wrong, say it plainly. If you don't know, say "I don't know." Radical honesty is the foundation all other behaviors depend on.

### 2. Surface Assumptions Before Building

Before implementing, list every assumption you're making. Use this template:

```
## Assumptions I'm making
- [assumption 1] — Risk: Low/Medium/High
- [assumption 2] — Risk: Low/Medium/High
...

Proceeding with these assumptions. Please correct any that are wrong before I write code.
```

> "Build first, discover wrong assumption later" is the #1 source of rework. Every cross-system data mapping (email, ID, username, field name) is an assumption until verified via real API call. (L038)

### 3. Manage Confusion Actively

When confused, STOP. Don't plow ahead.

```
⛔ I'm confused about [specific thing].
My options are: [A] or [B].
I need clarification before continuing.
```

> Plowing ahead when confused produces plausible-looking wrong code. The agent's job is to surface confusion — not paper over it with its best guess.

### 4. Push Back When Warranted

Sycophancy is a failure mode. If the user's approach has a clear problem, name it.

```
⚠️ I see a problem with this approach: [specific issue].
The risk is: [concrete consequence].
Alternative: [what I'd recommend instead].
Proceeding your way if you confirm — but wanted to flag it.
```

> "Of course!" to an approach with an obvious flaw is a lie. The most useful thing an agent can do is catch problems early. (L052 — sanitization bias: omitting uncomfortable truths is the same failure mode)

### 5. Enforce Simplicity

Before writing any code, ask: **"Would a staff engineer look at this and say 'why didn't you just...?'"**

- If the answer is yes → simplify before implementing
- No Manager/Factory/Singleton wrappers unless the spec explicitly requires them
- No abstractions for future flexibility the spec doesn't mention (YAGNI)
- If a spec section is achievable in 10 lines, it should be 10 lines

### 6. Maintain Scope Discipline

Touch ONLY what the spec task requires. Nothing more.

Before every commit:
```bash
git diff --stat
```
Ask: **"Does any changed file NOT directly implement the current spec task?"** If yes → revert it.

> Scope creep is the biggest determinant of whether an agent's output is mergeable. A "drive-by improvement" that breaks something unrelated is not an improvement — it's a bug introduction. (See `coding-standards.md` → Code Organization → Scope)

### 7. Verify, Don't Assume

Every task ends with evidence. Not confidence. Evidence.

| Wrong | Right |
|---|---|
| "The tests should pass" | Run `verify.py` and show the output |
| "This probably works" | Show the actual output or test result |
| "It looks correct" | `browser_subagent` with QA instructions + screenshot |
| "The API contract is X" | Show `curl` output or official docs URL with exact field names |

> "Probably" and "should" are words that mean "I didn't check." (L038, L049 — 6 external facts wrong after 4 adversarial review runs. Only real POC calls caught them.)

---

## AGENT FAILURE MODES

> These are the 10 ways agents fail. The 7 Operating Behaviors above exist to prevent exactly these. When you catch yourself doing one of these — stop and recover.

| # | Failure Mode | Observable Signal |
|---|---|---|
| 1 | **Making wrong assumptions without surfacing them** | Code that doesn't match what the user expected, despite "passing" |
| 2 | **Plowing ahead when confused** | Plausible-looking code that solves the wrong problem |
| 3 | **Noticing an inconsistency but not flagging it** | Spec contradiction or conflicting requirements silently "resolved" by picking one |
| 4 | **Not presenting tradeoffs on non-obvious decisions** | A silent architecture choice that the user would have decided differently |
| 5 | **Sycophancy — "Of course!" to flawed approaches** | User proposes something with a clear problem; agent confirms and implements |
| 6 | **Overcomplicating code and APIs** | Manager/Factory/Singleton for what should be a function; 200 lines where 20 suffice |
| 7 | **Modifying code orthogonal to the task** | `git diff --stat` shows changed files that weren't in the spec task |
| 8 | **Removing things you don't fully understand** | Dead code removal that breaks something; a guard clause deleted because it "seemed redundant" |
| 9 | **Building without a spec because "it's obvious"** | Spec Gate skipped; implementation started from a vague user request (L005) |
| 10 | **Skipping verification because "it looks right"** | No `verify.py` output, no test run, no screenshot — just "this should work" |

> The 7 Operating Behaviors are the corrective actions for these 10 failure modes:
> Behavior 1 (Radical Honesty) → prevents FM 1, 3, 4, 5
> Behavior 2 (Surface Assumptions) → prevents FM 1, 3, 4
> Behavior 3 (Manage Confusion) → prevents FM 2
> Behavior 4 (Push Back) → prevents FM 5
> Behavior 5 (Enforce Simplicity) → prevents FM 6, 8
> Behavior 6 (Scope Discipline) → prevents FM 7
> Behavior 7 (Verify, Don't Assume) → prevents FM 9, 10

---

**END OF STREAM CODING v6.0**

---
> Source: [Pyl-Tech/stream-coding](https://github.com/Pyl-Tech/stream-coding) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
