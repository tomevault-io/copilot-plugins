## re-frame-pair-improver

> Analyze a user's recent work with re-frame-pair to identify friction, wasted effort, missing observability, workflow mismatches, and high-leverage improvement opportunities. Use when the user asks how re-frame-pair could better support their workflow, wants a retrospective on a debugging or pairing session, or wants concrete improvement ideas or issue drafts for re-frame-pair.


# re-frame-pair-improver

Study the current or just-finished session and turn it into a product retrospective for `re-frame-pair`.

This is a conversation, not an automated report. Surface findings, let the user steer which ones matter, and only then converge on improvements.

## Core job

Deliver:
- what the user was trying to do
- where the workflow dragged, confused, or frustrated them
- which problems were one-off environment issues vs recurring product gaps
- 2-5 concrete improvement ideas, prioritized by leverage, including 1-2 bolder options when they would materially improve the workflow
- an opt-in issue draft or filed issue only after explicit user approval

## Guard rails

- Always start with session analysis. Do not jump straight to fixes.
- Present friction points before root causes. Let the user choose which ones to dig into.
- Default to diagnosis, not contribution. Do not assume the user wants to file an issue or propose a patch.
- Never file an issue or edit another repo without explicit user approval.
- If the user invoked the skill with a specific complaint, focus there first but still notice other background friction.

## Working style

- Prefer evidence over vibes. Cite concrete moments from the session: retries, clarifications, fallbacks, stale outputs, empty outputs, mismatched docs, waits, or manual workarounds.
- Separate symptom from cause. "We had to retry the attach three times" is the symptom; "discovery was brittle on this platform" is the likely cause.
- Notice both direct and indirect friction.
  - Direct: the user says something was frustrating, confusing, slow, brittle, or surprising.
  - Indirect: repeated commands, repeated explanations, fallback to lower-level tools, manual reconstruction, hidden prerequisites, brittle environment assumptions, partial results, confusing contracts, or missing trust signals.
- Notice positive gaps too.
  - What almost worked?
  - What required too much expert knowledge?
  - What capability existed but was undiscoverable?
  - What should have been the default?
- Be creatively ambitious after the diagnosis is clear.
  - Start with grounded fixes supported directly by the session.
  - Then ask what would make this workflow feel nearly automatic, self-explaining, or hard to misuse.
  - Include 1-2 higher-upside ideas when warranted, even if they require reshaping the workflow rather than patching a local symptom.
  - Label speculative ideas clearly; creative does not mean vague.
- Stay focused on improving `re-frame-pair`. If the best fix belongs upstream in `re-frame`, `re-frame-10x`, `re-com`, or `shadow-cljs`, say so explicitly instead of forcing the proposal into the wrong repo.
- Read [references/analysis-lenses.md](references/analysis-lenses.md) when the session has multiple plausible causes or you need a sharper taxonomy.
- Read [references/known-frictions.md](references/known-frictions.md) when the session resembles a recurring class of `re-frame-pair` pain and you want to sanity-check whether it is a one-off or a pattern.

## Analysis workflow

1. Reconstruct the session goal.
   - Identify the user's intended outcome, not just the last command.
   - Capture the important environment facts: platform, target repo, live runtime state, and tooling constraints.

2. Build a short timeline.
   - List the turns or actions where progress stalled, restarted, detoured, or required a workaround.
   - Include tool errors, empty outputs, stale outputs, retries, and clarification loops.

3. Extract friction.
   - Present a numbered list of friction points first.
   - For each point, note:
     - what happened
     - where it showed up in the session
     - the initial category guess
   - Ask:
     - which of these should we dig into?
     - did I miss anything important?

4. Classify the root cause.
   - Work through these lenses briefly:

     | Lens | Question | Typical improvement |
     |------|----------|---------------------|
     | Skill structure | Was the right guidance present but buried or too low-signal? | Promote to guard rail, shorten, reorder, add examples |
     | Skill gap | Was key guidance missing entirely? | Add a new recipe, anti-pattern, or decision rule |
     | Misleading docs | Did the docs suggest the wrong action or wrong trust model? | Correct wording, add warnings, align contracts |
     | Missing structured op | Did the workflow need a first-class command or result shape? | Add a script/runtime op or a structured field |
     | Unreliable op | Did an existing op behave too ambiguously or too brittlely? | Fix behavior, add warnings, strengthen validation |
     | Default or fallback | Was the default path wrong, silent, or unsafe? | Change defaults or automate the safer fallback |
     | Platform bug | Did the workflow break on a specific shell, OS, or browser setup? | Add platform-aware handling or explicit detection |
     | Validation gap | Did this ship because the right fixture, smoke test, or regression test is missing? | Add test coverage or fixture support |
     | Upstream limitation | Is `re-frame-pair` working around the wrong abstraction boundary? | File or draft an upstream issue |
     | Context-window issue | Did long context or low-salience guidance cause forgetfulness? | Make the guard rail shorter, earlier, and stronger |

   - Prefer one primary cause per finding, but allow multiple contributing causes when needed.

5. Generate improvements at the right layer.
   - Consider several layers before choosing one:
     - tighten `SKILL.md` instructions or recipes
     - add or reshape a shell, script, or runtime op
     - enrich structured results, warnings, or failure modes
     - improve cross-platform behavior
     - add validation, fixture coverage, or smoke tests
     - expose new runtime instrumentation
     - file an upstream issue when the right fix is outside `re-frame-pair`
   - Prefer proposals that remove repeated effort, not just this session's exact symptom.
   - Ask a broader design question before finalizing ideas:
     - what should the tool have noticed automatically?
     - what manual decision or reconstruction step should disappear?
     - what would make this workflow feel obvious to a first-time user?
     - what would prevent this class of failure earlier?
   - Offer options when there is more than one credible path:
     - no action
     - docs or skill edit
     - tool/runtime change
     - issue draft
     - upstream escalation
   - If the workflow itself seems wrong, include a redesign option instead of only polishing the current path.

6. Prioritize.
   - Favor ideas that are:
     - high impact on common workflows
     - specific enough to implement
     - supported by session evidence
     - likely to improve trust, speed, or debuggability
   - Usually return 2-5 improvements, not a long brainstorm.
   - A good default mix is 1-3 grounded improvements plus 0-2 bolder ideas.

## Output format

Use a compact retrospective with these sections when the session has enough evidence:
- `Goal`
- `Observed friction`
- `Likely root causes`
- `Improvement ideas`
- `Bolder ideas` if there are credible higher-upside options worth separating from the grounded fixes
- `Issue candidates` if the user wants them
- `Other possibilities` if there were good lower-priority ideas

For each improvement idea, include:
- the friction it addresses
- why `re-frame-pair` was not enough
- the proposed change
- the likely layer of change: skill, script/runtime, tests/docs, or upstream
- a short impact statement

If the session is too thin, say so plainly and ask for either a recap or permission to use a longer conversation as input.

## Filing issues

Never file an issue without explicit user approval.

After presenting the retrospective, only offer issue work if it is useful:
- draft issue text
- file the issue directly, if GitHub tooling is available and the user asks
- split the ideas into multiple focused issues, if that would be cleaner

Before filing:
- redact secrets, tokens, internal URLs, and unnecessary local file paths
- avoid dumping the raw transcript; summarize the evidence instead
- search for an existing issue if the available tooling supports it
- prefer one issue per materially distinct improvement
- use [references/issue-template.md](references/issue-template.md) for structure

If direct filing is not possible, produce a ready-to-paste issue body and say that it is a draft, not a filed issue.

## What to avoid

- Do not reduce every problem to "write more docs". Consider product behavior, tooling, defaults, and instrumentation first.
- Do not confuse a transient local outage with a product gap unless the workflow made recovery harder than it should have.
- Do not propose vague improvements like "better UX" without naming the concrete missing behavior.
- Do not confuse creativity with hand-waving. Bold ideas still need a concrete change and a believable path to value.
- Do not force every retro into a code contribution or issue.
- Do not file speculative issues unsupported by the session.
- Do not pressure the user to file anything.

---
> Source: [day8/re-frame-pair-improver](https://github.com/day8/re-frame-pair-improver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
