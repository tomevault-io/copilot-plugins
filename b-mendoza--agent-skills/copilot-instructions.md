## agent-skills

> This guide captures hard-won lessons from iterative improvement cycles across

# Best Practices for Writing Skills and Subagent Definitions

This guide captures hard-won lessons from iterative improvement cycles across
multiple skill and subagent definitions. Every practice below is grounded in
real-world testing, LLM research, and architectural review. They are
format-agnostic and apply to any skill/subagent system regardless of platform.

---

## Table of Contents

1. [Progressive Disclosure](#1-progressive-disclosure)
2. [Context Window Protection](#2-context-window-protection)
3. [Subagent-Default Execution](#3-subagent-default-execution)
4. [Positive Constraint Framing](#4-positive-constraint-framing)
5. [Instruction Reinforcement](#5-instruction-reinforcement)
6. [Structural Conventions](#6-structural-conventions)
7. [Input and Output Contracts](#7-input-and-output-contracts)
8. [Escalation Patterns](#8-escalation-patterns)
9. [Template Extraction](#9-template-extraction)
10. [Identity and Mental Model Statements](#10-identity-and-mental-model-statements)
11. [Example Strategy](#11-example-strategy)
12. [Validation Loops](#12-validation-loops)
13. [Naming Conventions](#13-naming-conventions)
14. [Artifact Lifecycle Management](#14-artifact-lifecycle-management)
15. [Empirical Validation over Self-Report](#15-empirical-validation-over-self-report)

---

## 1. Progressive Disclosure

### What it is

Structure skill content into layers that load only when needed, rather than
front-loading everything into a single file. This protects the agent's context
window from irrelevant information and follows the 80/20 rule: show 20% of
content prominently, reveal the rest on demand.

### Why it matters

Large skill files degrade agent performance. Research shows that models struggle
to follow instructions reliably when context grows long (AGENTIF benchmark,
arXiv:2505.16944 — best models follow fewer than 30% of agentic instructions
perfectly, with performance degrading as instructions grow longer). Splitting
content into layers keeps each decision point focused on only the information
needed at that moment.

### Three-level architecture

| Level | What loads                          | When                                | Size guidance      |
| ----- | ----------------------------------- | ----------------------------------- | ------------------ |
| 0     | SKILL.md body                       | When the skill triggers             | Under 500 lines    |
| 1     | Reference files (`references/`)     | Just-in-time, per phase or mode     | Unlimited          |
| 2     | Subagent definitions (`subagents/`) | Only when dispatching that subagent | Unlimited per file |

### What belongs at each level

**Level 0 (always loaded):** Core identity, behavioral description, input/output
contracts, subagent registry, phase guide or routing table. This is the
information the agent needs for every decision.

**Level 1 (just-in-time):** Mode-specific execution instructions, detailed phase
playbooks, error recovery procedures, data contract specifications. Loaded when
the agent enters that specific mode or phase.

**Level 2 (on-dispatch):** Individual subagent definitions. Read only when the
agent is about to dispatch that specific subagent. Never preloaded.

### Example: Phase guide as a routing table

Instead of inlining 500+ lines of phase-specific instructions, SKILL.md
contains a compact lookup table that tells the agent which reference file to
load:

```markdown
## Phase Guide

| Phase range | Reference file                   |
| ----------- | -------------------------------- |
| 1-4         | `./references/phases-1-4.md`     |
| 5-7         | `./references/task-loop.md`      |
| Error       | `./references/error-handling.md` |
```

The agent reads the appropriate file only when entering that phase range.

### When to use fewer levels

Not every skill needs three levels. Simple skills (single dispatch, compact
instructions) work fine with just Level 0. Reserve multi-level disclosure for
skills with:

- Multiple mutually exclusive modes (e.g., "upfront mode" vs "critique mode")
- Extended multi-phase workflows
- Large output templates or reference data
- Complex error recovery procedures

### References

- AGENTIF benchmark — arXiv:2505.16944, NeurIPS 2025 D&B Spotlight
- "Lost in the Middle" — TACL 2024, aclanthology.org/2024.tacl-1.9/

---

## 2. Context Window Protection

### What it is

Treat the orchestrating agent's context window as the most expensive resource
in the system. Every byte of raw content that leaks into it degrades
decision-making for every subsequent step.

### Why it matters

Context pollution compounds. When an orchestrator reads raw file contents,
command output, or API responses directly, that data stays in context for the
entire remaining conversation. It crowds out the instructions and summaries the
agent needs for later decisions, leading to instruction-following failures and
degraded output quality.

### Rules

1. **Dispatch, don't execute.** The orchestrator coordinates, decides, and
   synthesizes. Subagents do the heavy lifting in isolation.

2. **Collect summaries, not raw output.** When a subagent returns a result,
   extract the verdict and summary. Discard everything else. If details are
   needed later, dispatch a subagent to retrieve them.

3. **Pass structured data between steps.** Use file paths, ticket keys, and
   concise summaries as handoffs — not full file contents or raw command output.

4. **Never cache "just in case."** If information might be needed later, the
   appropriate action is to dispatch a subagent to retrieve it at that time,
   not to hold it in the orchestrator's context.

### The orchestrator holds only

- Decision-relevant summaries from subagents
- Current workflow state (phase, task, status)
- User instructions and confirmations
- Error reports that require judgment

### Example: Good vs bad handoff

```markdown
# Bad — raw data leaks into orchestrator context

Orchestrator reads docs/PROJ-123-tasks.md (200 lines) directly,
then analyzes it inline.

# Good — subagent returns a summary

Orchestrator dispatches artifact-validator with:
  TICKET_KEY: PROJ-123, PHASE: 2, DIRECTION: postcondition
Subagent returns:
  VALIDATION: PASS (3 lines)
Orchestrator uses the verdict to decide the next step.
```

### References

- Anthropic Claude Code agent architecture — code.claude.com/docs/en/sub-agents

---

## 3. Subagent-Default Execution

### What it is

Decide at each step whether to execute inline or dispatch to a subagent. The
decision is based on whether the step's output serves the orchestrator's
reasoning — not on step complexity or tool-call count.

**Important scope note:** This principle applies to **pre-authored skill
definitions** — orchestration instructions that prescribe when and where to
dispatch. It does not apply to ad-hoc, unconstrained model behavior. Anthropic's
Claude 4 best-practices documentation (platform.claude.com) warns that Opus 4.6
over-spawns subagents in open-ended coding tasks; that guidance addresses the
model's autonomous behavior, not pre-authored workflows where dispatch decisions
are made by the skill author.

### Why it matters

The cost of an unnecessary subagent is negligible (one extra dispatch). The cost
of polluted orchestrator context compounds with every step. But over-delegation
has its own cost: dispatch overhead, loss of conversational continuity, and
unnecessary complexity for simple operations. The right default depends on the
workflow.

### The two-question test

At each step in a skill's execution, ask:

1. **Will executing this inline help the orchestrator perform its
   coordination or decision-making better?** Does the orchestrator need to
   see, react to, or build upon the step's output in real time?
2. **Is the step's output information the orchestrator needs to retain for
   future steps?** Will the orchestrator reference this context when making
   later decisions?

If **either answer is yes** — execute inline. The step's output is working
material for the orchestrator.

If **both answers are no** — dispatch to a subagent. The orchestrator only
needs a summary, not the raw details.

### Heuristics

These are shortcuts for applying the two-question test, not hard rules:

- Steps that fetch large data sets the orchestrator won't reference → delegate.
- Steps that produce artifacts (files, tickets, commits) → delegate; return
  a path or key.
- Conversational turns where the orchestrator builds on prior exchanges
  (e.g., clarifying assumptions with the user) → inline.
- Validation checks that return a pass/fail verdict → delegate.

### Per-step, not per-skill

A single skill can mix inline and delegated steps. For example,
`clarifying-assumptions` runs Q&A turns inline (the orchestrator needs the
conversation history to coordinate) but dispatches to `critique-analyzer` and
`decision-recorder` (those produce artifacts the orchestrator doesn't need in
detail). Apply the two-question test at each step independently.

### Decision framework

| Choose...         | When...                                                            |
| ----------------- | ------------------------------------------------------------------ |
| **Inline**        | The step's output serves the orchestrator's reasoning or must be   |
|                   | retained for future coordination decisions.                        |
| **Subagent**      | The step produces output the orchestrator doesn't need in detail — |
|                   | a summary or verdict is sufficient.                                |
| **Skill**         | The step is purely about loading context or decision-making        |
|                   | guidance. It does not execute work — it informs the agent.         |
| **Slash command** | Quick, well-defined action the user invokes explicitly by name.    |

### Co-location

All subagent files live inside the skill folder, not in a global agents
directory. This keeps skills self-contained and portable:

```
skill-name/
├── SKILL.md
├── subagents/
│   ├── subagent-1.md
│   ├── subagent-2.md
│   └── subagent-3.md
├── references/
└── scripts/
```

### References

- Anthropic sub-agents documentation — code.claude.com/docs/en/sub-agents

---

## 4. Positive Constraint Framing

### What it is

When defining behavioral boundaries, tell the agent what it IS allowed to do
rather than listing what it must not do. Name the permitted tools and actions
explicitly, then state that everything else is a delegation.

### Why it matters

Anthropic's official prompting guidance recommends "tell Claude what to do
instead of what not to do." Research supports this across multiple dimensions:

- **Positive framing improves agent constraint adherence.** In agentic safety
  evaluations, positively framed principles achieved near-perfect adherence
  across all models, while negatively framed principles showed high variance
  (arXiv:2506.02357, ICML 2025 Technical Governance Workshop).
- **Prohibition can backfire.** Models under goal-directed pressure violate
  explicit prohibitions even when told they are under surveillance
  (arXiv:2507.02977). Practitioner analysis of the "Pink Elephant Problem"
  (eval.16x.engineer, anecdotal — no controlled experiments) suggests that
  suppressing a concept ("MUST NOT call Bash") may activate it.
- **Positive framing preserves intent.** Telling the agent "you use only Agent
  and Read" achieves the same boundary as "you must not use Bash, Grep, Write,
  Edit" — but without the negation risks.

### Example: Positive vs negative constraint

```markdown
# Negative (avoid)

The orchestrator MUST NOT directly call Read, Bash, Grep, Glob, Write, or
Edit. All tool use is delegated to subagents via the Agent tool.

# Positive (prefer)

The only tools the orchestrator calls directly are Agent (to dispatch
subagents) and Read (limited to loading skill, subagent, and reference
files). Every other operation is a dispatch to the appropriate subagent.
```

Both define the same boundary. The positive version names what IS permitted
(Agent, Read with exceptions) and states that everything else is a dispatch —
achieving specificity without relying on negation.

### When explicit restrictions still help

Pure positive framing works for design intent and identity. However, if
empirical testing shows the agent still deviates, brief reminders that name
the boundary can reinforce it at the violation point. The key is to lead with
the positive framing and add reinforcement only where proven necessary.

### References

- Anthropic Claude prompting best practices — official docs (2025-2026)
- Agent constraint adherence under positive/negative framing — arXiv:2506.02357,
  ICML 2025 Technical Governance Workshop
- Misalignment under prohibition — arXiv:2507.02977 (2025)
- Pink Elephant Problem — eval.16x.engineer (2025, practitioner anecdote)

---

## 5. Instruction Reinforcement

### What it is

Repeat critical constraints briefly at strategic points in long documents,
rather than stating them only once at the beginning.

### Why it matters

The "Lost in the Middle" effect (TACL 2024) demonstrates that instructions
stated once early in context lose influence as more content is processed. This
is especially problematic for orchestrating skills whose context includes
SKILL.md + reference files + subagent outputs — a constraint stated once at the
top faces maximum dilution by mid-execution.

Related research on prompt repetition (arXiv:2512.14982) found that repeating
input queries improved performance across 70 comparisons with 47 wins and
0 losses across Claude, GPT, Gemini, and DeepSeek. That study tested
input-level prompt duplication rather than mid-document reinforcement, but the
underlying attention mechanism — tokens later in context attend to earlier
repetitions — supports the same intuition.

### How to apply

1. State the primary constraint in the main SKILL.md (always loaded first).
2. Add brief (1-3 line) reminders at the top of long reference files that
   the agent reads mid-execution.
3. Keep reminders short — a single blockquote line is sufficient.
4. Do NOT repeat in every file. Target only the longest reference files
   where instruction dilution is most likely.

### Example: Brief reminder in a reference file

```markdown
# Phases 1-4 — Linear Pipeline

> Read this file when entering Phase 1, 2, 3, or 4.
>
> **Reminder:** All operations below are dispatches to subagents. The
> orchestrator does not call Read, Bash, Grep, Glob, Write, or Edit
> directly (except to load skill/subagent/reference files).
```

### What NOT to repeat

- Detailed explanations or rationale (those belong in one place)
- Content that downstream files already express implicitly through consistent
  "dispatch" language
- Constraints in short files where the "Lost in the Middle" effect is minimal

### References

- "Lost in the Middle" — TACL 2024, aclanthology.org/2024.tacl-1.9/
- arXiv:2603.10123 (preprint, March 2026) — lost-in-the-middle as architectural
  constraint
- Anthropic "Effective Context Engineering for AI Agents" — engineering blog,
  September 2025
- Prompt repetition (related) — arXiv:2512.14982 (2025)
- Palantir & ElevenLabs prompt engineering guides

---

## 6. Structural Conventions

### What it is

Follow a consistent section ordering for skills and subagents so that agents
(and humans) can quickly locate information and understand the structure of
any new definition.

### Skill section order

```
1. YAML frontmatter (name, description)
2. # Title + purpose paragraph
3. ## Inputs (table: name, required, example)
4. ## Pipeline / Workflow overview (if multi-phase)
5. ## Subagent Registry (table: name, path, purpose)
6. ## How This Skill Works (behavioral description, identity)
7. ## Execution steps or Phase guide
8. ## Example (dispatch round-trip)
```

### Subagent section order

```
1. YAML frontmatter (name, description)
2. # Title + mental model paragraph
3. ## Inputs (table or list)
4. ## Instructions / How to [action] (step-by-step)
5. ## Output Format (structured template with example)
6. ## Scope (allow-list: "Your job is to...")
7. ## Escalation (failure categories with report format)
```

### Why this order matters

The ordering reflects a natural reading flow:

- **Identity first** (frontmatter + title) — what is this and when does it
  trigger?
- **Contracts next** (inputs, outputs, registry) — what does it consume and
  produce?
- **Behavior then** (instructions, dispatch tables) — how does it work?
- **Boundaries last** (scope, escalation) — where does it stop?

Placing scope and escalation at the end is deliberate: these sections provide
guardrails after the agent understands its purpose and procedure. Moving them
earlier risks the agent optimizing for boundary-avoidance rather than
task-completion.

### Subagent Registry format

Every skill that dispatches to subagents needs a registry table. Place it near
the top of the skill body, after the overview:

```markdown
## Subagent Registry

| Subagent        | Path                           | Purpose                             |
| --------------- | ------------------------------ | ----------------------------------- |
| `log-analyzer`  | `./subagents/log-analyzer.md`  | Extract errors from build/test logs |
| `code-reviewer` | `./subagents/code-reviewer.md` | Review changed files for issues     |
```

The orchestrator uses this table to choose the right subagent without reading
every definition file. Paths are relative to the skill folder.

---

## 7. Input and Output Contracts

### What it is

Explicitly define what data a skill or subagent requires as input, what it
produces as output, and in what format. These contracts are the data boundaries
between pipeline stages.

### Why it matters

Without explicit contracts, subagents make assumptions about input format and
downstream consumers make assumptions about output format. Mismatches cause
silent failures that surface steps later, far from the root cause.

### Input contract format

```markdown
## Inputs

| Input      | Required | Example                                               |
| ---------- | -------- | ----------------------------------------------------- |
| `JIRA_URL` | Yes      | `https://workspace.atlassian.net/browse/PROJECT-1234` |
| `MODE`     | No       | `upfront` (default) or `critique`                     |
```

Include derivation rules when inputs carry multiple pieces of information:

```markdown
Extract these values from the URL:

- **Workspace:** subdomain before `.atlassian.net` → `workspace`
- **Ticket key:** full path segment → `PROJECT-1234`
```

### Output contract format

Specify the file path, required sections, and structural expectations:

```markdown
## Output Contract

Path: `docs/<TICKET_KEY>-tasks.md`

Must contain:

- `## Ticket Summary` section
- `## Tasks` section with at least 2 task entries
- Each task entry has: Title, Description, Acceptance Criteria
```

### Design principle: Prefer URLs over derived keys as inputs

When a value carries multiple pieces of context (e.g., a URL contains
workspace, project, and key), pass the full value rather than requiring the
caller to pre-extract components. The receiving skill can derive what it needs.
This reduces ambiguity and provides richer context to downstream operations.

---

## 8. Escalation Patterns

### What it is

Define explicit failure categories and reporting formats for every subagent, so
the orchestrator can make informed decisions about how to handle failures
without reading raw error details.

### Why it matters

Without structured escalation, subagents either fail silently (producing
degraded output the orchestrator doesn't notice) or dump raw errors that
pollute the orchestrator's context. Categorized escalation enables the
orchestrator to route failures correctly: retry, skip, or escalate to the user.

### Failure categories

Calibrate escalation categories to the subagent's failure profile:

**Utility subagents** (simple pass/fail operations):

```markdown
## Escalation

If you encounter an unexpected error, report:

VALIDATION: ERROR
Phase: <N> | Direction: <direction>
Reason: <what went wrong>

The orchestrator will decide how to handle the error.
```

**Planning subagents** (three-category pattern):

| Category  | Meaning                             | Example                      |
| --------- | ----------------------------------- | ---------------------------- |
| `BLOCKED` | Cannot start — missing prerequisite | Required skill not installed |
| `FAIL`    | Completed with issues in output     | Circular dependency detected |
| `ERROR`   | Unexpected failure                  | Filesystem error, crash      |

**API-interacting subagents** (5+ categories for granular error handling):

| Category        | Meaning                             | Severity |
| --------------- | ----------------------------------- | -------- |
| `404`           | Resource not found                  | Fatal    |
| `401/403`       | Authentication failure              | Fatal    |
| `TOOLS_MISSING` | Required API tools unavailable      | Fatal    |
| `RATE_LIMIT`    | Rate limited after retry            | Warning  |
| `PARTIAL`       | Some items failed, others succeeded | Warning  |

**Judgment-heavy subagents** (specialized escalation):

When a subagent's core purpose depends on specific capabilities (e.g., web
search for bias correction), the absence of those capabilities is a FAIL, not
a graceful degradation. If a bias-correction subagent cannot search the web,
its output silently becomes biased training-data regurgitation — which
undermines the entire reason it exists.

### Key principle: Fail loudly, recover gracefully

Surface failures immediately with enough context for the orchestrator to act.
Never silently degrade output quality. The orchestrator — not the subagent —
decides whether to retry, skip, or escalate.

---

## 9. Template Extraction

### What it is

Move large, self-contained output templates and reference data into separate
co-located files, loaded only when the subagent reaches the step that needs
them.

### Why it matters

Output templates (80-100+ lines) occupy significant context space but are only
needed at assembly time. Extracting them reduces the subagent definition's
always-loaded footprint while keeping the content accessible.

### When to extract

- Output format templates over 80 lines
- Reference data tables that are consulted, not continuously processed
- Content that is loaded at a single, well-defined execution point

### When NOT to extract

- Small templates (under 40 lines) scattered across multiple steps — extraction
  would require loading at 3+ different points, which is worse than inlining
- Behavioral content tightly coupled to instructions (format used immediately
  after its definition)
- Subagent definitions already under ~160 lines

### Example: Template extraction

```
# Before (267-line subagent)
subagents/ticket-retriever.md
  - Instructions (100 lines)
  - Document Template (90 lines)    ← extract this
  - Scope + Escalation (30 lines)

# After (185-line subagent + 90-line template)
subagents/ticket-retriever.md        (185 lines)
subagents/ticket-retriever-template.md (90 lines, loaded at assembly step)
```

The template file is read by the subagent only when it reaches the document
assembly step, not when it starts.

---

## 10. Identity and Mental Model Statements

### What it is

Begin every skill and subagent with a clear statement of what it is, what it
does, and how it approaches its work. This shapes the agent's reasoning about
its role, authority, and boundaries.

### Why it matters

Models respond better to reasoning than to commands (arXiv:2511.04694 — ~20%
improvement when models explicitly reason about instruction relationships).
An identity statement gives the agent a positive mental model of its role,
which improves instruction-following throughout the execution.

### Calibrate depth to complexity

**Utility subagents** (simple, mechanical operations) need minimal identity —
2-3 lines establishing the role:

```markdown
You are a progress-tracking subagent. Manage the workflow progress file and
per-task progress files, and return concise status summaries.
```

**Judgment-heavy subagents** (analysis, critique, complex decisions) need
extended identity that explains WHY the subagent exists and what biases it
must counter:

```markdown
You are a critique analyzer. Your purpose is to counter two systemic biases
that affect AI-assisted development:

- **The Matthew Effect:** AI tools disproportionately recommend mainstream
  frameworks. You counter this by searching for current alternatives and
  presenting trade-offs.
- **Solution-first thinking:** Tickets describe solutions without articulating
  user needs. You challenge the Problem Framing section to surface gaps.
```

**Orchestrating skills** use an identity paragraph that defines three
categories — what it does, how it reasons, and what it delegates:

```markdown
The orchestrator does exactly three things: **think** (analyze summaries,
assess state), **decide** (choose next phase, pick subagent, resolve
ambiguity), and **dispatch** (send work to a subagent via the Task tool).
```

### References

- Reasoning Up the Instruction Ladder — arXiv:2511.04694

---

## 11. Example Strategy

### What it is

Include concrete examples at every level of the skill/subagent hierarchy to
show expected quality, format, and edge cases.

### Why it matters

Examples serve as implicit output contracts. They show the agent what "good"
looks like more effectively than abstract descriptions. They also reduce
ambiguity in format-sensitive outputs (structured reports, validation summaries,
handoff data).

### What to include

Every improved skill or subagent should have:

1. **Dispatch round-trip example** (in the skill) — shows the complete flow
   from input to subagent dispatch to output summary.
2. **Output format example** (in the subagent) — shows the exact output
   structure with realistic data.
3. **Edge case or failure example** — shows how partial failures or unexpected
   inputs are handled.

### Example: Dispatch round-trip

```markdown
<example>
Input: JIRA_URL = https://workspace.atlassian.net/browse/PROJ-123

1. Orchestrator dispatches ticket-retriever with JIRA_URL
2. Subagent returns:
   FETCH: PASS
   File: docs/PROJ-123.md
   Sections: Description, Acceptance Criteria, Comments (5)

3. Orchestrator uses summary to decide: proceed to Phase 2

</example>
```

### Example: Subagent output format

```markdown
<example>
VALIDATION: FAIL
Phase: 2 | Direction: postcondition
File: docs/PROJ-123-tasks.md
Checks:
  - File exists: OK
  - Contains ## Tasks: OK
  - Has 2+ task entries: FAIL - found 1 task entry
</example>
```

---

## 12. Validation Loops

### What it is

Build validation into every phase boundary: verify preconditions before
starting work and postconditions after completing it. When output quality
matters, use a run-check-fix-recheck cycle with explicit retry limits.

### Standard phase execution cycle

Every phase follows this pattern — no step skipped:

```
1. Announce     — display the phase banner
2. Validate     — check preconditions (dispatch validator)
3. Execute      — invoke the skill or subagent
4. Validate     — check postconditions (dispatch validator)
5. Update       — record progress
6. Gate check   — decide: advance, re-plan, or escalate
```

### Targeted fix cycles

When quality gates fail, re-run only the failing gate — not the entire
pipeline:

```
1. Collect feedback from failing gate(s) — specific issues, file paths
2. Re-dispatch executor with ONLY the flagged issues
3. Re-run ONLY previously failing gates
4. Max 3 fix cycles per task
5. If limit exhausted, escalate to user
```

### Why targeted, not full-pipeline

Full pipeline reruns waste context and time on already-passing stages. Targeted
fix cycles preserve momentum and focus on the actual issue. The retry limit
prevents infinite loops when a fix introduces new problems.

---

## 13. Naming Conventions

### Skills

Use **gerund form** (present participle) — describes the ongoing activity:

- `analyzing-data`
- `deploying-service`
- `creating-jira-subtasks`
- `orchestrating-jira-workflow`

### Subagents

Use **role nouns** — describes the specialist performing the work:

- `log-analyzer`
- `code-reviewer`
- `task-executor`
- `progress-tracker`

### Why the distinction

The naming convention signals the artifact type at a glance. Gerunds describe
processes (skills coordinate processes). Role nouns describe actors (subagents
are specialists that execute). This makes registry tables, file listings, and
dispatch instructions self-documenting.

### Frontmatter naming

The `name` field in YAML frontmatter uses kebab-case and matches the directory
or file name exactly:

```yaml
---
name: "orchestrating-jira-workflow" # matches skills/orchestrating-jira-workflow/
---
```

---

## 14. Artifact Lifecycle Management

### What it is

Distinguish between orchestration artifacts (working documents that track
workflow state) and implementation artifacts (source code, tests, configs that
are the workflow's output). Apply different lifecycle rules to each.

### Two categories

| Category | Contents                              | Committed to git | Deleted |
| -------- | ------------------------------------- | ---------------- | ------- |
| A        | Orchestration: progress files, plans, | Never            | Never   |
|          | ticket snapshots, decision logs       |                  |         |
| B        | Implementation: source code, tests,   | Yes              | Normal  |
|          | configs, documentation                |                  | rules   |

### Why preserve Category A

Orchestration artifacts enable resumability. If a workflow is interrupted — by
user choice, error, or session timeout — the progress files and planning
artifacts allow the workflow to resume from exactly where it left off. Deleting
them forces a restart.

### Key rule

Never commit Category A artifacts to version control. They are working
documents that belong to the workflow session, not to the project's history.
Only Category B artifacts (the actual output of the workflow) are staged and
committed.

---

## 15. Empirical Validation over Self-Report

### What it is

Validate skill and subagent improvements by running them against real tasks and
measuring whether deviations decrease — not by asking the agent whether the
fix will work.

### Why it matters

Research strongly challenges treating agent self-reports as reliable:

- Chain-of-thought explanations systematically misrepresent true reasons for
  model predictions (arXiv:2305.04388, NeurIPS 2023).
- Claude Opus 4.1 succeeded at introspective detection only ~20% of the
  time under optimized conditions (transformer-circuits.pub, 2025).
- Models adapt responses to match the questioner's implied expectations
  (sycophancy, arXiv:2310.13548, ICLR 2024).

An agent's explanation of "why I deviated" may be plausible and coherent
without being faithful to the actual computation. The fix should be validated
by observing whether the behavior changes in practice.

### How to apply

1. Identify the specific deviation observed (e.g., "orchestrator called Bash
   directly instead of dispatching to subagent").
2. Implement the fix based on the best available evidence (research, prompt
   engineering literature, official documentation).
3. Run the workflow with a real task and observe whether the deviation recurs.
4. If it recurs, investigate further rather than trusting the agent's
   explanation of why the fix didn't work.

### Long-term path: Framework-level enforcement

The research consensus (2024-2026) is clear that prompt-level controls —
whether allow-list, deny-list, or hybrid — are architecturally insufficient
for hard tool-use enforcement. The reliable long-term solution is
framework-level enforcement: using `allowedTools`/`disallowedTools` fields in
agent definitions, enforced by the harness rather than by prose.

Prompt-level controls remain valuable as a pragmatic layer, but they should
not be the only enforcement mechanism for safety-critical boundaries.

### References

- Unfaithful Explanations in CoT — arXiv:2305.04388, NeurIPS 2023
- Anthropic introspection research — transformer-circuits.pub (2025)
- Sycophancy in LLMs — arXiv:2310.13548, ICLR 2024
- Agent-SafetyBench — arXiv:2412.14470 (2024)
- Claude Code agent teams documentation — code.claude.com/docs

---

## Quick Reference: Skill File Structure

```
skill-name/
├── SKILL.md                    # Under 500 lines; core identity + contracts + routing
├── subagents/
│   ├── specialist-a.md         # Full subagent definition with scope + escalation
│   ├── specialist-b.md
│   └── specialist-b-template.md  # Extracted output template (loaded on demand)
├── references/
│   ├── mode-specific-guide.md  # Loaded just-in-time per mode or phase
│   └── error-recovery.md       # Loaded only on error
└── scripts/                    # Optional: deterministic tasks that don't need context
```

## Reconciling Positive and Negative Framing

Primary definitions and brief reminders serve different cognitive purposes and
should use different framing styles intentionally:

- **Primary definitions** (SKILL.md, identity sections): Use positive framing.
  These shape the agent's understanding of its role over a long context
  window. Anthropic's guidance and the reasoning research (arXiv:2511.04694)
  apply here — "you use only Agent and Read" builds a clearer mental model
  than "you must not use Bash, Grep, Write, Edit."

- **Brief reminders** (1-3 line blockquotes in reference files): Naming
  forbidden tools explicitly is acceptable and may be more effective. The
  agent adherence research (arXiv:2506.02357) and practitioner analysis of
  the Pink Elephant Problem studied primary instructions, not short
  reinforcement triggers. A reminder's job is to
  create friction at the moment the agent is about to reach for a tool it
  shouldn't — and naming that tool ("does not call Bash") fires precisely at
  that moment. The reminder is too short for the "Lost in the Middle" effect
  to dilute, and too brief for ironic process theory to activate sustained
  attention on the forbidden concept.

This is a hybrid, not an inconsistency. Positive framing answers "what am I?"
(identity). Specific tool names in reminders answer "wait, should I be doing
this?" (friction). Both layers reinforce the same boundary through
complementary mechanisms.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/b-mendoza) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
