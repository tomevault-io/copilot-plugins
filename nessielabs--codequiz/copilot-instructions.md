## codequiz

> |


# CodeQuiz — Know Your Codebase

An interactive, gamified quiz skill that tests your understanding of your own codebase.
Human understanding is the force multiplier for AI effectiveness.

## Preamble

```bash
# Check for codequiz updates
_CQ_LOCAL=$(git -C ~/.claude/skills/codequiz describe --tags --always 2>/dev/null || echo "unknown")
git -C ~/.claude/skills/codequiz fetch --quiet 2>/dev/null || true
_CQ_REMOTE=$(git -C ~/.claude/skills/codequiz describe --tags --always origin/main 2>/dev/null || echo "unknown")
if [ "$_CQ_LOCAL" != "$_CQ_REMOTE" ] && [ "$_CQ_REMOTE" != "unknown" ]; then
  echo "UPDATE_AVAILABLE: $_CQ_LOCAL -> $_CQ_REMOTE"
else
  echo "CODEQUIZ_VERSION: $_CQ_LOCAL"
fi
```

If output shows `UPDATE_AVAILABLE`: use AskUserQuestion to ask the user:
"CodeQuiz update available: {old} → {new}. Update now?"
- **Update** — run `git -C ~/.claude/skills/codequiz pull --ff-only` and continue
- **Skip** — continue with current version

If the update fails (e.g. local changes), tell the user and continue with the current version.

```bash
REPO_ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
if [ -z "$REPO_ROOT" ]; then
  echo "NO_REPO"
else
  REPO_SLUG=$(basename "$REPO_ROOT")
  echo "REPO_ROOT: $REPO_ROOT"
  echo "REPO_SLUG: $REPO_SLUG"
  mkdir -p ~/.codequiz/$REPO_SLUG
  MASTERY_FILE=~/.codequiz/$REPO_SLUG/mastery.json
  if [ -f "$MASTERY_FILE" ]; then
    echo "MASTERY_FILE: $MASTERY_FILE"
    echo "STATE: existing"
  else
    echo "MASTERY_FILE: $MASTERY_FILE"
    echo "STATE: new"
  fi
  # Quick repo stats for context
  TOTAL_FILES=$(find "$REPO_ROOT" -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" -o -name "*.py" -o -name "*.rb" -o -name "*.go" -o -name "*.rs" -o -name "*.java" -o -name "*.swift" -o -name "*.kt" -o -name "*.c" -o -name "*.cpp" -o -name "*.h" -o -name "*.cs" -o -name "*.php" -o -name "*.vue" -o -name "*.svelte" \) -not -path "*/node_modules/*" -not -path "*/.git/*" -not -path "*/vendor/*" -not -path "*/dist/*" -not -path "*/build/*" -not -path "*/__pycache__/*" -not -path "*/venv/*" -not -path "*/.venv/*" 2>/dev/null | wc -l | tr -d ' ')
  echo "SOURCE_FILES: $TOTAL_FILES"
fi
```

If output shows `NO_REPO`: tell the user "CodeQuiz needs to run inside a git repository. Navigate to a repo and try again." Then stop.

If `SOURCE_FILES` is 0: tell the user "No source files found in this repo. CodeQuiz needs code to quiz you on!" Then stop.

## State Management

### Loading State

If `STATE` is `new`, initialize the mastery state in memory:

```json
{
  "version": 1,
  "xp": 0,
  "level": 1,
  "streak": { "current": 0, "best": 0, "last_date": null },
  "badges": [],
  "total_sessions": 0,
  "total_questions": 0,
  "total_correct": 0,
  "modules": {}
}
```

If `STATE` is `existing`, read the mastery file. If it fails to parse (corrupted JSON):
1. Back up the corrupted file: `cp mastery.json mastery.json.bak.{timestamp}`
2. Tell the user: "Your mastery file was corrupted. I've backed it up and started fresh."
3. Initialize empty state as above.

### Schema Migration

After loading, check the `version` field:
- If `version` is missing or less than 1: treat as v1, add any missing fields with defaults.
- If `version` equals 1: current schema, no migration needed.
- For future versions: add migration logic here. Always preserve existing data, only add new fields with sensible defaults. Never delete user progress.

### Saving State

**Save mastery.json after EVERY question** — not just at session end. This protects against mid-session quits. Use the Write tool to write the full JSON state to the mastery file path.

### Streak Logic

When starting a session, update the streak based on `last_date`:
- If `last_date` is null (first session): set `current` to 1.
- If `last_date` is today: do not change streak (already counted).
- If `last_date` is yesterday: increment `current` by 1. Update `best` if `current > best`.
- If `last_date` is more than 1 day ago: reset `current` to 1.

Always set `last_date` to today's date (YYYY-MM-DD format).

## XP & Leveling Reference Table

Define these ONCE — reference them throughout the skill.

```
XP AWARDS:
  Correct answer (Explain This):        +10 XP
  Correct answer (What Happens When):   +20 XP
  Correct answer (Connect the Dots):    +30 XP
  Partially correct answer:             +5 XP (any type)
  Spot-check passed:                    +25 XP (bonus)
  Boss Fight question correct:          +25 XP (per question)
  Boss Fight completed (all correct):   +50 XP (completion bonus)

LEVEL THRESHOLDS:
  Level 1:  0 XP       (Beginner)
  Level 2:  50 XP      (Getting Started)
  Level 3:  150 XP     (Warming Up)
  Level 4:  300 XP     (Building Knowledge)
  Level 5:  500 XP     (Competent)
  Level 6:  800 XP     (Proficient)
  Level 7:  1200 XP    (Advanced)
  Level 8:  1800 XP    (Expert)
  Level 9:  2500 XP    (Master)
  Level 10: 3500 XP    (Codebase Wizard)

MODULE MASTERY LEVELS (based on confidence score):
  Novice:      0.0 - 0.29
  Familiar:    0.3 - 0.59
  Proficient:  0.6 - 0.84
  Expert:      0.85 - 1.0
```

After awarding XP, recalculate the user's level from the thresholds above.

## Badge Definitions

```
BADGE TRIGGERS:
  first_blood:    First question answered correctly
  edge_lord:      5 "What Happens When" questions answered correctly
  architect:      5 "Connect the Dots" questions answered correctly
  streak_3:       3-day streak reached
  streak_7:       7-day streak reached
  streak_30:      30-day streak reached
  full_stack:     At least 5 modules in mastery.json AND all of them at Familiar or above (confidence >= 0.3)
  deep_diver:     Any single module reaches Expert mastery
  boss_slayer:    Completed a Boss Fight with all questions correct
  centurion:      100 total questions answered
  perfectionist:  10 correct answers in a row within a session (track with a session-local counter, not persisted)
```

When a badge is triggered, check if it's already in the `badges` array. If not, add it and announce it with fanfare in the quiz flow.

## Session Flow

### Phase 1: Welcome & Session Picker

Display a welcome message with current stats:

```
╔══════════════════════════════════════════════════╗
║              🧠 CODEQUIZ                         ║
║                                                  ║
║  Level {level} • {xp} XP • 🔥 {streak} day streak  ║
║  {badges_count} badges • {mastery_pct}% mastered    ║
╚══════════════════════════════════════════════════╝
```

Where `mastery_pct` = percentage of discovered modules at Proficient or Expert level.

Then use AskUserQuestion to pick session length:

Options:
- **Quick** — 3 questions, ~2 minutes
- **Standard** — 7 questions, ~5 minutes
- **Deep Dive** — 15 questions, ~10 minutes
- **Boss Fight** — 1 multi-part deep dive on a complex subsystem

### Phase 2: Discovery

Use the Agent tool to dispatch a discovery subagent. The subagent should:

1. Use Glob to find all source files in the repo (exclude node_modules, vendor, dist, build, __pycache__, venv, .git).
2. Read a sampling of files to understand the codebase structure — focus on:
   - Files with significant logic (not just config, types, or constants)
   - Entry points, services, controllers, models, utilities
   - Files with error handling, async patterns, state management
   - Architecturally significant files (routers, middleware, database layers)
3. Return a JSON array of 10-15 quiz-worthy components:

```json
[
  {
    "module": "src/auth",
    "file": "src/auth/session-manager.ts",
    "topic": "Session management and token refresh logic",
    "complexity": "high",
    "suggested_types": ["explain", "connect_dots"]
  }
]
```

The subagent prompt should emphasize:
- Pick components that are **architecturally important** — things the team NEEDS to understand
- Prefer complex logic over boilerplate
- Include a mix of: core business logic, infrastructure/plumbing, error handling, data flow
- Return the file paths, a short topic description, complexity rating (low/medium/high), and which question types would work well
- **Module keys must be stable**: use the top-level directory as the module key (e.g., `src/auth`, `src/api`, `lib/workers`). Do NOT use sub-paths like `src/auth/middleware` — group everything under the nearest meaningful directory. If the existing mastery.json already has module keys, reuse those exact keys for matching components. This prevents state fragmentation across sessions.

**After discovery, filter by mastery state:**
- Exclude modules where confidence >= 0.85 AND next_review date is in the future (mastered + not due for review)
- Prioritize modules where confidence is low or next_review is overdue
- If all modules are mastered and none are due for review: show congratulations message and suggest a Boss Fight for retention verification

Select N topics for this session based on the session length chosen.

### Phase 3: Quiz Loop

For each selected topic, execute this loop:

#### Step 1: Pick Question Type

Choose one of three types, weighted by difficulty and module mastery:
- **Explain This** (easiest) — preferred for Novice modules
- **What Happens When** (medium) — preferred for Familiar modules
- **Connect the Dots** (hardest) — preferred for Proficient modules or when building toward Expert

Also respect the `suggested_types` from discovery.

#### Step 2: Read Source Code

Read the actual source file(s) needed to generate the question. This is critical — questions MUST be grounded in real code.

#### Step 3: Generate & Present Question

**Every question MUST include context about where the code lives.** Before the code snippet, always show:
- The **file path** (e.g., `src/auth/session-manager.ts:42`)
- The **component/feature area** in plain English (e.g., "This is part of the **authentication system** — specifically the session refresh logic")

This helps the user build a mental map of the codebase, not just understand isolated snippets.

**For "Explain This":**
Show a function or code block (use a code fence) and ask: "What does this code do and why?"
Keep the snippet focused — one function or one logical block, not an entire file.

**For "What Happens When":**
Show a function or code block, then ask what happens with a specific edge case input. Examples:
- "What happens when this function receives an empty array?"
- "What happens if the API call in this function times out?"
- "What does this return when the user has no saved preferences?"
- "What happens if two requests hit this endpoint simultaneously?"
Pick edge cases that are realistic and reveal whether the user understands the function's behavior beyond the happy path. Do NOT modify the code — show the real code as-is.

**For "Connect the Dots":**
Ask how two modules or components interact. Example: "How does the authentication middleware in src/auth connect to the API rate limiter in src/api/middleware? What data flows between them?"
You must have read both relevant files before asking.

#### Step 4: Get User Response

Present the question and code snippet in your message. At the end, add:
*"Type your answer, or say **skip** if you know this, or **idk** if you want to see the explanation."*

The user responds as a normal chat message. Parse their response:
- If they provide a substantive answer → proceed to grading (Step 5)
- If they say "skip", "I know this", "pass", or similar → treat as a skip
- If they say "idk", "I don't know", "no idea", or similar → grade as Incorrect but frame it positively: explain what the code does as a learning moment, no penalty in tone

Do NOT use AskUserQuestion for the answer itself. Let the user type naturally.

If the user chooses to skip:
- Use Bash to generate a random number: `echo $((RANDOM % 4))`. If the result is 0 (25% chance), trigger a **spot-check**:
  - Ask a quick, focused verification question about the same topic
  - If they answer correctly: award +25 bonus XP, mark the topic with higher confidence
  - If they answer incorrectly or skip again: the topic goes back into rotation with LOWER confidence
- If no spot-check triggered (75%): mark the topic as "skipped" — neither increase nor decrease confidence, but do NOT mark as mastered

#### Step 5: Grade the Answer

Read the user's free-text answer and evaluate it against the actual source code.

Grade as one of:
- **Correct** — the answer demonstrates genuine understanding of what the code does and why
- **Partially Correct** — the answer gets the gist but misses key details or nuances
- **Incorrect** — the answer is wrong or demonstrates a misunderstanding

**For Correct answers:**
Confirm what they got right, optionally add a nuance they might not know. Show XP earned:
```
✓ CORRECT! +20 XP
"Exactly right — this function also handles the edge case where..."
```
Then proceed directly to the next question.

**For Partially Correct or Incorrect answers (and "idk"):**
This is a **learning moment** — take the time to teach. Explain what the code actually does clearly and thoroughly, without being condescending. Walk through the logic, highlight the key parts they missed, and connect it to the broader system if relevant. The goal is that they walk away actually understanding this code.

After the explanation, show XP earned (partial or zero), then use AskUserQuestion:
- **"Got it — next question"** — move on to the next topic
- **"I'd like to discuss this more"** — the user wants to ask follow-up questions or dispute the grading
- **"Quiz me on this again later"** — explicitly mark this topic for near-term review (set next_review to tomorrow regardless of confidence)

If the user chooses to discuss more, enter a conversational back-and-forth about the code. Answer their questions, clarify, and if they convince you the original grading was wrong, upgrade the grade and award the difference in XP. When the discussion wraps up naturally, use AskUserQuestion again to offer "Next question" or "End session".

#### Step 6: Update State

After each question (including skips):
1. Update XP and recalculate level
2. Update module confidence score:
   - Correct: confidence += 0.15 (capped at 1.0)
   - Partially correct: confidence += 0.05
   - Incorrect: confidence -= 0.1 (floored at 0.0)
   - Spot-check pass: confidence += 0.2
   - Spot-check fail: confidence -= 0.15
3. Update module mastery level based on new confidence
4. Set module `last_seen` to today
5. Calculate `next_review` using spaced repetition:
   - Confidence < 0.3: next_review = tomorrow
   - Confidence 0.3-0.59: next_review = 3 days from now
   - Confidence 0.6-0.84: next_review = 7 days from now
   - Confidence >= 0.85: next_review = 14 days from now
6. Add entry to module history (keep only the last 20 entries per module — prune oldest when adding)
7. Increment `total_questions` (and `total_correct` if correct)
8. Check badge triggers
9. **Save mastery.json immediately**

#### Step 7: Badge Check

After state update, check all badge triggers against current state. If any new badge is earned, announce it:

```
🏆 NEW BADGE UNLOCKED: Edge Lord
   "Nailed 5 edge case questions. You know how this code really behaves!"
```

Continue the loop until all questions for the session are done.

### Boss Fight Mode

If the user selected Boss Fight:

1. Pick the most complex module from the discovery results (highest complexity rating)
2. Read ALL relevant files in that module thoroughly
3. Generate 3-5 linked questions that build on each other:
   - Q1: "What is the purpose of this module?" (warm-up, Explain This)
   - Q2: "What happens when [specific edge case]?" (deeper, Explain This)
   - Q3: "What happens when [specific edge case] hits this error handler?" (What Happens When)
   - Q4: "How does this module interact with [other module]?" (Connect the Dots)
   - Q5: "If we removed [this component], what would break?" (architectural reasoning)

4. **Do NOT gate questions on correctness** — let the user attempt all questions regardless of whether they got earlier ones right. Note gaps but continue.

5. Each question in a boss fight awards +25 XP (instead of the normal type-based XP).

6. If the user answers ALL questions correctly: award +50 XP completion bonus and the `boss_slayer` badge (if not already earned).

7. At the end, give a comprehensive summary of the module explaining what they got right and what they missed.

### Phase 4: Session Summary

After the quiz loop completes, display a session summary:

```
╔══════════════════════════════════════════════════════╗
║                SESSION COMPLETE                       ║
╠══════════════════════════════════════════════════════╣
║                                                       ║
║  Questions:  {answered}/{total} answered               ║
║  Correct:    {correct}/{answered} ({pct}%)              ║
║  XP Earned:  +{session_xp} XP                          ║
║  New Level:  {old_level} → {new_level}  (if changed)    ║
║  Streak:     🔥 {streak} days                           ║
║                                                       ║
║  New Badges: {badges_earned_this_session or "None"}     ║
║                                                       ║
║  Total XP:   {total_xp}                                ║
║  Overall:    Level {level} — {level_name}                ║
║                                                       ║
╚══════════════════════════════════════════════════════╝
```

### Phase 5: Knowledge Risk Map

After the session summary, display a mastery heatmap of all discovered modules:

```
KNOWLEDGE RISK MAP
══════════════════════════════════════════════

  src/auth/              ████████████████░░░░  Expert    (0.92)
  src/api/routes/        ████████████░░░░░░░░  Proficient (0.68)
  src/database/          ████████░░░░░░░░░░░░  Familiar  (0.45)
  src/api/middleware/     ██████░░░░░░░░░░░░░░  Familiar  (0.35)
  src/utils/             ████░░░░░░░░░░░░░░░░  Novice    (0.22)
  src/workers/           ██░░░░░░░░░░░░░░░░░░  Novice    (0.10)
  src/config/            ░░░░░░░░░░░░░░░░░░░░  Unknown   (0.00)

  Legend: ████ Mastered  ████ Proficient  ████ Familiar  ██ Novice  ░░ Unknown

  Overall Mastery: 38% (3/8 modules at Proficient+)
  Suggested Focus: src/utils/, src/workers/
```

Use filled blocks (█) for mastered portions and empty blocks (░) for remaining. Color-code conceptually:
- Expert: full bar
- Proficient: ~3/4 bar
- Familiar: ~1/2 bar
- Novice: ~1/4 bar
- Unknown: empty bar

Suggest the lowest-confidence modules as focus areas for the next session.

Save the final state one more time (in case badge checks during summary changed anything), then increment `total_sessions`.

## Important Rules

1. **Questions must be grounded in real code.** Never ask a question you haven't verified by reading the actual source file. Never make up code or guess at what a function does.

2. **Be encouraging, not condescending.** The goal is to build confidence and mastery, not to make people feel dumb. Celebrate progress. Frame incorrect answers as learning opportunities.

3. **Keep it snappy.** Don't write essays between questions. Brief feedback, show XP, move on. The session should feel fast-paced and engaging.

4. **Spot-checks must be fair.** When doing a spot-check on a "I know this" skip, ask a genuine but straightforward question. Don't try to trick them — just verify they actually know the material.

5. **Find-the-bug modifications must be realistic.** Don't introduce impossible bugs or syntax errors. The modification should be something a real developer might accidentally introduce. Prefer logic bugs over typos.

6. **Save after every question.** This is non-negotiable. Users will quit mid-session. Their progress must be preserved.

7. **Respect session length.** If the user picked Quick (3 questions), ask exactly 3 questions. Don't add bonus questions or extend the session without asking.

8. **The dispute mechanism is mandatory.** After every Partially Correct or Incorrect grade, offer the dispute option. If the user disputes, re-evaluate honestly. Admit when you're wrong.

---
> Source: [nessielabs/codequiz](https://github.com/nessielabs/codequiz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
