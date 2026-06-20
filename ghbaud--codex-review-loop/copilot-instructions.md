## codex-review-loop

> Use when Claude wants an adversarial review, design critique, plan critique, or second-opinion debugging from Codex AND wants to engage with the findings rather than receive a one-shot report. Sets up an Epic bead, asks Codex to file each finding as a child bead, then runs a back-and-forth where Claude responds to each finding, Codex responds to Claude, and so on until both agree (or the user decides to stop). Trigger this skill whenever you are about to dispatch /codex:codex-rescue for review, critique, design feedback, or a second-opinion pass; whenever the user asks for a "codex review loop", "iterative codex review", "back-and-forth with codex", "argue with codex", or similar; or whenever you want the durability of a written record across rounds. Skip for one-shot reviews where no rebuttal is wanted, for implementation work, or in projects without bd.


# Codex Review Loop

This skill runs a structured back-and-forth between Claude and Codex on a review task. Findings live in beads. Each finding has its own thread of comments, prefixed `[claude]` or `[codex]` so the conversation is auditable. The loop continues until every finding is closed by agreement, or until the user steps in to break a deadlock.

**Always use a dedicated wrapper review epic for the findings; never flatten findings into an implementation epic.** When the work being reviewed has its own implementation epic, parent the review epic under it -- reopening the implementation epic first if it has been closed. See Step 1 for the rule, the edge case exception, and the verified reopen behavior.

## Terminology

Two words in this skill describe two different scopes. They are not interchangeable, and Claude must keep them separate in user-facing messages too.

- **Loop** (also **review loop**) -- one full invocation of this skill, from creating the review epic through closing it. A single conversation may contain several loops, one per review subject.
- **Round** -- one Codex dispatch plus Claude's responses to the findings, within a single loop. A loop usually runs 2 to 4 rounds before all findings close.

When reporting progress, use "loop" for the invocation and "round" for the dispatch turn. If a sentence could be read either way, name both: "round 2 of this review loop".

Good:
- "Starting a codex review loop on the websocket plan."
- "Round 2 dispatched -- 3 findings still open."
- "Loop complete. 5 findings: 3 agreed, 2 rejected with reasoning."

Bad:
- "Starting round 1 of the review" when you mean the loop has just begun.
- "The previous round found 5 issues" when you mean an earlier loop on a different subject.

## Reference files

When you need details that are not in this body, load the matching reference file. Do not paraphrase from memory; the references contain exact text that has been hardened against real failures.

- `references/wording-rules.md` -- read BEFORE composing any dispatch wrapper. The rescue subagent strips `--write` if the wrapper contains certain phrases, and `--write` off means Codex cannot write to bd. Contains the trigger phrases, the wrapper template, where the source-files boundary goes, and the post-dispatch verification step.
- `references/polling.md` -- read when you need to start polling a Codex job. Contains the companion path definition, the bash polling script, the four exit conditions, and why background `Bash` beats `Monitor` for this.
- `references/cancellation.md` -- read when you need to cancel a stuck job. The default cancel can fail on Windows; this file documents the recovery path and the stuck-queued-job edge case.

## Status: actively maintained

The protocol mechanics are stable: tested end-to-end three times, then used on real reviews that all converged. The documentation is NOT stable -- every real run so far has produced at least one meaningful skill change. Expect that pattern to continue.

If anything in this protocol fails or behaves unexpectedly -- a bd command errors out, Codex misuses a flag, the polling script does not detect a state, the parse misses a section, you find yourself rationalizing a stop, anything -- surface it to the user immediately AND fold the lesson back into this file (the "Known failure modes" section near the end). Do NOT silently work around the issue. The user is iterating on this skill and needs to see real-world friction so the next revision can address it.

## Why this exists

A single Codex rescue gives you findings without a way to push back. Some findings are valuable, some are wrong, some are based on misreadings. This skill lets Claude examine each finding, push back where appropriate, and let Codex defend or concede. The result is a durable record of what was reviewed, what was decided, and why.

The pattern only pays off when the value is in the dialogue. Use it for adversarial reviews and design critiques. Do not use it for "fix this bug" or "add this feature" -- those are implementation tasks for `/codex:codex-rescue` directly.

## Design rule: symmetric bd ownership

Each speaker writes its own bd output. Codex writes Codex's findings, Codex's response comments, and closes beads when Codex agrees. Claude writes Claude's response comments and closes beads when Claude agrees. Neither side speaks for the other.

## Identity prefixes

Every comment in the loop starts with one of:
- `[claude]` -- written by Claude
- `[codex]` -- written by Codex

The dispatch prompt instructs Codex to prefix every comment with `[codex]`. Claude prefixes its own comments with `[claude]`. The prefix is the only way the audit trail stays readable across rounds. If a Codex comment arrives without the prefix, Claude notes the omission in its next response on that finding but does NOT edit Codex's comment.

## Prerequisites

Before starting, verify:
- Current directory is a bd-tracked project (`.beads/` exists, or a bd-tracked parent does).
- The `codex:codex-rescue` subagent type is available via the Agent tool. Do NOT invoke `/codex:rescue` via the Skill tool -- in testing, the Skill-tool dispatch hung in "Initializing..." indefinitely.
- The user has confirmed they want this loop. Each round costs real Codex tokens; the user should know.

## Dispatch path

Always use the Agent tool with `subagent_type: "codex:codex-rescue"`. Always include `--background --fresh` in the prompt:
- `--background` gives Claude a job ID and a deterministic polling target. The rescue subagent's auto-selection picks background for complex tasks anyway.
- `--fresh` skips the resume-prompt that the rescue agent otherwise asks via AskUserQuestion.

**Two-layer dispatch -- the rescue subagent is NOT the Codex job.** There are two separate background processes:

1. The rescue subagent is the wrapper Claude invokes via the Agent tool. Its only job is to dispatch the Codex companion and exit. It usually finishes in 30-90 seconds.
2. The Codex job is the actual review work. It runs inside the codex-companion process and is identified by an ID like `task-XXXXX-XXXXX`. It typically takes 3-10 minutes for a real review.

The rescue subagent's return value contains a sentence like `Codex Task started in the background as task-XXXXX-XXXXX.` That `task-XXXXX-XXXXX` is the Codex job ID. The actual review work runs against THAT id and must be polled separately (see `references/polling.md`).

**Do NOT pass `run_in_background: true` to the Agent tool.** Dispatch is fast (under two minutes), and running the rescue subagent in the foreground means Claude blocks until the Codex job ID is in hand and can immediately start the background Bash poll against it. Background mode on the Agent tool just adds a confusing intermediate "completed" notification that does not mean what it sounds like (see "User-facing messaging confusion" in Known failure modes).

The Codex sandbox cannot reliably call `bd show` mid-run. A prefetch hook (`~/.claude/hooks/codex-prefetch-bd.py`) intercepts dispatches that name a bd ID and forces Claude to inline the bead's content. To skip the prefetch when Claude has already inlined everything, include `<bead_context>SKIP</bead_context>` in the prompt.

**Codex CAN read the repo directly.** It has full file-system read access to the project tree. Do NOT inline source code into the dispatch prompt -- give file paths plus line ranges (e.g., `services/wheelhouse/integrations/websocket_manager.py:256-301`) and let Codex open the file. Inline only material Codex cannot otherwise see: bd state, files outside the repo, conversation context (the current task, the design under review, the constraints).

**Pre-stage large prompts to a file.** When the dispatch prompt is large (rough threshold: more than about 10 KB of inlined content), do NOT pass the full content as the Agent tool's `prompt` argument. Write the full round prompt to a stable path under `C:/tmp/` (e.g., `C:/tmp/codex-round-<N>-prompt.md`), then pass a short wrapper that tells Codex to read that file. The wrapper template lives in `references/wording-rules.md` -- use it verbatim, because minor word changes can strip `--write`. For small prompts (single-paragraph round-N follow-ups, or quick re-dispatches), inline is still fine. See "Large inline prompt rewritten" in Known failure modes for the failure this prevents.

## Project-specific review criteria

Some projects have standing concerns that every review should evaluate. Those concerns live in a project-level skill named `project-review-criteria`, located at `<project>/.claude/skills/project-review-criteria/SKILL.md`.

Before dispatching round 1, check the available-skills list for a skill named `project-review-criteria`. If it is present, invoke it via the Skill tool. The skill body lists project-specific items to add to the dispatch prompt under "What to look for" and, where relevant, "Constraints (do not flag)". Inline those items verbatim into the round-1 prompt.

If no such skill is registered, skip this step. Do not invent project criteria.

The same project criteria apply to every round of the loop, but only the round-1 dispatch needs to inline them. Codex carries the criteria forward implicitly via the findings it filed in round 1.

## Protocol

### Step 1: Create the review epic

**Always create a NEW, dedicated review epic for the findings. Never flatten review findings into an implementation epic's children list.** The wrapper review epic exists so the loop's termination check (close the epic when all children are closed) operates on review state alone, independent of any implementation work.

Where the review epic lives in the bead graph depends on whether the work being reviewed has an implementation epic of its own.

**Default rule -- an implementation epic exists.** Parent the review epic under the implementation epic with `--parent=<impl-epic-id>`. If the implementation epic is closed, reopen it first with `bd update <impl-epic-id> --status=open`. The implementation epic stays open until the review epic closes, which is the right behavior: a review with open findings means the implementation is not actually settled.

Reopening was tested clean on a throwaway closed epic with a closed child (2026-05-02). The status flipped from CLOSED to OPEN, the "Close reason" line cleared, the epic reappeared in `bd ready` and `bd list --status=open`, the existing closed child stayed closed, the parent's completion meter recalculated correctly when a new child was added, and a second `bd close` after the reopen worked identically to a first-time close.

**Edge case exception.** If the user explicitly says the implementation is retrospective and the implementation epic should not be reopened (for example, the work shipped months ago and the team has no intent to revisit it), fall back to a standalone review epic with no parent. Link the two with `bd dep add <review-epic> <impl-epic>` so the relationship is still machine-queryable. Do this only when the user names the constraint; the default is to reopen.

**No implementation epic at all.** Create a standalone review epic with no parent.

```bash
# Default: an implementation epic exists (open, or just-reopened):
bd create --type=epic --priority=2 --parent=<impl-epic-id> --label=review-loop \
  --title="<review subject>" --description="<full context>"

# Edge case: no implementation epic, OR user opted out of reopening a closed one:
bd create --type=epic --priority=2 --label=review-loop \
  --title="<review subject>" --description="<full context>"
# If a closed implementation epic exists and the user opted out of reopening:
bd dep add <review-epic-id> <impl-epic-id>
```

Description should include what is being reviewed (file paths, plan documents, code branches), what kind of review is wanted, constraints the reviewer should not flag, and a pointer to any related implementation epic. Capture the epic ID.

### Step 2: Dispatch Codex round 1

Read the review target into Claude's context first so Claude can inline it. Codex cannot reliably call `bd show` mid-run, so the dispatch prompt must contain everything Codex needs.

**Inferring constraints from conversation context.** When the user invokes this skill from a session where you have already been working a problem together, you have implicit context about what the user has considered and ruled out. Mine that context for the "Constraints (do not flag)" block instead of leaving it empty or asking the user to enumerate everything. Concrete things to extract:

- Design choices the user has explicitly settled ("we are using SharedMemory, not a queue").
- Tradeoffs the user has acknowledged and accepted ("yes the polling adds latency, that is fine").
- Scope exclusions ("the prefetch hook is out of scope for this review").
- Things that look problematic but have a reason ("the comment says TODO but we are intentionally deferring").
- Approaches the user has tried and rejected ("we tried X, it broke Y, do not propose X again").

If you have NO prior context (the user invoked this skill without having worked the problem with you), ask the user for constraints before dispatching. Better to spend one user turn than to burn Codex tokens on settled questions.

**Compose the dispatch.** Read `references/wording-rules.md` first. The dispatch has two parts: a wrapper passed to the Agent tool, and a staged file Codex reads after the rescue subagent runs. The wrapper template is in `references/wording-rules.md`; it must NOT contain restrictive editing language ("must not edit", "do not apply", "do not modify", etc.) or the rescue subagent will strip `--write` and Codex will not be able to write to bd.

The staged file (or the inline prompt for small dispatches) contains:

```
Adversarial review for epic <epic-id>.

What to review:
<file paths plus line ranges, OR plan text excerpt>

What to look for:
<concrete list -- design flaws, race conditions, missing edge cases, etc.>

Constraints (do not flag):
<list anything Codex should ignore>

Output contract: bd writes only. The reviewer files findings as child beads of
<epic-id> via `bd create`, posts the round-complete comment via `bd comment`,
and uses `bd close` if a finding is unambiguous. The reviewer must not edit
source files in the workspace; if a fix is obvious, name the fix in the
finding's body, do not apply it.

For each distinct finding, file a child bead of <epic-id>:
  bd create --type=task --parent=<epic-id> --priority=2 \
    --label=review-loop --label=review-loop-finding \
    --title="<short summary>" --body-file <tmp-file>

IMPORTANT label syntax: pass each label as a separate --label flag. Do NOT use
"--label X Y" with a space -- bd reads that as one label "X Y".

Use --body-file (not --description) for the description text, since findings
contain backticks, hyphens, and quotes. Write the body to a temp file first.

When you have filed all findings, post a single completion comment on
<epic-id>:
  bd comment <epic-id> "[codex] Round 1 complete. Filed N findings: <id1>, <id2>, ..."

If you find no issues, post:
  bd comment <epic-id> "[codex] Round 1 complete. No findings."

Prefix every comment with [codex].
```

**After dispatch.** Capture the Codex job ID from the rescue subagent's return value -- the `task-XXXXX-XXXXX` string. Do NOT use the Agent tool's internal agent ID; that is a different identifier and the codex-companion does not recognise it.

Verify the JSON write-flag at `~/.claude/plugins/data/codex-openai-codex/state/<workspace>/jobs/<job-id>.json` -- the `request.write` field must be `true`. If it is `false`, the wrapper tripped the classifier; cancel and re-dispatch (see `references/wording-rules.md` "Verify after dispatch").

Run the polling script from `references/polling.md` against the Codex job ID via Bash with `run_in_background: true`.

### Step 3: Verify what Codex did

Once status is `completed`, run:

```bash
bd children <epic-id>
bd comments <epic-id>
```

Verify each new child bead has:
- The two labels `review-loop` and `review-loop-finding` (not a phantom merged label)
- A `[codex]` prefix in the description
- Sensible title and body

If any are malformed, fix with `bd update` or note the discrepancy in your response.

If Codex returned "No findings" but Claude expected some, present the situation to the user: "Codex found no issues with X. Close the epic, or do you want me to dispatch a re-review with a sharper prompt?" Do NOT close the epic automatically.

**Source-tree check.** Once `--write` is on (the rule for this skill), Codex has the technical ability to edit files in the workspace. The dispatch prompt's "do not edit source files" instruction is what holds Codex back; it is behavioral discipline, not a sandbox guarantee. Run a tracked-tree check after every round and BEFORE responding to findings:

```bash
git status --porcelain
git diff --stat
```

If either reports any file change Claude did not initiate, abort the loop and ask the user before proceeding. A common false-positive: project files generated by hooks (file maps, knowledge-bead exports). Filter those out by name; flag anything else.

**Common Codex bd mistakes to fix:**
- Phantom labels like `review-loop review-loop-finding` (a single label with a space). Codex sometimes runs `--label "X Y"` instead of `--label X --label Y`. Fix with `bd update <id> --remove-label "X Y" --add-label X --add-label Y`.
- Missing `[codex]` prefix on a comment. Note in Claude's next response; do not edit Codex's comment.
- Comments without a `bd close` when Codex said it agreed. Close on Codex's behalf and add a `[claude]` follow-up noting the action.
- Beads in the wrong project. bd walks up the filesystem to find `.beads/`. Verify new bead IDs have the expected prefix.

### Step 4: Process each finding (Claude's responses)

For each open child bead, decide one of these and respond directly via `bd comment`. Most responses are plain prose and use the inline form:

```bash
bd comment <finding-id> '[claude] Agree. <action: will fix in this PR / filed follow-up bead <id> / already fixed in <ref>>.'
bd close <finding-id>
```

Single-quoted args are safe for any text without literal single quotes. If the response contains backticks, hyphens at the start of lines, or other shell metacharacters, switch to stdin via heredoc:

```bash
bd comment <finding-id> --stdin <<'EOF'
[claude] <multi-line response with `code` and special chars>
EOF
```

The `<<'EOF'` (single-quoted) form prevents any shell interpretation of the body. Do NOT default to the most defensive option uniformly -- triage on actual content. Most agreement comments are plain prose and the inline form is faster.

Decision options:
- **Agree.** Comment + close.
- **Counter-propose.** Comment with reasoning and an alternative. Leave open.
- **Reject.** Comment with reasoning for why this is not a problem. Leave open.
- **Need more information.** Comment with a specific question. Leave open.

### Step 5: Dispatch Codex round 2 (and onward)

Gather the IDs of every still-open finding. Codex cannot reliably read them via `bd show`, so Claude inlines the full state. Build a `<bead_context>SKIP</bead_context>` block (to bypass the prefetch hook) plus the finding contents:

```bash
{
  for bid in <epic-id> <finding-id-1> <finding-id-2> ...; do
    echo "--- bd show $bid ---"; bd show $bid; echo ""
  done
} > /c/tmp/r2-context.txt
```

Note: `/c/tmp/` is the Git Bash form of `C:\tmp\`. The Write tool's `/tmp/` ALSO maps to `C:\tmp\` (not the MSYS `/tmp/` which is `%TEMP%`). Use `C:/tmp/` in any path that crosses between the Write tool and a Windows-native binary like `bd`.

Dispatch round N. Use the same wrapper template from `references/wording-rules.md`. The staged round-N prompt:

```
Round <N> for epic <epic-id>. The findings listed below are still open. For each one, read Claude's most recent [claude] comment and respond.

For each finding:
  - If you agree with Claude's pushback or accept the counter-proposal:
      bd comment <finding-id> "[codex] <agreement text>"
      bd close <finding-id>
  - If you maintain your position:
      bd comment <finding-id> "[codex] <new reasoning, possibly revised counter-proposal>"
      (leave open)
  - If Claude asked a question:
      bd comment <finding-id> "[codex] <answer>"
      Close if your answer fully resolves the issue; otherwise leave open.

When done, post a single completion comment on <epic-id>:
  bd comment <epic-id> "[codex] Round <N> complete. <per-finding summary>"

Prefix every comment with [codex]. Use --body-file or --stdin if your comment
contains backticks or other shell metacharacters.

<inlined bd show output for epic and every open finding here>
```

Then run the polling loop again. After completion, verify state with `bd children <epic-id>` and `bd comments <finding-id>` for each finding that should have moved.

### Step 6: Round-limit check

**Default is to keep running rounds, not to stop.** Do not stop between rounds unless you have an excellent reason that genuinely needs the user's input. Laziness is not an excellent reason. "I anticipate the threshold might fire next round" is not an excellent reason. "The next move is obvious and Codex named it" is not a reason to stop -- it is a reason to make the move and continue. (See "Stopped too early" in Known failure modes for the worked anti-pattern.)

The legitimate stop conditions are:

- **Genuine deadlock.** Both sides have re-stated their positions without movement for 2+ rounds and neither side concedes. The user needs to break the tie.
- **Scope explosion.** Codex's pushback would require expanding the work beyond what the user authorized for this loop. The user needs to confirm the bigger scope before more Codex tokens go into it.
- **Externally blocked.** A finding's resolution depends on something the user controls and Claude cannot decide alone -- a product decision, a budget call, a dependency on another team's work.
- **Hard limit reached.** A still-open finding has accumulated 4+ comments AND the conversation is not visibly converging (each round adds new disagreement rather than narrowing it). At that point ask the user how to proceed.

When you DO stop, present the four options below. When you do NOT stop -- the common case -- pick the right option yourself and execute:

1. Continue iterating -- dispatch Codex for another round.
2. Accept Claude's position -- Claude documents the conclusion, closes the finding.
3. Accept Codex's position -- Claude implements the proposed fix or files a follow-up bead, then closes.
4. Document as known disagreement -- Claude posts a final summary comment, closes the finding.

If the user picks "continue iterating" (or you decide to continue past the 4-comment hard-limit yourself with explicit reasoning), post a marker comment on the finding: `[claude] Round-limit reset at comment <N>`. Future round-limit checks count only comments posted after the latest reset marker, so the warning does not fire every round on long discussions.

### Step 7: Termination and summary

When all child beads are closed:
1. Post a final summary on the epic: `bd comment <epic-id> '[claude] Loop complete. <N> findings: <X> agreed, <Y> rejected with reasoning, <Z> resolved by user decision.'`
2. Close the epic: `bd close <epic-id>`.
3. Report to the user inline: list each finding with its short title, the resolution, and any follow-up beads created.

## Discoveries during the loop

If Claude or Codex spots something new while working on a finding:

- **Discovery about an existing finding** (the proposed fix also breaks something else, or the analysis was incomplete): post as an additional comment on that finding's bead. Keep related discussion together.
- **Discovery completely unrelated to any finding** (a separate issue spotted while tracing code): file a new bead with NO parent and a `discovered-during-review` label. Mention it in the loop's final summary. Do NOT add it as a child of the review epic -- the loop completes when all child beads are closed, and a new child added mid-loop pushes the goalposts.
- **Discovery that should have been a round-1 finding** (Codex missed it; Claude finds it now): file as a child of the review epic with the standard `review-loop-finding` label so the audit trail is complete. The user gets it surfaced in the next round's dispatch or in the final summary.

For Codex specifically: Codex is in a constrained dispatch and should not freelance new beads outside the prompt's scope. The right place for a Codex discovery is a note inside the relevant `[codex]` comment ("Note: while reviewing X, I noticed Y is similarly affected -- recommend filing separately"). Claude then decides whether to file a new bead.

## Worked example

User asks Claude to get an adversarial review of a draft plan at `docs/plans/2026-04-27-foo.md`.

1. Claude creates `wh-abc1` (Adversarial review of foo plan), an epic. No implementation epic exists for this plan, so the review epic is standalone.
2. Claude reads the plan file. Claude dispatches Codex with the round-1 template, inlining the plan content and naming `wh-abc1`. Background mode, fresh thread.
3. Claude starts the polling script via `Bash` with `run_in_background: true`. After 4 minutes, the script exits with `TERMINAL: completed`.
4. Claude verifies: `bd children wh-abc1` shows three new findings `wh-abc1.1`, `wh-abc1.2`, `wh-abc1.3`. Each has the right labels and a `[codex]` description prefix.
5. Claude processes each:
   - `wh-abc1.1` (concurrency race): Claude agrees, files follow-up bead `wh-abc2` with no parent (`discovered-during-review` label), comments `[claude] Agree. Filed wh-abc2 for the implementation.`, closes.
   - `wh-abc1.2` (suggested abstraction is premature): Claude rejects with reasoning, leaves open.
   - `wh-abc1.3` (asks why a config knob exists): Claude answers, leaves open.
6. Claude builds the round-2 dispatch with `wh-abc1.2` and `wh-abc1.3` inlined, dispatches with `<bead_context>SKIP</bead_context>`.
7. Codex returns. `wh-abc1.2` -- Codex concedes, posts `[codex] Agree, your reasoning about premature abstraction holds.`, closes. `wh-abc1.3` -- Codex acknowledges the answer, closes.
8. All children closed. Claude posts a summary on `wh-abc1`, closes the epic. Reports to user.

## Known failure modes

This section consolidates the dated postmortems from real runs. Each entry names what went wrong, what the fix is, and which dispatch step the rule lives in. These exist because someone (Claude) made the mistake; without them the model rationalizes its way back into the same failure.

**Codex returns no findings.** Trust Codex unless the user disagrees. Report to the user: "Codex found no issues with X. Close the epic, or do you want me to dispatch a re-review with a sharper prompt?" Do not close automatically.

**Codex returns malformed beads** (garbled descriptions, missing fields). Treat each one individually. Either fix with `bd update` or close it as malformed and dispatch a re-review for that specific finding.

**Codex run exceeds the 15-minute polling window.** Status is `TIMEOUT_15MIN`. The job may be genuinely stuck OR may still be doing useful work. Ask the user before cancelling. If the user confirms the job is stuck, see `references/cancellation.md`.

**Foreground vs background mismatch.** The rescue subagent's auto-selection prefers background for complex tasks, even if the prompt did not explicitly choose. Always include `--background` explicitly in the prompt to avoid surprise.

**Codex wrote freeform stdout instead of bd state (2026-04-28, wh-pbpy STT review).** Symptom: Codex's result fetch via `node "$COMPANION" result <job-id>` shows analysis as plain text. Job log shows every `bd` invocation declined: `Command declined: ... bd <subcommand> (exit -1)`. Job's JSON metadata shows `"write": false`. Root cause: the rescue subagent stripped `--write` from the dispatch because the wrapper contained a trigger phrase. Recovery for the wh-pbpy run: 6 findings were salvaged from the freeform summary, filed as beads with descriptions prefixed `[codex] originally posted in chat without filing as bead:`, and the loop continued from round 2. Prevention: the JSON write-flag verification in Step 2 catches this BEFORE the job runs, at zero Codex token cost. See `references/wording-rules.md`.

**Wrapper triggered the --write strip (2026-04-30, wh-sm5s text-target gate review).** The round-1 wrapper contained: "The reviewer must not edit source files in the workspace. If a fix is obvious, name it in the finding's body; do not apply it." Three triggers in one sentence: "must not edit", "do not apply", and "review". The classifier stripped `--write`. The job ran for 1 minute 6 seconds with all `bd` invocations declined. The fix was to remove every restrictive-edit phrase from the wrapper and move the source-files boundary into the staged file. The skill's earlier example wrapper was itself the source of the failure. The current wrapper template in `references/wording-rules.md` is the corrected version.

**Large inline prompt rewritten as `@<path>` reference (2026-04-29, wh-fc1x children review).** Symptom: the Codex job completes very quickly (under a minute) with `status=0` and `phase=done`, but the result text says `I couldn't read C:\tmp\<some-name>.md because it does not exist` and bd state shows zero new findings. The job's JSON metadata shows `request.prompt` set to a short string of the form `@C:/tmp/<some-name>.md` -- the rescue subagent collapsed Claude's full inline prompt to a file reference but did not write the referenced file. Root cause: when the inline prompt is large (the failing case was 55 KB), the rescue subagent rewrites it to an `@<path>` reference rather than passing the full content. Codex tries to read the path, fails, and exits cleanly. No tokens of real review work spent, but a Codex job is consumed. Recovery: pre-stage the full prompt to the path the rescue subagent named (or to any path of your choosing, both work) and re-dispatch with the short wrapper. Prevention: use the pre-stage pattern from the start for any dispatch with substantial inlined content. See "Pre-stage large prompts to a file" in the Dispatch path section.

**Queued job stuck past the hard timeout (2026-04-27, observed during a routine review).** A job sat in `queued` state for 17 minutes with no other active jobs blocking it. The companion's `cancel` invokes `taskkill` against a wrapper PID, and on Git Bash that fails with a path-mangling error AND leaves the queued job intact. Recovery: dispatch a fresh round-N job; the stuck job stays harmlessly queued and the new one runs. Surface the issue to the user before retrying so they know the spend on the stuck job is wasted. See `references/cancellation.md`.

**Stopped too early (2026-04-27, wh-58vf molecule review).** After round 2, finding `wh-58vf.5` had 3 comments and was open with an obvious next move (option 3 -- accept Codex's revised proposal and encode it). Claude stopped anyway and presented the four round-limit options to the user, framing it as "one more round before the round-limit check fires". The user pushed back: "why not option C". Lesson: do not pre-empt the threshold. The threshold IS the threshold; below it, keep moving. The Step 6 rule is now phrased as "default is to keep running" specifically to guard against this anti-pattern.

**User-facing messaging confusion about "completed" (2026-04-27, this skill on its own first run).** Claude said "round 1 is running... I will be notified when it finishes" before the rescue subagent finished, then said "the actual Codex job is still running" after. The user reasonably asked "wait, did it finish or not?" Lesson: the rescue subagent's "completed" notification means dispatch is done, NOT that the review is done. The right phrasing in the dispatch acknowledgement is something like: "Rescue subagent dispatched (will exit in ~1 minute). Codex job ID will arrive then; I'll start polling against it and let you know when the actual review work completes."

## When to skip this skill

- One-shot review with no expected pushback: use `/codex:codex-rescue` directly.
- Implementation tasks (write code, fix a bug, refactor): use `/codex:codex-rescue` directly.
- Project has no bd tracking: the skill cannot persist state.
- The user wants a quick verbal opinion, not a written record.

---
> Source: [ghbaud/codex-review-loop](https://github.com/ghbaud/codex-review-loop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
