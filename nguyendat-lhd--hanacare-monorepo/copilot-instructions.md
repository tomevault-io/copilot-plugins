## scrum-collaboration

> You are an expert in monorepo architecture, cross-functional team collaboration, and Scrum Agile methodology. Organize a monorepo and workflow to enable effective teamwork between Product Owners (PO), Business Analysts (BA), Developers (DEV), and Testers (QA) within Scrum sprints. The structure should allow:

You are an expert in monorepo architecture, cross-functional team collaboration, and Scrum Agile methodology. Organize a monorepo and workflow to enable effective teamwork between Product Owners (PO), Business Analysts (BA), Developers (DEV), and Testers (QA) within Scrum sprints. The structure should allow:
- Clear folder ownership
- Sprint-based planning, execution, and review
- Traceable requirements → user stories → scenarios → code → tests
- Parallel work by different teams in one codebase
- Minimal merge conflicts with isolated folders

## Monorepo Structure

project-monorepo/
├── apps/
├── packages/
├── docs/
├── features/
├── tools/
├── .github/
│   ├── workflows/
│   ├── CODEOWNERS
│   └── pull_request_template.md
├── .cursor/
│   └── rules/
│       └── scrum-collaboration.mdc
├── README.md

## Team Roles & Scrum Collaboration

- **Product Owner (PO)**: Owns the backlog, defines sprint goals, prioritizes epics/features. Collaborates with BA for user stories and accepts/rejects sprint deliverables.
- **Business Analyst (BA)**: Details requirements for each backlog item; refines user stories; defines acceptance criteria. Works with PO on story clarifications.
- **Developer (DEV)**: Estimates and commits to backlog items in sprint planning, implements features, fixes bugs, and writes code/test. Works closely with QA.
- **Tester (QA)**: Collaborates in sprint planning to write Gherkin scenarios; creates/updates automated test suites; executes test cases during sprint.

### Scrum Work Cycle per Sprint

**Sprint Planning:**
- PO presents prioritized backlog
- Each item refined with BA (requirements, acceptance criteria)
- DEV & QA estimate effort and split work

**Sprint Execution:**
- Teams work in parallel in their own folders
- Jira (or GitHub Projects) used for Kanban board
- BA, DEV, QA update status on Stories, Tasks, Bugs

**Daily Scrum:**
- Each team updates progress
- PO/BA answer questions/clarifications
- Merge only into feature branches for isolation

**Sprint Review:**
- Demo completed features
- PO accepts/rejects based on requirements
- QA demonstrates passing test scenarios (Gherkin)
- BA links acceptance criteria to features

**Sprint Retrospective:**
- All teams discuss what went well and what to improve next sprint

## Folder Ownership & CODEOWNERS

- `/apps/`    → DEV team
- `/packages/`→ DEV team
- `/features/`→ QA team
- `/docs/requirements/` → BA team
- `/docs/user-stories/` → BA team
- `/docs/specs/` → BA + DEV
- `/docs/sprint-notes/` → Scrum Master/All
- `.github/CODEOWNERS`: ensure only respective teams can approve merges to their folders

## User Story & Acceptance Criteria Template (BA)
```markdown
# User Story: <Story Name>
As a <user type>, I want <goal> so that <benefit>

## Acceptance Criteria
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## Sprint Tag: <Sprint-X>
## Status: [Planned/In Progress/Done]
```

## Gherkin Feature File Template (QA)
```gherkin
Feature: <Feature Name>
  As a <user type>
  I want <goal> so that <benefit>

  Background:
    Given <initial setup>

  Scenario: <happy path>
    Given <context>
    When <action>
    Then <expected result>

  @Sprint_X
```

## Development Tasks (DEV)
- Implement code as per stories and Gherkin scenarios in `/apps/` or `/packages/`
- Reference story and feature file in commits and PR (e.g., `[Sprint-X][Story-Y] Implemented login feature`)
- Unit + Integration tests co-located in `/test/`

## Workflow Example Per Sprint

1. **Sprint Planning**:
   - PO selects Sprint Backlog
   - BA writes/refines stories in `/docs/`
   - QA writes preliminary features in `/features/`
   - DEV estimates/code structure in `/apps/`

2. **During Sprint**:
   - DEV develops story, pushes to feature branch
   - QA writes/updates Gherkin scenarios, implements automated tests, tags `@Sprint_X`
   - BA clarifies requirements and syncs changes
   - PO tracks story progress

3. **Sprint Review**:
   - All teams demo completed stories & features
   - QA verifies acceptance criteria via tests
   - BA checks requirements met
   - PO accepts/rejects stories

4. **Retrospective**:
   - Teams add notes to `/docs/sprint-notes/`
   - Update process for next sprint

## Scrum-related Prompts for Cursor

### For Any Team
```
- "Organize this repo for Scrum collaboration across PO, BA, DEV, QA."
- "Create new user story and acceptance criteria for Sprint X." 
- "Generate Gherkin .feature file for Story-123 for Sprint 5."
- "Set up folder structure for new app as per team roles."
- "Generate Sprint notes file for Sprint 4."
- "Update README to reflect updated Scrum workflow."
```

### Individual Role Prompts

**For PO**
- "Create Sprint Backlog and link requirements, user stories, test scenarios, and responsible teams for Sprint X."

**For BA**
- "Write/refine user stories for upcoming sprint, tag each with Sprint number."

**For DEV**
- "Implement code for Story-456, reference matching Gherkin feature and acceptance criteria."

**For QA**
- "Generate/expand Gherkin feature file for Story-789, tag Feature and Scenarios for Sprint X. Execute tests and report status."

## Principles for Scrum Collaboration

1. **Sprint tagging** in all requirements, features, commits, and PRs enables traceability for each sprint
2. **Kanban board** links repo files to current workflow
3. **Daily Scrums** summarized in `/docs/sprint-notes/`
4. **Feature branches isolate changes by story/sprint**
5. **CODEOWNERS enforces review discipline across teams**
6. **QA tests (Gherkin) must cover all acceptance criteria before PR merge**
7. **Retrospective notes drive continuous improvement, stored in repo**

---
> Source: [nguyendat-lhd/hanacare-monorepo](https://github.com/nguyendat-lhd/hanacare-monorepo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
