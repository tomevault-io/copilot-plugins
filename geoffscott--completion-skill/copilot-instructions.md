## completion-skill

> >


# completion

A task completion skill for OpenCLAW-based AI agents that maintains a single, consolidated
view of commitments across multiple roles, with role-weighted priorities and conflict surfacing.

## Core Concepts

### Roles

Most people split their attention across at least two domains — work and personal life —
and many have more (side projects, volunteer commitments, caregiving). Each role represents
a domain of responsibility with a default attention weight. Weights are relative — they
express how the user wants to allocate focus across roles when tasks compete for the same
time window.

The skill ships with two default roles (Work and Personal), but users can add, rename, or
remove roles at any time. When the skill is first initialized, prompt the user to confirm
or adjust the default roles. Common additions might include a side project, a volunteer
board, or a freelance practice.

### Tasks

Every task belongs to exactly one role. Tasks have:

- **title**: short description of what needs to happen
- **status**: one of `backlog`, `to-do`, `in_progress`, `blocked`, `done`
- **priority**: `p1` (must do), `p2` (should do), `p3` (nice to do) — within a role
- **role**: which role this task belongs to
- **due_date**: optional hard deadline (ISO 8601 date)
- **notes**: optional longer context, blockers, or next steps
- **created_at**, **updated_at**: timestamps managed automatically
- **tags**: optional comma-separated labels for cross-cutting concerns

Status meanings:
- `backlog` — captured and triaged, but not committed to yet. The "I know about this but not now" bucket.
- `to-do` — ready to work on. Picked from backlog, committed to doing soon. The short list of what's actually on deck—waiting for available capacity to start.
- `in_progress` — actively being worked on right now (today).
- `blocked` — waiting on someone or something external. Record who/what in notes.
- `done` — completed or intentionally abandoned. If dropped rather than finished, note why.

### Priority Across Roles

A `p1` in one role is not automatically more important than a `p1` in another. When tasks
at the same priority level compete, role weights inform which one surfaces first. But the
skill should never silently resolve these conflicts — instead, surface them to the user:

> "You have two P1s competing for attention: [Work task] and [Personal task].
> Work currently has higher weight. Want to keep that ordering, or does the personal
> task need to win this week?"

This is the most important design principle: **surface conflicts, don't resolve them.**

## Database

The skill uses a SQLite database stored at `~/.openclaw/completion/tasks.db`. Initialize it on first use
by running `scripts/init_db.py`. The script is idempotent — safe to run if the database
already exists.

Before any task operation, check if the database exists:

```bash
python3 /path/to/skill/scripts/init_db.py
```

Then interact with it using SQL via Python or the `sqlite3` CLI. The schema is documented
in `references/schema.sql`.

## Workflows

### Adding Tasks

When the user says something like "I need to..." or "remind me to..." or "add a task for...":

1. Extract the task title from what they said
2. Infer the role from context (ask if ambiguous)
3. Default status to `backlog` unless context suggests otherwise (e.g., "I need to do this today" → `to-do`)
4. Default priority to `p2` unless the user signals urgency
5. Confirm back to the user what was captured, including the inferred role and priority
6. Insert into the database

If the user rattles off several tasks at once, capture all of them and confirm the batch.

### Listing and Filtering Tasks

When the user asks "what's on my plate" or "show me my tasks" or similar:

1. Query the database, excluding `done` by default
2. Group by role
3. Within each role, sort by priority (p1 first), then by due date (soonest first)
4. Display with role weights so the user can see relative importance
5. Call out any conflicts: same-priority tasks across roles that compete

If the user asks for a specific role, filter to that role. If they ask for a specific
status (e.g., "what am I waiting on"), filter to that status.

Keep the output concise. Don't dump a wall of text — if there are many tasks, summarize
by role and offer to drill into any specific role.

### Updating Tasks

When the user says "mark X as done" or "move X to blocked" or "change priority of X":

1. Find the task (by title match, partial match, or ID if they specify one)
2. If ambiguous (multiple matches), show the options and ask which one
3. Update the record
4. Confirm the change

For status transitions, be conversational:
- Moving to `done`: "Nice — marked [task] as done."
- Moving to `blocked`: "Got it. Who or what are you waiting on?" (capture in notes)
- Moving to `backlog`: "Moved [task] back to the backlog."
- Moving to `to-do`: "On deck — [task] is in your to-do list."

### Daily Standup

When the user says "standup" or "what should I work on today" or "morning review":

Have a short conversation, not a data dump. Walk through this sequence:

1. **Carryover**: "Yesterday you had [X, Y] in progress. Did those close out, or are they carrying over?"
2. **Newly urgent**: Surface any tasks with approaching due dates or that recently moved to p1.
3. **Conflicts**: If p1 tasks across roles compete, surface the conflict.
4. **The one thing**: "Across everything, what's the one thing that moves the needle most today?"

The standup should feel like a 2-minute conversation with a thoughtful colleague, not a
report printout. Be brief. Ask questions. Let the user drive.

### Weekly Review

When the user says "weekly review" or "how did my week go" or "what's stuck":

Walk through three questions:

1. **What moved?** — Tasks that transitioned to `done` this week, grouped by role.
2. **What's stuck?** — Tasks that have been `in_progress` or `blocked` for a long time, or tasks
   that were expected this week but didn't get done. Call out any task that's been `blocked`
   for more than a week.
3. **What needs attention?** — Backlog items that might deserve promotion to `to-do`, roles that
   have no active tasks (might be getting neglected), upcoming due dates in the next two weeks.

Again: conversational, not a report. Surface observations, ask questions, let the user decide
what to do about each finding.



## Autonomous Rituals & Heartbeat Events

Beyond user-initiated workflows, the completion skill should run predictive health checks and surface insights automatically. These rituals apply kaizen (continuous improvement) and Theory of Constraints thinking to task flow.

### Morning Nudge (Optional, Configured)

**Trigger:** User's configured morning time, or at skill startup if user has active tasks.

**Purpose:** Confirm focus and surface immediate conflicts before the day starts.

**Interaction:**
1. Query tasks in `in_progress` and `todo` statuses
2. Show WIP count vs. configured limit (default: 2-3 per role)
3. Surface P1s across roles: "You have [count] P1s competing. Still good with the priority order?"
4. Ask: "What's the one thing that moves the needle most today?"
5. If multiple roles have urgent work: surface the conflict explicitly
6. Optional: "Any blockers from yesterday that need escalation?"

**Agent behavior:**
- Be brief (1-2 minutes)
- Only interrupt if there's genuine conflict or WIP overflow
- Don't repeat if user already ran standup today

### Stuck Alert (Daily or Per-Workflow)

**Trigger:** Check on every task operation; also run nightly if configured.

**Purpose:** Prevent tasks from rotting in `blocked` or `in_progress` limbo.

**Detection:**
1. **Stalled in progress** — tasks in `in_progress` > 3 days without update
2. **Blocked too long** — tasks in `blocked` > 5 days (alert at day 3: "Still waiting on [blocker]?")
3. **Backlog accumulation** — backlog growing 20%+ faster than completion rate this week
4. **Role starvation** — any role with zero `todo` or `in_progress` tasks

**Interaction:**
When stuck items detected, surface them conversationally:
> "You've got [Task X] waiting on [blocker] for 4 days. Want to bump it, drop it, or find another way to unblock?"

Or:
> "[Role] hasn't had any active work in 5 days — is that intentional, or do you want to pull something from the backlog?"

**Agent behavior:**
- Don't nag (once per day max)
- Offer concrete actions: escalate blocker, break task into smaller pieces, drop it, replan
- Link to entities if relevant ("Still waiting on Dani?")

### Weekly Kaizen (Friday or End-of-Week)

**Trigger:** User says "weekly review" or automatically Friday EOD (if configured).

**Purpose:** Move beyond "what got done" to "how can we improve the system."

**Core questions (after standard weekly review):**

1. **Flow health**
   - Planned vs. completed (how much of your committed `todo` actually finished?)
   - Average cycle time per priority (P1s taking longer than expected?)
   - Batch size (did you break work into manageable pieces, or did big tasks take all week?)

2. **Constraint analysis** (Theory of Constraints)
   - What was the bottleneck this week? (Blocked tasks, context switching, unclear scope?)
   - Is it the same bottleneck as last week?
   - Can we remove it, work around it, or should we accept it?

3. **Pattern detection**
   - Same person/system always blocking? → escalate or document workaround
   - P2 tasks consistently deprioritized by P1s? → re-examine role weights
   - Backlog items consistently re-estimated as harder? → improve estimation or slice smaller

4. **Improvement hypothesis**
   - "You shipped 60% of committed work this week. What would help get closer to 80%?"
   - Possible answers: smaller batches, clearer acceptance criteria, fewer interruptions, rebalance roles

**Interaction:**
Conversational, not a report. Ask questions and let the user drive:
> "You completed 8 of 10 committed tasks. The two that slipped were both [Role] P2s blocked by [external]. Is that a pattern, or a one-off?"

Then:
> "Want to adjust how we commit next week, or do you want to tackle the blocker differently?"

**Schema tracking:**
Update `status_history` to track:
- Timestamp of status transitions
- Days in each status (for cycle time calculation)
- Whether task was completed or moved back to backlog

### Role Rebalance (Weekly, After Kaizen)

**Trigger:** After weekly review, or if user says "rebalance roles."

**Purpose:** Align role weights to actual time allocation and changing circumstances.

**Analysis:**
1. Compare intended role weights to actual task completion by role
2. Check if priorities shifted mid-week (many P1→P2 moves? → role weight misaligned)
3. Surface changes in role circumstances (e.g., "Saranam board work doubled this month — should weight change?")

**Interaction:**
> "This week: Work was 60% of your time, Personal 30%, Saranam 10%.
> Currently weighted 1.5:1.0:0.5.
> Still happy with those weights, or should we adjust?"

**Agent behavior:**
- Suggest adjustments, don't enforce them
- Keep weights relative (1.0 as baseline)
- Document the change reason in metadata (e.g., "Q2 focus shift", "board meeting season")

## Configuration

Add optional user settings to `metadata.json`:

```json
{
  "rituals": {
    "morning_nudge": {
      "enabled": true,
      "time": "07:00",
      "wip_limit_per_role": 2,
      "only_on_conflict": false
    },
    "stuck_alert": {
      "enabled": true,
      "in_progress_threshold_days": 3,
      "blocked_threshold_days": 5,
      "check_frequency": "daily"
    },
    "weekly_kaizen": {
      "enabled": true,
      "day": "friday",
      "time": "17:00"
    },
    "role_rebalance": {
      "enabled": true,
      "after_review": true,
      "min_variance_percent": 10
    }
  }
}
```

User can enable/disable rituals and tweak thresholds as their workflow matures.

## Autonomy vs. Interruption

- **Don't be chatty.** Rituals surface one key insight or decision point per check, not a data dump.
- **Respect momentum.** Don't interrupt mid-task; nudges happen at natural boundaries (start of day, end of week).
- **Let the user lead.** Offer observations and questions; let them decide actions.
- **Learn from patterns.** Over time, track which rituals the user finds valuable and adjust frequency/detail accordingly.


While the completion skill is triggered by user requests (task CRUD, standup, reviews), it also runs autonomous rituals on a schedule defined in metadata.json and orchestrated by Ananda's heartbeat loop.

When the heartbeat is triggered, Ananda:

1. **Checks the current time and day** against ritual schedules in metadata.json
2. **Invokes the appropriate ritual function** from `scripts/rituals.py`
3. **Surfaces findings conversationally** if alerts are present
4. **Returns HEARTBEAT_OK** if all rituals are clear

### Morning Nudge Invocation

**Trigger:** Daily at configured time (default 07:00), or on agent startup if user has active tasks

**Command:** `python3 skills/completion/scripts/rituals.py morning`

**Agent behavior:**
- If WIP overflow or P1 conflict detected, present findings:
  > "Morning Nudge: [finding]. What's the one thing moving the needle today?"
- If all clear, no alert needed
- Don't repeat if user already ran standup today

### Stuck Alert Invocation

**Trigger:** Every heartbeat check, or nightly (configurable)

**Command:** `python3 skills/completion/scripts/rituals.py stuck`

**Agent behavior:**
- If stalled or blocked tasks detected, surface them:
  > "You have [task] blocked on [blocker] for 4 days. Want to escalate, replan, or drop it?"
- If role starvation detected:
  > "[Role] has no active work. Intentional, or should we pull from backlog?"
- Don't nag (max once per day)
- Offer concrete actions, not just warnings

### Weekly Kaizen Invocation

**Trigger:** Friday EOD (default 17:00), or when user explicitly says "weekly review"

**Command:** `python3 skills/completion/scripts/rituals.py kaizen`

**Agent behavior:**
- Present flow analysis conversationally (completion rate, cycle time, bottlenecks)
- Ask improvement question based on findings:
  > "You completed 70% of committed work. What would help get closer to 80%?"
- Let user lead; don't prescribe solutions
- Move to role_rebalance after kaizen if enabled

### Role Rebalance Invocation

**Trigger:** After weekly kaizen, or on explicit request ("rebalance roles")

**Command:** `python3 skills/completion/scripts/rituals.py rebalance`

**Agent behavior:**
- Compare intended vs. actual role weights:
  > "This week: Work 65%, Personal 30%, Saranam 5%. Weights are 60%/30%/10%. Still good?"
- Don't enforce changes; let user decide
- Document reason for any weight adjustments

### Error Handling & Fallback

- If database is missing or corrupted, initialize it: `python3 scripts/init_db.py`
- If ritual function raises an exception, log it and return no alert
- If metadata.json is malformed, use defaults
- All rituals degrade gracefully if data is sparse (e.g., no tasks yet)

### Extending Rituals

To add a new ritual:
1. Add function to `scripts/rituals.py` with signature: `def ritual_name(db_path=None, config=None) -> Optional[Tuple[str, str]]`
2. Add configuration defaults to `metadata.json`
3. Add invocation instructions here
4. Add to heartbeat check in HEARTBEAT.md



## Interaction Style

- Be concise. Task management is a utility, not a ceremony.
- Default to action. If the user says "I need to update the deployment docs," just capture
  it — don't ask five clarifying questions first. Infer sensible defaults and confirm.
- Surface conflicts but don't nag. Mention competing priorities once, clearly, then move on.
- Remember that the user has context you don't. If something seems off (a p3 with a deadline
  tomorrow), ask about it briefly rather than assuming it's wrong.
- When listing tasks, keep it scannable. Use brief formatting, not paragraphs.

## Error Handling

- If the database doesn't exist, initialize it silently and proceed.
- If a task query finds no matches, say so plainly: "I don't see a task matching [X]. Want
  to add it?"

## Entity Integration

The todo skill integrates with the shared entity registry at `~/.openclaw/entities.json`
to understand people, organizations, and projects mentioned in tasks. This enables
entity-aware task management — linking tasks to the people and contexts they involve.

### How It Works

1. **Entity Discovery**: On task creation, scan the title and notes for mentions of known
   entities (by any of their registered names/aliases). Match against `people`,
   `organizations`, and `projects` in the registry.

2. **Auto-Enrichment**: When entities are recognized, store their IDs in the task's
   `entities` field as a JSON array:
   ```json
   {"entities": ["dani-pascarella", "oneeleven"]}
   ```

3. **Entity-Based Queries**: Support natural queries like:
   - "Show me all tasks for OneEleven"
   - "What do I need to do with Dani?"
   - "Saranam tasks this week"
   - "What's open for Growth Science?"

   These translate to SQL queries filtering on the `entities` JSON field:
   ```sql
   SELECT * FROM tasks WHERE entities LIKE '%"oneeleven"%' AND status != 'done'
   ```

4. **Context-Aware Routing**: When creating tasks, use entity context to infer the
   correct role. For example, if a task mentions "Dani" (context: oneeleven), default
   to the Work role. If it mentions "Chai" (context: saranam/personal), default to
   Personal.

### Entity Enrichment Behavior

When the user says something like "Talk to Dani about Q2 planning":

1. Recognize "Dani" → entity `dani-pascarella` (CEO OneEleven)
2. Infer related org entity → `oneeleven`
3. Create task with `entities: ["dani-pascarella", "oneeleven"]`
4. Infer role → Work (based on entity contexts)
5. Later, "What do I need to do with Dani?" returns this task

When enriching, be conservative:
- Only attach entities you're confident about from the text
- Don't attach every possible entity — just the ones clearly referenced
- If unsure, create the task without entities rather than guessing wrong

### Reading the Entity Registry

Before enriching tasks, load the registry:

```python
import json, os
registry_path = os.path.expanduser("~/.openclaw/entities.json")
if os.path.exists(registry_path):
    with open(registry_path) as f:
        entities = json.load(f)
```

Build a lookup of all names/aliases → entity IDs for fast matching. The registry
contains `people`, `organizations`, and `projects`, each with a `names` array of
known aliases (including voice-to-text variants).

### Cross-Skill Integration

Entity references create natural bridges between skills:
- A task mentioning "Dani" can be cross-referenced with reflection entries about Dani
- Weekly reviews can surface entity patterns: "You had 5 tasks about OneEleven this week
  but none about Growth Science — is that intentional?"
- Standup can group tasks by entity when relevant: "Three open items with Dani"

---
> Source: [geoffscott/completion-skill](https://github.com/geoffscott/completion-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
