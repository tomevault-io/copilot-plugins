## ultra-search

> > **For Claude Code**: This document provides guidelines on how to effectively use Ultra Search tools, especially for long-running research tasks.

# Claude Code Integration Guide for Ultra Search

> **For Claude Code**: This document provides guidelines on how to effectively use Ultra Search tools, especially for long-running research tasks.

---

## Quick Decision Tree: Which Tool to Use?

```
Is this research expected to take > 5 minutes?
├─ YES → Use start_deep_research_async (returns immediately)
│         Then later: check_research_status to get results
│
└─ NO  → Use deep_research (blocks until complete)
          - quick depth: ~30 seconds - 2 minutes
          - standard depth: ~2-5 minutes
          - comprehensive depth: Can take 5-30+ minutes (use async!)
```

---

## Understanding Async Research

### The Problem
When research takes 10+ minutes, synchronous tools cause timeouts and poor UX:
- ❌ Claude Code blocks for the entire duration
- ❌ User can't interact with you during research
- ❌ If anything fails, they've wasted significant time
- ❌ Can't run multiple long research tasks in parallel

### The Solution: Async Task Pattern

**✅ Use async tools for long-running research:**

1. **Start** the research → Returns immediately with `task_id`
2. **User continues** working with you on other things
3. **Check status** later → Get progress and results when ready
4. **Results persist** in database and files

### Key Benefit

**Tasks can run INDEFINITELY** - there's no timeout limit! The research runs in a separate background process, so it continues even if:
- The user closes Claude Code and reopens later
- The user's computer goes to sleep (process pauses but resumes)
- The MCP connection resets

---

## Available Tools

### Synchronous Tools (Block Until Complete)

| Tool | Duration | Use When |
|------|----------|----------|
| `deep_research` | Variable | Quick or standard depth, < 5 minutes |
| `quick_answer` | 30-120 sec | Fast factual questions |
| `search_web` | 5-30 sec | Web search queries |
| `search_news` | 5-30 sec | News search queries |

### Async Tools (Return Immediately)

| Tool | Purpose |
|------|---------|
| `start_deep_research_async` | Start long-running research in background |
| `check_research_status` | Check progress/status of a task |
| `list_research_tasks` | See all tasks (running, completed, failed) |
| `get_research_result` | Get full results of completed task |
| `cancel_research_task` | Cancel a running task |

---

## Usage Patterns

### Pattern 1: Single Long Research Task

**User Request:**
> "I need comprehensive research on quantum computing error correction. Save it to quantum_research.md"

**Your Response:**

```markdown
I'll start a comprehensive deep research task on quantum computing error correction.
This will take approximately 30 minutes.

[Call: start_deep_research_async]
```

```python
start_deep_research_async(
    query="quantum computing error correction techniques and recent advances",
    depth="comprehensive",
    output_file="quantum_research.md"
)
```

**Response to User:**
```markdown
✓ Research task started (ID: task_a3f8d9c2)

This comprehensive research will take approximately 30 minutes. The results will
be automatically saved to `quantum_research.md` when complete.

You can check progress anytime by asking me to check on the task. I've noted the
task ID for reference.

What else would you like to work on in the meantime?
```

**Later - User asks:**
> "Check on that quantum research"

```python
check_research_status(task_id="task_a3f8d9c2")
```

**If still running:**
```markdown
Research task task_a3f8d9c2 is still running:
- Status: running
- Progress: 65%
- Started: 12 minutes ago
- Estimated remaining: ~18 minutes

I'll let you know when it's complete, or you can check again later.
```

**If completed:**
```python
get_research_result(task_id="task_a3f8d9c2")
```

```markdown
✓ Research completed!

Results saved to: quantum_research.md

Summary:
[Show summary from results]

The research found 47 sources and generated 8 follow-up questions.
Would you like me to explore any of the follow-up questions?
```

---

### Pattern 2: Multiple Parallel Research Tasks

**User Request:**
> "I need research on AI safety, quantum computing, and climate tech. Each one should be comprehensive."

**Your Response:**

```markdown
I'll start three comprehensive research tasks in parallel. Each will take
approximately 30 minutes.

[Call all three in parallel]
```

```python
# Start all three simultaneously
start_deep_research_async(
    query="AI safety research, alignment, and current challenges",
    depth="comprehensive",
    output_file="research/ai_safety.md"
)

start_deep_research_async(
    query="quantum computing advances and practical applications",
    depth="comprehensive",
    output_file="research/quantum_computing.md"
)

start_deep_research_async(
    query="climate technology solutions and innovations",
    depth="comprehensive",
    output_file="research/climate_tech.md"
)
```

**Response:**
```markdown
✓ Started 3 comprehensive research tasks:

1. AI Safety (task_x1y2z3) → research/ai_safety.md
2. Quantum Computing (task_a4b5c6) → research/quantum_computing.md
3. Climate Tech (task_d7e8f9) → research/climate_tech.md

All will complete in approximately 30 minutes and save automatically.

Would you like to work on something else while these run?
```

**Later - Check all:**

```python
list_research_tasks(status_filter="running")
```

```markdown
Active research tasks:

1. task_x1y2z3: AI safety research... (75% complete)
2. task_a4b5c6: quantum computing advances... (80% complete)
3. task_d7e8f9: climate technology solutions... (70% complete)

Estimated completion: 5-10 minutes
```

---

### Pattern 3: Opportunistic Status Checks

**Proactive checking** (if user seems idle or asks unrelated question):

```markdown
By the way, I can check on those research tasks if you'd like?

[If yes, call check_research_status or list_research_tasks]
```

---

## Decision Logic

### Use `start_deep_research_async` when:

- ✅ `depth="comprehensive"` is requested
- ✅ User mentions "detailed", "thorough", "in-depth"
- ✅ Research topic is complex/multi-faceted
- ✅ Expected duration > 5 minutes
- ✅ Multiple research tasks requested simultaneously
- ✅ User wants to do other work while researching

### Use `deep_research` (sync) when:

- ✅ `depth="quick"` or `depth="standard"`
- ✅ User wants immediate results
- ✅ Expected duration < 5 minutes
- ✅ Simple, focused question
- ✅ User says "quick" or "fast"

---

## File Output Integration

Both sync and async tools support file output:

```python
# Sync with file output
deep_research(
    query="AI trends",
    depth="standard",
    output_file="ai_trends.md"
)

# Async with file output
start_deep_research_async(
    query="Comprehensive market analysis",
    depth="comprehensive",
    output_file="reports/market_analysis.html",
    output_format="html"
)
```

**When task completes**, results are automatically written to the specified file.

---

## Error Handling

### Check for Errors

```python
status = check_research_status(task_id="task_abc123")

if status.status == "failed":
    print(f"Task failed: {status.error}")
    # Offer to retry or suggest alternative approach
```

### Cancelled Tasks

```python
cancel_research_task(task_id="task_abc123")
```

Only use when user explicitly requests cancellation:
- "Stop that research"
- "Cancel that task"
- "Never mind, I don't need that anymore"

---

## Best Practices

### 1. Always Capture task_id

Store the `task_id` from `start_deep_research_async` in your mental context:

```markdown
✓ Research started (task_a3f8d9c2) ← Remember this!
```

### 2. Provide Context

When starting async research, always tell the user:
- What research is starting
- Estimated duration
- Where results will be saved
- How to check status later

### 3. Proactive Updates

If user returns after some time:
```markdown
Welcome back! I can check on those research tasks from earlier if you'd like.
```

### 4. List Before Details

When checking multiple tasks, use `list_research_tasks` first:
```python
# Good: Get overview first
list_research_tasks(status_filter="running")

# Then get details for specific ones
check_research_status(task_id="task_abc123")
```

### 5. File Organization

Encourage good file organization:
```python
output_file="research/2026/feb/quantum_computing.md"  # Organized
# Not: output_file="output.md"  # Generic
```

---

## Common Scenarios

### Scenario 1: User Leaves and Returns

**User:** "Check on my research from earlier"

**You:**
```python
list_research_tasks(limit=10)
```

Show recent tasks and their status, then offer to get detailed results for completed ones.

### Scenario 2: Multiple Tasks, Some Complete

```python
tasks = list_research_tasks()

# Separate by status
completed = [t for t in tasks if t.status == "completed"]
running = [t for t in tasks if t.status == "running"]

# Report both
```

### Scenario 3: User Wants to Change Research Mid-Flight

**User:** "Actually, cancel that and research something else instead"

```python
# Cancel the old task
cancel_research_task(task_id="task_old123")

# Start new one
start_deep_research_async(
    query="new research topic",
    depth="comprehensive"
)
```

---

## Technical Details

### Task Persistence

- Tasks are stored in SQLite: `~/.ultra_search/tasks.db`
- Survives Claude Code restarts
- Can check tasks from days/weeks ago

### Background Execution

- Runs in separate Python process via `multiprocessing`
- Doesn't block Claude Code
- Continues even if MCP connection drops temporarily

### Progress Tracking

Progress percentages:
- 0%: Task created
- 10%: Task started
- 30%: Provider initialized
- 30-90%: Research in progress (provider-dependent)
- 90%: Saving results
- 100%: Complete

---

## Troubleshooting

### "Task not found"

- User provided wrong task_id
- Task was from a different session/database
- Suggest using `list_research_tasks()` to find it

### "Task still running after a long time"

- Check actual status with `check_research_status`
- Provider might be slow or stuck
- Offer to cancel and restart

### "Task failed"

- Show the error message
- Suggest retry with different parameters
- Or use different provider

---

## Examples for Different Depths

### Quick (30 sec - 2 min) → Use Sync

```python
deep_research(
    query="Who won the Nobel Prize in Physics 2025?",
    depth="quick"
)
```

### Standard (2-5 min) → Use Sync

```python
deep_research(
    query="Overview of CRISPR gene editing applications",
    depth="standard",
    output_file="crispr_overview.md"
)
```

### Comprehensive (10-30+ min) → Use Async

```python
start_deep_research_async(
    query="Comprehensive analysis of renewable energy technologies, costs, and adoption trends",
    depth="comprehensive",
    output_file="renewable_energy_report.md"
)
```

---

## Remember

1. **Async tools return immediately** - Don't wait for results
2. **Task runs in background** - User can do other things
3. **Check status later** - Not immediately after starting
4. **Results auto-save** to files when complete
5. **Tasks can run for hours** if needed - no timeout!

---

## Quick Reference

```python
# Start long research
start_deep_research_async(query="...", depth="comprehensive", output_file="...")

# Check one task
check_research_status(task_id="task_abc123")

# List all tasks
list_research_tasks()

# Get completed results
get_research_result(task_id="task_abc123")

# Cancel task
cancel_research_task(task_id="task_abc123")
```

---

**When in doubt:** If you think research might take > 5 minutes, use async!

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rlawless22)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rlawless22)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
