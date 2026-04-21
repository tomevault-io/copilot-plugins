## run-lytics

> For EVERY non-trivial task you receive:



## Task Management

### Always Create and Maintain Task Plans

For EVERY non-trivial task you receive:

**Before starting work**: Create `plans/[task-name].md` with:

- Clear goal statement
- Current state analysis (what works, what's broken)
- Detailed implementation plan broken into phases/steps
- Progress tracking table
- Dependencies and blockers
- Files to create/modify
- Next steps

**While working**: Update `plans/[task-name].md` after completing each step:

- Mark completed steps with ✓ or DONE
- Update progress tables
- Add new findings or changes to approach
- Note any issues encountered
- Update next steps

**After completing**: Final update to `plans/[task-name].md`:

- Mark all steps complete
- Summary of what was accomplished
- Any remaining TODO items
- Lessons learned or notes for future work

### Format Guidelines

**For simple tasks** (single file changes, quick fixes):

```markdown
# Task: [Brief description]

## Steps
- [ ] Step 1
- [x] Step 2 (completed)

## Current Status
Working on: [step name]
```

**For complex tasks** (multi-file refactoring, feature additions):

- Include progress summary tables
- Organize into phases
- Track files created/modified separately
- Document API changes needed
- List dependencies and blockers
- Maintain current state diagnosis section

See [plans/template.md](plans/template.md) for detailed examples and formats.

This helps maintain context across sessions and track progress systematically.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tmushayahama) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
