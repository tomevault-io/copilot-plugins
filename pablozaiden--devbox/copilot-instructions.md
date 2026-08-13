## devbox

> <!-- ralpher-optimized-v1 -->

<!-- ralpher-optimized-v1 -->
## Agentic Workflow — Planning & Progress Tracking

When working on tasks, follow this workflow to ensure clarity, goal alignment, and resilience to context loss:

### Planning

- At the start of any multi-step task, write your goals and plan in `./.planning/plan.md`.
- Track the status of each task in `./.planning/status.md`.
- Make sure that goals are written down in a way that you can properly verify them later.
- Don't say something is done until you have verified that all goals are met.
- **Never start implementation before the plan is confirmed.** Present the plan to the user and wait for explicit approval before writing any code. If the plan needs changes, revise and re-confirm before proceeding.

### Incremental Progress Tracking

- After completing each individual task, **immediately** update `./.planning/status.md` to mark it as completed and note any relevant findings or context.
- Do **not** wait until the end of a session to batch-update progress — update after every task so that progress is preserved even if the session is interrupted or context is lost.

### Pre-Compaction Persistence

- Before ending your response, update `./.planning/status.md` with:
  - The task you are currently working on and its current state
  - Updated status of all tasks in the plan
  - Any new learnings, discoveries, or important context gathered
  - What the next steps should be when work resumes
- This ensures progress is preserved even if the conversation context is compacted or summarized between iterations. Treat the status file as your persistent memory.

### Goal Verification

- Before considering work complete, check `./.planning/plan.md` and `./.planning/status.md` to ensure all tasks are actually marked as completed.
- Follow this general loop:
  1. Write down goals in the plan
  2. Implement the work
  3. Verify all goals are met
  4. Update status with progress
  5. If all goals are met, you are done; otherwise, continue from step 2

---
> Source: [PabloZaiden/devbox](https://github.com/PabloZaiden/devbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
