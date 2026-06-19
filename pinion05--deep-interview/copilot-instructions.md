## deep-interview

> Standalone Socratic deep interview for turning vague ideas into execution-ready specs with ambiguity scoring, ontology tracking, and readiness gating.


# Deep Interview

## Purpose

Deep Interview is a standalone requirements-clarification skill for any autonomous agent.

It transforms vague user intent into a clear, testable, execution-ready specification by:
- asking one targeted question at a time,
- measuring ambiguity across weighted clarity dimensions,
- exposing hidden assumptions,
- tracking ontology stability across rounds,
- and refusing to proceed to execution until readiness is high enough.

This skill is runtime-agnostic. It does not depend on any specific planner, orchestrator, or execution framework.

## Use When

Use this skill when:
- the user has a vague idea and wants thorough clarification before execution,
- the task is complex enough that implementing immediately would likely cause rework,
- the user says things like:
  - "interview me"
  - "ask me everything"
  - "don't assume"
  - "make sure you understand first"
  - "I have a vague idea"
  - "help me figure out what I actually want"
- the agent needs a visible readiness gate before building.

## Do Not Use When

Do not use this skill when:
- the user already provided concrete file paths, function names, APIs, schemas, or acceptance criteria,
- the user explicitly says "just do it" or "skip the questions,"
- the task is a small direct modification,
- the user already has a sufficient PRD, plan, issue, or spec.

## Why This Exists

AI systems can generate code quickly, but vague intent still causes expensive mistakes.

The bottleneck is often not execution speed but specification quality:
- unclear goals produce wrong outputs,
- hidden assumptions create rework,
- unstable domain concepts cause architecture drift,
- missing acceptance criteria make "done" impossible to verify.

Deep Interview exists to make ambiguity visible, reduce it systematically, and produce a durable spec that any downstream agent can execute.

## Core Guarantees

This skill guarantees that it will:
- ask exactly one question per round,
- always target the weakest clarity dimension,
- explain why that dimension is the next bottleneck,
- show ambiguity scores after every round,
- track ontology stability across rounds,
- persist state for resume,
- produce a final standalone spec artifact,
- provide a runtime-neutral handoff package for downstream execution.

## Runtime Requirements

The host agent/runtime should ideally support the following capabilities.

### Required capabilities
- ask_user(question, options?)
- write_artifact(path, content)
- save_state(key, value)
- load_state(key)

### Strongly recommended capabilities
- list_files()
- read_file(path)
- search_workspace(query)
- read_workspace()

### Optional capabilities
- spawn_subagent(role, prompt) for scoped exploration
- score_with_model(prompt, temperature) for stable structured scoring
- get_timestamp()
- generate_uuid()

If no subagent system exists, the primary agent may perform exploration itself.

## Runtime-Neutral Behavior

This skill does not assume:
- a specific orchestrator,
- a specific planning framework,
- a specific folder layout,
- or a specific execution engine.

It only assumes that:
1. it can ask the user questions,
2. it can inspect the environment when relevant,
3. it can persist state,
4. and it can write a final artifact.

## Interview State Schema

Persist state under a runtime-defined key such as:

`deep-interview:<session-or-task-id>`

### State JSON

```json
{
  "active": true,
  "current_phase": "deep-interview",
  "state": {
    "interview_id": "<uuid>",
    "type": "greenfield|brownfield",
    "initial_idea": "<user input>",
    "rounds": [],
    "current_ambiguity": 1.0,
    "threshold": 0.2,
    "codebase_context": null,
    "challenge_modes_used": [],
    "ontology_snapshots": [],
    "status": "interviewing"
  }
}
```

## Project Type Detection

### Greenfield
Use greenfield when:
- there is no existing repo/workspace context,
- the request is clearly for something new,
- or codebase exploration is unavailable.

### Brownfield
Use brownfield when:
- a repo or source tree exists,
- relevant source files or manifests exist,
- or the request appears to modify or extend an existing system.

If exploration fails, default to greenfield and note the limitation.

## Brownfield Exploration Policy

Before asking the user about implementation context:
- inspect likely relevant files,
- identify existing modules, auth paths, APIs, routes, schemas, services, jobs, or UI surfaces,
- summarize findings into `codebase_context`.

Never ask the user questions that the workspace can already answer directly.

### Good
> "I found JWT auth middleware in src/auth/. Should this feature extend that path or intentionally diverge?"

### Bad
> "What authentication system does your project use?"

Facts are for the repo. Decisions are for the user.

## Clarity Dimensions

### Greenfield dimensions
1. Goal Clarity
2. Constraint Clarity
3. Success Criteria Clarity

### Brownfield dimensions
1. Goal Clarity
2. Constraint Clarity
3. Success Criteria Clarity
4. Context Clarity

### Dimension definitions

#### Goal Clarity
Can the primary objective be stated in one sentence without major ambiguity?
Are the key nouns and verbs stable?

#### Constraint Clarity
Are limitations, boundaries, assumptions, dependencies, and non-goals clear?

#### Success Criteria Clarity
Could a reviewer, tester, or evaluator determine whether the work is correct?

#### Context Clarity
For brownfield work: is the relationship between the requested change and the existing system understood well enough to modify it safely?

## Ambiguity Formula

### Greenfield

```text
ambiguity = 1 - (goal × 0.40 + constraints × 0.30 + criteria × 0.30)
```

### Brownfield

```text
ambiguity = 1 - (goal × 0.35 + constraints × 0.25 + criteria × 0.25 + context × 0.15)
```

### Default threshold

```text
ambiguity <= 0.20
```

This threshold means the work is clear enough to proceed without likely major rework.

## Execution Policy

- Ask one question at a time.
- Never batch multiple unrelated questions.
- Target the weakest clarity dimension every round.
- Explicitly explain why that dimension is now the bottleneck.
- Ask assumption-exposing questions rather than feature-list questions.
- For brownfield tasks, gather codebase facts before asking the user.
- Cite discovered workspace evidence when asking brownfield confirmation questions.
- Re-score clarity after every answer.
- Do not proceed to execution unless ambiguity is below threshold or the user explicitly chooses early exit.
- Persist state after every round.
- Support resume after interruption.
- Activate challenge modes at defined round thresholds.

## Interview Flow

### Phase 1: Initialize

1. Parse the user's idea from arguments.
2. Detect whether the task is greenfield or brownfield.
3. If brownfield, inspect the workspace and collect `codebase_context`.
4. Initialize interview state.
5. Announce the interview.

Example:

> Starting deep interview. I'll ask targeted questions to reduce ambiguity before execution. After each answer, I'll show a clarity score and the current ambiguity level.

Display:
- initial idea
- project type
- current ambiguity = 100%

### Phase 2: Interview Loop

Repeat until:
- ambiguity <= threshold,
- the user exits early,
- or maximum rounds is reached.

## Question Generation

Generate the next question from:
- the original idea,
- all previous Q&A rounds,
- current clarity scores,
- current ontology snapshot,
- codebase context if brownfield,
- active challenge mode if applicable.

### Targeting rule
- identify the lowest-scoring dimension,
- ask a question designed to improve that dimension,
- state why that dimension is now the limiting factor.

### Style rule
Questions should expose assumptions, not merely gather preferences.

### Ontology rule
If the main entities keep shifting or the user keeps describing symptoms instead of the core thing, switch to ontology-style questioning:
- "What is the core thing here?"
- "Which entity is primary?"
- "Which terms are just views or containers?"

## Question Styles by Dimension

| Dimension | Question Style | Example |
|---|---|---|
| Goal Clarity | What exactly is this thing? | "When you say 'manage tasks', what is the first concrete action the user takes?" |
| Constraint Clarity | What boundaries exist? | "Does this need to work offline, or is network access assumed?" |
| Success Criteria | How do we know it works? | "What would make you say 'yes, this is correct' if I showed the finished result?" |
| Context Clarity | How does this fit the current system? | "I found an existing JWT middleware path in src/auth/. Should this feature reuse that path?" |
| Ontology | What is the fundamental concept? | "You've mentioned tasks, projects, and workspaces. Which one is the core entity?" |

## Asking the Question

Use a format like:

```text
Round {n} | Targeting: {weakest_dimension} | Why now: {rationale} | Ambiguity: {score}%

{question}
```

Optional multiple-choice helpers may be offered, but free-text answers must always be allowed.

## Scoring Clarity

After each answer, score each dimension from 0.0 to 1.0.

For each dimension provide:
- score
- justification
- gap

Also provide:
- weakest_dimension
- weakest_dimension_rationale

### Scoring criteria

#### Goal Clarity
- Is the objective unambiguous?
- Are the main entities and relationships stable?
- Can the core problem be stated precisely?

#### Constraint Clarity
- Are boundaries and limitations clear?
- Are assumptions visible?
- Are non-goals identified?

#### Success Criteria Clarity
- Could a test or review checklist verify success?
- Is "done" operationally meaningful?

#### Context Clarity
- Does the change map cleanly to the current system?
- Is there enough context to modify the system safely?

## Ontology Extraction

After each round, extract all key entities discussed so far.

For each entity record:
- name
- type
- fields
- relationships

Example:

```json
{
  "name": "Task",
  "type": "core domain",
  "fields": ["title", "status", "dueDate"],
  "relationships": ["Task belongs to Project", "Task has many Tags"]
}
```

Ontology extraction should reuse existing names when the concept is clearly the same.
Only introduce a new entity name when the concept is genuinely new.

## Ontology Stability

### Round 1
No stability comparison is calculated.
All entities are treated as new.

### Round 2 and later
Compare the current ontology against the previous round.

Classify entities as:
- stable_entities: same name persists
- changed_entities: different name, but same type and >50% field overlap; treat as renamed, not replaced
- new_entities: not matched to prior entities
- removed_entities: present before, absent now

### Stability ratio

```text
stability_ratio = (stable + changed) / total_current_entities
```

### Matching transparency
Before reporting stability, briefly show:
- which entities matched directly,
- which matched fuzzily as renames,
- which are new,
- which were removed.

## Progress Report

After each round, report progress in this format:

```text
Round {n} complete.
| Dimension | Score | Weight | Weighted | Gap |
|-----------|-------|--------|----------|-----|
| Goal | {s} | {w} | {s*w} | {gap or "Clear"} |
| Constraints | {s} | {w} | {s*w} | {gap or "Clear"} |
| Success Criteria | {s} | {w} | {s*w} | {gap or "Clear"} |
| Context | {s} | {w} | {s*w} | {gap or "Clear"} |
| Ambiguity |  |  | {score}% |  |

Ontology:
- entities: {count}
- stability: {ratio}
- stable: {stable}
- changed: {changed}
- new: {new}
- removed: {removed}

Next target:
{weakest_dimension} — {weakest_dimension_rationale}
```

If threshold is met:
> Clarity threshold met. Ready to produce final spec.

## Challenge Modes

Challenge modes are perspective shifts, not separate tool invocations.

### Round 4+: Contrarian Mode
Ask a question that challenges a core assumption.

Examples:
- "What if the opposite were true?"
- "What if this constraint is habitual rather than real?"
- "If we removed this requirement, what actually breaks?"

Purpose:
- expose false premises,
- test whether the framing is valid.

### Round 6+: Simplifier Mode
Ask what can be removed.

Examples:
- "What is the simplest version that would still be valuable?"
- "Which of these requirements are truly necessary?"
- "What can be deferred without harming the core value?"

Purpose:
- collapse accidental complexity,
- find the minimum viable specification.

### Round 8+: Ontologist Mode
If ambiguity is still above 0.30, switch to ontology-first questioning.

Examples:
- "What is this thing, really?"
- "Which entity is the core concept?"
- "Which terms are just views, metaphors, or containers?"

Purpose:
- stabilize the domain model,
- stop symptom-level questioning,
- recover from drifting terminology.

Each challenge mode is used once and recorded in state.

## Soft Limits and Stop Conditions

### Soft warning at Round 10
Display:
- current ambiguity,
- most unclear dimensions,
- option to continue or proceed.

### Hard cap at Round 20
Stop asking more questions and generate a spec with a risk warning.

### Early exit
Allowed after Round 3.

If ambiguity is still above threshold, warn the user:
- which dimensions are still unclear,
- why execution may cause rework,
- and what risk remains.

### Ambiguity stall rule
If ambiguity stays within ±0.05 for 3 rounds:
- activate ontologist reframing.

### Fast-path completion
If all dimensions reach 0.90+:
- generate final spec immediately.

## Finalization

When the interview ends due to:
- threshold met,
- early exit,
- or hard cap,

generate the final artifacts:
1. clarified-spec.md
2. readiness-report.json
3. interview-state.json

Paths may be adapted to the host runtime.

## Final Spec Structure

```markdown
# Deep Interview Spec: {title}

## Metadata
- Interview ID: {uuid}
- Rounds: {count}
- Final Ambiguity Score: {score}%
- Type: greenfield | brownfield
- Generated: {timestamp}
- Threshold: {threshold}
- Status: {PASSED | EARLY_EXIT | MAX_ROUNDS_REACHED}

## Clarity Breakdown
| Dimension | Score | Weight | Weighted |
|-----------|-------|--------|----------|
| Goal Clarity | {s} | {w} | {s*w} |
| Constraint Clarity | {s} | {w} | {s*w} |
| Success Criteria Clarity | {s} | {w} | {s*w} |
| Context Clarity | {s} | {w} | {s*w} |
| Total Clarity |  |  | {total} |
| Ambiguity |  |  | {ambiguity} |

## Goal
{clear goal statement}

## Constraints
- {constraint 1}
- {constraint 2}

## Non-Goals
- {non-goal 1}
- {non-goal 2}

## Acceptance Criteria
- [ ] {criterion 1}
- [ ] {criterion 2}
- [ ] {criterion 3}

## Assumptions Exposed and Resolved
| Assumption | Challenge | Resolution |
|------------|-----------|------------|
| {a} | {challenge} | {resolution} |

## Technical Context
{brownfield findings or greenfield architecture assumptions}

## Ontology (Key Entities)
| Entity | Type | Fields | Relationships |
|--------|------|--------|---------------|
| {entity.name} | {entity.type} | {entity.fields} | {entity.relationships} |

## Ontology Convergence
| Round | Entity Count | New | Changed | Stable | Stability Ratio |
|-------|-------------|-----|---------|--------|----------------|
| 1 | {n} | {n} | - | - | - |
| 2 | {n} | {new} | {changed} | {stable} | {ratio}% |
| ... | ... | ... | ... | ... | ... |

## Interview Transcript
<details>
<summary>Full Q&A</summary>

### Round 1
Q: {question}
A: {answer}
Ambiguity: {score}%

### Round 2
Q: {question}
A: {answer}
Ambiguity: {score}%

...
</details>
```

## Readiness Report Structure

```json
{
  "interview_id": "<uuid>",
  "status": "ready|not-ready|early-exit|max-rounds",
  "type": "greenfield|brownfield",
  "final_ambiguity": 0.18,
  "threshold": 0.2,
  "rounds": 6,
  "weakest_dimension": "Constraint Clarity",
  "recommended_next_action": "execute|plan-more|continue-interview",
  "artifacts": {
    "spec": "clarified-spec.md",
    "state": "interview-state.json"
  }
}
```

## Generic Execution Handoff

This skill does not implement the requested work directly.

Instead, it emits a normalized handoff package that any downstream agent can consume.

```json
{
  "handoff_type": "clarified-spec",
  "ready_for_execution": true,
  "spec_path": "clarified-spec.md",
  "ambiguity": 0.18,
  "project_type": "brownfield",
  "acceptance_criteria_count": 7,
  "core_entities": ["User", "Task", "Project"],
  "recommended_mode": "planning-first"
}
```

### Possible next actions
- hand off to a planner agent,
- hand off to an executor agent,
- continue interviewing,
- stop and save for later.

The host runtime decides which downstream agent receives the handoff.

## Autoresearch Mode

If `--autoresearch` is enabled:

### Readiness gates
The skill must clarify:
1. mission clarity
2. evaluator clarity
3. normal ambiguity threshold

### First question
If no mission is available yet, start with:
> "What should the research improve, validate, or disprove?"

### Evaluator collection
Then gather an evaluator:
- benchmark command,
- rubric,
- expected output format,
- target metric,
- or review condition.

If the evaluator is still vague, keep interviewing until it is actionable.

### Output
When ready, produce a research brief instead of an implementation spec.

### Research brief fields
- mission
- evaluator
- scope boundaries
- success condition
- workspace context
- safety constraints
- assumptions still open

## Resume Behavior

If interrupted, resume using persisted state.

Resume should:
- load prior interview state,
- continue from the next unresolved round,
- preserve ambiguity history,
- preserve ontology snapshots,
- and avoid repeating already-answered questions.

## Configuration

```json
{
  "deepInterview": {
    "ambiguityThreshold": 0.2,
    "maxRounds": 20,
    "softWarningRounds": 10,
    "minRoundsBeforeExit": 3,
    "enableChallengeModes": true,
    "autoGenerateSpec": true,
    "scoringTemperature": 0.1
  }
}
```

## Good Patterns

### Good: target weakest dimension
> "Constraints are currently the weakest area, so the next question focuses on environment boundaries rather than features."

### Good: inspect first, ask second
> "I found auth middleware in src/auth/. Should the new capability reuse that existing path?"

### Good: challenge assumptions
> "You said 10,000 concurrent users. What if the true requirement is 100? Would the architecture fundamentally change?"

### Good: stabilize ontology
> "You've described this as a workflow, planner, and inbox. Which is the core concept?"

## Bad Patterns

### Bad: batch questions
> "Who is it for, what stack, how should auth work, and where should it deploy?"

### Bad: ask for code facts already available
> "What database do you use?" when manifests or configs already reveal it.

### Bad: proceed while still highly unclear
> "We're at 45% ambiguity, but let's build anyway."

## Stop Conditions

Stop immediately if:
- the user says stop, cancel, or abort,
- ambiguity reaches threshold,
- the hard round cap is reached,
- the user explicitly chooses to proceed despite risk.

## Final Checklist

- [ ] One question per round
- [ ] Weakest dimension explicitly named each round
- [ ] Ambiguity shown after each round
- [ ] Ontology extracted each round
- [ ] Ontology stability tracked across rounds
- [ ] Challenge modes activated at thresholds
- [ ] Final spec written
- [ ] Readiness report written
- [ ] State persisted for resume
- [ ] Generic handoff package emitted
- [ ] No dependency on any specific planner, orchestrator, or execution framework

---
> Source: [pinion05/deep-interview](https://github.com/pinion05/deep-interview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
