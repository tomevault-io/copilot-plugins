## ship-feature

> >


# Ship Feature

Autonomous 4-agent pipeline: Planner → Coder → Tester → Reviewer, followed by an
orchestrator-run **live-proof loop** that iterates the feature against the real
e2e surface (devnet, staging, local app, whatever your project has) until green.
Every agent is pre-seeded with the full codebase context for the repo(s) in scope, plus the
reference files that match the feature.

**Trigger:** `/ship <feature description>` or any request to implement + ship autonomously.

---

## Pipeline overview

```
Step 0: Live recon → safety preflight → load context docs → select reference files → build REPO_CONTEXT
    ↓
Step 1: Planner (strongest model)   → .pipeline/spec.md
    ↓
Step 2: Coder (mid-tier model)      → .pipeline/changes.md
    ↓
Step 3: Tester (mid-tier model)     → .pipeline/test-results.md
    ↓
Step 4: Reviewer (strongest model)  → .pipeline/review.md
    ↓
Step 4.5: Live-Proof Loop (orchestrator) → iterate-until-green → .pipeline/live-proof.json
    ↓
Step 5: Report VERDICT to user   (SHIP (LIVE-PROVEN) / SHIP (UNIT-PROVEN) / NEEDS WORK / BLOCK)
```

---

## Reference files

Reference files live in `references/`. They DEEPEN correctness; they never GATE it — an
orchestrator that reads ONLY this SKILL.md still runs a correct (if shallower) pipeline. Step 0d
decides which to inject; Step 0e pastes each selected file verbatim under its sentinel, exactly
like `REPO_CONTEXT`.

- **`references/debugging-doctrine.md`** — fast reproducer first; A/B both directions; never
  diagnose through an unverified read layer; read post-state before theorizing; monitor failure
  signatures; peel one layer per run. Domain-agnostic. → Tester + the Step 4.5 live-proof loop.
- **`references/e2e-spec-engineering.md`** — ONE stapled spec, cold-boot, structural assertions,
  locale-agnostic selectors, toast-as-evidence-capture, multi-actor testing, report JSON, machine
  safety. → Coder + Tester on app repos with a browser e2e surface.
- **`references/example-domain-brain.md`** — a worked EXAMPLE of a domain-specific reference
  (the author's, for a Bitcoin metaprotocol). Write your own for your domain (see "Adapting
  this skill" below) and inject it into all four agents when a feature touches that domain.

---

## Adapting this skill to your project (do this once)

This skill ships with placeholders where project knowledge belongs. Fill in three things:

1. **Repo table** (in "Repo Context Templates" below): list your repo(s), their root paths, and
   one line on what each is. Step 0a re-verifies this against disk every run anyway.
2. **Repo context template(s)**: for each repo, write a `=== <REPO> — MANDATORY CONTEXT ===`
   block holding the invariants an agent must never violate — the "never do X" rules, the
   source-of-truth rules, the key file paths, the safety rules. Mine your CLAUDE.md, past
   incident writeups, and code-review feedback. Specifics beat paraphrase: "never call
   `fetch(rpc)` directly — use `lib/rpc.ts` wrappers" is useful; "follow good practices" is not.
3. **Domain brain(s)** (optional but high-leverage): if your project has a domain with
   non-obvious failure modes (a blockchain protocol, a distributed system, a hardware target),
   distill the hard-won facts into a `references/<domain>-brain.md` modeled on
   `example-domain-brain.md`. Every rule should state its WHY (the real burn that produced it)
   so a zero-context agent trusts it.

---

## Step 0 — Setup, Live Recon & Context Preloading

This step is mandatory and must complete before any agent is spawned.

### 0a — Live repo recon (REPLACES blind trust in the repo table)

The repo table below is a recon-refreshed **default**, not gospel — branches and primary repos
move. For each candidate repo, verify on disk:
`git -C <path> rev-parse --abbrev-ref HEAD` (branch), `git -C <path> status --porcelain | wc -l`
(dirty count), `ls <path>` (exists). BUILD the working table fresh from these; override the
defaults on any mismatch. Ask the user if the repo(s) in scope are unclear — a single feature may
touch multiple repos.

### 0b — Concurrency & safety preflight

Run these BEFORE touching any repo:

1. **Concurrent-session detection.** Check mtimes of `.pipeline/*` and dirty trees (optionally
   `ps aux | grep claude`). Two sessions in the same checkout fight over HEAD and kill each
   other's browsers — isolate in a `git worktree` if detected.
2. **Checkpoint WIP first.** If the target repo is dirty or on a non-default branch, CHECKPOINT
   the working tree on a branch/commit BEFORE editing — never disturb someone else's uncommitted
   work. When in doubt, run the pipeline in a separate worktree.
3. **Resource gate.** If the live-proof surface is memory-heavy (an in-browser VM, a local
   chain, a big test harness), check free memory BEFORE any live run (on macOS:
   `vm_stat` free pages > 5000). Below the floor, the live loop is DEFERRED and the verdict is
   UNIT-PROVEN — a heavy run on a starved machine can swap-death freeze the host.
4. **One heavy harness at a time.** Never two memory-heavy harnesses in parallel; never run the
   live e2e alongside a manually-booted instance of the same surface.
5. **Port etiquette.** Do not assume the default dev port. Check who owns candidate ports
   (`lsof -ti:<port>`) and drive the spec via an env var (e.g. `PLAYWRIGHT_BASE_URL`).
6. **Worktrees may need a REAL dependency install** — some dev servers (e.g. Next.js/Turbopack)
   reject symlinked `node_modules`.

### 0c — Load context docs

For each repo in scope, **you (the orchestrator) must read the following files yourself** before
spawning any agent, and inline the content into each agent's prompt as the `REPO_CONTEXT` block:

- The repo's `CLAUDE.md` / `AGENTS.md` (primary source of truth for rules and invariants).
- Architecture docs (`docs/architecture.md`, `docs/CORE_PRINCIPLES.md`, or equivalent).
- The workspace manifest (`package.json` / `Cargo.toml` / `go.mod`) for layout and members.
- Any plan doc relevant to the feature.

List the exact files per repo in your repo table so this step is mechanical.

### 0d — Select reference files to inject

Decide injection per feature:
- **Touches a domain with a brain file** (e.g. your protocol layer, your distributed-system
  core) → inject that `references/<domain>-brain.md` into **all four agents**.
- **Has a browser UI with an e2e surface** → inject `references/e2e-spec-engineering.md`
  into **Coder + Tester** (Reviewer gets a one-line pointer).
- **Always** inject `references/debugging-doctrine.md` into the **Tester** and the **Step 4.5
  live-proof loop** prompt.
- A pure backend/library feature pulls neither app reference; `debugging-doctrine.md` is
  domain-agnostic and stays "Always" per the rule above.

### 0e — Build the REPO_CONTEXT block

After reading, synthesize the content into a structured `REPO_CONTEXT` block prepended to every
agent prompt. Use the templates in "Repo Context Templates" below. Do not summarize — keep the
exact rules, paths, and invariants verbatim. Reference files selected in 0d are pasted by the
IDENTICAL mechanism: read the file, paste its content verbatim under its sentinel at the
agent's paste-point. Agents need the specifics, not a paraphrase.

### 0f — Pipeline housekeeping

1. Create `.pipeline/` at the repo root (or a shared `.pipeline/` if multiple repos are in scope
   — put it at the primary repo root). Add `.pipeline/` to the repo's `.gitignore` if absent.
2. Delete stale handoff files from prior runs.
3. Confirm feature description: if vague, ask one clarifying question.

---

## Repo Context Templates

These are the critical invariants per repo. You must inject these verbatim into agent prompts,
supplemented with the full context docs you read in Step 0c. The paths are
**last-known defaults; Step 0a verifies each against disk and overrides on mismatch.**

| Repo | Last-known root path (verify in 0a) | What it is |
|---|---|---|
| `<your-app>` | `/path/to/your-app` | e.g. Next.js frontend |
| `<your-backend>` | `/path/to/your-backend` | e.g. Rust/Go service |
| `<your-contracts>` | `/path/to/your-contracts` | e.g. smart contracts / protocol layer |

> The table is a hint; disk is truth. Branches in particular move.

### Template skeleton (write one per repo)

```
=== <REPO NAME> — MANDATORY CONTEXT ===
(This is the summary; the full CLAUDE.md you read in 0c supplements it.)

RULE -1 (HIGHEST PRIORITY): grep for how the same thing is already done BEFORE writing any code —
the codebase IS the documentation. (Give 2-3 concrete grep recipes for the repo's most common
tasks: how to fetch data, how to call the core API, how existing tests invoke the thing.)

SOURCE OF TRUTH RULES (never violate):
  SoT-1: <where balances/state/config MUST come from, and the banned alternatives + why>
  SoT-2: <...>

DATA/RESOURCE MODEL: <the non-obvious model an agent will get wrong by default — e.g. "tokens
  ARE UTXOs; spending one for fees destroys it", or "this cache is the only writer">

BANNED PATTERNS: <raw calls that must go through a wrapper layer; symbolic placeholders that
  resolve to the wrong thing; APIs with known phantom/stale-data bugs — name file paths>

CRITICAL SAFETY: <prod namespaces/environments never to touch; mandatory pre-sign/pre-deploy
  steps that must never be removed>

KEY FILES: <config entry, RPC/API wrapper layer, state layer, mutation hooks, styling system>
=== END <REPO NAME> CONTEXT ===
```

The WHYs matter: where a rule exists because of a real past incident, say so inline
("broken twice — PRs #112, #120 — weeks to fix each"). Agents follow rules they can see the
cost of breaking.

---

## Agent prompts — injection convention

Each agent prompt below opens with paste-points. Always paste the full `REPO_CONTEXT` (Step 0e)
first. Then paste each reference file Step 0d selected, verbatim under its `<<…>>` line; OMIT any
paste-point not selected. Paste-points: `BRAIN` = your `references/<domain>-brain.md` (domain
features), `E2E` = `references/e2e-spec-engineering.md` (app repos with a browser e2e surface),
`DOCTRINE` = `references/debugging-doctrine.md`.

Model guidance: run Planner and Reviewer on your strongest available model; Coder and Tester on
a fast mid-tier model. Adjust to taste.

---

## Step 1 — Planner (strongest model)

Prepend the full `REPO_CONTEXT` block (from Step 0) to the following prompt, then spawn an
agent:

```
<<REPO_CONTEXT>>  <<BRAIN if domain feature>>

You are the Planner in a 4-agent feature-shipping pipeline. Your job is deep codebase
analysis — not implementation. The REPO_CONTEXT above is your starting point; treat it as
authoritative for architecture and invariants, but always verify specifics against the
actual source files before citing them.

Feature to ship: <FEATURE>
Repo root: <REPO_ROOT>

Your deliverable is <REPO_ROOT>/.pipeline/spec.md. Write it before you exit.

spec.md must contain:
1. FEATURE SUMMARY — one paragraph, plain English, what this feature does and why.
2. FILES TO CHANGE — exact paths of every file that will need edits. For each file:
   - Current relevant function/type names and signatures (read the actual file)
   - What specifically needs to change and why
   - Existing patterns in the codebase the Coder must follow (grep to find them)
3. FILES TO CREATE — paths + purpose for any new files.
4. TESTS TO WRITE — what behavior must be covered, where tests live, what test
   patterns already exist (grep for describe/it/#[test]/#[tokio::test]).
   If a domain brain is injected above: the spec MUST require the test strategies it
   mandates (e.g. byte-vector tests pinned to source layout; reads via the verified
   read-path, not known-broken wrappers).
5. INVARIANTS — things the Coder must NOT break. Include ALL relevant invariants
   from the REPO_CONTEXT above that apply to this feature.
6. OPEN QUESTIONS — ambiguities the Coder should resolve conservatively.

Rules:
- Read before you write. Use Grep and Glob extensively. Never assume — verify.
- If the REPO_CONTEXT says "never use X", include that as an explicit invariant.
- Your spec is the Coder's only input. Vague spec = wrong implementation.
- Do not write any implementation code. Analysis and instructions only.
- End the file with exactly: PLANNER COMPLETE
```

**Gate:** `.pipeline/spec.md` must exist and end with `PLANNER COMPLETE`.

---

## Step 2 — Coder (mid-tier model)

Prepend the full `REPO_CONTEXT` block, then spawn an agent:

```
<<REPO_CONTEXT>>  <<BRAIN if domain feature>>  <<E2E if browser e2e surface>>

You are the Coder in a 4-agent feature-shipping pipeline. Your job is clean, correct
implementation. The REPO_CONTEXT above contains the architectural rules for this codebase.
Treat every rule in it as a hard constraint — not a suggestion.

Read <REPO_ROOT>/.pipeline/spec.md first. That document is your specification.

Rules:
- FOLLOW EXISTING PATTERNS. The spec will identify them — use them exactly.
  If the spec says "use the wrapper in lib/rpc.ts", use it. If it says "thread context
  from the canonical hook", do it.
- Honor every BANNED PATTERN in the REPO_CONTEXT — no raw calls around wrapper layers,
  no placeholder values that resolve to the wrong target.
- EVIDENCE-CAPTURE RULE (app repos): every user-facing mutation MUST surface a success
  signal the e2e can capture (e.g. a success toast carrying the transaction/operation id) —
  a mutation with no machine-readable success signal is UNTESTABLE.
- Do not refactor unrelated code. Do not rename things outside scope.
- No TODOs or placeholder implementations. Everything must compile and run.
- After implementation: run the type checker (npx tsc --noEmit for TS, cargo check for Rust,
  etc.). Fix ALL errors before writing changes.md.
- Run the existing test suite. Note results.

Your deliverable is <REPO_ROOT>/.pipeline/changes.md.

changes.md must contain:
1. FILES CHANGED — path + description of what changed and why.
2. FILES CREATED — same.
3. KEY DECISIONS — how you resolved OPEN QUESTIONS from the spec.
4. TYPE CHECK RESULT — full output. Must say PASS.
5. EXISTING TESTS RESULT — pass/fail counts. List any pre-existing failures separately.

End the file with exactly: CODER COMPLETE
```

**Gate:** `.pipeline/changes.md` must exist, end with `CODER COMPLETE`, and TYPE CHECK RESULT must be PASS.

---

## Step 3 — Tester (mid-tier model)

Prepend the full `REPO_CONTEXT` block, then spawn an agent:

```
<<REPO_CONTEXT>>  <<DOCTRINE (always)>>  <<E2E if browser e2e surface>>  <<BRAIN if domain feature>>

You are the Tester in a 4-agent feature-shipping pipeline. Your job is verification,
not fixing. The REPO_CONTEXT above tells you what the codebase expects — use it to
identify what invariants the tests should assert.

Read <REPO_ROOT>/.pipeline/spec.md and <REPO_ROOT>/.pipeline/changes.md before writing
any tests.

Your job:
1. Write tests covering every behavior in TESTS TO WRITE from the spec.
2. Run all tests (existing + new). Every test must pass.
3. If a test fails because the Coder's implementation is wrong — document it, do NOT
   fix the implementation. Your job is to find problems, not paper them over.
4. If a test fails because your test is wrong — fix the test.
5. Include negative tests: if the REPO_CONTEXT says "never use X", write a test that
   confirms X is not used (e.g. grep the changed files for the banned pattern).

Methodology (DEBUGGING DOCTRINE above):
- Prefer a fast node/CLI-side reproducer (ONE harness at a time, resource gate respected)
  over a full live boot for diagnosing failures (seconds vs minutes per iteration).
- If a domain brain is injected: follow its mandated test strategies exactly.
- If this app repo has a browser e2e surface: WRITE/EXTEND the project's ONE stapled spec
  as a new Flow/Stage N — do NOT fork a per-feature spec. Follow every E2E SPEC ENGINEERING
  rule.

Follow existing test patterns exactly (files the Planner identified, or grep describe(/it(/#[test]/#[tokio::test]).

Your deliverable is <REPO_ROOT>/.pipeline/test-results.md.

test-results.md must contain:
1. TESTS WRITTEN — each test file path + what scenarios it covers.
2. TEST RUN OUTPUT — full test runner output (copy/paste).
3. RESULT — one of: ALL PASS or FAILURES FOUND
4. If FAILURES FOUND: each failing test, the failure message, and whose fault it is
   (Coder's implementation vs. environment vs. test itself).

End the file with exactly: TESTER COMPLETE
```

**Gate:** `.pipeline/test-results.md` must exist and end with `TESTER COMPLETE`.
- `ALL PASS` → proceed to Reviewer.
- `FAILURES FOUND` → stop. Report to user. Do not proceed automatically.

---

## Step 4 — Reviewer (strongest model)

Prepend the full `REPO_CONTEXT` block, then spawn an agent:

```
<<REPO_CONTEXT>>  <<BRAIN if domain feature>>

You are the Reviewer in a 4-agent feature-shipping pipeline. You are skeptical,
thorough, and read-only — you do not write or edit implementation files.

The REPO_CONTEXT above is your checklist for pattern and invariant compliance.
Every rule listed there is something you must verify the Coder followed.
(For e2e specifics: confirm the changes follow the E2E SPEC ENGINEERING rules — ONE stapled
spec, structural assertions, a success signal per mutation, multi-actor for multi-role
protocols.)

Before writing your review, read: .pipeline/spec.md, changes.md, test-results.md; `git diff HEAD`
(or `git diff main..HEAD`); and the actual changed files.

Your review must address:
1. SPEC COMPLIANCE — did the Coder implement what the spec asked? Any gaps?
2. REPO_CONTEXT COMPLIANCE — go through every applicable rule: every SOURCE OF TRUTH rule,
   every BANNED PATTERN, every CRITICAL SAFETY rule. Verify each against the diff, not the
   Coder's claims.
3. INVARIANTS — did anything in the spec's INVARIANTS section get broken?
4. TEST QUALITY — do the tests actually catch regressions, or trivially pass?
5. STALE ARTIFACTS — is any .pipeline/* file contradicted by the actual code/tests? A stale
   test-results.md (claims that no longer match reality) is a HIGH finding.
6. RISK ASSESSMENT — what could go wrong in production? Untested edge cases?
7. SECURITY — hardcoded secrets, missing auth, injection vectors, key material exposure?

Your deliverable is <REPO_ROOT>/.pipeline/review.md.

review.md must contain:
1. SUMMARY — two paragraphs: what was built, and your overall assessment.
2. FINDINGS — numbered list (critical / major / minor / nit). Empty list if none.
3. LIVE-PROOF STATUS — state explicitly whether the feature is verified by unit/contract
   tests only (UNIT-PROVEN) or also against the real live/e2e surface (LIVE-PROVEN). If a
   live gate remains (e.g. a cold-boot smoke not yet run), NAME it. This feeds the Step 5
   verdict taxonomy.
4. VERDICT — exactly one of these on its own line:
   VERDICT: SHIP          (correct, tested, follows all patterns, safe to merge)
   VERDICT: NEEDS WORK    (minor issues, not blockers — list them)
   VERDICT: BLOCK         (critical issues — wrong behavior, broken invariant,
                           security flaw, or test failures)

End the file with exactly: REVIEWER COMPLETE
```

**Gate:** `.pipeline/review.md` must exist and end with `REVIEWER COMPLETE`. Extract VERDICT.

---

## Step 4.5 — Live-Proof Loop (orchestrator-run)

After a SHIP/NEEDS-WORK review, the orchestrator drives an iterate-until-green live-proof loop —
pipeline machinery it runs directly (it MAY spawn focused Coder/Tester sub-invocations, but it is
NOT a 5th agent). Paste `references/debugging-doctrine.md` into every focused-Coder it spawns.

### 4.5a — When the loop runs / when it's skipped

Run the loop ONLY when the feature has a runnable live/e2e surface (an app repo with a stapled
browser spec, a service with an integration harness). For pure library/contract repos there is
NO browser loop — the proof is the unit/integration test suite + (where applicable) a fast
CLI reproducer; the loop degrades to "run the tests, report UNIT-PROVEN". Do NOT fabricate
a live loop where there is none. Also skip (DEFER) if the 0b resource gate fails (free memory
below the floor, or no display for a headed run) — exit UNIT-PROVEN with the blocker named.

### 4.5b — Background e2e + stall detection + failure-signature coverage

Launch the project's ONE stapled spec as a background process (cold-boot, resource-gated),
then monitor it in a loop that (1) tails the runner log; (2) detects STALLS — define a
startup deadline (e.g. no setup/funding progress after 15 min → kill + relaunch) and a
mid-flow silence threshold (e.g. 5+ min of silence signals an OOM or stuck-mutation hang; a
true timeout is a REAL stuck operation, NOT a budget shortfall — do not raise the cap to mask
it); (3) scans for your project's FAILURE signatures, not only the pass string — grep the known
revert/error/empty markers, because a monitor that only waits for "PASSED" sits forever on a
silent failure.

### 4.5c — todoChecks scorecard JSON

The live run emits `.pipeline/live-proof.json` in the e2e-spec-engineering report shape:
`FlowResult[]` (name, status, evidence id, skip_reason, error, trace) + `aberrations: string[]`
+ `todoChecks: Record<checkName, "PASS" | "FAIL: …">` (+ feature-specific fields). Every named
check that FAILS must ALSO push a string into `aberrations`. This scorecard is the loop's
machine-readable exit condition.

### 4.5d — Focused-Coder iteration (evidence-bearing, not a full rerun)

On a live failure, spawn a FOCUSED Coder — NOT a full pipeline rerun — carrying the EVIDENCE: the
failing check name, the aberration string, the relevant trace/log digest, and the runtime
console lines. Paste the domain brain (if applicable) + the debugging doctrine into its prompt.
Constrain it to ONLY that failure; give it the original spec.md so it stays spec-compliant; have
it OVERWRITE changes.md / test-results.md (keeping `.pipeline/` current). Then re-run the same
scorecard. Efficiency lesson from a real campaign: six failure layers were peeled across
sequential live runs; full pipeline reruns would have wasted a 10–20 min cold boot each time,
and only a live read-back exposed a wrong-opcode bug that every unit test had asserted as
"correct".

### 4.5e — Loop exit conditions

- **All-PASS + `aberrations` empty → LIVE-PROVEN.** Commit the green iteration with the evidence
  in the message (the check that now passes + its proof).
- **A documented hard environmental block** (no display / memory floor breached) → exit
  UNIT-PROVEN with the blocker named. iterate-until-green is the DEFAULT (see Iteration); the
  user is NOT asked between iterations.

---

## Step 5 — Report to User

Send a final message with:
1. **VERDICT** prominently — for SHIP, the evidence tier:
   - `VERDICT: SHIP (LIVE-PROVEN)` — proven against the real live/e2e surface (all scorecard
     checks PASS, zero aberrations) → "Ready to merge. Full report in `.pipeline/review.md` +
     `.pipeline/live-proof.json`."
   - `VERDICT: SHIP (UNIT-PROVEN)` — unit/contract tests green but a live gate remains; NAME it
     (e.g. "cold-boot smoke not yet run; needs free memory + display").
   - `VERDICT: NEEDS WORK` → list the issues (the loop already iterated; only escalate here if
     hard-blocked). `VERDICT: BLOCK` → explain the blocker; do not suggest merging.
2. Feature summary (from spec.md).
3. Test results (pass count) + live-proof scorecard summary if the loop ran.
4. Key FINDINGS from the Reviewer (if any).

---

## Iteration

**Autonomous iterate-until-green is the DEFAULT.** Do NOT ask the user which step to restart
from. The live-proof loop (Step 4.5) drives focused iterations automatically until the scorecard
is green OR a documented hard environmental block is hit. Stopping at "plumbing proven, real flow
deferred" is not acceptable when the user asked for the real behavior — if blocked, ask a
clarifying question; otherwise keep peeling layers until green.

Per iteration:
- **Evidence-bearing commits.** Each green iteration commits with the evidence in the message
  (the check that now passes + its proof).
- **Keep `.pipeline/` artifacts CURRENT.** Every iteration OVERWRITES changes.md /
  test-results.md / live-proof.json so they match reality — a stale `test-results.md` that
  contradicts the code is a HIGH Reviewer finding (Step 4 checklist item 5).
- **Re-read context before re-spawning.** Don't reuse a stale REPO_CONTEXT or stale reference
  injection — re-read the context docs and reference files.

To re-run a non-live failure manually: append the Reviewer's FINDINGS to the Coder prompt as
`REVIEWER FEEDBACK FROM PREVIOUS RUN:` and re-run from Step 2 (overwriting `.pipeline/`).

---

## Cross-repo features

If the feature touches multiple repos (e.g. a protocol change in both the app and the mobile
client): build a combined REPO_CONTEXT with a section per repo; the Planner produces
separate FILE sections per repo; the Coder works each repo sequentially; tests run per repo; the
Reviewer diffs both. The Step 4.5 live-proof loop runs ONCE PER REPO that has an e2e surface
(pure library repos degrade to "run the tests, report UNIT-PROVEN", Step 4.5a). Put `.pipeline/`
in the primary repo (the one the user named first).

---
> Source: [0xcasuwu/ship-feature](https://github.com/0xcasuwu/ship-feature) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
