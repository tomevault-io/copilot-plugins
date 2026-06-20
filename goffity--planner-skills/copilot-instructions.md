## planner-skills

> Plans and breaks down work before implementation. Use this skill whenever the user wants to plan a feature, break down a task into subtasks, create a project plan, estimate effort, or organize work into trackable issues — then optionally create GitHub or Jira issues from the plan. MUST trigger when the user describes a feature or task and asks to "break it down", "plan", "figure out tasks", "estimate effort", "where to start", "what steps", "create issues for this work", or wants to think through implementation before coding. Also triggers for migration planning, refactoring strategy, CI/CD pipeline design, or any multi-step engineering work that benefits from upfront task decomposition. Trigger even if the user doesn't say "plan" explicitly — if they describe something complex and seem unsure how to approach it, this skill structures their thinking.


# Planner

This skill helps you think through work before jumping into code. It analyzes your codebase, breaks a task into well-scoped subtasks with dependencies, writes a plan file you can track, and optionally creates GitHub or Jira issues from it. The goal is to turn a vague idea like "add authentication" into a concrete, ordered list of things to build.

## Usage

```
/planner [task/feature description]
```

## Instructions

### Step 1: Get Task Description

If no argument provided, ask:
```
What task or feature do you want to plan?
```

If argument provided, use it as the task description.

### Step 1.5: Clarifying Questions

Before jumping into analysis and planning, ask **5-10 clarifying questions** to fully understand the task scope and constraints. This step prevents wasted effort from misunderstanding requirements.

Generate questions based on the task description. Focus on areas that are ambiguous, have multiple valid approaches, or could significantly affect the plan structure. Categories to consider:

**Scope & Boundaries:**
- What's included vs explicitly out of scope?
- Are there specific user roles or personas to consider?
- Should this work with existing data/users or only new ones?

**Technical Constraints:**
- Are there specific technologies, libraries, or patterns to use (or avoid)?
- Are there performance requirements (latency, throughput, data volume)?
- Does this need to be backward-compatible?

**Dependencies & Integration:**
- Does this depend on other teams, services, or external APIs?
- Are there existing systems this needs to integrate with?
- Is there a deadline or release train to target?

**Prioritization:**
- What's the MVP vs nice-to-have?
- Are there phases (launch with X, add Y later)?

Present questions as a numbered list:

```
Before I plan this out, a few questions to make sure I scope it right:

1. [Question about scope/boundaries]
2. [Question about technical approach]
3. [Question about constraints]
4. [Question about integration]
5. [Question about priority/MVP]
...

Answer what you can — skip any that aren't relevant. I'll make reasonable assumptions for anything unanswered.
```

After receiving answers (or if the user skips), incorporate the context into the planning process. Unanswered questions should be noted as assumptions in the plan's Context section.

### Step 2: Choose Language

Ask the user which language to use for the plan output:

```
What language should the plan be written in? (default: English)
```

Use the selected language for all generated content (plan file, issue bodies, summaries). Default to English if no preference given.

### Step 3: Analyze Codebase

Before doing a full analysis, check if a previous plan already analyzed this codebase. This avoids redundant work when the codebase hasn't changed significantly.

```bash
# Check for existing plans and current git state
CURRENT_COMMIT=$(git rev-parse --short HEAD 2>/dev/null)
echo "Current commit: $CURRENT_COMMIT"
ls docs/plans/*.md 2>/dev/null
```

If a previous plan exists, read its "Codebase Analysis" section and compare its commit hash with the current one:

```bash
# Check what changed since the plan was written
PLAN_COMMIT=$(grep -o 'Commit: [^ ]*' docs/plans/*.md 2>/dev/null | tail -1 | sed 's/.*Commit: //')
if [ -n "$PLAN_COMMIT" ]; then
    git diff --stat "$PLAN_COMMIT"..HEAD 2>/dev/null
fi
```

- **If no changes** (or only unrelated files changed): reuse the previous analysis and tell the user. Skip to Step 4.
- **If there are changes**: do a focused analysis on what changed, and merge with the previous analysis.
- **If no previous plan exists**: do a full analysis from scratch.

For a full or focused analysis, use an Explore agent. Focus on:

- Project structure and tech stack
- Architecture patterns and conventions already in use
- Files and modules that will need changes
- Existing test setup and coverage
- Related code that might be affected

Present a brief summary to the user:

```
## Codebase Analysis

**Commit:** [short hash]
**Tech Stack:** [languages, frameworks]
**Architecture:** [patterns found]
**Affected Areas:**
- [file/module] - [why it's relevant]
**Existing Tests:** [test framework, coverage notes]
```

This analysis informs how tasks get scoped — for instance, if the project has no tests yet, a "write tests" task carries more effort than adding tests to an existing suite.

### Step 3.5: Detect Project Structure

Determine whether the project is a **monorepo / multi-repo** or a **single repo**:

```bash
# Detect project structure
# Check for monorepo indicators: multiple go.mod, package.json in subdirs, workspace config, etc.
MONOREPO=false
if ls */go.mod 2>/dev/null | head -1 >/dev/null 2>&1 || \
   grep -q '"workspaces"' package.json 2>/dev/null || \
   [ -f "pnpm-workspace.yaml" ] || \
   [ -f "lerna.json" ] || \
   [ -f "nx.json" ] || \
   [ -f "turbo.json" ]; then
    MONOREPO=true
fi
echo "Monorepo: $MONOREPO"
```

This detection affects subtask formatting:
- **Monorepo / multi-repo**: subtask titles use `[service-name]: description` prefix
- **Single repo**: subtask titles use plain descriptions (no prefix needed)

### Step 4: Break Down Tasks

Split the work into one parent task and subtasks. Each subtask should be an atomic unit that can be completed and verified independently.

#### Task Sizing Guidelines

Each subtask should target the **2-8 hour sweet spot**. Use this table to evaluate task granularity:

| Category | Duration | Signal | Action |
|----------|----------|--------|--------|
| **Too Large** | > 2 days | Hard to estimate; blocks other work; unclear progress | Break down further |
| **Well-Sized** | 2-8 hours | Clear deliverable; single owner; daily visibility; easy to estimate | Keep as-is |
| **Too Small** | < 1 hour | Over-planning; excessive tracking overhead | Combine with related tasks |

If a subtask exceeds 8 hours (Effort: XL), consider splitting it into smaller subtasks. If multiple subtasks are under 1 hour each and closely related, merge them into one.

#### Structure: Parent Task + Subtasks

Always create a single **parent task (T1)** that represents the overall goal, with all work items as **subtasks (T1.1, T1.2, T1.3, ...)**. This gives the team one issue to track the big picture, with subtasks for the actual work.

- **Parent task (T1)**: the overall objective. Its effort is the sum of all subtasks.
- **Subtasks (T1.1, T1.2, ...)**: the actual work items. Each has its own type, effort, acceptance criteria, and dependencies.
- The parent task's **Dependencies** section defines the execution order — which subtasks must finish before others can start.

#### Parent Task: Current Flow + New Flow (MANDATORY)

The parent task (T1) **MUST always include** two sections that describe the before/after behavior:

- **Current Flow:** Numbered list of how the system works today (before implementation). Describe the step-by-step user/system journey and highlight limitations.
- **New Flow (after implementation):** Numbered list of how the system will work after implementation. Show the complete journey including new capabilities and how they integrate with existing behavior.

These sections help the team understand the full picture without reading subtask details. They should be included in both the plan file AND the Jira/GitHub parent issue description.

#### Subtask Title: Service/Repo Prefix (conditional)

**For monorepo / multi-repo projects:** Every subtask title **MUST start with a `[service-name]:` prefix** to clearly indicate which service/package the work belongs to. This makes it easy to assign work and understand scope at a glance.

Format: `[service-name]: action description`

Examples:
- `[api]: Create auth middleware`
- `[web]: Add login UI components`
- `[shared]: Add validation utilities`

Use the short service/package name. If a subtask spans multiple services, use the primary one.

**For single-repo projects:** Use plain descriptive titles without a prefix.

Examples:
- `Create auth middleware`
- `Add JWT token validation`
- `Add login UI components`

#### Subtask Description: Affected Files (MANDATORY)

Every subtask description **MUST include**:

- **Affected Files:** Specific file paths discovered during codebase analysis (not generic placeholders)
- **Affected Service/Package:** (only for monorepo/multi-repo) The full service or package name

This ensures developers know exactly where to make changes.

#### Test Coverage (MANDATORY — per subtask, NOT bundled)

**NEVER bundle tests into a single "test" subtask.** Each subtask that produces code MUST include its own tests as part of its acceptance criteria. 1 task = 1 job — the job includes writing and passing its own tests.

- Each subtask's **Acceptance Criteria** must list the specific test cases it needs to pass
- Each subtask's **Affected Files** must include the test files to create/modify
- A subtask is not "done" until its tests pass

If integration tests span multiple subtasks, create a **separate integration test subtask** for each integration boundary — not one big "test everything" task at the end.

For each subtask, define:

| Field | Values |
|-------|--------|
| ID | T1.1, T1.2, T1.3, ... |
| Title | Short, action-oriented name (with `[service]:` prefix for monorepo) |
| Type | feature, bug, chore, refactor, docs, test |
| Priority | high, medium, low |
| Effort | S (< 2h), M (2-4h), L (4-8h), XL (> 8h) |
| Dependencies | Which subtask IDs must finish first |
| Labels | For issue tracker categorization |

#### Estimation Techniques

Use **T-shirt sizing** (S/M/L/XL) as the default. For tasks where more precision is needed, offer the **three-point estimation** technique:

**Three-Point Estimation Formula:** `(Optimistic + 4×Likely + Pessimistic) / 6`

Example:
```
T1.3: Add social login providers
  Optimistic:  4h (just Google OAuth)
  Likely:      8h (Google + GitHub with edge cases)
  Pessimistic: 16h (token refresh bugs, provider-specific quirks)
  Estimate:    (4 + 4×8 + 16) / 6 = 8.7h → Effort: L
```

Use three-point estimation when:
- The task involves unfamiliar technology or external dependencies
- There's significant uncertainty in scope
- The team needs confidence intervals for timeline planning

For most tasks, T-shirt sizing is sufficient. Only apply three-point when asked or when uncertainty is high.

Order subtasks by dependency — independent subtasks first, dependent ones after their prerequisites. Write acceptance criteria for each subtask so it's clear when it's "done."

**Monorepo example:**

```
| ID   | Title                                        | Type     | Priority | Effort | Deps     |
|------|----------------------------------------------|----------|----------|--------|----------|
| T1   | Add authentication layer                     | feature  | high     | XL     | -        |
| T1.1 | [api]: Create auth middleware                 | feature  | high     | M      | -        |
| T1.2 | [api]: Add JWT token validation               | feature  | high     | M      | T1.1     |
| T1.3 | [api]: Add social login providers             | feature  | medium   | L      | T1.2     |
| T1.4 | [web]: Add login UI components                | feature  | high     | M      | T1.2     |
```

Each subtask above includes its own unit tests in its acceptance criteria. No separate bundled test task.

**Single-repo example:**

```
| ID   | Title                                        | Type     | Priority | Effort | Deps     |
|------|----------------------------------------------|----------|----------|--------|----------|
| T1   | Add authentication layer                     | feature  | high     | XL     | -        |
| T1.1 | Create auth middleware                        | feature  | high     | M      | -        |
| T1.2 | Add JWT token validation                     | feature  | high     | M      | T1.1     |
| T1.3 | Add social login providers                   | feature  | medium   | L      | T1.2     |
| T1.4 | Add login UI components                      | feature  | high     | M      | T1.2     |
```

Each subtask includes its own tests — 1 task = 1 job (code + tests).

### Step 5: Present Plan for Review

Show the full plan and ask for feedback:

```
## Plan: [Plan Name]

**Total Subtasks:** [N]
**Estimated Effort:** [sum]

| ID   | Title | Type | Priority | Effort | Deps |
|------|-------|------|----------|--------|------|
| T1   | [overall goal] | feature | high | XL | - |
| T1.1 | ... | feature | high | M | - |
| T1.2 | ... | chore | medium | S | T1.1 |
| T1.3 | ... | test | medium | M | T1.1,T1.2 |

### Subtask Details

#### T1.1: [Title]
**Type:** feature | **Priority:** high | **Effort:** M
**Description:** [what and why]
**Acceptance Criteria:**
- [ ] [criterion 1]
- [ ] [criterion 2]
```

#### Parallel Execution Waves

After the task table, show which tasks can run concurrently by grouping them into **waves**:

```
### Execution Waves

| Wave | Tasks | Can Run in Parallel | Estimated Duration |
|------|-------|--------------------|--------------------|
| 1 | T1.1, T1.4 | Yes — no shared dependencies | 4h (longest task) |
| 2 | T1.2 | Solo — depends on T1.1 | 4h |
| 3 | T1.3 | Solo — depends on T1.2 | 8h |
| 4 | T1.5 | Solo — depends on all above | 4h |

**Total sequential estimate:** 24h
**With parallelization:** 20h (saved 4h)
```

Wave grouping rules:
- Tasks with no unfinished dependencies go in the same wave
- Each wave starts only after all previous waves complete
- Show time savings from parallel execution

#### Dependency Graph

After the wave table, include an ASCII dependency graph to visualize task relationships:

```
### Dependency Graph

T1.1 ──→ T1.2 ──→ T1.3
              └──→ T1.4
T1.1~T1.4 ──→ T1.5 (tests)
```

For larger plans, use a vertical layout:

```
T1.1 (Create auth middleware)
 ├──→ T1.2 (Add JWT validation)
 │     ├──→ T1.3 (Social login)
 │     └──→ T1.4 (Login UI)
 └──────────────────┐
                    ▼
              T1.5 (Tests)
```

Keep graphs compact. Use task IDs with short descriptions in parentheses for readability.

#### Risks & Rollback

Identify potential risks and mitigation strategies:

```
### Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| [what could go wrong] | high/medium/low | high/medium/low | [how to prevent or handle] |
| External API rate limits | high | medium | Implement retry with backoff; cache responses |
| Breaking existing auth flow | high | low | Feature flag; run old + new in parallel first |

### Rollback Plan
1. [How to revert if things go wrong]
2. [e.g., "Revert migration with `migrate down`"]
3. [e.g., "Disable feature flag to restore old behavior"]
```

Focus on risks that are **actionable** — skip obvious ones like "server could crash." Include rollback steps that are specific to this plan's changes.

Then ask:
```
Review the plan above:
1. Approve - proceed to create plan file and issues
2. Edit - modify tasks (add/remove/change)
3. Cancel - discard
```

If the user chooses Edit, take their feedback and revise the task breakdown. Repeat until approved.

### Step 5.5: Subagent Review (Automatic)

Before presenting the final plan for user approval, use an **Explore agent** to review the plan for gaps:

```
Review this plan for completeness. Check for:
1. Missing dependencies between tasks
2. Tasks that reference files/modules not discovered in codebase analysis
3. Missing edge cases or error handling tasks
4. Circular dependencies
5. Tasks that are too large (>8h) and should be split
6. Subtasks missing test cases in their acceptance criteria (each subtask must own its tests)
7. Bundled "test everything" tasks that should be split (1 task = 1 job)
```

If the review finds issues, silently fix them before presenting to the user. If a fix requires significant restructuring, note it in the plan presentation:

```
**Review Notes:**
- Moved T1.4 after T1.2 (discovered hidden dependency on auth middleware)
- Split T1.3 into T1.3a and T1.3b (exceeded 8h estimate)
```

This step runs automatically — do not ask the user whether to review.

### Step 6: Create Plan File

Save the approved plan to `docs/plans/<plan-name>.md`. Read `references/plan-format.md` for the exact file format.

```bash
mkdir -p docs/plans
```

Use the Write tool to create the file. The plan name should be kebab-case, max 40 characters (e.g., `add-oauth2-auth.md`, `fix-payment-timeout.md`).

### Step 7: Create Issues (Optional)

First, check if Jira is available:
```bash
JIRA_CONFIGURED=false
if [[ -f ".jira-config" ]] || [[ -f "$HOME/.config/claude-km/jira.conf" ]]; then
    JIRA_CONFIGURED=true
fi
echo "Jira configured: $JIRA_CONFIGURED"
```

Ask where to create issues:
```
Create issues on:
1. GitHub Issues
2. Jira          (only show if configured)
3. Skip - keep plan file only
```

#### Two-Pass Issue Creation

Because tasks reference each other through dependencies, create issues in two passes.

**Jira — locate the script dynamically:**
```bash
JIRA_SCRIPT=""
for path in "./scripts/jira-client.sh" "$HOME/.claude/skills/"*/scripts/jira-client.sh; do
    if [[ -f "$path" ]]; then
        JIRA_SCRIPT="$path"
        break
    fi
done
```

If no script is found, check if the `/jira` skill is available and use it instead. If neither is available, fall back to GitHub Issues or skip issue creation.

Read `references/templates.md` to select the right issue body template based on task type (feature, bug, epic, story, chore, refactor).

**Label Selection Rules:**

Labels MUST match the actual services/repos being modified in each task — NOT the service that is affected indirectly. For example:
- If a task only modifies `kol-frontend` code → label `Frontend` only
- If a task modifies `kol-backend` code → label `Backend` only
- If a task modifies both → label both `Frontend,Backend`
- Do NOT add labels for services that are only called/consumed but not modified (e.g., don't add `kol-player` just because the frontend serves player pages)

Check the available labels list from project memory and only use labels that exist. When in doubt, use fewer, more accurate labels rather than more, less accurate ones.

**Pass 1 — Create parent tasks first, then subtasks:**

For **GitHub**:

1. Create parent task issues first:
```bash
gh issue create \
  --title "[type]: [task title]" \
  --label "[labels]" \
  --body "[body from template]"
```

2. Then create subtask issues, referencing the parent in the body:
```bash
gh issue create \
  --title "[type]: [subtask title]" \
  --label "[labels]" \
  --body "Parent: #[parent_issue_number]
Subtask of [parent title]

[body from template]"
```

For **Jira**:

1. Create parent task issues first:
```bash
$JIRA_SCRIPT create "[PROJECT]" "[title]" "[body]" "[type]"
```

2. Then create subtasks under the parent:
```bash
$JIRA_SCRIPT create "[PROJECT]" "[subtask title]" "[body]" "Sub-task" --parent "[PARENT_KEY]"
```
If `--parent` is not supported by the script, create as regular issues and link them:
```bash
$JIRA_SCRIPT link "is-subtask-of" "[SUBTASK_KEY]" "[PARENT_KEY]"
```

Keep a mapping of task ID to issue number/key as you create them.

**Pass 2 — Update dependencies, priority, and plan file:**

After all issues exist, go back and:

**Priority (Jira only):**

`jira-client.sh` does not support `--priority` flag when creating issues — all issues default to "Medium". You MUST update priority via Jira REST API after creation to match the plan.

Priority ID mapping:
- `1` = Highest
- `2` = High
- `3` = Medium (default — no update needed)
- `4` = Low
- `5` = Lowest

Load Jira credentials and update priority for each issue:
```bash
source <(grep -E '^(JIRA_DOMAIN|JIRA_EMAIL|JIRA_API_TOKEN)=' .jira-config 2>/dev/null || \
         grep -E '^(JIRA_DOMAIN|JIRA_EMAIL|JIRA_API_TOKEN)=' ~/.config/claude-km/jira.conf 2>/dev/null)
AUTH=$(echo -n "${JIRA_EMAIL}:${JIRA_API_TOKEN}" | base64)

# Update priority for each issue that is NOT Medium
for issue in [ISSUE_KEYS_WITH_HIGH_PRIORITY]; do
    curl -s -X PUT \
        -H "Authorization: Basic $AUTH" \
        -H "Content-Type: application/json" \
        -d '{"fields":{"priority":{"id":"2"}}}' \
        "https://${JIRA_DOMAIN}/rest/api/3/issue/${issue}"
done
```

**Dependencies:**

- For Jira: create dependency links between subtasks with `$JIRA_SCRIPT link blocked-by [KEY] [DEP_KEY]`
- For GitHub: add a comment on the parent issue listing all subtasks with checkboxes and dependency order:
```bash
gh issue comment [parent_issue_number] --body "## Subtasks (execution order)

- [ ] #[subtask_1] - [title] (no dependencies)
- [ ] #[subtask_2] - [title] (after #[subtask_1])
- [ ] #[subtask_3] - [title] (after #[subtask_1], #[subtask_2])"
```

**Pass 3 — Update parent issue description with Current Flow + New Flow:**

If the parent issue already exists on Jira/GitHub (e.g., created before planning), **always PUT update its description** to include:
- **Current Flow** section (from the plan's parent task)
- **New Flow (after implementation)** section (from the plan's parent task)

This ensures the parent issue on the tracker matches the plan file. Use the same ADF/markdown format as the rest of the description.

- Update the plan file's Issue column with the real issue references

### Step 8: Summary

Show what was created:

```
## Plan Created

**Plan File:** docs/plans/[name].md
**Parent Task:** T1 - [title] ([issue ref])
**Subtasks:** [N] subtasks
**Issues:** [N+1] on [GitHub/Jira] (or "skipped")

| ID   | Title | Issue | Status |
|------|-------|-------|--------|
| T1   | [parent title] | #123 | Created |
| T1.1 | [subtask title] | #124 | Created |
| T1.2 | [subtask title] | #125 | Created |
| T1.3 | [subtask title] | #126 | Created |

**Execution Order:**
1. T1.1 - [title] (no dependencies)
2. T1.2 - [title] (after T1.1)
3. T1.3 - [title] (after T1.1, T1.2)

**Next Steps:**
- Start working on T1.1 (no dependencies)
- View the full plan at docs/plans/[name].md
```

## Examples

```
# Plan a new feature
/planner Add user authentication with OAuth2

# Plan a refactoring
/planner Refactor database layer to use repository pattern

# Plan a bug fix
/planner Fix payment timeout when users checkout with multiple items

# Interactive mode
/planner
```

---
> Source: [goffity/planner-skills](https://github.com/goffity/planner-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
