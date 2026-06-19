## spec-vibe-coding

> Structured software design before coding. Use when the user wants to build a new project, app, or feature from scratch — including 'I want to build...', 'spec vibe', 'help me plan this project', 'I need a PRD', or any new coding project request. Also triggers when the user jumps straight to 'write me an app that does X' — redirect them to design first.


# Spec Vibe Coding — Vibe coding with specs, not chaos

## What this skill does

Spec Vibe Coding guides users through a professional software design process before they write a single line of code. It produces a complete set of design documents — PRD, tech spec, UI spec, and development task breakdown — saved as markdown files in `.spec-vibe/` in the project directory.

The core insight: AI coding tools are great at writing code, but they need good instructions. Most projects fail not because the code is bad, but because nobody thought through what to build, how to organize it, or what the user experience should be. Spec Vibe Coding fixes this by making "thinking before coding" structured, guided, and even enjoyable.

## Who this is for

People who have an idea and want to build software — whether they're non-technical founders, junior developers, indie hackers, or experienced engineers who tend to skip the design phase. The skill adapts its language and depth based on the user's technical level.

## How it works

Spec Vibe Coding runs as a multi-phase conversation. Each phase produces a markdown document saved to `.spec-vibe/`. The user can go through phases sequentially or jump between them. At any point they can say "let's start coding" to generate development tasks from whatever design work is complete.

---

## Phase overview

```
Phase 0: Project kickoff       → .spec-vibe/00-blueprint.md
Phase 1: Requirements           → .spec-vibe/01-requirements.md
Phase 2: System design          → .spec-vibe/02-system-design.md
Phase 3: UI/UX design           → .spec-vibe/03-ui-design.md
Phase 4: Development tasks      → .spec-vibe/04-dev-tasks.md
Phase 5: Quality check          → .spec-vibe/05-quality-report.md
```

Always create the `.spec-vibe/` directory at the start. If it already exists, read existing documents to understand current state and pick up where the user left off.

---

## Phase 0: Project kickoff

**Goal:** Turn a vague idea into a structured project blueprint.

**How to run this phase:**

1. Ask the user to describe their idea in their own words. Accept any format — a sentence, a paragraph, bullet points, or even "something like X but for Y."

2. Ask 2-4 targeted follow-up questions to fill gaps. Pick from these based on what's missing:
   - Who is this for? (target users)
   - What's the core problem it solves?
   - What platform? (web, desktop, mobile, CLI)
   - What's the scale? (personal tool, startup product, enterprise)
   - Any technology preferences or constraints?
   - What's the timeline expectation?

   Present questions with suggested options the user can pick from, plus room for custom answers. Don't ask more than 2-3 questions at a time. The user can say "that's enough" at any point.

3. Generate a project blueprint and save it.

**Blueprint document structure** (save to `.spec-vibe/00-blueprint.md`):

```markdown
# Project blueprint: {Project name}

## One-line description
{What this software does, in one sentence}

## Problem statement
{What problem does it solve, and for whom}

## Target users
{2-3 user types with brief descriptions}

## Core features (initial)
{Bulleted list of 5-10 key features, ranked by importance}

## Technical direction
{Platform, suggested tech stack, key constraints}

## Estimated complexity
{Simple / Medium / Complex — with brief justification}

## Next steps
{Which design phases to focus on}
```

4. Show the blueprint to the user. Ask: "Does this capture your vision? Anything to change before we dive deeper?"

5. After confirmation, tell the user which phase to tackle next (usually Phase 1) and that they can say the phase name anytime to jump there.

---

## Phase 1: Requirements (PRD)

**Goal:** Produce a complete product requirements document.

**Read `.spec-vibe/00-blueprint.md` first** to load project context.

Work through these 6 sections by asking the user focused questions. For each section, propose an AI-generated draft based on the blueprint, then let the user refine it. Don't dump all 6 sections at once — go one at a time.

### Sections

1. **Product positioning** — One-liner, core problem, competitive difference
2. **User personas** — Generate 2-3 persona cards (role, scenario, pain point, tech level). Ask the user to confirm, add, or remove.
3. **Feature list** — Organize into Core (MVP must-have), Enhanced (nice-to-have), Future (later). Each feature needs: name, description, and acceptance criteria ("how do we know it's done?"). This is where most users need the most help — prompt them on acceptance criteria specifically.
4. **Core user flows** — Extract 2-3 key journeys. For each: trigger → steps → outcome. Proactively ask about error cases and edge scenarios.
5. **Constraints and boundaries** — Tech constraints, time constraints, what's explicitly NOT in scope, MVP boundary definition.
6. **Success metrics** — How will the user measure if this project succeeded?

**Save to `.spec-vibe/01-requirements.md`** using this structure:

```markdown
# Requirements: {Project name}

## 1. Product positioning
### One-line description
### Core problem
### Competitive difference

## 2. User personas
### Persona 1: {Name}
- **Role:** ...
- **Scenario:** ...
- **Pain point:** ...
- **Tech level:** ...

## 3. Feature list
### Core features (MVP)
| # | Feature | Description | Acceptance criteria |
|---|---------|-------------|---------------------|
| 1 | ...     | ...         | ...                 |

### Enhanced features
...

### Future features
...

## 4. Core user flows
### Flow 1: {Name}
1. User does X
2. System responds with Y
3. ...

**Error cases:**
- If X fails → ...

## 5. Constraints and boundaries
### Technical constraints
### Time constraints
### Out of scope
### MVP boundary

## 6. Success metrics
| Metric | Target | How to measure |
|--------|--------|----------------|
| ...    | ...    | ...            |
```

After completing each section, update the file and tell the user their progress (e.g., "Requirements: 4/6 sections complete"). The user can skip sections or come back later.

---

## Phase 2: System design

**Goal:** Translate requirements into a technical architecture.

**Read `.spec-vibe/00-blueprint.md` and `.spec-vibe/01-requirements.md` first.**

Adapt depth to the user's technical level:
- **Non-technical users:** Use plain language. "Your app needs a place to store data" instead of "We'll use PostgreSQL with a normalized schema." Still produce the technical details, but explain each decision in simple terms alongside it.
- **Technical users:** Go straight to architecture diagrams, DB schemas, API specs.

### Sections

1. **Tech stack recommendation** — Based on requirements, recommend specific technologies. For each choice, explain: why this one, what alternatives exist, and what the tradeoff is. Let the user override any choice.

2. **Architecture overview** — Describe the high-level system structure. What are the major components and how do they communicate? Use a text-based diagram:
   ```
   [Frontend: React] → [API Layer: Express] → [Database: PostgreSQL]
                                              → [Auth: Clerk]
                                              → [Storage: S3]
   ```

3. **Data model** — List all entities, their fields, and relationships. For non-technical users, frame as "information your app needs to remember."

4. **API design** — Key endpoints with request/response shapes. For simple projects, a table is enough. For complex ones, group by resource.

5. **Third-party services** — What external services are needed (auth, payments, email, etc.)? Include cost estimates if the user mentioned budget concerns.

6. **Non-functional requirements** — Performance targets, security considerations, scalability approach. Scale these to the project's actual needs — a personal tool doesn't need Kubernetes.

**Save to `.spec-vibe/02-system-design.md`** with clear sections matching the above.

---

## Phase 3: UI/UX design (or interface design)

**Goal:** Define the interface structure, user flows, and visual direction.

**Read previous phase documents first.**

### Non-UI projects (CLI, API, library, backend service)

If the project has no graphical UI, replace the standard UI/UX sections below with the following and save to `.spec-vibe/03-ui-design.md` using the same filename:

1. **Interface inventory** — List all interfaces the user or consumer interacts with: CLI commands/subcommands, API endpoints, library public API, config file format, etc.

2. **Interaction flows** — For each core user flow from the PRD, map the interface-level journey. For CLIs: command → flags → output → next command. For APIs: request → response → follow-up request.

3. **Input/output specification** — For each interface point, define:
   - Input format (flags, args, request body, config schema)
   - Output format (stdout, stderr, response body, exit codes)
   - Example invocations with expected output

4. **State matrix** — For each interface point, define behavior across states:
   | Interface | Success | Empty/no data | Invalid input | Error | Auth failure |
   |-----------|---------|---------------|---------------|-------|-------------|

5. **Error messages and help text** — Draft user-facing error messages, help text (`--help` output), and API error response shapes. Good error messages are the CLI/API equivalent of good UI.

6. **Developer experience (DX) direction** — Equivalent of "design direction" for non-UI projects. Describe the DX philosophy: verbose vs terse output, interactive vs scriptable, progressive disclosure of complexity, etc.

Then skip to Phase 4.

---

### UI projects

Follow this order — it matters:

1. **Page map** — List all pages/views the app needs. Show hierarchy:
   ```
   - Home / Dashboard
     - Project list
     - Project detail
       - Task board
       - Settings
   - User settings
   - Login / Register
   ```

2. **Page flows** — For each core user flow from the PRD, map the page-level journey. Mark where the user enters, what they see at each step, and where they exit.

3. **Page state matrix** — For each page, define 4 states:
   | Page | Normal | Empty | Loading | Error |
   |------|--------|-------|---------|-------|
   Generate a draft and highlight states the user probably hasn't thought about.

4. **Layout descriptions** — For each key page, describe the layout in words:
   ```
   ## Dashboard layout
   - Top bar: logo, search, user avatar
   - Left sidebar: navigation (projects, settings)
   - Main area: project cards in a grid (3 columns)
   - Each card shows: name, progress bar, last updated, member avatars
   ```

5. **Design direction** — Suggest 2-3 visual style options with descriptions (e.g., "Clean minimal — lots of white space, thin borders, neutral palette" vs "Warm professional — soft colors, rounded cards, friendly typography"). Let the user pick one.

6. **Component inventory** — List all UI components needed: buttons, cards, forms, modals, navigation, tables, etc. For each, note any variants (primary/secondary button, etc.).

**Save to `.spec-vibe/03-ui-design.md`.**

---

## Phase 4: Development tasks

**Goal:** Break the entire project into executable development tasks with AI-friendly prompts.

**Read all previous phase documents.**

This is where Spec Vibe Coding's value crystallizes — turning design documents into actionable development instructions.

### Task generation process

1. **Generate task list.** Break the project into tasks where each task:
   - Can be completed in 1-2 hours
   - Has clear input (what exists) and output (what to build)
   - Has explicit acceptance criteria
   - Lists file dependencies

2. **Order by dependency.** Infrastructure first (project setup, DB schema, auth), then backend APIs, then frontend components, then integration.

3. **Generate a prompt for each task.** Each prompt should be self-contained — a developer (or AI coding tool) should be able to execute it without reading any other document. Use the following template for every task prompt:

   ```
   ## Context
   {Project name} is a {one-line description}. Tech stack: {stack}.

   ## Your task
   {Clear description of what to build or implement in this task.}

   ## Prior work
   {List tasks already completed that produced code this task depends on.
   E.g., "Task 1 set up the project and created the DB schema in schema.prisma."
   Write "This is the first task." if there are no dependencies.}

   ## Data models
   {Paste only the models/tables/types this task touches. Omit unrelated models.}

   ## API / Interface spec
   {Paste only the endpoints, CLI commands, or function signatures this task needs
   to implement or consume. Include request/response shapes.
   Write "N/A" if this task doesn't involve APIs.}

   ## UI spec
   {For frontend tasks: paste the layout description, component list, and states
   from the UI design doc. For non-UI tasks: write "N/A".}

   ## Technical constraints
   {List any constraints: libraries to use/avoid, performance targets, auth
   requirements, file structure conventions, etc.}

   ## Acceptance criteria
   - [ ] {Criterion 1}
   - [ ] {Criterion 2}
   - [ ] {Criterion N}

   ## Out of scope
   {Explicitly state what this task should NOT do, to prevent scope creep.}
   ```

   Fill every section from the design documents. If a section is not applicable, write "N/A" — do not remove the section header, so the prompt structure stays predictable.

**Save to `.spec-vibe/04-dev-tasks.md`** using this structure:

```markdown
# Development tasks: {Project name}

## Overview
- Total tasks: {N}
- Estimated time: {X hours} (with AI assistance)

## Task execution order
1. [Task 1 name] — ~{time}
2. [Task 2 name] — ~{time}
...

---

## Task 1: {Name}

**Priority:** P0 (critical) / P1 (important) / P2 (nice-to-have)
**Type:** Setup / Backend / Frontend / Integration / Testing
**Estimated time:** ~{X}h
**Dependencies:** None / Task {N}
**Files involved:** {list}

### Prompt

\`\`\`
{The complete, self-contained prompt to give to an AI coding tool}
\`\`\`

### Acceptance criteria
- [ ] {Criterion 1}
- [ ] {Criterion 2}
...

---

## Task 2: {Name}
...
```

4. After generating all tasks, show the user a summary: total task count, estimated hours, and the first 3 tasks in detail. Ask if the task breakdown looks right before generating prompts for the rest.

---

## Phase 5: Quality check

**Goal:** Cross-reference all documents for consistency and completeness.

**Read all `.spec-vibe/*.md` files.**

Run these checks and report findings:

1. **Coverage check** — Every feature in the PRD should appear in:
   - System design (has supporting architecture)
   - UI design (has a page/view that exposes it)
   - Dev tasks (has at least one task that implements it)
   Report any features that are missing from any layer.

2. **Consistency check** — Data fields used in UI descriptions should exist in the data model. API endpoints referenced in task prompts should be defined in the system design. Report mismatches.

3. **Completeness check** — Are there tasks without acceptance criteria? Pages without error state designs? Features without acceptance criteria in the PRD?

4. **Scope check** — Is the total task count realistic for the user's stated timeline? If not, suggest what to cut.

**Save to `.spec-vibe/05-quality-report.md`:**

```markdown
# Quality check report: {Project name}

## Overall health: {A/B/C/D}

## Coverage matrix
| Feature | PRD | System design | UI design | Dev tasks |
|---------|-----|---------------|-----------|-----------|

## Issues found

### Blocking (must fix before development)
- ...

### Recommended (should fix)
- ...

### Suggestions (nice to have)
- ...
```

---

## Interaction guidelines

### Adaptive language
Pay attention to how the user communicates. If they use technical terms correctly, respond at that level. If they're describing things in everyday language, match that — explain technical concepts when they arise but don't overwhelm.

### Document language
All generated documents and conversational responses should use the same language as the user. If the user speaks Chinese, write documents in Chinese. If the user speaks English, write in English. When the user switches languages mid-conversation, follow the switch for all subsequent output and document updates.

### Don't block, guide
If the user wants to skip a phase or jump ahead, let them. The documents will just have gaps, and the quality check in Phase 5 will catch anything important that was missed. Progress beats perfection.

### Keep documents alive
When the user changes something in an earlier phase (like adding a feature to the PRD), proactively note which downstream documents might need updating: "You just added push notifications to the feature list. This will need: a notification service in the system design, a notification settings page in the UI design, and at least one new dev task."

### Show progress
After each interaction that updates a document, briefly note the overall status:
```
Progress: Blueprint ✓ | Requirements 4/6 | System design — | UI — | Tasks — | QC —
```

### The "let's just start coding" escape hatch
If the user says they want to start coding at any point, don't resist. Generate the best possible dev tasks from whatever design work is complete, note the gaps, and let them go. They can always come back to fill in the design later.

### Resume capability
If `.spec-vibe/` already has documents from a previous session, read them all and summarize the current state: "I see you've completed the blueprint and requirements, and started on system design. Want to continue from where you left off?"

---

## File management

All files go in `.spec-vibe/` in the current working directory. Create the directory if it doesn't exist. Use these exact filenames:

```
.spec-vibe/
├── 00-blueprint.md
├── 01-requirements.md
├── 02-system-design.md
├── 03-ui-design.md
├── 04-dev-tasks.md
└── 05-quality-report.md
```

When updating a document, rewrite the full file — don't try to patch sections. Always read the current version before writing to avoid losing changes from a previous session.

At the end of each phase, confirm the file was saved and tell the user: "Saved to `.spec-vibe/{filename}`. You can review it anytime."

---
> Source: [CheneyNine/Spec-Vibe-Coding](https://github.com/CheneyNine/Spec-Vibe-Coding) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
