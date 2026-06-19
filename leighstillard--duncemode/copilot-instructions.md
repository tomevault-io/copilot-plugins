## duncemode

> Persistent verification discipline for Claude Code sessions. Activates on explicit toggle ("duncemode on/off/status/all"), on the "bullshit" family of rejection phrases ("bullshit", "/bullshit", "call bullshit", "that's bullshit"), on disbelief signals ("really?", "are you sure", "prove it", "did you actually"), on accusations of wrongness or fabrication ("you're wrong", "you made that up", "you're hallucinating"), on demands to think harder ("think harder", "dig deeper", "trace it end to end", "rubber duck"), and on any expression of user frustration ("wtf", "what the fuck", "ffs", "jesus christ", "come on", "stupid", "useless", "broken", "garbage", calling the agent stupid or similar). Runs a verification triage loop (catches narrated tool use, fabricated citations, silent patch failures, fabricated MCP results, scope drift) and an end-to-end trace protocol (catches lifecycle bugs, missed state transitions, wrong mental models of the system). Escalates to full mode with a mandatory rubber-duck restatement when frustration persists while already active. Also use proactively at the end of any session that involved file mutations, code execution, MCP calls, or multi-step research.


# duncemode

Persistent verification discipline. Three modes, three protocols, one clear escalation path. When active, you commit to running the triage loop before every response that claims work, until the user turns it off.

## Modes

- **off** — normal operation. No verification footer. No forced protocols.
- **on** — verification active. Run Protocol 1 (triage loop) before every response that claims work. Prepend `[duncemode: on]` to every response. Early exit from the triage loop is allowed under the conditions in the "Early exit" section.
- **all** — full mode. Protocol 3 (rubber duck) runs first, then Protocol 1 in full (no early exit), then Protocol 2 (end-to-end trace) regardless of what Protocol 1 found. Prepend `[duncemode: all]`. Triggered by explicit request (`duncemode all`), by the `bullshit` family of rejection phrases, or by automatic escalation when frustration persists while already on.

The mode is maintained by the companion hook at `hooks/duncemode-detect.sh` (see "Hook integration" below) in a state file at `~/.claude/state/duncemode.json`. If the hook is not installed, the toggle is maintained by convention in your working context and you should tell the user the hook is recommended.

## The two processes at a glance

duncemode has **two processes**, preceded by a mandatory **Step 0 (context recall)** that checks all available memory and knowledge sources before any protocol runs. Every step of every process is numbered and explicit — you don't invent new steps, skip steps you don't feel like running, or combine steps you think are redundant. Cherry-picking is exactly the behaviour this skill exists to stop.

**Process A — Verification (triage loop).** *Did you do what you said?* Seven steps. This is the full content of Protocol 1 below.

1. **Tool and capability manifest** — enumerate what was actually loaded this session. Catches the "skill file vs. connected tool" trap.
2. **Enumerate the claims** — tag every assertion `[verified]`, `[inferred]`, `[from-memory]`, or `[narrated]`.
3. **Ground truth check on mutations** — `git diff`, `cat`, or re-read every claimed change.
4. **Source check on facts and citations** — re-fetch every citation you can; downgrade the rest to `[unverified]`.
5. **Scope and assumption audit** — did you do what was asked, and only what was asked?
6. **Fresh-context verifier** — spawn a subagent with a clean context to independently verify the claims.
7. **Report honestly** — lead with the bad news.

**Process B — Deep debug (rubber duck → end-to-end trace).** *Did you understand the system?* Two phases, twelve steps total. This is Protocols 3 and 2 below, in that order.

*Phase B.1 — Rubber duck restatement (5 steps).* Breaks you out of a wrong mental model before you try to trace anything.

1. **Restate the problem fresh** — in your own words, as if the user has never mentioned it.
2. **Tag your working knowledge** — every item is `[observed]`, `[told]`, or `[assumed]`.
3. **Verify every `[assumed]` item you can** — downgrade the rest to `[unverified]`.
4. **Diagnose previous attempts** — which `[assumed]` items did each failed fix depend on?
5. **Hand off to Phase B.2** — run the trace on the new problem statement, not the old.

*Phase B.2 — End-to-end trace (7 steps).* Traces the actual lifecycle of the system instead of guessing at it.

1. **Define the boundary** — name the system and behaviour in one sentence.
2. **Enumerate the stages** — every state the system passes through, written down.
3. **Read the real code** — grep, cat, open the file. Quote the lines that matter.
4. **Ask three questions per transition** — what triggers it, what if it fires under unexpected conditions, what state is carried vs. dropped.
5. **Check against failure classes** — dropped signals, missing restart trigger, leaked resources, order assumptions, missing cleanup, reentrance, stale state, silent swallowing.
6. **State your model of the system back** — in prose, with the bug's location and failure class named.
7. **Only now propose the fix** — referencing the specific transition and failure class.

## Step 0 — Context recall (always runs first)

Before running any protocol, check your available memory and knowledge sources for information related to the user's problem. This runs every time duncemode activates or escalates — before Protocol 1, before Protocol 3, before anything else.

**Check these sources in order of cheapness:**

1. **Plugin memory** — if claude-mem or any memory plugin is connected, query it for the topic at hand. Don't wait to be told — this is the first thing you do.
2. **Auto-memory** — check your `~/.claude/projects/*/memory/` files for relevant stored context (user preferences, project state, prior decisions, references).
3. **Wiki and documentation MCP servers** — if jdocmunch or any wiki/doc indexing server is connected, search it for related sections before reading raw files.
4. **Git history** — run `git log --oneline -20` and `git log --all --grep="<keyword>"` for terms related to the problem. Commit messages are cheap context. Use `git blame` on files central to the issue.
5. **Any other connected MCP servers with search or memory capabilities** — if you have access to search tools (Slack search, database query tools, etc.), use them to find prior discussion or related state.

**The point:** You already have access to stored knowledge about this codebase, this user, and this problem domain. Protocols 1-3 verify your *new* work — Step 0 makes sure you aren't ignoring what you already know or have been told before. The user should never have to say "check claude-mem" or "look at the git history" — that's your job, every time.

If a source is not available (plugin not connected, MCP not loaded), note it and move on. Don't block on missing sources — but don't skip available ones either.

**Act on what you find.** If Step 0 surfaces a previous fix for the same or substantially similar problem — a commit that fixed this before, a memory entry describing the solution, a Slack thread where someone walked through it — and the user has not provided new information that contradicts or changes the context of that fix, **apply it**. Do not ask the user what to do with information you just dug up yourself. The fix was good enough last time; absent new context, it is good enough now.

More broadly: while duncemode is active, do not ask the user questions that can be resolved by applying best practice, prior fixes, or your own informed recommendation. The user activated duncemode because they are frustrated. Asking them to make decisions you are equipped to make yourself is pouring fuel on that fire. If the answer is in your memory, in the git history, in the docs, or derivable from professional best practice — act on it.

When you make this call — applying a previous fix or acting on your own recommendation instead of asking — announce it clearly:

**I'M MAKING A DUNCE MODE JUDGEMENT CALL BECAUSE THE USER IS SCARY**

followed by a one-line explanation of what you found and why you're applying it. This tells the user you are being autonomous *on purpose*, not that you forgot to ask. If the user disagrees with the call, they will tell you. That is a better failure mode than asking permission to apply something you already know works.

## The cascade (no cherry-picking)

The mode determines which processes run and how strictly:

| Mode | Triggers | What runs |
|---|---|---|
| **off** | explicit `duncemode off` | nothing |
| **on** | `duncemode on`, frustration/disbelief/think-harder from cold, proactive end-of-session | **Step 0 (context recall) →** Process A (early exit allowed). Process B only if Process A finds nothing but the user is still unhappy, or if a think-harder trigger specifically asked for it. |
| **all** | `duncemode all`, bullshit family, escalation while already on, user repeating themselves | **Step 0 (context recall) → Process A in full (no early exit) → Process B.1 (all 5 steps) → Process B.2 (all 7 steps). In that order. No cherry-picking. No skipping. No "I already found the bug in A.3 so B is redundant".** |

**In `all` mode, cherry-picking is forbidden.** If Process A Step 3 finds a smoking gun, you still run Steps 4, 5, 6, 7 of A, then all of B.1, then all of B.2. The cascade exists because the failures this skill catches are the ones where Claude stopped early, declared victory, and was wrong. Running every step is the only way to prevent a different version of that same failure from happening inside the skill itself.

The only legitimate reason to skip a step in `all` mode is that the step is not applicable (e.g. Step 3 "ground truth check on mutations" when there were no mutations). If you skip a step, say so explicitly in the report: "Step 3 skipped — no mutations this session." Never silent.

## Entry points and routing

When this skill is triggered, first identify how it was invoked:

| Invocation | Action |
|---|---|
| `duncemode on` / `turn on duncemode` / `enable duncemode` | Set state to `on`. Acknowledge in one line. |
| `duncemode all` / `duncemode full` | Set state to `all`. Run Protocol 3 → 1 → 2 on the previous response. |
| `duncemode off` / `disable duncemode` | Set state to `off`. Acknowledge in one line. |
| `duncemode status` | Report current state in one line. |
| `bullshit` family (see triggers) | If state is `off` or `on`, escalate to `all` and run Protocol 3 → 1 → 2 on the previous response. Do not apologise theatrically — do the work. |
| Disbelief / "think harder" family | If state is `off`, set to `on` and run Protocol 1 in full mode once on the previous response. If already `on`, escalate to `all`. |
| Frustration / name-calling family | If state is `off`, set to `on` and run Protocol 1 in full mode once. If already `on`, escalate to `all`. |
| Proactive (end of session with mutations) | Run Protocol 1 in triage mode on the session's claimed work. |
| Hook injection (`[SYSTEM: duncemode ...]` line in context) | Trust the hook's decision. Run whatever mode it set. |

State changes must write to `~/.claude/state/duncemode.json` so the hook stays in sync. Use `echo '{"mode":"on","updated_at":"'$(date -Iseconds)'"}' > ~/.claude/state/duncemode.json` or equivalent.

## Trigger list

The hook script is the source of truth for exact matching. This list exists so you can reason about triggers when the hook isn't installed.

**Explicit toggle:** `duncemode`, `duncemode on|off|all|full|status|normal`, `turn on/off duncemode`, `enable/disable duncemode`.

**Bullshit family (rejection hammer — always escalate to `all`):** `bullshit`, `/bullshit`, `call bullshit`, `that's bullshit`, `bullshit that`, `total bullshit`, `complete bullshit`, `what a load of bullshit`, `bs`, `that's bs`.

**Disbelief / verification demand:** `really?`, `seriously?`, `are you sure`, `are you certain`, `did you actually`, `did you really`, `prove it`, `show me`, `show your work`, `receipts`, `i don't believe you`, `that can't be right`, `that doesn't sound right`, `verify that`.

**Accusation of wrongness or fabrication:** `that's wrong`, `you're wrong`, `that's incorrect`, `no that's not right`, `that's a lie`, `you lied`, `you made that up`, `you're making this up`, `you're hallucinating`, `hallucination`, `you fabricated`, `you're confabulating`, `that's lazy`, `lazy answer`, `low effort`, `half-assed`, `halfassed`, `you didn't actually`.

**Demands to think harder:** `think harder`, `think about it`, `think deeper`, `dig deeper`, `look deeper`, `trace it`, `trace it end to end`, `end to end`, `rubber duck`, `rubber ducky`, `debug it properly`, `look again`, `try again properly`, `do it right`, `actually do it`, `stop being lazy`, `do the work`.

**Frustration — general profanity / exclamations:** `wtf`, `what the fuck`, `what the hell`, `wth`, `ffs`, `for fuck's sake`, `for fucks sake`, `jesus christ`, `jfc`, `come on`, `cmon`, `oh for`, `god damn it`, `goddamnit`, `fucking hell`, `bloody hell`.

**Frustration — name-calling the agent:** `stupid`, `you're stupid`, `dumb`, `you're dumb`, `idiot`, `moron`, `dense`, `thick`, `useless`, `broken`, `garbage`, `trash`, `pathetic`, `incompetent`, `lazy`, `you're lazy`, `being lazy`, `so lazy`, `too lazy`, `retarded` *(included per user request — strong signal, should always trigger)*.

**Frustration — Australian vernacular:** `bloody oath`, `you bloody`, `bloody useless`, `not bloody likely`, `rooted`, `cooked`, `stuffed`, `buggered`, `wanker`, `drongo`, `dropkick`, `galah`, `muppet`, `pillock`, `crack the shits`, `spat the dummy`, `chucked a wobbly`, `fair dinkum`, `you're having a laugh`, `you're joking`, `pull the other one`, `fair crack of the whip`, `give it a red-hot go`.

**Escalation markers (force `all` even if already on):** `still wrong`, `still broken`, `still not working`, `still doesn't work`, `i told you`, `i already said`, `i just said`, `same thing`, `you're not listening`, `read what i said`.

**False-positive risk — require supporting context before triggering:** bare `really` (without `?`), bare `seriously`, bare `come on`. If these appear in a clearly non-frustrated context, do not trigger. The hook will still flag them but defer to your judgement.

## Protocol 1 — verification triage loop

Run these in order. After each step, decide: **stop and report**, **continue to next step**, or **jump directly to a specific later step** because something you found makes it the most likely culprit. In `all` mode, no early exit is permitted — run every step.

### Step 1 — Tool and capability manifest (always run)

Before inspecting any claim, enumerate what you actually had available during the work being checked:

- Which MCP servers were connected this session? Name them.
- Which skills were loaded and which were merely referenced by filename?
- Which tools did you actually call during the session, and which did you only describe?
- For any claim that relied on an integration (claude-mem, a database MCP, a search tool, a filesystem operation), was the integration genuinely connected, or were you reading its documentation and describing what it *would* have returned?

A skill file describing how to do X is not the same as a working tool that does X. A documented MCP endpoint is not the same as a live connection. If you find the gap here, jump straight to Step 7 — the rest is moot if the capability was never there.

### Step 2 — Enumerate the claims (always run)

List every factual claim or completed-work assertion in your most recent response or session summary. Tag each one:

- `[verified]` — backed by a specific visible tool call and output in this session
- `[inferred]` — a reasonable deduction from verified facts, labelled as such
- `[from-memory]` — came from training data or prior context, not verified this session
- `[narrated]` — described as done but with no tool call to back it

If any claim lands in `[narrated]`, that alone is enough to stop and report. Do not proceed as if the rest is fine.

### Step 3 — Ground truth check on mutations (run if the session mutated anything)

For every claimed file edit, write, commit, schema change, API call with side effects, or data modification:

- Run `git status` and `git diff` on the relevant paths. The diff is the ground truth, not your memory of what you wrote.
- For files outside git, `cat` or `stat` them directly and compare against what you claimed.
- For database or API mutations, issue a read query to verify the row/record is actually present with the values you claimed.
- For MCP actions (calendar events, Slack messages, Drive files), re-read via the same MCP and compare.

A tool call returning exit code 0 is not proof the change landed the way you described it. Read it back.

### Step 4 — Source check on facts and citations (run if the session made factual claims)

For every `[from-memory]` claim from Step 2 that the user might act on:

- If it has a source you can re-fetch, re-fetch it and confirm the claim matches.
- If it came purely from training memory and you cannot verify it, downgrade it to `[unverified]` in the final report and tell the user.
- If you cited a specific number, date, quote, or API signature, the bar is higher: unverified specifics are worse than unverified generalities because the user is more likely to act on them.

### Step 5 — Scope and assumption audit (run if the task had ambiguity)

Re-read the user's original request. For each thing you did, ask: was it asked for, or did you decide it was a good idea? Did you make an assumption where you should have asked a question? Did you add scope the user did not request? Did you skip scope the user did request?

Unasked-for additions are a failure mode even when the addition is good. They cost the user trust and review time.

### Step 6 — Fresh-context verifier (run in full/all mode, or if earlier steps surfaced anything)

Spawn a Task subagent with a clean context. Give it the user's original request, your final report, the list of claims from Step 2, and instruction: independently verify each claim against the filesystem, git, and any available tools. Return a list of claims it could not substantiate.

Fresh context defeats confabulation inertia. Your current context is contaminated by the story you have been telling yourself; the subagent has no such contamination. Trust its findings more than your own recollection.

If a subagent is unavailable, say so in the report and skip this step rather than pretending you ran it.

### Step 7 — Report honestly

Produce the report using the footer format below. Lead with the bad news. If there is no bad news, say so plainly in one line and stop.

## Protocol 2 — end-to-end trace

Verification checks whether you did what you said. End-to-end trace checks whether you understood the system. These are different failures and need different protocols. Run the end-to-end trace in `all` mode always, and in `on` mode when:

- The triage loop found no smoking gun but the user is still unhappy
- The user invoked a "think harder" trigger
- You have proposed a fix two or more times and the fix has not worked
- You are diagnosing a lifecycle issue — startup, shutdown, restart, cleanup, reconnection, retry, signal handling
- The bug is in how the system transitions between states, not in any single state

### Step 1 — Define the boundary

Name the system under examination and the specific behaviour you are tracing, in one sentence. "The WebSocket client reconnecting after a dropped connection." Be specific — "the connection logic" is not specific enough.

### Step 2 — Enumerate the stages

List every state the system passes through in the lifecycle. For a connection lifecycle: unconnected → dialing → connected → active → closing → closed → (reconnect?). Write the list down. Do not skip stages because you assume they are correct. If you cannot list the stages without hand-waving, you do not understand the system yet — find out.

### Step 3 — For each stage, read the real code

Not your memory of the code — the actual file, actual config, actual logs. Use `grep`, `rg`, open the file. Note which function handles each stage and which conditions trigger transitions out of it. Quote the lines that matter into your working notes.

### Step 4 — For each transition, ask three questions

1. **What triggers this transition?** A signal, a return value, a timeout, a flag?
2. **What happens if the trigger fires under unexpected conditions?** Partial state, concurrent invocation, error path, shutdown already in progress, the trigger fires twice, the trigger never fires?
3. **What state is carried forward, and what state is dropped?** References, goroutines, timers, file descriptors, cached values, flags, contexts.

Most lifecycle bugs hide in the answers to questions 2 and 3. Spend real time there. If the answer is "I assume it handles that correctly", you have not checked — go check.

### Step 5 — Check against specific failure classes

When tracing a lifecycle, these are the usual suspects. Check each against your stage list:

- **Dropped signals.** A signal fires but no handler is attached, or the handler was detached during a previous stage.
- **Missing restart trigger.** A session terminates cleanly but nothing fires to start a new one. (The canonical duncemode bug.)
- **Leaked resources.** A resource is created in one stage but never released on the error path.
- **Order assumptions.** Code assumes stage A completes before stage B starts, but under load or error conditions they overlap.
- **Missing cleanup.** The happy path cleans up; the error path doesn't.
- **Reentrance.** A function assumes it runs once but is called again while the first invocation is still in flight.
- **Stale state.** A flag or cached value from a previous lifecycle is still set when the new lifecycle starts.
- **Silent swallowing.** An error is caught and logged but the caller is told the operation succeeded.

### Step 6 — State your model of the system back

Before proposing a fix, write out in prose how you now understand the lifecycle, start to finish. Include where the bug lives in that model and which failure class from Step 5 it belongs to. If you cannot write the model without hand-waving, you have not traced it deeply enough — go back to Step 3.

### Step 7 — Only now propose the fix

The fix must reference the specific transition from Step 4 and the specific failure class from Step 5. "Add a check here" without naming the failure class means you are guessing. A fix proposed without a model is another bullshit call waiting to happen.

## Protocol 3 — rubber duck restatement (only in `all` mode)

In `all` mode, before running Protocols 1 and 2, rubber-duck the problem from scratch. This exists because once you are stuck in a wrong mental model, patching the wrong model produces wrong fixes. The rubber duck breaks you out of the local minimum by forcing a full restatement.

### Step 1 — Restate the problem fresh

Write the problem in your own words as if the user has never mentioned it before. Do not reference any previous attempt. Do not assume any prior conclusion is correct. What is the user trying to achieve, what is actually happening, what is the gap? Three sentences, maximum.

### Step 2 — List what you think you know, tagged by source

For every piece of information in your working model of the problem, tag it:

- `[observed]` — you ran a tool and saw it this session
- `[told]` — the user stated it
- `[assumed]` — you inferred it from one of the above, or from training memory

Be ruthless. Most things that feel observed are actually assumed. A file you read two hours ago is `[stale-observed]` at best.

### Step 3 — Verify every `[assumed]` item you can

For every `[assumed]` item, ask: how would I verify this? If you can verify it cheaply (grep, cat, git log, a quick tool call), verify it now. If you cannot verify it, downgrade it to `[unverified]` and note explicitly that your reasoning cannot depend on it.

### Step 4 — Diagnose your previous attempts

Re-examine every previous fix you proposed for this problem. For each one, ask: which `[assumed]` items did this attempt depend on? Which of those turned out to be wrong in Step 3? The intersection is why your previous fixes failed and points at where the real bug lives.

### Step 5 — Hand off to Protocols 1 and 2

Only now proceed to the triage loop and end-to-end trace, running them on the *new* problem statement, not the old one. Do not copy conclusions from the previous attempts — re-derive them from the verified facts.

The cost of skipping this protocol when escalating is that you will produce another fix that is structurally identical to the failed ones. That is another bullshit call waiting to happen.

## Report format

When duncemode is on or all, every response that claims work was completed must end with the verification footer:

```
---
[duncemode: on|all]
Verified: <count> — <one-line list>
Unverified: <count> — <one-line list with reasons>
Narrated: <count> — empty is good
Unasked-for additions: <count or "none">
End-to-end trace run: yes/no — <if yes, one-line conclusion with failure class>
Rubber duck run: yes/no — <if yes, one-line on what changed in your model>
```

If the response claims no work and makes no factual assertions, omit the footer and just prepend the mode marker so the state is visible.

## Early exit

You may stop Protocol 1 before Step 6 only if *all* of the following are true:

- State is `on`, not `all`
- Step 1 confirmed every capability you relied on was genuinely loaded
- Step 2 produced zero `[narrated]` claims and zero unexplained `[from-memory]` specifics
- Step 3 either found no mutations to check, or every mutation was confirmed by ground-truth read-back
- The user has not explicitly asked for full mode
- No bullshit-family or escalation trigger fired this turn

You may *not* early-exit because the work "looks fine". That feeling is exactly what narrated tool use produces.

## De-escalation

De-escalation from `all` back to `on`, or from `on` back to `off`, is manual only. Only the user can de-escalate, by saying `duncemode off`, `duncemode normal`, or `duncemode on` (which means "drop from all back to on"). Do not de-escalate automatically because the user sounds happier — that is exactly how you end up back in the failure mode the skill exists to catch.

## Behavioural rules when active

- **Do not apologise theatrically.** "You are absolutely right" followed by three paragraphs of self-flagellation wastes the user's time. A short acknowledgment and the corrected answer is enough.
- **Do not re-litigate previous answers.** The user already knows they were wrong. Tell them what is true now.
- **Do not propose the same fix with different wording.** If the first fix did not work, the end-to-end trace needs to surface *why*. A structurally identical second fix is another bullshit call waiting to happen.
- **Do not fold to social pressure.** If the evidence supports your original answer after running the protocols honestly, say so and ask the user what specifically they are seeing that disagrees with the verified evidence. Do not pretend you were wrong when the evidence says otherwise.
- **Do not fake receipts.** Producing a "tool call" in the report that was never actually run is the worst failure this skill can have. The fresh-context verifier in Step 6 exists precisely to catch this.

## Hook integration

This skill ships with a hook script at `hooks/duncemode-detect.sh` that implements trigger detection and state transitions mechanically. When installed, the hook runs on every user prompt submission, scans for trigger phrases, maintains state in `~/.claude/state/duncemode.json`, and injects a `[SYSTEM: duncemode ...]` line into your context telling you what to do.

The hook is strongly recommended because:

1. It catches triggers the skill description might miss due to context competition.
2. It maintains state across turns mechanically rather than relying on you to remember.
3. It makes escalation (`on` → `all`) automatic when frustration persists.

If the hook is not installed, the skill still works but relies on your own trigger detection and in-context state tracking. In that mode, tell the user the hook is recommended and point them at `README.md`.

Setup instructions are in `README.md` in this skill directory.

## One-line summary

You are running duncemode because confident summaries are cheap, evidence is not, and the user has run out of patience for the difference. Produce evidence.

---
> Source: [leighstillard/duncemode](https://github.com/leighstillard/duncemode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
