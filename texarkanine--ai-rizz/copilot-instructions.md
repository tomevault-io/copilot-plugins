## ai-rizz

> Determine the appropriate complexity level (1-4) for a task, then write the determination to the memory bank and load the corresponding workflow.


# Task Complexity Determination

Determine the appropriate complexity level (1-4) for a task, then write the determination to the memory bank and load the corresponding workflow.

## Decision Tree

```mermaid
graph TD
    Start["New Task"] --> Q1{"Bug fix or<br>error correction?"}
    Q1 -->|Yes| Q1a{"Affects single<br>component?"}
    Q1a -->|Yes| L1["Level 1:<br>Quick Bug Fix"]
    Q1a -->|No| Q1b{"Affects multiple<br>components?"}
    Q1b -->|Yes| L2["Level 2:<br>Simple Enhancement"]
    Q1b -->|No| Q1c{"Affects system<br>architecture?"}
    Q1c -->|Yes| L3["Level 3:<br>Intermediate Feature"]
    Q1c -->|No| L2

    Q1 -->|No| Q2{"Adding small<br>feature or<br>enhancement?"}
    Q2 -->|Yes| Q2a{"Self-contained<br>change?"}
    Q2a -->|Yes| L2
    Q2a -->|No| Q2b{"Affects multiple<br>components?"}
    Q2b -->|Yes| L3
    Q2b -->|No| L2

    Q2 -->|No| Q3{"Complete feature<br>requiring multiple<br>components?"}
    Q3 -->|Yes| Q3a{"Architectural<br>implications?"}
    Q3a -->|Yes| L4["Level 4:<br>Complex System"]
    Q3a -->|No| L3

    Q3 -->|No| Q4{"System-wide or<br>architectural<br>change?"}
    Q4 -->|Yes| L4
    Q4 -->|No| L3
```

## Complexity Indicators

### Level 1: Quick Bug Fix
- **Signals**: "fix", "broken", "not working", "bug", "error", "crash"
- **Scope**: Single component or UI element
- **Risk**: Low, isolated changes
- **Examples**: broken button, styling glitch, validation error, typo, broken link

### Level 2: Simple Enhancement
- **Signals**: "add", "improve", "update", "change", "enhance", "modify"
- **Scope**: Single component or subsystem
- **Risk**: Moderate, contained to specific area
- **Examples**: add form field, improve validation, update styling, enhance existing component

### Level 3: Intermediate Feature
- **Signals**: "implement", "create", "develop", "build", "feature"
- **Scope**: Multiple components, complete feature
- **Risk**: Significant, affects multiple areas
- **Examples**: user authentication flow, dashboard, search functionality, complex form system

### Level 4: Complex System
- **Signals**: "system", "architecture", "redesign", "integration", "framework"
- **Scope**: Multiple subsystems or entire application
- **Risk**: High, architectural implications
- **Examples**: payment processing framework, microservice architecture, database migration system, real-time communication system

## Assessment Questions

When the decision tree doesn't give a clear answer, consider:

1. **Scope** — Single component, multiple components, or system-wide?
2. **Design decisions** — Can you start coding immediately, or do you need to explore and choose between approaches first?
3. **Risk** — What breaks if this goes wrong? Is it isolated or cascading?
4. **Effort** — Hours, days, or weeks?

## Write Determination to Memory Bank

Before writing, check whether the ephemeral files already exist:

- If **all four** ephemeral files exist (`projectbrief.md`, `activeContext.md`, `tasks.md`, `progress.md`), work is already in-progress. **Warn the user** — the previous task should be archived first, or explicitly abandoned before proceeding.
- If the ephemeral files are absent and only the persistent files exist, the memory bank is ready for new work. Create or overwrite the ephemeral files as described below.

---

**`memory-bank/projectbrief.md`** — Capture the user story and requirements: Populate it from the user's input, prompting for clarification if the requirements are incomplete or ambiguous.

**`memory-bank/activeContext.md`** — Record the current session state:

- **Current Task** — task name derived from the user's request
- **Phase** — `COMPLEXITY-ANALYSIS - COMPLETE`
- **What Was Done** — complexity level determined (Level N) and rationale
- **Next Step** — load the Level N workflow

**`memory-bank/tasks.md`** — Stub the task list:

Create a stub file with the task name. Do **not** populate checklists yet — the level-specific planning rules prescribe the exact format and content.

**`memory-bank/progress.md`** — Initialize if absent:

Include a brief summary of the work to be done as described in the project brief - you've just *completed* complexity analysis!

## Next Steps

After writing the determination, inform the user:

```markdown
✅ **Complexity Analysis Complete** - Level N determined.
Loading Level N workflow...
```

Then, load the level-specific workflow and proceed to the next phase as specified by that document.

- Level 1: `.cursor/rules/shared/niko/level1/level1-workflow.mdc`
- Level 2: `.cursor/rules/shared/niko/level2/level2-workflow.mdc`
- Level 3: `.cursor/rules/shared/niko/level3/level3-workflow.mdc`
- Level 4: `.cursor/rules/shared/niko/level4/level4-workflow.mdc`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Texarkanine)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Texarkanine)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
