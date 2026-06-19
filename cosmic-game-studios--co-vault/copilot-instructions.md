## co-vault

> >


# co-vault — agent operating instructions (v0.7, scientific loop + structured reasoning)

You operate against up to two self-describing vaults:

- **Project vault** (`$COVAULT_PATH`) — per-project facts, decisions,
  proposals, reports, conflicts. Loaded per task.
- **Person vault** (`$COVAULT_PERSON`) — durable knowledge about the user
  across all their projects. Cross-agent. Loaded once per session.

The loop has 6 phases. Each phase maps to a documented function in
cognitive science and writes structured data that feeds the next.

Follow these instructions literally. Do not improvise. If you ever lose
track of which phase you are in, restart from PHASE 1.

## CONVENTIONS — applied to both vault types

- **Self-description first**: read `.covault/manifest.yaml` once per
  session per vault. Read `.covault/schemas/<type>.md` and the matching
  example before writing any note of that type for the first time.
- **Timestamp**: `$(date -u +%Y-%m-%dT%H:%MZ)`.
- **Slug**: 2–4 words, lowercase, hyphenated.
- **Filename**: `<YYYY-MM-DD-HHMM>-<slug>.md`. Append `-2`, `-3`, ... on collision.
- **Path hygiene**: never read or write outside the vault paths during
  the loop, except project code in PHASE 3.
- **Phase announcement**: print `[co-vault: PHASE <N>/6 — <NAME>]` at the
  start of every phase. Non-negotiable.
- **Commit after every write**: `git -C <vault> add . && git -C <vault> commit -q -m "<msg>"`.
- **Token efficiency**: never bulk-load a vault. Always go through indexes
  and named files.
- **Run maintenance after CONSOLIDATE**: `bin/maintain-vault.sh <vault>`.

## ACTIVATION CHECK — run first, every session

```bash
PROJECT_VAULT_OK=0
if [ -n "${COVAULT_PATH:-}" ] && [ -f "$COVAULT_PATH/.covault/manifest.yaml" ]; then
  SV=$(grep -E '^schema_version:' "$COVAULT_PATH/.covault/manifest.yaml" | awk '{print $2}')
  case "$SV" in
    1|2) PROJECT_VAULT_OK=1 ;;
    *) echo "co-vault: project vault schema_version=$SV — refusing" ;;
  esac
fi

PERSON_VAULT_OK=0
if [ -n "${COVAULT_PERSON:-}" ] && [ -f "$COVAULT_PERSON/.covault/manifest.yaml" ]; then
  SV=$(grep -E '^schema_version:' "$COVAULT_PERSON/.covault/manifest.yaml" | awk '{print $2}')
  case "$SV" in
    1|2) PERSON_VAULT_OK=1 ;;
    *) echo "co-vault: person vault schema_version=$SV — refusing" ;;
  esac
fi

[ "$PROJECT_VAULT_OK" = "0" ] && [ "$PERSON_VAULT_OK" = "0" ] && \
  echo "co-vault: no vaults active. Skill is dormant."
```

If the project vault is active, run the 6-phase loop on every task.
If the person vault is active, run SESSION START once before any task.

## AUTHORITY RULES — apply to both vault types, non-negotiable

| author value      | your permitted operations                                        |
|-------------------|------------------------------------------------------------------|
| `user`            | READ, CITE via `[[wikilink]]`. NEVER write, edit, move, archive. |
| `agent+reviewed`  | READ, CITE. NEVER write or edit. (auto-promoted from `agent`)    |
| `agent`           | READ, WRITE, EDIT, SUPERSEDE, ARCHIVE.                           |
| (no author field) | TREAT AS BROKEN. Report to user.                                 |

In project vaults, default author for new notes:
- `user` for decisions, domains, index
- `agent` for proposals, reports, facts, conflicts

In person vaults, default author for new notes is `agent`.

## SCHEMA LOOKUP — before every write

Before writing a note of type `T` for the first time in a session:
```bash
cat "<vault>/.covault/schemas/$T.md"
cat "<vault>/.covault/examples/$T.md"
```
Match the schema. Use the example as a template.

---

## SESSION START — only if COVAULT_PERSON is active

Run ONCE per session, before any task. Loads durable knowledge about the
user without bulk-loading the entire vault.

```bash
cd "$COVAULT_PERSON"
cat .covault/manifest.yaml          # know the schemas
cat _index.md                        # one-line summaries of every note
find corrections -type f -name '*.md' -not -name '.gitkeep' \
  -exec sh -c 'echo "=== $1 ==="; cat "$1"; echo' _ {} \;
[ -f identity/basic.md ] && { echo "=== identity/basic.md ==="; cat identity/basic.md; }
```

That is all the bulk loading. Everything else is fetched on demand based
on the index.

**Token budget rule**: total person vault overhead per session must stay
under ~3000 tokens. If exceeded, run REVIEW and prune.

---

## CALIBRATION AWARENESS — read on every session

If a project vault is active, read its calibration log:
```bash
[ -f "$COVAULT_PATH/calibration_log.md" ] && cat "$COVAULT_PATH/calibration_log.md"
```

This file (auto-maintained by `maintain-vault.sh`) shows your
historical prediction accuracy. Use it to calibrate the confidence
values you assign in PHASE 2 HYPOTHESIZE. If past predictions at "90%"
were only correct 70% of the time, adjust this session's "90%" downward.

---

# THE 6-PHASE LOOP

```
PHASE 1   PHASE 2          PHASE 3   PHASE 4   PHASE 5      PHASE 6
ORIENT  → HYPOTHESIZE    → EXECUTE → VERIFY  → CONSOLIDATE → REVIEW
                                                             (only if conflict)
perception generative      action    prediction memory       conflict
           model with                error      consolidation resolution
           predictions               checking
```

Each phase has a documented function in cognitive science. Each phase
writes structured data that feeds the next. You will run all 6 phases.
You will not collapse phases. You will not skip CONSOLIDATE.

---

## PHASE 1 — ORIENT
*Function: situated perception. Build a model of the current knowledge state.*
*Enhanced with: Deep Read protocol (DeepResearch R1).*

Announce: `[co-vault: PHASE 1/6 — ORIENT]`

### Step 1a — Load context (unchanged)

```bash
cd "$COVAULT_PATH"
cat index.md
DOMAINS="<inferred from user request, space-separated>"

# Read domain notes for each domain touched
for D in $DOMAINS; do
  [ -f "domains/$D.md" ] && { echo "=== domains/$D.md ==="; cat "domains/$D.md"; echo; }
done

# Pull user-authored notes in those domains
for D in $DOMAINS; do
  find decisions facts -type f -name '*.md' 2>/dev/null | while read f; do
    grep -qE '^author:[[:space:]]*user[[:space:]]*$' "$f" \
      && grep -qE "domain:.*$D" "$f" \
      && { echo "=== $f ==="; cat "$f"; echo; }
  done
done

# Check for OPEN conflicts in those domains
for D in $DOMAINS; do
  find conflicts -type f -name '*.md' 2>/dev/null | while read f; do
    grep -qE '^status:[[:space:]]*open[[:space:]]*$' "$f" \
      && grep -qE "domain:.*$D" "$f" \
      && echo "OPEN CONFLICT: $f"
  done
done
```

### Step 1b — Check anti-patterns (new — DeepResearch knowledge base)

Before proposing anything, check for previously failed approaches:
```bash
# Search for anti-patterns in this domain
for D in $DOMAINS; do
  find facts -type f -name '*.md' 2>/dev/null | while read f; do
    grep -qE '^pattern_type:[[:space:]]*anti-pattern' "$f" \
      && grep -qE "domain:.*$D" "$f" \
      && { echo "⚠ ANTI-PATTERN: $f"; cat "$f"; echo; }
  done
done
```

If anti-patterns exist for the current domain, factor them into your
reasoning. Do NOT repeat an approach that has already been recorded as
an anti-pattern unless the user explicitly requests it.

### Step 1c — Deep Read (new — structured reasoning before action)

After loading context, perform structured reasoning BEFORE moving to
PHASE 2. Print this analysis in your response (not in a vault file):

1. **Problem decomposition** — Break the task into component parts.
   What are the independent sub-problems?
2. **Constraint inventory** — What user decisions, domain rules, and
   anti-patterns constrain the solution space?
3. **Causal hypothesis** — What is the root cause or core mechanism?
   Why does this task need to be done? What specific bottleneck or gap
   does it address?
4. **Approach candidates** — List 2-3 possible approaches with
   trade-offs. Do NOT commit to one yet; that happens in PHASE 2.

This is a reasoning step, not a writing step. It costs ~200 tokens
and prevents the "first idea = only idea" failure mode.

### Step 1d — Person vault cross-reference

ALSO, if `COVAULT_PERSON` is active, opportunistically scan the person
vault index for hits on the current task's domains:
```bash
[ -n "${COVAULT_PERSON:-}" ] && grep -iE "($(echo $DOMAINS | tr ' ' '|'))" \
  "$COVAULT_PERSON/_index.md" 2>/dev/null
```
For any hit, `cat` that specific file.

**Stopping conditions:**
- Open conflict in domain → STOP, ask user.
- User-authored note contradicts the task → STOP, quote it, ask user.
- Anti-pattern directly applies to the only viable approach → WARN user.

---

## PHASE 2 — HYPOTHESIZE
*Function: build a generative model with explicit, testable predictions.*
*Science: predictive coding (Friston 2010); active inference.*
*Enhanced with: informed mutation selection (DeepResearch L1.5).*

Announce: `[co-vault: PHASE 2/6 — HYPOTHESIZE]`

Read schema and example:
```bash
cat "$COVAULT_PATH/.covault/schemas/proposal.md"
cat "$COVAULT_PATH/.covault/examples/proposal.md"
```

Write `proposals/<timestamp>-<slug>.md` matching the schema. Critically:

### Change type classification (new — DeepResearch mutation types)

Every proposal MUST classify its `change_type` in frontmatter. This
determines the safety rails and review requirements:

| change_type              | level | risk   | description                                    |
|--------------------------|-------|--------|------------------------------------------------|
| `parametric`             | 1     | low    | Change a value, config, or threshold           |
| `structural_addition`    | 2     | medium | Add new function, module, endpoint             |
| `structural_removal`     | 2     | medium | Remove dead code or unnecessary complexity     |
| `structural_replacement` | 2     | high   | Replace one implementation with a better one   |
| `integration`            | 2     | medium | Connect two existing components                |
| `architectural`          | 3     | high   | Design and build a new component from scratch  |

**Risk determines confirmation behavior:**
- `low` risk → proceed if `estimated_effort: small`
- `medium` risk → proceed if `estimated_effort: small`, confirm if `medium`
- `high` risk → ALWAYS confirm, regardless of estimated effort

### Causal hypothesis (new — DeepResearch reasoning layer)

The `## Hypothesis` section is now REQUIRED alongside `## Predictions`.
It must contain:
- **What** you believe the root cause or core mechanism is
- **Why** you believe this (citing evidence from ORIENT)
- **What would change your mind** (falsification condition)

This is distinct from predictions. Predictions are measurable outcomes;
the hypothesis is the causal model that generates those predictions.

### Alternative approaches (new — explore before committing)

The `## Alternatives considered` section is now REQUIRED. List at
least one other approach you considered and why you rejected it.
Reference anti-patterns from ORIENT if they ruled out an approach.

### Predictions (unchanged requirements, enhanced calibration)

- **`## Predictions` is required.** Minimum 3 predictions, each with a
  confidence number (0-100%) and a clearly testable claim.
- Use the calibration log (loaded earlier) to ground confidence values.
  If your historical "80%" predictions were only correct 60% of the time,
  use 60% this time.
- **Per-domain calibration**: if the calibration log has domain-specific
  accuracy data, use the domain-specific accuracy to calibrate, not the
  global average.
- Testable means: there is a clear way to mark each prediction as
  correct / partial / wrong / untestable in the matching report.

Commit:
```bash
git -C "$COVAULT_PATH" add . && git -C "$COVAULT_PATH" commit -q -m "hypothesize: <slug>"
```

Print the proposal path. Apply confirmation rules based on BOTH
`estimated_effort` AND `change_type` risk level (see table above).

---

## PHASE 3 — EXECUTE
*Function: motor action. Touch project code.*

Announce: `[co-vault: PHASE 3/6 — EXECUTE]`

Do the actual work in project code (NOT in vaults).

Rules:
1. Stay strictly inside the proposal's `Out of scope` constraints.
2. Reference the proposal in commit messages on the project repo:
   `<type>(<scope>): <subject>  [co-vault: <proposal-filename>]`
3. If you discover a contradiction with a `user` note, STOP and jump
   straight to PHASE 6.
4. If the user says "abort", jump to PHASE 4 with `status: aborted`.
5. **Track prediction-relevant data while you work**: time elapsed,
   files touched, tests passing/failing. You will need this in PHASE 4.

---

## PHASE 4 — VERIFY
*Function: prediction error checking. Compare each prediction to reality.*
*Science: predictive processing; Bayesian updating.*
*Enhanced with: reflection protocol (DeepResearch R3).*

Announce: `[co-vault: PHASE 4/6 — VERIFY]`

### Step 4a — Prediction verdicts (unchanged)

For each prediction in the matching proposal, mark it as:
- `correct` — prediction matched reality
- `partial` — directionally right but off in degree
- `wrong` — contradicted by reality
- `untestable` — no evidence either way (rare; usually means a bad prediction)

These verdicts go into the report's `## Verification` section in PHASE 5.

**Be honest.** Marking a wrong prediction as "partial" to make yourself
look good corrupts the calibration log and degrades all future planning.

### Step 4b — Hypothesis verdict (new)

Evaluate the causal hypothesis from the proposal:
- `confirmed` — the mechanism worked as theorized
- `partially_confirmed` — the mechanism was part of the answer but not all
- `refuted` — the root cause was something else entirely

This goes into the report's `## Hypothesis verdict` section.

### Step 4c — Reflection (new — DeepResearch R3 protocol)

After verdicts, perform structured reflection. Print this analysis in
your response AND include a summary in the report's `## Reflection`:

1. **What surprised you?** — Any outcomes you did not predict at all.
2. **Causal analysis of errors** — For each wrong/partial prediction,
   WHY was it wrong? Not just "I was wrong about X" but "I was wrong
   because I assumed Y, which turned out to be false because Z."
3. **Model update** — What has changed in your mental model of this
   domain? What will you predict differently next time?
4. **Anti-pattern candidates** — Did you discover an approach that
   should NEVER be tried again in this domain? If yes, record it as
   an anti-pattern fact in PHASE 5.

This reflection costs ~150 tokens and is the primary learning signal
that makes future tasks in the same domain more accurate.

---

## PHASE 5 — CONSOLIDATE
*Function: write to memory. Update calibration. Promote stable facts.*
*Science: Complementary Learning Systems (McClelland 1995); memory consolidation.*

Announce: `[co-vault: PHASE 5/6 — CONSOLIDATE]`

Read schemas:
```bash
cat "$COVAULT_PATH/.covault/schemas/report.md"
cat "$COVAULT_PATH/.covault/examples/report.md"
cat "$COVAULT_PATH/.covault/schemas/fact.md"        # if creating new facts
```

**Step 5a — Write the report.** Filename: `reports/<same-name-as-proposal>.md`.
Match the schema. Include the `## Verification` section with verdicts from
PHASE 4. Set `predictions_correct`, `predictions_partial`, `predictions_wrong`
in frontmatter.

**Step 5b — Write any new facts.** For each genuinely new piece of
knowledge from PHASE 3, write a separate atomic file in `facts/`. One
claim per file. Set `confirmation_count: 1` and `last_confirmed: <now>`.

**Step 5b+ — Write anti-pattern facts (new — DeepResearch knowledge base).**
If PHASE 4 reflection identified an approach that should never be repeated,
write it as a fact with `pattern_type: anti-pattern` in frontmatter:

```yaml
---
type: fact
author: agent
domain: <domain>
pattern_type: anti-pattern
created: <now>
discovered_in: "[[reports/<matching-report>]]"
confidence: high
confirmation_count: 1
last_confirmed: <now>
---

## Claim
<approach> does not work in <context> because <reason>.

## Evidence
<what happened when it was tried>

## Instead
<what worked instead, or what should be tried>
```

Anti-patterns are loaded in PHASE 1 (Step 1b) to prevent the agent
from repeating known-bad approaches.

**Step 5c — Re-confirm existing facts.** If during PHASE 3 you observed
something that confirms an existing `agent`-authored fact, update its
`confirmation_count` (increment) and `last_confirmed` (now). Do NOT
duplicate the fact.

**Step 5d — Commit.**
```bash
git -C "$COVAULT_PATH" add . && git -C "$COVAULT_PATH" commit -q -m "consolidate: <slug>"
```

**Step 5e — Run auto-maintenance.** This is mandatory:
```bash
"$COVAULT_REPO/bin/maintain-vault.sh" "$COVAULT_PATH"
```
(`COVAULT_REPO` defaults to `~/.claude/skills/co-vault`.)

The maintainer will:
- Decay confidence on stale notes
- Promote facts with `confirmation_count >= 3` to `agent+reviewed`
- Archive notes past `valid_until`
- Recompute the calibration log
- Validate the vault and abort if anything is malformed

If `COVAULT_PERSON` is active, also run **PERSON LEARNING** (next section)
and then run maintain on the person vault too.

If no contradiction was found, announce:
`[co-vault: PHASE 6/6 — skipped, no conflict]`
The loop is done.

---

## PERSON LEARNING — runs inside CONSOLIDATE if COVAULT_PERSON is active

Ask yourself four questions:

1. **Did the user correct me on something during this task?**
   → Write `corrections/<topic>.md` matching the `correction.md` schema.
2. **Did I observe a stable preference I haven't recorded?**
   → Check `_index.md` for a matching preference. If exists, update
   `last_confirmed`. If not, write `preferences/<topic>.md`.
3. **Did I observe a behavioral pattern (3+ instances)?**
   → Same: check, update or create `patterns/<topic>.md`.
4. **Did the person's life/work context change?**
   → Update or create `context/<topic>.md`.

**Strict criteria for any new note** — all four must be true:
- **Durable** (not session-specific)
- **Non-obvious** (not "user uses a computer")
- **Useful** (would change agent behavior in the future)
- **Not duplicate** (search the index first)

After any write, run:
```bash
"$COVAULT_REPO/bin/maintain-vault.sh" "$COVAULT_PERSON"
```

If you can't decide whether something is worth recording: don't.

---

## PHASE 6 — REVIEW (only if contradiction found)
*Function: cognitive control. Resolve a clash between observation and ground truth.*

Announce: `[co-vault: PHASE 6/6 — REVIEW]`

Trigger conditions (any of):
- A `user`-authored note states X and you observed not-X.
- A `user`-authored note forbids approach Y and the task requires Y.
- Two `user`-authored notes contradict each other.

When triggered:
1. Stop all work in the affected domain immediately.
2. Read schema:
   ```bash
   cat "$COVAULT_PATH/.covault/schemas/conflict.md"
   ```
3. Write `conflicts/<timestamp>-<slug>.md` matching the schema.
4. Commit.
5. Print conflict path.
6. State: "I have stopped work on this task. Please resolve the conflict
   and tell me how to proceed."
7. Do NOT continue. Do NOT run CONSOLIDATE on a conflicted task. Do NOT
   work on anything else in the same domain.

---

# AUTONOMOUS MODE

When the user says "autonomous: <intent>" or "fire and forget: <intent>"
or "run autonomously and report back: <intent>".

## Hard limits — non-negotiable
- **Must run on a feature branch.** If you are on `main`, create one
  named `co-vault/<timestamp>-<slug>` first. Refuse to start if a clean
  branch can't be created.
- **Maximum 5 sub-tasks per autonomous run.**
- **Hard token budget: 30000 tokens per run.** Track and abort if exceeded.
- **Any conflict in any sub-task immediately exits autonomous mode** and
  drops back to interactive. Do not continue with other sub-tasks.
- **Any out-of-scope discovery exits autonomous mode.**
- **Final report is mandatory.**

## Procedure

1. **Create branch** if not on one:
   ```bash
   BRANCH="co-vault/$(date -u +%Y%m%d-%H%M)-<slug>"
   git checkout -b "$BRANCH"
   ```

2. **Write a master plan** to `proposals/<timestamp>-autonomous-<slug>.md`
   listing every sub-task, with `type: master_proposal` in frontmatter.
   Each sub-task gets its own one-line entry with effort estimate.

3. **Wait for ONE user confirmation of the master plan.** This is your
   only checkpoint before running autonomously. Do not proceed without
   explicit "yes" or equivalent.

4. **For each sub-task in order**, run the full 6-phase loop:
   - Auto-confirm `small` and `medium` proposals (NOT `large`).
   - Treat any conflict, any out-of-scope discovery, any `large` effort
     as an exit signal.
   - Track cumulative tokens. If approaching budget, exit.

5. **Final report**: write `reports/<timestamp>-autonomous-<slug>.md`
   summarizing every sub-task that ran, its result, every file in
   project code that was changed, every vault file that was written,
   any sub-tasks that were skipped due to exit conditions.
   End the report with exactly:
   > Review the branch `<branch-name>` and decide: merge, fix, or discard.

6. **Stop and notify the user.** Do not auto-merge. Do not run further
   tasks until the user reviews.

## Why these limits exist
- The branch limit prevents runaway changes from polluting `main`.
- The sub-task limit prevents the agent from getting lost in a long chain
  where prediction errors compound.
- The token budget prevents pathological loops.
- The conflict-exit prevents the agent from working around your decisions
  in autonomous mode where you're not actively watching.

If the user pushes back ("just do it, ignore the limits"), refuse and
explain that autonomous mode without these guards is a footgun.

---

# HARD RULES — violating any of these is a failure of the skill

1. **Never edit a file with `author: user`.** Open a conflict instead.
2. **Never edit a file with `author: agent+reviewed`.** Same rule.
3. **Never delete a file.** Move to `_archive/` with a top-comment reason.
4. **Never write a note without reading its schema first** in the current session.
5. **Never write a note without complete frontmatter.**
6. **Never write a proposal without `## Predictions`** containing at least
   3 testable predictions with confidence values.
7. **Never write a proposal without `## Hypothesis`** containing a causal
   model with a falsification condition.
8. **Never write a proposal without `## Alternatives considered`** listing
   at least one rejected approach.
9. **Never write a proposal without `change_type`** in frontmatter.
10. **Never write a report without `## Verification`** marking every
    prediction from the matching proposal.
11. **Never write a report without `## Hypothesis verdict`** evaluating
    the causal hypothesis from the matching proposal.
12. **Never write a report without `## Reflection`** containing causal
    error analysis and model updates.
13. **Never write more than one claim per `facts/` file.**
14. **Never write more than one topic per file** in person vault.
15. **Never silently supersede.** Use `superseded_by:` and move old to `_archive/`.
16. **Never auto-consolidate semantic content.** Only the deterministic
    `bin/maintain-vault.sh` may modify confidence, promotion, archival.
17. **Never proceed past an open conflict in the affected domain.**
18. **Never copy text between notes.** Use `[[wikilinks]]`.
19. **Never read or write outside the vaults** during the loop, except
    project code in PHASE 3.
20. **Never skip the phase announcement.**
21. **Never operate on a vault with mismatched `schema_version`.**
22. **Never bulk-load the person vault.**
23. **Never duplicate a note.** Search the index first; update if exists.
24. **Never inflate prediction verdicts.** Mark wrong predictions as
    wrong, even if it makes the calibration log look bad.
25. **Never enter autonomous mode without a feature branch.**
26. **Never repeat a recorded anti-pattern** without explicit user approval.
27. **Never skip Deep Read** (Step 1c) in ORIENT. The reasoning step is
    mandatory, not optional.

---

## ABORT command — when the user says "abort", "stop", "cancel"

1. Stop PHASE 3 immediately.
2. Jump to PHASE 5 (CONSOLIDATE) with report `status: aborted`.
3. In `## What actually happened`, document what was done, what was
   left undone, what needs cleanup.
4. Mark all open predictions as `untestable`.
5. Commit. Run maintain-vault. Stop.
6. Do NOT run PERSON LEARNING after an abort.

"Abort" is not "pause". Once aborted, the loop is over.

---

## REVIEW command — only when the user explicitly says it

Trigger phrases: "review the vault", "co-vault review", "vault status".

Run `bin/maintain-vault.sh` first (it includes validation), then present
the maintenance output as numbered action items. Do NOT make decisions
on archive / promote / fix yourself. The user decides.

---

## BOOTSTRAP — when vaults are uninitialized

Tell the user this exact instruction:

> To bootstrap a project vault:
>
> ```bash
> ./install.sh "$COVAULT_PATH"
> ```
>
> To bootstrap a person vault:
>
> ```bash
> ./install.sh --person "$COVAULT_PERSON"
> ```
>
> After bootstrap, edit `index.md` (project) or `identity/basic.md`
> (person) before starting work.

Wait for the user to confirm bootstrap is complete.

---

## END OF SKILL

If you reached this point without violating any hard rule, you are
operating co-vault correctly. If uncertain which phase you are in,
restart from PHASE 1 — re-running ORIENT is cheap; acting on stale
context is expensive.

The whole point of this skill is to make agent behavior predictable,
auditable, and improvement-trackable over time. Every prediction you
make goes into the calibration log. Every fact you observe goes into
consolidation. Every conflict you find blocks work until the human
decides. Nothing happens silently. Nothing happens without a record.

That is the contract.

---
> Source: [Cosmic-Game-studios/co-vault](https://github.com/Cosmic-Game-studios/co-vault) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
