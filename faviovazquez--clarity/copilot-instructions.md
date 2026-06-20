## clarity

> >


# clarity

A knowledge mapping skill that makes the gap between what your agent built and
what you understand into a first-class, persistent artifact. Where `learnship`
manages the agent's memory, `clarity` manages yours — and helps you build it.

**Core principle:** "The code works" and "I understand this code" are not the same
statement. This skill makes the difference visible before it becomes a problem,
and helps fix it when it does.

Six actions:
- **map** — classify every module by how well you understand it
- **debt** — measure comprehension from a session, the full codebase, or a description
- **review** — autonomous agent scan of the codebase, no questions required
- **explain** — agent teaches you a specific module interactively
- **handoff** — generate or consume a context transfer document
- **status** — five-line project knowledge snapshot

Based on research cited in [references/cognitive-debt.md](references/cognitive-debt.md)
and [references/feynman-technique.md](references/feynman-technique.md).

---

## Actions

### `map` — Knowledge map

**Trigger:**
```
# Claude Code
/clarity map
/clarity map --quick
/clarity map --module <module>
/clarity map --explain

# Windsurf, Cursor, and others
@clarity map
@clarity map --quick
@clarity map --module <module>
@clarity map --explain
```

**What to do:**

1. **Scan the codebase.** Read the directory structure and identify top-level modules,
   components, or meaningful areas. If `AGENTS.md` exists, read it to understand
   decisions already documented. If `CLARITY_MAP.md` already exists, read it before
   updating — preserve prior scores and dates for unchanged modules.

2. **For each module or area** (skip unchanged modules if `--quick` is set):

   If `--explain` is set: before asking anything, give a 2-3 sentence summary of what
   the agent sees in this module — its apparent purpose, key patterns, and any notable
   complexity. This gives the user something to react to rather than starting from a
   blank page. Then proceed to Question A.

   Otherwise proceed directly to Question A.

   **Question A (what):** "Walk me through what `<module>` does — as if explaining it
   to someone joining the project today."

   **Question B (why):** "What was the key decision that shaped how this was built?
   Why that approach and not the obvious alternative?"

   Wait for each answer before moving to the next. Do not batch questions.

3. **Classify** each module into one of three zones based on the answers:
   - **Green (Understood):** The user explained what it does and why it is built that
     way, with no significant gaps or "I think" hedges.
   - **Yellow (Partial):** The user understood the what but was vague on the why, or
     could not articulate the key decision. Also assign Yellow when the user's
     explanation matches the code's surface behavior but misses the mechanism.
   - **Red (Risk zone):** The user could not explain what the module does, said
     "I'm not sure," said "the AI wrote it," or gave a description that contradicts
     what the code actually does.

4. **Write `CLARITY_MAP.md`** to the project root using the template at
   [templates/CLARITY_MAP.md](templates/CLARITY_MAP.md). Preserve all prior entries.
   Update only modules that were evaluated in this session.

5. **Generate or update `clarity-graph.html`** in the project root using the template
   at [templates/clarity-graph.html](templates/clarity-graph.html).
   Inject the current module data as JSON into the `CLARITY_DATA` variable. Each
   module entry must include:
   - `id` (string, module name slug)
   - `label` (string, display name)
   - `zone` ("green" | "yellow" | "red")
   - `lines` (approximate line count — use `wc -l` if available, estimate otherwise)
   - `last_evaluated` (ISO date string)
   - `what` (one sentence from the user's answer, or empty string)
   - `why` (one sentence on the key decision, or empty string)
   - `dependencies` (array of module id strings this module imports from or calls)

6. **Report** the full classification summary, highlighting any new Red zones.
   Tell the user to open `clarity-graph.html` for the visual map.

**Flags:**
- `--quick`: Re-evaluate only modules that are new, Red, or Yellow. Skip unchanged Green.
- `--module <module>`: Evaluate only the named module. Update its entry and regenerate graph.
- `--explain`: Before asking each question, give the agent's reading of the module so
  the user has a starting point. Useful when entering unfamiliar territory.

**Never** pre-classify modules without asking. The point is to surface what the user
actually knows, not what the agent infers.

---

### `debt` — Cognitive debt measurement

**Trigger:**
```
# Claude Code
/clarity debt
/clarity debt --scan
/clarity debt --session
/clarity debt --history
/clarity debt --threshold <0-100>

# Windsurf, Cursor, and others
@clarity debt
@clarity debt --scan
@clarity debt --session
@clarity debt --history
@clarity debt --threshold <0-100>
```

**How the source is chosen:**

`debt` has three modes. Choose automatically based on context unless a flag forces one:

1. **Diff mode** (default when git is available and there are recent commits):
   Run `git diff HEAD~1 HEAD`. If the diff is non-trivial (more than ~20 lines of
   meaningful logic), use it as the question source.

2. **Scan mode** (`--scan`, or auto-selected when no usable diff exists):
   Read the codebase directly. Identify the three areas of highest complexity,
   coupling, or risk — regardless of recent changes. These are the areas most likely
   to harbor silent cognitive debt. Ask questions about them. Use this when git is not
   initialized, the project is new, or when the user wants a general comprehension check.

3. **Session mode** (`--session`, or auto-selected when the user says what was built):
   Ask the user to briefly describe what was built or changed in this session. Use
   their description to form targeted questions. Useful mid-session before committing.

Regardless of mode, if `CLARITY_MAP.md` exists, read it first to understand which
modules are already Red or Yellow and prioritize questions around those areas.

**What to do (all modes):**

1. **Select source material** using the mode above.

2. **Select three areas** that represent meaningful logic — not boilerplate, config,
   or import changes. Prioritize:
   - Non-trivial functions or methods (more than 5-10 lines of logic)
   - Branching logic, error handling, or state transitions
   - Integration points between modules
   - Areas already classified as Red or Yellow in the map

3. **Ask three questions** derived from actual code, one at a time:

   **What question:** Point to a specific function or block. "What does
   `<function_name>` do? Walk me through it." Do not summarize the code first.

   **Why question:** Point to a specific decision in the code. "Here you're using
   `<approach>` rather than `<obvious_alternative>`. Why?" Pick a real decision, not
   a trivial one.

   **What-if question:** Point to a specific line or branch. "What happens to
   `<system_or_data>` if `<condition>` is true here?" Pick a failure mode or edge
   case that is non-obvious.

   Wait for each answer before asking the next.

4. **Score each answer** from 0 to 100:
   - 80-100: Complete, accurate, no significant hedges
   - 50-79: Partially correct or missing the key mechanism
   - 20-49: Vague, incorrect, or "I think the AI handled that"
   - 0-19: "I don't know" or no answer

5. **Calculate the session Comprehension Score**: average of the three question scores.

6. **Update `CLARITY_MAP.md`** under `## Cognitive Debt Log`:
   ```
   | <ISO date> | <score>/100 | <mode used> | <brief note on what was covered> |
   ```
   Mark ALERT if the score is below the threshold (default: 70).
   If the map does not exist, create a minimal one with only the debt log section.

7. **Update `clarity-graph.html`** if it exists: inject the new debt log data into
   the `CLARITY_DEBT_LOG` variable.

8. **Report** the session score, the 5-session running average, and a recommendation:
   - Score >= 70: Acknowledge and move on.
   - Score 50-69: Name the specific gaps. Suggest a `map --module` on the weakest area.
   - Score < 50: Flag plainly. Recommend a review session before continuing to build.
     Name which modules to address first.

**Flags:**
- `--scan`: Force scan mode — read the codebase and ask about the most complex areas,
  regardless of git state.
- `--session`: Force session mode — ask the user to describe what was built before
  forming questions.
- `--history`: Show the full debt log without running a new evaluation.
- `--threshold <n>`: Set the alert threshold for this session. Default is 70.

**Do not** be lenient in scoring. Technically correct but unreproducible-under-pressure
answers score 50, not 80.

---

### `review` — Autonomous codebase risk scan

**Trigger:**
```
# Claude Code
/clarity review
/clarity review --module <module>
/clarity review --deep

# Windsurf, Cursor, and others
@clarity review
@clarity review --module <module>
@clarity review --deep
```

**What to do:**

This action does not ask the user any questions. The agent reads the codebase and
produces a risk and complexity report on its own. Useful for:
- Joining an unfamiliar project for the first time
- Getting a quick read on which areas need `map` or `explain` first
- Generating a structural overview before a code review or refactor

1. **Read the codebase.** Scan the full directory structure. Read the files with the
   most logic (skip generated files, lockfiles, and assets). If `AGENTS.md` exists,
   read it. If `CLARITY_MAP.md` exists, read it — use it to anchor the review in
   already-known zones rather than starting from scratch.

2. **Identify risk signals** for each module:
   - **Complexity:** deeply nested logic, many branches, functions over 50 lines
   - **Coupling:** modules that import from many others, or are imported by many
   - **Opacity:** missing comments on non-obvious code, generic variable names,
     logic that depends on implicit side effects
   - **Churn risk:** modules that touch many concerns at once (auth + state + UI in
     one file, for example)
   - **Unverified AI patterns:** boilerplate-heavy code with a recognizable generation
     signature but no comments explaining why

3. **Classify each module** with an agent-estimated zone. Mark all zones with
   `(agent estimate)` — these are the agent's reading of the code, not user-verified
   comprehension. The zones mean the same thing, but the source is different:
   - **Green (agent estimate):** Code is readable, well-structured, intent is clear
   - **Yellow (agent estimate):** Some opacity or coupling — worth a closer look
   - **Red (agent estimate):** High complexity, high coupling, or significant opacity
     — the agent cannot confidently explain the intent from the code alone

4. **Write a `CLARITY_REVIEW.md`** to the project root using the template at
   [templates/CLARITY_REVIEW.md](templates/CLARITY_REVIEW.md). Include:
   - Review date and method (autonomous scan, no user input)
   - Module table: name, agent-estimated zone, primary risk signal, one-line note
   - Top 3 recommended next steps: which modules to `explain` or `map` first, and why
   - Any patterns that suggest systematic AI-generated code without comprehension

5. **Do not write to `CLARITY_MAP.md`** from this action. Agent estimates are separate
   from user-verified understanding. Tell the user how to promote estimates to verified
   zones: run `map --module <module>` on any module from the review.

6. **If `--deep` is set:** for each Red-estimated module, read it in full and produce
   a 3-5 sentence plain-language description of what it appears to do and what the
   primary risk is. Include this in `CLARITY_REVIEW.md`.

7. **Report** a short summary: how many modules were scanned, how many are estimated
   Red/Yellow/Green, and the top recommended action.

**Flags:**
- `--module <module>`: Review only the named module. Write findings to `CLARITY_REVIEW.md`.
- `--deep`: For each Red-estimated module, produce a detailed plain-language description.

**Important:** Always make the agent-estimate nature of this output explicit to the user.
A `review` is a starting point, not a substitute for `map`.

---

### `explain` — Agent teaches a module

**Trigger:**
```
# Claude Code
/clarity explain <module>
/clarity explain <module> --quiz

# Windsurf, Cursor, and others
@clarity explain <module>
@clarity explain <module> --quiz
```

**What to do:**

This action is the Feynman technique in reverse: instead of asking the user to
explain, the agent explains the module and the user asks questions. At the end,
the agent checks whether the explanation landed. Use this when:
- The user does not understand a module well enough to start `map`
- A module is classified Red and the user wants to address it
- A new team member needs to understand a specific area before the full map session

1. **Read the module in full.** Identify:
   - Its primary purpose (one sentence)
   - The key pattern or abstraction it uses
   - The inputs, outputs, and main side effects
   - Any non-obvious decisions or dependencies
   - Anything that looks complex or risky

2. **Give a layered explanation** in plain language:
   - Start with purpose: "This module is responsible for X."
   - Explain the mechanism: "It does this by Y. The key idea is Z."
   - Name the non-obvious part: "The part worth understanding carefully is..."
   - Name the risk or gotcha: "If you change X without knowing Y, you will likely..."

   Keep each layer short. Pause after each one: "Does that make sense so far? Any
   questions before I go deeper?"

3. **Answer questions from the user.** Adjust the depth of explanation based on
   what they ask. If they ask about something outside this module, note it briefly
   and stay focused.

4. **If `--quiz` is set:** after the explanation, ask the user three questions to
   check comprehension — the same format as `debt` (what, why, what-if), but derived
   from this specific module rather than a diff. Score the answers. If the average
   is 70 or above, mark this module Green in `CLARITY_MAP.md` (creating the file if
   it does not exist). If below 70, mark Yellow and note which question revealed the gap.

5. **Without `--quiz`:** do not classify or update the map. The session is informational.
   Suggest running `map --module <module>` when ready to record the comprehension formally.

6. **Tell the user** what they can do next: run `map --module <module>` to record their
   understanding, or run `debt --scan` for a full comprehension check.

**Flags:**
- `--quiz`: After explaining, run a three-question comprehension check and update
  `CLARITY_MAP.md` with the result.

---

### `handoff` — Context transfer

**Trigger:**
```
# Claude Code
/clarity handoff
/clarity handoff --import
/clarity handoff --sync
/clarity handoff --cold

# Windsurf, Cursor, and others
@clarity handoff
@clarity handoff --import
@clarity handoff --sync
@clarity handoff --cold
```

**What to do (default — export):**

1. **Check for `CLARITY_MAP.md`.** If it exists, read it in full — this is the
   preferred source. If it does not exist, do not stop: proceed to step 1a.

   **1a. No map exists:** Read the codebase directly. Identify the main modules,
   their apparent purpose, and any obvious risk signals (same signals as `review`).
   Produce a structural handoff based on the agent's reading. Mark all zone
   assessments as `(agent estimate — no map session on record)`. Tell the user at the
   top of the handoff that comprehension was not formally evaluated and recommend
   running `map` before or after the handoff.

2. **Read `AGENTS.md`** if it exists. Do not duplicate what is already there.
   The handoff captures the human knowledge layer — not the technical project state.

3. **Generate `CLARITY_HANDOFF.md`** using the template at
   [templates/CLARITY_HANDOFF.md](templates/CLARITY_HANDOFF.md). Fill in:
   - **Project snapshot date**: today's ISO date
   - **Source**: "user-verified map" or "agent estimate (no map session)"
   - **Red zones** (or high-risk agent estimates): with the user's own words from the
     map session, or the agent's reading if no map exists
   - **Yellow zones** (or medium-risk agent estimates): with the partial explanation
     or agent note
   - **Cognitive debt summary**: the running average score if a map exists, or "not
     evaluated" if the handoff is cold
   - **Open questions**: things flagged as "I'm not sure" in any session, or questions
     the agent cannot answer from the code alone
   - **What the next person should do first**: 2-3 concrete recommendations, always
     including which `clarity` actions to run first

4. **Tell the user** the file was written and where it is. Suggest committing it.

**What to do (`--import` — joining an existing project):**

1. **Check for `CLARITY_HANDOFF.md`.** If not found, check for `CLARITY_MAP.md`. If
   neither exists, offer to run `review` to produce a structural overview, then use
   it as the basis for onboarding. Do not stop.

2. **Read the handoff or map fully.** Then guide the new user through structured
   onboarding:
   - Present Red zones first: "These are the areas flagged as not fully understood."
     For each, give a brief plain-language explanation of what the agent sees in the
     code. Ask: "Does that make sense? Questions before we move on?"
   - Present Yellow zones: same treatment, lighter touch.
   - Present open questions: "These were unresolved. Some may still be."
   - At the end, offer to run `debt --scan` to establish a baseline Comprehension Score.

3. **Write a session entry** to `CLARITY_MAP.md` (creating it if needed) under
   `## Onboarding Session`, noting the date and any new understandings.

**What to do (`--sync`):**

After an onboarding or pairing session, re-evaluate any modules that changed status.
Run `map --quick` internally and update `CLARITY_MAP.md` and the graph.

**What to do (`--cold`):**

Produce a handoff with no existing map and minimal user input. The agent reads the
full codebase and generates a complete `CLARITY_HANDOFF.md` autonomously — using the
same signals as `review`. Mark everything as agent-estimated. Useful when a developer
is leaving and there is no time for a full map session.

---

### `status` — Project knowledge snapshot

**Trigger:**
```
# Claude Code
/clarity status

# Windsurf, Cursor, and others
@clarity status
```

**What to do:**

1. Check for `CLARITY_MAP.md` and `CLARITY_REVIEW.md`. Read whichever exists.
   If neither exists, say so and recommend starting with `review` for a quick read
   or `map` for a formal evaluation.

2. Report a concise summary:

```
CLARITY STATUS — <project name or directory>
Last evaluated: <date of most recent map session, or "never">

Knowledge zones (user-verified):
  Green  (understood):  <n> modules
  Yellow (partial):     <n> modules
  Red    (risk):        <n> modules — <list their names>

Agent-estimated zones (from review, not user-verified):
  <n> modules with estimates — run map or explain to verify

Cognitive debt:
  Last session score:       <score>/100  (<mode>)
  5-session average:        <avg>/100
  Sessions below threshold: <n>

Last handoff:  <date, or "never">
Last review:   <date, or "never">

Recommended action: <one concrete next step>
```

3. If there are Red zones and the last map session was more than 14 days ago,
   note that the map may be stale.

4. Do not speculate about zones for unevaluated modules. Mark them "not evaluated."

---

## Files written to the project

All files are written to the project root. `clarity` never writes to the skill
directory itself.

```
CLARITY_MAP.md      knowledge map — user-verified zones, debt log, open questions
CLARITY_REVIEW.md   agent-estimated risk scan — written by review, not map
CLARITY_HANDOFF.md  context transfer document — written by handoff
clarity-graph.html  interactive visual map — generated by map and debt
```

These files are yours. Commit them, share them, review them in retrospectives.
The graph is a single static HTML file — no server, no build step.

---

## How clarity relates to learnship

`learnship` manages the agent's memory: persistent context, structured phases,
workflow state across sessions. `clarity` manages the developer's memory: what
you understand, where the gaps are, and how to build and transfer that.

They are complementary but independent. `clarity` reads `AGENTS.md` if it exists
to avoid duplicating what `learnship` already tracks, but does not require it.

A natural rhythm with both: build a phase with `learnship`, run `debt` at the end,
run `map` before any phase where you are entering unfamiliar territory, run `review`
when onboarding someone new.

---

## When to use each action

| Situation | Claude Code | Windsurf / others |
|-----------|-------------|-------------------|
| Starting a new phase or returning after a break | `/clarity map` | `@clarity map` |
| End of a long AI-assisted build session | `/clarity debt` | `@clarity debt` |
| No git, or want a general comprehension check | `/clarity debt --scan` | `@clarity debt --scan` |
| Joining or scanning an unfamiliar codebase | `/clarity review` | `@clarity review` |
| Need to understand a specific module | `/clarity explain <module>` | `@clarity explain <module>` |
| Before a code review — see the risk zones | `/clarity status` | `@clarity status` |
| Someone new is joining the project | `/clarity handoff` | `@clarity handoff` |
| No time for a full map before handoff | `/clarity handoff --cold` | `@clarity handoff --cold` |
| You are the person joining an existing project | `/clarity handoff --import` | `@clarity handoff --import` |
| After an onboarding session | `/clarity handoff --sync` | `@clarity handoff --sync` |

---

## What clarity is not for

- **Brainstorming or new ideas.** Use `@agentic-learning brainstorm` or `/deliberate`.
- **Learning a new concept.** Use `@agentic-learning learn`.
- **Deciding between two approaches.** Use `/deliberate`.
- **Tracking code quality.** Use your linter and test suite.

`clarity` has one job: make the human's understanding of the project a
first-class artifact, as persistent and inspectable as the code itself.

---
> Source: [FavioVazquez/clarity](https://github.com/FavioVazquez/clarity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
