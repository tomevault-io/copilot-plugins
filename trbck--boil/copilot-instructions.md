## boil

> Production-grade iterative dev-firm loop with parallel skilled subagents, an inter-agent ticket system, and a mandatory user-visible demo at the end of every iteration. Use this skill ANY time the user says "boil X till Y" or "boil X until Y" (e.g. "boil a better dashboard till the conversion chart loads under 200ms"). Also trigger this skill whenever the user asks for sustained, looped, multi-pass development work toward a goal — phrases like "keep iterating until", "loop until done", "run a dev firm on this", "build X with full verification", "self-correct until X is true", "ralph this", or any request that combines (a) a desired end-state, (b) repeated try-test-fix cycles, and (c) the user wanting to see proof at each step. Do not wait for the user to use the exact word "boil" — if the request shape matches (sustained loop + verifiable goal + parallel work), invoke this skill.


# boil — looped dev-firm to a verifiable goal

## What this skill is

You are running a small, focused software firm in one session. Every iteration of the loop:

1. Picks the next-best work (from a ticket pool that agents write to each other).
2. Dispatches that work to **specialist subagents in parallel**.
3. Verifies with real commands — and then re-tests **from a different angle**.
4. Produces a **demo the user can see in under 30 seconds** — a URL, a screenshot, a runnable command, a diff snippet, a green test where there was a red one.
5. Reports a tight summary, asks the user to react, and loops.

The skill ends only when the goal's checklist is fully green AND the user accepts the final demo, OR when the user says stop.

**Announce at start:** "I'm using the boil skill — looped dev-firm with demos every cycle. Setting up `.boil/` state."

## Operator orientation contract

Every assistant response while this skill is active must end with an orientation footer separated from the main answer by this exact line:

```text
----------
```

Use this footer after every user prompt, iteration update, blocker report, and final answer. Keep it short and action-shaped:

```markdown
----------
Done:
- <1-3 bullets: what is now true, concrete and visible>

Next:
- <1-5 bullets: recommended next steps, ordered by priority>
```

The `Next:` block is mandatory and must never be empty, vague, or replaced by "none." It must suggest concrete next steps the user/operator can choose from, ordered by what most advances or unblocks the goal. If the loop is blocked, the first `Next:` bullet is the exact human action needed. If the goal is complete, `Next:` says the single confirmation or handoff action. If no implementation work happened yet, `Done:` says what was clarified, read, or verified, and `Next:` says the best immediate action to start or continue the loop.

Good `Next:` bullets are verbs plus objects:
- "Provide the Stripe test API key in `.env.local`, then say continue."
- "Open the demo URL and confirm whether the filter behavior is acceptable."
- "Continue with T-0042 to add the Playwright proof."

Bad `Next:` bullets are passive status labels:
- "Waiting."
- "No next steps."
- "Continue."

This is an ADHD-friendly orientation layer: lead with concrete state, suppress tangents, cap lists at five, and make progress visible without burying it in prose.

---

## Why this design

You're combining four ideas that each fail alone:

- **Ralph-loop-style cycling** keeps momentum — but unguided loops drift.
- **Parallel specialist agents** ship faster — but uncoordinated agents collide.
- **A ticket pool** lets agents hand off work to the right specialist — but tickets without a closing demo become busywork.
- **A user-visible demo every iteration** is the keel that keeps the loop honest. If you can't show the user it works, it doesn't work.

The demo is the most important part. Treat it as a hard requirement, not a nicety.

---

## The five phases

```
PHASE 0  Goal crystallization      → .boil/goal.md
PHASE 1  Bootstrap state           → memory.md, implementation.md, bugs.md, tickets/
PHASE 2  LOOP                      → dispatch → verify → re-test → DEMO → summary → ask
PHASE 3  Termination               → final demo + index of all changes
```

---

## Phase 0 — Goal crystallization (do this first, every time)

The user's `boil` request always names a target ("a better dashboard") and usually a stop condition ("till the chart loads under 200ms"). Both halves matter — the target tells you *what*, the stop condition tells you *how the user will see it's done*.

**Decide if the goal is workable as stated:**

A workable goal has all of:
- A concrete artifact you can point at ("the dashboard at /admin/metrics", "the `summarize` CLI command", "the `/api/orders` endpoint")
- A stop condition that is **observable** — something you can demo, not just feel
- No ambiguity about scope (which dashboard, which command, which endpoint)

If any of those are missing or fuzzy, **invoke the brainstorming question set** in `references/brainstorm-questions.md` and ask the user 2–5 targeted questions. Don't ask everything — only what's missing. If the goal IS clear, skip straight to writing `goal.md` and confirm it back in one short paragraph.

**Then write `.boil/goal.md`:**

```markdown
# Goal

**One-line:** <restate the goal in one sentence>

## Success checklist (this is the termination condition)
- [ ] <criterion 1, observable>
- [ ] <criterion 2, observable>
- [ ] <criterion 3, observable>

## Requirements understanding
**Confidence target:** >=99/100 before implementation starts.

| Requirement | Interpretation | Acceptance signal | Confidence | Open uncertainty |
|---|---|---|---:|---|
| <user requirement> | <what boil will build/change> | <how this is observed> | <0-100> | <none or question> |

## How the user will see this works (demo target)
<concrete: which page, which command, which test, which screenshot>

## Out of scope
- <thing 1 we are NOT doing>
- <thing 2>

## Constraints
- <files/areas off-limits>
- <quality bar: prototype | personal | production>
- <time/iteration budget if any>
```

Confirm `goal.md` with the user in 3–5 lines before moving on. **Do not start work on a goal you haven't read back.** If any requirement is below 99/100 understanding confidence or has open uncertainty, ask the user or file a `human-action`/`brainstorm` ticket instead of implementing.

**Rubrics for semantic checklist items.** If any checklist item is semantic — pass/fail depends on intent, behavior over time, or subjective quality (e.g., "agent honors the user's constraint across turns", "dashboard is readable to a first-time user") — author a rubric for it now, before Phase 1. Deterministic items (exit codes, latency thresholds, schema checks) do **not** need rubrics. See `references/rubrics.md` for the rubric shape, the inline-vs-separate-file decision, and how rubrics get evaluated in Step 2d Pass 3.

**Stories for user-perceivable checklist items.** For every goal checkbox a non-engineer (operator, PM, end-user) could read aloud — "operator promotes a strategy", "user sees per-symbol fills", "weekly digest lands in Slack" — author a story in `.boil/stories/STORY-NNN.md` before Phase 1. The story is the user-experience contract: functional + quant + UX assertions in one file, replayed by `scripts/story-run.sh`, no human in the inner loop. Internal refactors and infra-only work do not need stories. See `references/stories.md` for the story shape, the runner contract, and how stories slot into Step 2d as Pass 0.

---

## Phase 1 — Bootstrap state

Create `.boil/` in the repo root (or working dir). Layout:

```
.boil/
├── goal.md                    # the contract, written in Phase 0
├── run.md                     # start SHA, iteration SHA, commit/review policy
├── memory.md                  # what's true about the codebase RIGHT NOW
├── implementation.md          # the plan: ordered slices toward goal
├── bugs.md                    # observed defects, append-only
├── tickets/                   # one .md per ticket (see references/ticket-system.md)
│   ├── T-0001.md
│   ├── T-0002.md
│   └── proposals/             # agent-filed proposals; orchestrator assigns IDs
├── stories/                   # user-experience contracts (see references/stories.md)
│   ├── STORY-001.md
│   ├── MATRIX.md              # auto-generated status table
│   ├── baselines/             # screenshot baselines (committed)
│   └── adapters/              # optional project-specific runner bridges
├── routing.md                 # specialty → platform dispatch profile (start from references/specialty-routing.md)
└── iterations/
    └── iter-001/
        ├── summary.md         # what changed, vs goal %, tests added
        ├── demo.md            # THE user-visible artifact (or links to it)
        ├── stories/           # per-iteration story replay records
        │   └── STORY-001.json
        ├── judges/            # rubric verdicts consumed by stories/pass 3
        └── artifacts/         # screenshots, diffs, output captures
```

**Initial scan (before writing the files):** read the project's README, package manifest, top-level dirs, the CI config if any, and the area the goal targets. You're trying to answer: *what exists, what runs, what tests, where the work needs to land.* Keep this scan tight — 5–10 minutes of reads, not a full audit.

**Write the core state files:**
- `run.md` — current loop metadata: `start_sha`, current iteration, commit policy, and review policy. Capture `start_sha` with `git rev-parse --short HEAD` if this is a git repo; otherwise record `(not-a-git-repo)`.
- `memory.md` — current state. Tech stack, where the goal-relevant code lives, how to run/test it, any gotchas. ~30–60 lines.
- `implementation.md` — ordered slices of work, each small enough that one specialist can finish it in one iteration. Each slice maps to one or more tickets.
- `bugs.md` — anything obviously broken you noticed in the scan. Empty is fine.
- `tickets/T-0001.md`, `T-0002.md`, … — initial tickets. Keep the first batch small (3–6 tickets); more will be proposed by agents during the loop and assigned canonical IDs by the orchestrator. Every ticket must set `proof_strategy`. Tickets that touch user-perceivable code must list `closes_stories: [STORY-NNN, …]`.
- `tickets/proposals/` — empty directory for agent-filed ticket proposals. Agents write proposals here; only the orchestrator creates canonical `T-NNNN.md` files.

**Proof strategy map (write this before dispatch):**
- For every goal checkbox, name the proof strategy that should prove it before implementation starts.
- For behavior and bug-fix checkboxes, name the failing test that should prove it first (`proof_strategy: red-green`).
- For every frontend/user-flow checkbox, name the Playwright or browser-level test that proves the visible workflow.
- For every non-UI checkbox, name the unit/integration/contract test that fails first and turns green later.
- For refactor/docs/research/tooling/performance work, name the equivalent proof (`characterization`, `rendered-doc`, `research-artifact`, `verification-only`, or `perf-baseline`).
- If a checkbox cannot be expressed as deterministic proof, attach a story and/or rubric before any code changes.

**Confidence gate map (write this before dispatch):**
- For every goal checkbox and initial ticket, name the evidence that would make the orchestrator at least 99/100 confident that:
  1. the user requirement is understood,
  2. the implementation satisfies that requirement,
  3. the implementation is verified working with no known bug in the covered scope.
- If the evidence cannot be named up front, the work is not ready for implementation. Ask for clarification, write a story/rubric, or file a `research`/`brainstorm` ticket.

**Write the stories** (only if Phase 0 identified user-perceivable checklist items):
- `stories/STORY-001.md`, `STORY-002.md`, … — one story per user-perceivable goal checkbox. The story is the spec the tickets implement. Stories are written **before** the tickets that close them.
- `stories/adapters/{functional,quant,ux}.sh` — only if the default runner needs project-specific bridges (DB driver, gate evaluator, custom dev-server boot). Skip until needed; a greenfield project starts with none.

**See `references/state-files.md` for state-file templates and `references/stories.md` for the story shape + runner contract.**

**Choose a routing profile:** Copy the profile in `references/specialty-routing.md`
that matches the current client into `.boil/routing.md`. If the client exposes
`superpowers:*` agents/skills, prefer the `superpowers-compatible` profile so
development tickets route through roles like `superpowers:test-driven-development`,
`superpowers:verification-before-completion`, `superpowers:systematic-debugging`,
`superpowers:dispatching-parallel-agents`, and
`superpowers:requesting-code-review`. If those exact agents are not available,
use the Codex or rich-agent profile and keep the same specialty names.

**Sync project agent instructions:** When the project should be usable across
Codex/Cursor/other agents, run `scripts/boil-sync-agents.py --root <project>`.
This writes `AGENTS.md`, `.cursor/rules/boil.mdc`, and `.boil/routing.md`
without overwriting existing files unless `--force` is passed.

---

## Phase 2 — The loop

Each iteration is one full pass: pick → dispatch → verify → re-test → demo → summarize → ask.

At the start of every iteration, update `.boil/run.md` with `Current iteration` and `Iteration start SHA` (`git rev-parse --short HEAD`, or `(not-a-git-repo)`). This is the diff boundary for demos, adversarial retests, and roborev review.

### Step 2a — Pick the next batch

Read `tickets/`. Pick all tickets that are:
- `status: open`
- Not blocked (`blocked_by` empty or all listed tickets are `done`)
- Reachable in this iteration (don't pick 12 — pick 1–4 you can dispatch in parallel)

Prioritize: P0 > P1 > P2 > P3, and within priority, prefer tickets whose completion would close a `goal.md` checkbox.

### Step 2b — Route and dispatch in parallel

For each picked ticket, look up `ticket.specialty` in `routing.md` to get the right platform dispatch target (`agent_type`, `subagent_type`, or local equivalent). Then dispatch all of them **in a single message with multiple subagent tool calls** so they run concurrently.

When `.boil/routing.md` uses the `superpowers-compatible` profile, treat the
superpower route as the agent's operating role:

- Implementation tickets use `superpowers:test-driven-development`.
- Verification and regression tickets use `superpowers:verification-before-completion`.
- Non-obvious failures use `superpowers:systematic-debugging`.
- Parallel batching/orchestration tickets use `superpowers:dispatching-parallel-agents`.
- Review tickets use `superpowers:requesting-code-review`.

If a named superpower route is unavailable in the current runtime, record the
fallback in `.boil/routing.md` and dispatch to the nearest available platform
agent. Do not serialize the work just because the ideal role is missing.

For each picked ticket, prefer generating a compact handoff packet first:

```bash
python3 <boil-skill-repo>/scripts/boil-dispatch-packet.py T-0001 --root <project>
```

The packet in `.boil/dispatch/` is what should be pasted or attached to a
subagent prompt. It contains only the goal slice, memory slice, ticket, proof
requirements, and return contract.

Each agent prompt must be self-contained — see `references/ticket-system.md` for the dispatch template. Critically, each agent gets:
- The ticket file path and contents
- The relevant slice of `goal.md` and `memory.md`
- Permission to **file ticket proposals** (write new `.md` files in `tickets/proposals/`) when they discover work for another specialist
- The acceptance criteria, including the demo target if any
- Instruction to return a structured report (changed files, proof/tests added/run, ticket proposals filed, blockers)

Read the ticket-system reference for the exact prompt template — getting this right is the difference between a firm and a mob.

### Step 2c — Collect, integrate, update tickets

When agents return:
1. Read each summary.
2. **Verify their changes are real** — `git status`, `git diff --stat` — don't trust the agent's word (`superpowers:verification-before-completion`).
3. Update each ticket's status (`done`, `blocked`, `in-progress`).
4. Read any files in `tickets/proposals/`, assign canonical `T-NNNN` IDs, resolve priority/specialty/proof strategy, write real ticket files, then move accepted proposals to `tickets/proposals/accepted/` (or delete rejected ones with a short note in the iteration summary).
5. Append observed defects to `bugs.md`.

### Step 2d — Verify, then re-test from a different angle

Five passes total. Pass 0 is the user-experience contract; Passes 1–2 are required for every iteration that ships code; Passes 3–4 are conditional.

**Pass 0 — Story replay (only if stories exist and tickets reference them).** For every story listed in any completed ticket's `closes_stories` field this iteration, first scan the story for `kind: rubric` assertions. If any exist, dispatch the story-rubric judges now and write their verdict files to `.boil/iterations/iter-NNN/judges/R-*.md`; missing or unparseable judge files are infra errors, not passes. Then run `scripts/story-run.sh STORY-NNN --iteration iter-NNN`. The runner replays the story end-to-end across four lanes (functional, quant, UX-mechanical, UX-rubric), folds in the judge verdicts, and updates `.boil/stories/MATRIX.md` + the story's frontmatter. If a story is still red after the iteration's code lands, the iteration is **not** done — file a `demo-prep` ticket and loop. A story that was green before and is red now is a regression — file a `regression` ticket and loop. UNCERTAIN/INDETERMINATE rubric verdicts are treated as FAIL. Full protocol in `references/stories.md`. If the project has no stories (refactor-only iteration, or stories layer not adopted), skip cleanly.

**Pass 1 — direct verification.** Run the project's own test suite, lint, type-check, build. Whatever the project actually uses. Capture exit codes and output. **No "should pass" claims.** For each completed ticket, confirm its `proof_strategy` evidence is present: RED→GREEN for behavior/bug tickets, characterization baseline for refactors, rendered output for docs, research artifact for spikes, before/after workload for performance, or verification command for tooling/deps. For frontend or browser-visible work, run the Playwright/browser-level test that maps to the goal checkbox; unit tests alone cannot close a user-visible frontend checkbox.

**Confidence audit.** For every ticket an agent returned as done, inspect its `confidence` block before closing it. `requirements_understood`, `implementation_matches`, and `verification_working` must each be `>=99`, `confidence.evidence` must list concrete artifacts/commands, and `confidence.uncertainty` must be empty. If any part fails, leave the ticket open/in-progress, file the missing-proof or clarification ticket, and do not count the goal checkbox as green.

**Pass 2 — adversarial re-test.** Pick a different angle than what the implementing agent tested. Examples:
- They wrote a unit test → you write an integration test (or vice versa).
- They tested the happy path → you test the empty input, the malformed input, the concurrency case.
- They added a UI feature → you click through it manually (or via Chrome MCP / Playwright) instead of trusting a snapshot.
- They fixed a bug → you reproduce the original symptom on the prior commit, then on HEAD, and confirm the difference.

If Pass 2 reveals new problems, file new ticket proposals and continue the loop — don't try to fix everything in one iteration. The whole point of looping is that you don't have to.

If direct verification or adversarial retest fails twice for the same ticket,
enter debugging mode before another implementation attempt:

```bash
python3 <boil-skill-repo>/scripts/boil-debug-mode.py --root <project> --iteration iter-NNN --ticket T-0001 --failure "<symptom>"
```

Route the resulting `.boil/debug/iter-NNN/T-0001-debug.md` worksheet to a
debugger/systematic-debugging specialist.

**Pass 3 — semantic judgment (only if rubrics apply).** For every goal checkbox this iteration moved or closed that has a rubric attached (inline or in `.boil/rubrics/`), dispatch a `judge` subagent in parallel with the others, context-isolated, given only the rubric + the artifacts it names + the iteration diff. Each judge writes an evidence-backed verdict to `.boil/iterations/iter-NNN/judges/R-NNN.md`. Pass → check the box. Fail → leave the box, file one ticket per failed rubric using the judge's "actionable next step" sentence as the ticket title. Indeterminate → file a `demo-prep` ticket (the work might be done, you just couldn't see it). Skip rubrics whose artifacts didn't change this iteration unless they're marked `standing: true`. **Do not route the judge to the specialty that did the implementation work** — that's the bias the rubric layer exists to avoid. Full protocol in `references/rubrics.md`.

**Pass 4 — cross-LLM review (roborev + a different reviewer).** If `roborev` is installed and the repo is initialized for it (skip cleanly otherwise), enqueue a code review on the iteration's reviewable diff using a **different LLM than the one doing implementation**. This catches the bias the implementer cannot see in its own work.

Run after Pass 1–3 have settled and `run.md` says the iteration has a reviewable diff:

1. Read `.boil/run.md` for `Iteration start SHA`, `Commit Policy`, implementation agent/model, and preferred roborev reviewer.
2. Choose a roborev reviewer that is not the implementation model family. If implementation was Codex, do **not** use `--agent codex`; prefer a configured non-Codex reviewer. If implementation was Claude, do **not** use a Claude reviewer. If no different reviewer is available, log "roborev: no different reviewer available, skipped" and file/keep a P2 `tooling` ticket.
3. If `Commit Policy` is `checkpoint-commits`, create or use the iteration checkpoint commit after verification passes, then run:

```bash
roborev review --agent <different-reviewer> --fast --wait --since <iteration_start_sha>
```

If the repo is using `user-managed-commits`, run roborev only when the installed roborev supports the available review scope (for example branch or working-tree review). Otherwise record that review was unavailable; do not claim cross-LLM review ran.

Handling the verdict:
- **Pass** → record one line in the iteration summary ("roborev <reviewer>: clean") and move on.
- **Fail with findings** → file one ticket per finding under the implementing specialty's specialist (e.g., a frontend finding becomes a ticket routed to `frontend`), priority derived from severity: Critical/High → P0, Medium → P1, Low → P2. Add a `roborev_job: <id>` field to the ticket so the next-iteration agent can comment + close the review when the fix lands. Do **not** try to fix roborev findings in the same iteration — that defeats the cross-LLM layer; let the loop handle them next cycle, exactly like Pass 2 findings.
- **Agent unavailable** (the selected different reviewer fails or is unhealthy) → log "roborev: <reviewer> unavailable, skipped" in the iteration summary; don't block the loop. If it's persistently broken, file a `tooling` ticket.

The post-commit hook (if installed) may already enqueue per-commit reviews automatically — in that case `roborev wait` first to consume the hook-fired job, then run the explicit per-iteration scope review above. Close the hook-fired job after the explicit one completes.

### Step 2d.5 — Human-action blockers

If verification or implementation is blocked by something only the user/operator can do (API keys, OAuth approval, billing setup, hardware access, domain/DNS changes, product decisions, production credentials), file or convert a `human-action` ticket before stopping.

Protocol:

1. Mark the blocker ticket `type: human-action`, `status: blocked`, `priority: P0` if it blocks the loop, and set `working_on: "blocked on user action: <safe summary>"`.
2. Fill the ticket's `human_action` block from `references/ticket-system.md`. Keep it secret-free. Never write actual keys, tokens, private account IDs, `.env` values, session cookies, or private URLs into `.boil/` state.
3. If the local ignored Susi bridge exists at `<boil-skill-repo>/.susi-human-blockers/add_blocker.py`, run it with the project root, ticket path, and safe summary so the user gets a Susi/Microsoft To Do task for the respective project.
4. When the bridge creates the To Do item and Pushover is configured locally, it also sends a Pushover notification that names the project, ticket, created To Do, and safe action summary.
5. Write the bridge result back into `human_action.susi_task_id`, `human_action.susi_sync_status` (`created`, `failed`, or `skipped`), and `human_action.pushover_status` (`sent`, `not_configured`, `failed`, or `skipped`) when available.
6. Surface the human action in the iteration summary and termination blocker report, including whether the Susi To Do and Pushover notification were created. Do not claim the loop is done; it is blocked until the user completes the action.
7. In the orientation footer, make the first `Next:` bullet the same safe human action from `human_action.safe_summary`, rewritten as an imperative. Example: "Add the missing OpenAI API key to the project environment, then say continue."

The Susi bridge itself is intentionally local and ignored by Git (`.susi-human-blockers/`). It may contain a Susi dashboard URL, session cookie, local project labels, and generated sync logs, so it must never be committed or exposed on remote GitHub.

### Step 2e — DEMO (the most important step)

Produce one user-visible artifact for this iteration. **Never skip this.** If you find yourself unable to produce one, that's a signal the work isn't actually done from the user's point of view — file a `demo-prep` ticket and continue.

**If stories cover this iteration's work, the demo IS the green story runner output.** `iterations/iter-NNN/demo.md` is generated from the MATRIX.md diff (red→green), the per-story JSON summaries, and the screenshot artifacts each green story produced. A demo without an underlying green story is forbidden for user-perceivable work — if you can't replay it via the runner, you can't claim it works.

**If frontend behavior changed, the demo must include Playwright proof.** Prefer a named Playwright test plus a screenshot/video artifact. If the repo has no Playwright setup and the work is user-visible frontend work, file the setup/test ticket before claiming the checkbox closed.

If no story applies (internal refactor, infra-only), fall back to the format guide below. Pick the demo format that fits the work — see `references/demo-formats.md` for full recipes. Quick guide:

| Work type | Demo |
|-----------|------|
| Web UI / dashboard | Start dev server, take a screenshot via Chrome MCP, give the user the localhost URL. Save screenshot to `iterations/iter-NNN/artifacts/`. |
| API / backend | Print a `curl` one-liner the user can paste, plus the actual response captured from running it. |
| CLI tool | Show the command + its real output (input → output, before/after). |
| Library / pure code | Unified diff snippet (≤30 lines) with `file:line` references, plus the test that now passes. |
| Bug fix | The failing test from before, the same test now green, and the one-line root cause. |
| Test-only work | The pass count diff (e.g., "47 → 53 passing, 4 → 0 failing") with the names of the new tests. |
| Docs | The rendered output (Markdown preview path, or pasted excerpt) of the new section. |
| Performance | Numbers from a real run: before/after, same workload. |

Write `.boil/iterations/iter-NNN/demo.md` with:
1. **What changed** — 2–4 bullets, file-level.
2. **How to see it works** — the exact action the user takes (open URL, run command, look at screenshot).
3. **Where it sits vs goal** — which `goal.md` checkboxes this closed or moved.
4. **Tests added/run** — names + the verification command + result.

### Step 2f — Iteration summary

Every iteration ends with TWO things in the chat: a tight **machine
summary** AND a **human-readable narrative** of what moved toward
the goal. The narrative is NOT optional — it is how the operator
stays oriented across a long auto-loop and the only thing a returning
human reads to catch up.

**Machine summary (~10 lines):**

```
## Iteration N

**Done this cycle:** <2-3 bullets, file-level>
**Goal progress:** <X / Y checkboxes green> — <which one(s) just turned green>
**Tests:** <added N, all green | added N, M failing — see bugs.md>. Paste the actual test stdout one-liner (e.g. `47 passed in 2.3s`), not "should be green".
**New tickets filed:** <T-00XX (frontend), T-00YY (qa)> — or "none"

**Demo (30 seconds to verify):**
→ <the single concrete action: open http://..., run `cmd`, view artifacts/iter-N/screenshot.png>

**Next focus:** <which ticket(s) next iteration will pick>
```

**Human-readable goal-progress narrative (3-6 sentences):**

```
## What changed toward the goal

<plain English, written for an outside reader (PM / risk officer /
returning dev who skipped this iter):>

- One sentence: what user-perceivable behaviour now works that didn't
  before — OR — "infrastructure-only iter, no user-visible change".
- One sentence: what fraction of the goal is now done, and which
  named checkbox(es) moved (cite by goal.md identifier).
- One sentence: what the next 1-2 iters are about and why.
- (optional) One sentence: any honest setback — codex finding, story
  regression, broken dependency. If there's nothing to flag, omit;
  don't manufacture concern.
```

Both blocks ship together. The machine summary is for grepping later;
the narrative is for human attention. Never skip the narrative even
in an auto-loop — that's the operator's only window into "where are
we, and is this still on track?".

After the narrative, include an explicit `## Suggested next steps`
block before the footer:

```markdown
## Suggested next steps

1. <best next action for the user/operator, concrete>
2. <best next ticket or verification action if continuing>
3. <optional choice: refine/pivot/stop when relevant>
```

This block is mandatory for every iteration summary, including blocked
iterations and final handoffs. It can overlap with the footer, but it gives
the main response a visible next-action section instead of relying on the
footer alone.

```
Continue, refine the goal, pivot, or stop?
```

Then append the orientation footer from "Operator orientation contract" after the summary/narrative. The footer is not a substitute for the machine summary; it is the short attention reset for the next prompt.

For mechanical gates, run:

```bash
bash <boil-skill-repo>/scripts/boil-run-iteration.sh iter-NNN <project> --test-cmd "<project test command>"
```

This runs doctor, ticket lint, story replay when stories exist, user-supplied
test commands, and iteration verification.

### Step 2g — Wait for user, or auto-loop

- If you're invoked via `/boil` or run inside a `/loop` wrapper, you can auto-continue when the user is silent and `goal.md` isn't done — but **always still emit the summary + demo and pause briefly** so the user can interrupt. The demo is the user's interrupt window.
- If the user reacts: incorporate their input into `goal.md` if they refined the goal, or into a new ticket if they pointed out a defect.

---

## Phase 3 — Termination

Stop when **any** of these are true:

1. Every checkbox in `goal.md` is checked, AND every checkbox has proof mapped to it (RED→GREEN TDD evidence, Playwright/browser proof for frontend behavior, story/rubric verdict where applicable), AND every closing ticket has `confidence.requirements_understood >=99`, `confidence.implementation_matches >=99`, `confidence.verification_working >=99`, concrete evidence, and no uncertainty, AND the most recent direct + adversarial verification both pass, AND every rubric attached to a checked checkbox has a current PASS verdict (in this iteration, or the most recent iteration that touched its artifacts), AND the user accepted the most recent demo (explicit "looks good" or equivalent).
2. The user says stop / good enough / ship it.
3. You've hit a hard blocker that no specialist can resolve without user input (e.g., needs a credential, needs a product decision). File/update a `human-action` ticket, sync it to Susi if the ignored local bridge is available, surface the blocker clearly, and stop.

On termination, write `.boil/iterations/FINAL.md` with:
- One-line goal restatement
- The final demo (URL / command / screenshot path) — pulled together so it's a single click for the user
- Index of every iteration's demo
- Test totals (added, passing)
- All open tickets (and why they're being left open)
- The git diff stats vs where you started

For production work, prefer PR-first handoff: create a branch/PR and generate
the PR body with `scripts/boil-pr-summary.py`. Do not push directly to `main`
unless the user explicitly chose that mode.

---

## Hard rules (the non-negotiables)

These exist because each one corresponds to a known failure mode of looped agentic dev work.

1. **No iteration without a demo.** If you can't demo, you can't claim progress. File a `demo-prep` ticket and loop again.
2. **No completion claims without fresh verification output in the same message.** See `superpowers:verification-before-completion`. Run the command, paste the relevant output line, then claim.
3. **Always re-test from a different angle.** A test the implementer wrote and ran is not adversarial. You must add an angle the implementer didn't.
4. **Parallel dispatch goes in one message.** Multiple subagent tool calls in a single assistant turn so they actually run concurrently. Use the current platform's dispatch API (`spawn_agent`, `Agent`, or equivalent). Sequential dispatch defeats the firm metaphor.
5. **Agents can file tickets but the orchestrator (you) routes them.** Agents propose, the orchestrator decides specialty + priority before dispatch. This prevents specialty thrash.
6. **Goal.md is sacred.** If the user wants to change scope mid-loop, edit `goal.md` first, confirm with the user, then continue. Don't silently re-interpret the goal because an agent suggested it.
7. **Honesty over progress theater.** If a cycle made no real progress (or regressed), say so plainly in the summary. The user trusts the loop only as long as the loop tells the truth.
8. **Cross-LLM review every iteration that ships code when a different reviewer is available.** Step 2d Pass 4 enqueues a roborev review with a different LLM than the implementation model. Findings become next-iteration tickets, never silently dismissed. If `roborev` isn't installed, the repo has no reviewable diff, or no different reviewer is available, skip cleanly, log the exact reason, and file/keep a `tooling` ticket when the gap is persistent. Do not claim the pass ran unless it did.
9. **User-perceivable work goes through a story.** Step 2d Pass 0 replays every story this iteration's tickets claim to close. The story is the spec written *before* the code; the runner is the only authority on "the user can actually do this." A green Playwright + selftest endpoint without a green story is not a finished feature. If the project has stories and you ship user-perceivable code without one, you broke a hard rule. (Refactor / dependency / infra work without a user surface is exempt — the ticket body must say so.)
10. **Proof-first order on every ticket.** Every ticket declares `proof_strategy` before dispatch. Behavior and bug tickets use strict RED→GREEN TDD. Refactors use characterization proof, docs use rendered-output proof, research uses an artifact, performance uses before/after numbers, and tooling/deps use an explicit verification command. Implementation comes after the pre-change proof. Then the full test suite or relevant regression set runs where it exists. No code ships with red tests; no completion claim without fresh proof output in the ticket's `Proof / tests:` field. Agents that interleave or skip their proof strategy must be re-dispatched.
11. **`working_on` is the operator's window.** Every ticket carries a `working_on:` frontmatter field that's one line and kept current at every state transition (dispatch, mid-implementation, return, close). The operator reads `working_on` across the ticket pool to answer "what is the LLM working on right now?" without diving into the iteration log. If `working_on` is stale or empty mid-`in-progress`, the orchestrator must surface the gap.
12. **Iteration summary always includes a human narrative.** Step 2f ships two blocks: the machine summary and the plain-English "What changed toward the goal" narrative (3-6 sentences). The narrative is for a returning operator catching up after an auto-loop — never skip it, even in unattended runs.
13. **Every response ends with the orientation footer and concrete next steps.** The `----------` footer is mandatory after every prompt while boil is active. Its `Next:` block must contain 1-5 concrete suggested actions, ordered by priority, and the first item must be the unblock action when a human-action ticket exists. It separates "what just happened" from "what to do next" so the operator can re-enter the loop quickly.
14. **Frontend claims need Playwright or browser-level proof.** If the goal touches a visible UI, at least one Playwright/browser test must prove the user flow before the checkbox can be checked. Manual screenshots are demos; they are not substitutes for the automated browser proof.
15. **Confirm-and-loop until proven.** Do not stop at "implemented." Loop until the proof map is green, the demo is visible, and the user accepts or explicitly stops. If proof is missing, file a ticket and continue.
16. **Human blockers become safe tasks, not leaked secrets.** If progress waits on the user, create a `human-action` ticket and sync only the safe summary to Susi through the ignored local bridge when available. Never place private credentials, tokens, session cookies, account IDs, or local-only Susi config in tracked boil files.
17. **99% confidence is an evidence gate, not a vibe.** Before a ticket or goal checkbox can be called done, the loop must be at least 99/100 confident that the requirement is understood, implemented, and verified working. That confidence must be backed by `goal.md` interpretation, ticket acceptance criteria, tests/proof output, adversarial retest, and an empty uncertainty list. If confidence is lower, continue the loop or ask the user; never round uncertainty up to done.
18. **Prefer PR-first production changes.** For production or shared repos, boil should work on a branch and produce a PR body from `.boil/` state. Direct pushes to `main` are opt-in, not the default.

---

## Reference files

Read these as you need them:

- `references/brainstorm-questions.md` — Phase 0 question set for fuzzy goals.
- `references/state-files.md` — templates for `goal.md`, `memory.md`, `implementation.md`, `bugs.md`.
- `references/ticket-system.md` — ticket schema, dispatch prompt template, agent-to-agent handoff rules.
- `references/specialty-routing.md` — the specialty → platform dispatch profile. Copy the profile matching the current client into `.boil/routing.md` at bootstrap and adapt per-project.
- `references/demo-formats.md` — recipes for producing a user-visible demo for each work type.
- `references/rubrics.md` — semantic LLM-as-judge layer: when to write a rubric, the rubric shape, how the judge subagent is dispatched (context-isolated, evidence trace required), and how verdicts feed back into tickets and termination.
- `references/stories.md` — user-experience contracts (BPM-style): one file per user-perceivable behavior, replayed end-to-end by `scripts/story-run.sh` across four lanes (functional, quant, UX-mechanical, UX-rubric). No human in the inner loop; rubric-judge handles the "feels right" check.
- `references/lsdf-codebase-index.md` — when and how to use [L-SDF](https://github.com/ec1980/lsdf-core) (`lsdf-core` on PyPI) to maintain a compact `INDEX.lsdf` of the repo so subagent dispatch contexts navigate the codebase by index rather than full file reads (~13× cheaper on Python repos). Read this if dispatch context is the cost driver of your loop.

---

## Integration with other skills

`boil` orchestrates — it doesn't replace specialist skills.

- **Brainstorming** (`superpowers:brainstorming`) — Phase 0 uses this kind of inquiry; if the goal is genuinely greenfield/creative, invoke it explicitly before writing `goal.md`.
- **Verification before completion** (`superpowers:verification-before-completion`) — applies to every "done" claim inside the loop.
- **Dispatching parallel agents** (`superpowers:dispatching-parallel-agents`) — the mechanics for Step 2b are exactly this; read it if your dispatches feel off.
- **Systematic debugging** (`superpowers:systematic-debugging`) — when an iteration's verification reveals a non-obvious failure, route a ticket to a debugger agent who follows that skill.
- **TDD** (`superpowers:test-driven-development`) — when a ticket adds new behavior, prefer red-green-refactor; the adversarial re-test (Step 2d) is the green-side check.
- **ADHD-friendly orientation** (inspired by `ayghri/i-have-adhd`) — action-first updates, visible state, short next-step bullets, and no tangents. Boil encodes this as the mandatory `----------` footer after every response.
- **roborev cross-LLM review** — Step 2d Pass 4 calls `roborev review --agent <different-reviewer> --fast --wait` to have a different LLM critique the iteration's code. Findings become tickets, not in-place edits. Outside of boil, the equivalent self-driven loop is `/milestone-review`; the user-driven version is `/roborev-refine`.
- **Superpowers-compatible agents** — when available, route development through `superpowers:test-driven-development`, verification through `superpowers:verification-before-completion`, debugging through `superpowers:systematic-debugging`, parallel batching through `superpowers:dispatching-parallel-agents`, and review through `superpowers:requesting-code-review`. These are role contracts; if the runtime lacks those agents, use the nearest local subagent and keep the same proof/return requirements.
- **Loop / schedule** — `/loop` can wrap `boil` for unattended runs; `/schedule` can run `boil` on a recurring basis (e.g., nightly maintenance loops).
- **L-SDF codebase index** ([`lsdf-core`](https://pypi.org/project/lsdf-core/)) — when present, boil uses `lsdf gen . --recursive` at bootstrap and `lsdf sync --check` per iteration to keep a compact index alongside source. Subagent dispatch contexts then point at the index rather than asking each agent to grep / read source from scratch. ~13× compression on Python repos; non-Python repos currently skip. Full protocol in `references/lsdf-codebase-index.md`.

---

## Quick mental model

> *Each loop is one work-day at a small firm. The orchestrator is the team lead. Specialists pick up tickets and ship in parallel. QA re-tests from a different angle. At end of day, the team lead does a 30-second demo to the customer (you), updates the board, and goes home — until tomorrow's loop, when the cycle starts again. The firm closes when the customer signs off on the demo.*

---
> Source: [trbck/boil](https://github.com/trbck/boil) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
