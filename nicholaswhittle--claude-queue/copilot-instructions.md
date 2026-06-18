## claude-queue

> Run tasks in order — each step waits for the previous to finish, with results carried forward. Add more to the queue anytime, even while running. Use this skill whenever the user wants to run multiple things sequentially, says "do X then Y then Z", "run these in order", "queue these up", or lists multiple tasks that depend on each other.


# /queue — Sequential Prompt Execution

Execute multiple prompts **strictly in order**. Each step fully completes before the next begins. Results from earlier steps are passed forward so later steps have context.

## Parsing the input

Parse the user's input into an **ordered list of prompts**. Try these formats in order:

1. **Quoted strings**: `"prompt 1" "prompt 2" "prompt 3"` — split on quote boundaries. Handle escaped quotes inside (`\"` or `''`). Ignore whitespace between quotes.

2. **Numbered list**: Lines starting with `1.`, `2.`, `- `, `* ` etc. Strip the prefix; each line is one prompt.

3. **Newline-separated**: Each non-empty line is one prompt.

4. **Pipe-separated**: `prompt 1 | prompt 2 | prompt 3`

If parsing produces **zero prompts**, respond: "Usage: `/queue \"step 1\" \"step 2\" \"step 3\"`" and stop.

If parsing produces **one prompt**, just execute it directly — no queue needed. Do not spawn an agent, just do the work yourself.

## Execution loop

Before starting, briefly list all parsed steps so the user can see what will run:
```
Queue (3 steps):
  1. run the tests
  2. fix any failures
  3. run tests again to verify
```

Then for each step N of M:

### Step execution

1. **Create a task** via `TaskCreate`:
   - `subject`: first 80 chars of the prompt
   - `description`: full prompt text
   - `activeForm`: "Step N/M: <short description>"

2. If N > 1, **set dependency** via `TaskUpdate`: `addBlockedBy: [previous_task_id]`

3. **Set task to `in_progress`** via `TaskUpdate`

4. **Execute** by spawning a **foreground** `Agent` (subagent_type: "general-purpose"). The agent prompt MUST include:
   - The full prompt text for this step
   - "You are executing step N of M in a sequential queue."
   - If N > 1: a summary of what previous steps accomplished (the `prior_results` string below)
   - "Complete this task fully. When done, provide a clear summary of what you did and any outputs."
   - NEVER set `run_in_background: true`

5. **Capture the result** returned by the agent. Append to `prior_results`:
   ```
   Step N result: <agent's summary>
   ```

6. **Mark task `completed`** via `TaskUpdate`

7. Proceed to step N+1.

### On failure

If an agent returns an error or reports it could not complete:
- Mark the task as `completed` (with a note about failure — there is no "failed" status)
- **Stop the queue** — do not proceed to remaining steps
- Report:
  ```
  Queue stopped at step N/M: <reason>
  Completed: steps 1 through N-1
  Skipped: steps N+1 through M
  ```

## Slash commands in the queue

If a prompt starts with `/` (e.g., `/review`, `/ship`), tell the agent to invoke that skill via the Skill tool rather than treating the slash as literal text.

## Adding to a running queue

If the user sends another `/queue` command (or plain text) while a queue is already executing, treat the new items as additions to the existing queue, not a new queue.

**Placement:** The user can specify where new items go using flags or natural language:
- **Run next** (insert after currently running step): `/queue --next <tasks>`, or natural language like "do this next", "add to the top", "run this before the others", "prioritize this"
- **Run last** (append after all existing steps): `/queue --last <tasks>`, or natural language like "add to the end", "do this after", "add to the bottom", "also do this when you're done"
- **IMPORTANT: No indication — ALWAYS default to bottom** (append after existing steps). Never insert at the top unless the user explicitly asks for it.

### How it works

1. Check for existing in-progress queue tasks via `TaskList`. Look for tasks with `activeForm` matching "Step N/M:" that are not yet completed.
2. Determine the current step number and the last step number (M) from the existing queue.
3. **If bottom/last (default):** Number new steps starting from M+1. The first new step should `addBlockedBy` the last existing step.
4. **If top/next:** Renumber all not-yet-started steps to make room. Insert new steps right after the currently running step. The first new step should `addBlockedBy` the currently running step. The next original step should `addBlockedBy` the last inserted step.
5. Update M to the new total across all tasks.
6. Update `activeForm` on all tasks to reflect the new total (e.g., "Step 2/3" becomes "Step 2/5").

## Editing the queue

Users can modify **pending** (not-yet-started) steps via natural language: "remove step 3", "change step 2 to run lint", "swap step 2 and 3". Use `TaskUpdate` to update `subject`/`description`/`activeForm`, or `status: "deleted"` to remove. Rewire `addBlockedBy` dependencies and renumber `activeForm` after any change. Completed or in-progress steps cannot be edited.

## Mixed-placement parsing

If a single command has items for different positions (e.g., "add to top X and add to bottom Y"), split on placement keywords and process each segment with its own placement. Segments without a placement keyword default to bottom.

## Critical rules

- **NEVER use `run_in_background: true`** — this would parallelize and defeat the queue
- **NEVER start step N+1 before step N's agent returns and the task is marked completed**
- **ALWAYS pass prior step results forward** so later steps have context
- **ALWAYS use foreground agents** — they block until complete, which is the mechanism that enforces ordering
- If the user types while the queue is running, that input gets queued by the system — it does not interrupt the queue

## Final summary

After all steps complete (or the queue stops on failure), show:

```
Queue complete: N/M steps succeeded

Step 1: <subject> — done
Step 2: <subject> — done
Step 3: <subject> — skipped (step 2 failed)
```

---
> Source: [nicholaswhittle/claude-queue](https://github.com/nicholaswhittle/claude-queue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
