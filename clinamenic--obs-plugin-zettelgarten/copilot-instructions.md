## obs-plugin-zettelgarten

> Defines standards for creating AI-optimized implementation plans, ensuring clear context, unambiguous directives, and traceable dependencies.


## Planning Workflow

### Active Planning (`.workspace/to-do/`)

Create implementation plans in `.workspace/to-do/` for features ready to be worked on.

1. Intake: capture the user's request

- Collect a concise feature description, goals, and constraints
- Identify impacted areas (plugin functionality, UI, settings, vault interaction)

2. Codebase reconnaissance

- Search for related modules and configurations
- Note existing interfaces and integration points
- Record key files and their purposes

3. Clarification questions (if needed)

- Ask targeted questions to remove ambiguity about scope and edge cases
- Block further steps until answers are provided

4. Synthesize answers and finalize scope

- Incorporate user responses
- Decide on approach, dependencies, and trade-offs

5. Write planning doc in `.workspace/to-do/`

- File name: `YYYY-MM-DD-feature-slug.md`
- Include: Baseline Knowledge, Implementation Order, Integration Points, Success Criteria, Risk & Rollback

6. Review readiness

- Confirm the plan is testable, traceable to requirements, and has explicit dependencies

7. Proceed to implementation

- Use the plan as the source of truth for edits and implementation steps

8. Archive completed plans

- After successful implementation, move plan to `.workspace/archive/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Clinamenic) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
