## creative-loop-1

> Run a structured creative thinking loop on any problem. Generates diverse ideas via parallel personas, evaluates them, refines top candidates, and runs experiments to validate. Use when facing open-ended design decisions, architecture choices, or any problem that would benefit from genuine divergent thinking before converging on a solution.


When the user invokes `/creative`, you enter a structured creative thinking mode. This skill transforms any problem into a multi-perspective exploration. Follow these instructions precisely.

---

## COMMANDS

### `/creative init`
Initialize creative loop in the current project. Create `.creative-loop/` directory structure with all required files. Copy default personas and prompts. Create `config.json`. Tell the user it's ready.

### `/creative <problem>`
Run the full creative loop on the problem. Follow the FULL LOOP PROTOCOL below.

### `/creative quick <problem>`
Run a fast version: 3 personas, no experiments, 1 iteration. Follow QUICK MODE PROTOCOL.

### `/creative evaluate <idea>`
Skip generation. Evaluate a user-provided idea using the EVALUATION PHASE only.

### `/creative experiment <idea>`
Skip generation and evaluation. Run the EXPERIMENT PHASE on the idea.

### `/creative optimize`
Run SELF-OPTIMIZATION PROTOCOL to improve the loop's own prompts.

### `/creative sync`
Sync self-optimized improvements back to the canonical repo so they can be shared with others. Handles all three cases: local changes on this machine, improvements from another project on the same machine, and improvements from a different machine entirely. Follow the SYNC PROTOCOL.

### `/creative status`
Show the most recent session summary from `.creative-loop/sessions/`.

### `/creative help`
List these commands with brief descriptions.

---

## FULL LOOP PROTOCOL

Work through these phases in order. Be transparent with the user about what phase you're in.

---

### PHASE 0: CHECK SETUP

1. Check if `.creative-loop/` exists in the current directory.
2. If not: tell the user and offer to run `/creative init` first. Stop here.
3. Read `.creative-loop/config.json`. Use its settings throughout.
4. Create a session directory: `.creative-loop/sessions/{YYYYMMDD_HHMMSS}/`

---

### PHASE 1: CREATIVE BRIEF

Generate a structured creative brief from the user's problem statement.

**Read the problem** alongside:
- CLAUDE.md (project context, constraints, conventions)
- Recent git log summary if available: `git log --oneline -10`
- Any relevant file the user mentioned

**Write** `.creative-loop/sessions/{timestamp}/brief.json`:
```json
{
  "problem_statement": "Clear one-sentence description",
  "constraints": ["constraint1", "constraint2"],
  "success_criteria": ["criterion1", "criterion2"],
  "domain": "e.g. backend-performance, api-design, ux, architecture",
  "tags": ["relevant", "tags"],
  "exploration_budget": "low|medium|high",
  "context_summary": "2-3 sentences about relevant project context",
  "known_attempts": ["anything the user already tried"]
}
```

**Present the brief to the user** and ask: "Does this capture it, or should I adjust anything before we explore?"

Wait for confirmation. Update brief.json if needed.

---

### PHASE 2: DIVERGE — GENERATE IDEAS

Select personas and launch generators in parallel.

**Load persona files** from `.creative-loop/personas/` — first check `custom/` then `builtin/`.

**Select 5 personas** using this logic:
1. Pick personas matching the domain (check persona `domains` field)
2. Always include at least 1 "wild card" persona (Provocateur or Constraint Inverter)
3. Check `.creative-loop/patterns/persona_effectiveness.json` — deprioritize personas with hit_rate < 0.2 for this domain
4. Fill remaining slots with highest-effectiveness personas

**Load creative memory** from `.creative-loop/patterns/successful.json` and `failed.json`. Select entries matching the brief's domain and tags.

**Tell the user**: "Launching 5 generator agents in parallel. Each explores the problem from a different angle..."

**Launch 5 sub-agents in parallel** (one per persona). Each sub-agent receives this prompt — fill in the placeholders:

```
## Your Creative Persona: {PERSONA_NAME}

### Your Thinking Frame
{PERSONA_THINKING_FRAME}

### The Problem to Explore
{PROBLEM_STATEMENT}

### Constraints
{CONSTRAINTS}

### Project Context
{CONTEXT_SUMMARY}

### Successful Patterns (from past sessions — consider these as starting points)
{RELEVANT_SUCCESSFUL_PATTERNS or "None yet"}

### Approaches to Avoid (known failures)
{RELEVANT_FAILED_APPROACHES or "None yet"}

### Your Task
Generate 3 candidate ideas using your specific thinking frame. Do NOT produce safe, obvious ideas — your persona demands a specific unconventional angle.

For each idea, produce:
- **title**: Short descriptive name
- **core_idea**: What this approach is (2-3 sentences)
- **mechanism**: How it would work technically (3-5 sentences)
- **novelty_claim**: What assumption this challenges or what makes it non-obvious
- **risks**: 1-3 specific risks
- **confidence**: 0.0-1.0 how strongly you believe this is worth exploring
- **thinking_process**: 1-2 sentences on how your persona shaped this idea

Think boldly. The evaluation phase will filter — your job is to expand the possibility space.

Return a JSON object with key "ideas" containing an array of idea objects, and key "dead_ends" listing approaches you considered but rejected (with brief reasons).
```

**Collect all results** and merge into a single candidates array. Write to `.creative-loop/sessions/{timestamp}/candidates.json`.

**If include_combination_round is true in config:** Launch one additional sub-agent to look for unexpected combinations:

```
You are looking for creative combinations. Here are {N} ideas generated by different thinkers for this problem: {IDEAS_SUMMARY}

Identify 2-3 pairs or groups of ideas that would be stronger together than separately. For each combination, explain the synergy in 2 sentences.

Return JSON with key "combinations" containing an array of {ideas: [id1, id2], synergy: "..."} objects.
```

---

### PHASE 3: EVALUATE

Score all candidates independently.

**Load evaluation config** from `config.json` (weights, evaluator count).

**Tell the user**: "Evaluating {N} ideas across 5 scoring axes..."

**Launch 2 evaluator sub-agents in parallel**. Each receives ALL candidates (randomized order — different randomization per evaluator). Use this prompt:

```
## Evaluation Task

You are evaluating {N} creative ideas generated for this problem:
"{PROBLEM_STATEMENT}"

Constraints: {CONSTRAINTS}
Success criteria: {SUCCESS_CRITERIA}

### Ideas to Evaluate
{ALL_CANDIDATES_JSON}

### Scoring Instructions
For EACH idea, score these axes from 0.0 to 1.0:
- **novelty**: Is this genuinely different from obvious approaches? 0=first thing anyone would try, 1=fundamentally reframes the problem
- **feasibility**: Can this be built given the constraints? 0=impossible with current tools, 1=straightforward to implement
- **relevance**: Does this solve the stated problem? 0=interesting but wrong problem, 1=directly and completely
- **risk**: How severe are the failure modes? 0=catastrophic/irreversible, 1=low-risk/easy to roll back
- **elegance**: Is the solution clean and maintainable? 0=unnecessary complexity, 1=beautifully simple

IMPORTANT CALIBRATION:
- Use the full range. Do not cluster scores in the 0.5-0.7 band.
- Unconventional ≠ infeasible. Score these axes independently.
- A 0.9 novelty score is warranted when the idea is genuinely surprising.

For each score, write a 1-sentence rationale.

Also identify:
- Any ideas that should be COMBINED (note the IDs)
- Any REDUNDANT ideas (essentially the same approach)
- Any with HIDDEN POTENTIAL (low obvious appeal but interesting kernel worth developing)

Return JSON with key "evaluations" (array of scored ideas) and keys "combinations", "redundancies", "hidden_potential".
```

**Aggregate scores:**
1. For each idea, take the median score per axis across evaluators
2. Compute composite: `(novelty×0.25) + (feasibility×0.25) + (relevance×0.25) + ((1-risk)×0.15) + (elegance×0.10)`
3. Flag any idea where evaluators differ by >0.3 on any axis (mark as `"disputed": true`)
4. Collect combination/hidden_potential flags from either evaluator

**Write** `.creative-loop/sessions/{timestamp}/evaluations.json`.

---

### PHASE 4: SELECT

Choose which ideas to develop further.

**Apply selection policy** (from config — default: "balanced"):

| Mode | Logic |
|---|---|
| `balanced` | Top 3 by composite score + 1 wild card (highest novelty not already in top 3) |
| `exploit` | Top 3 by composite only |
| `explore` | Top 3 by novelty score |
| `consensus` | Only ideas where both evaluators scored composite > 0.6 |

**Always add** evaluator-suggested combinations to the shortlist.

**Write** `.creative-loop/sessions/{timestamp}/selected.json`.

**Present selection to user:**

```
## Ideas Selected for Development

**1. {TITLE}** (Composite: {SCORE})
  Novelty: {N} | Feasibility: {F} | Relevance: {R}
  {CORE_IDEA_ONE_SENTENCE}
  Generated by: {PERSONA}

**2. {TITLE}** (Composite: {SCORE})
  ...

**[Wild Card] {TITLE}** (Composite: {SCORE}, Novelty: {N})
  {CORE_IDEA_ONE_SENTENCE}
  Kept because: high novelty even with lower composite

**Suggested combination:** {IDEA_1} + {IDEA_2}
  "{SYNERGY}"

Shall I proceed to develop and test these, adjust the selection, or stop here?
```

Wait for user input. Respect changes.

---

### PHASE 5: REFINE

Deepen the selected ideas with targeted sub-agents.

**For each selected idea**, launch a refinement sub-agent:

```
## Refinement Task

You are developing this idea into a concrete proposal:

### Idea
Title: {TITLE}
Core: {CORE_IDEA}
Mechanism: {MECHANISM}
Novelty claim: {NOVELTY_CLAIM}
Risks: {RISKS}

### Problem
{PROBLEM_STATEMENT}
Constraints: {CONSTRAINTS}

### Your Task
Deepen this idea into an actionable proposal. Go beyond restating the idea — add:

1. **Concrete implementation sketch** — Specific steps, components, or code structure needed
2. **Key decisions** — What choices would need to be made during implementation?
3. **Addressed risks** — For each identified risk, suggest a mitigation
4. **Open questions** — What would need to be validated before committing?
5. **Effort estimate** — Rough sizing (S/M/L) with rationale
6. **Quick win** — What's the smallest version that would prove the core idea works?

Be specific. Vague refinement is useless. Name actual functions, files, or patterns where relevant.

Return JSON with these keys: implementation_sketch, key_decisions, mitigations, open_questions, effort_estimate, quick_win.
```

**If combinations were suggested**, launch an additional sub-agent to develop the combination:

```
## Combination Development

These two ideas complement each other:

Idea A: {TITLE_A} — {CORE_A}
Idea B: {TITLE_B} — {CORE_B}
Suggested synergy: {SYNERGY}

Develop how these would work together. What does the combined approach look like? How do they integrate? What does each contribute that the other lacks?

Return a combined proposal in the same format as a refined idea.
```

**Write** `.creative-loop/sessions/{timestamp}/refined.json`.

---

### PHASE 6: EXPERIMENT

Validate assumptions by testing.

**For each refined proposal**, determine if experimentation is warranted:
- If the idea involves a claim that can be tested with code → run a code prototype
- If the idea modifies existing behavior → run the test suite
- If the idea is based on assumptions about the codebase → do static analysis
- If the idea is purely conceptual (naming, process, architecture pattern) → skip

**For each experiment**, tell the user what you're testing and why.

Create `.creative-loop/sessions/{timestamp}/experiments/{idea_id}/`.

Run experiments using Bash. Write results to the experiment directory. Keep it scoped:
- Small: one script, under 50 lines, runs in seconds
- Medium: multi-file, integration test, runs in under 2 minutes
- Never modify project source files from within an experiment

**Write** `.creative-loop/sessions/{timestamp}/experiments/{idea_id}/result.json`:
```json
{
  "hypothesis": "...",
  "method": "...",
  "outcome": "validated|invalidated|inconclusive",
  "key_findings": ["..."],
  "recommendation": "proceed|revisit|discard"
}
```

---

### PHASE 7: CAPTURE AND PRESENT

Synthesize everything and present to the user.

**Present results:**

```
## Creative Exploration Complete
Problem: {PROBLEM_STATEMENT}
Iterations: {N} | Ideas generated: {TOTAL} | Developed: {SELECTED_COUNT}

---

### Proposal 1: {TITLE}
Score: {COMPOSITE} | Effort: {SIZE}

{CORE_IDEA}

**How it works:** {IMPLEMENTATION_SKETCH_SUMMARY}
**Quick win:** {QUICK_WIN}
**Main risk:** {TOP_RISK} → {MITIGATION}
**Experiment result:** {VALIDATED|NOT TESTED|INVALIDATED}
*Generated by {PERSONA}*

---

### Proposal 2: {TITLE}
...

---

### What I'd suggest
{1-2 sentence recommendation based on scores, experiments, and constraints}

---

What would you like to do?
→ **Implement** one of these (I'll create the implementation plan)
→ **Iterate** with new constraints or direction
→ **Combine** elements from multiple proposals
→ **Save and continue later**
→ **None of these** (help me improve the loop)
```

**Update creative memory** based on outcomes:

1. **Session log**: Write `outcome.md` to the session directory with a human-readable summary
2. **Patterns**: For any proposal the user decides to implement → add to `successful.json`
3. **Failures**: For ideas invalidated by experiments → add to `failed.json` with reason
4. **Persona effectiveness**: Update `persona_effectiveness.json` — which personas' ideas made it to the final shortlist?
5. **Prompt evolution**: If any phase felt weak (user said "none useful", or ideas lacked diversity), flag it in `prompt_evolution.json` as a candidate for improvement

**Run lightweight self-assessment** (see SELF-OPTIMIZATION PROTOCOL appendix). If issues are found, mention them briefly to the user: "I noticed the generated ideas were quite similar in this session. Would you like me to run `/creative optimize` to strengthen the persona prompts?"

---

## QUICK MODE PROTOCOL

For `/creative quick <problem>`:

1. Generate brief (no confirmation needed, just show it)
2. Select 3 personas (1 domain-match, 1 effective-from-memory, 1 wild card)
3. Run PHASE 2 (generate) with 3 agents instead of 5
4. Run PHASE 3 (evaluate) with 1 evaluator
5. Run PHASE 4 (select) — top 2 + wild card
6. Skip PHASE 5 (refine) — surface raw ideas
7. Skip PHASE 6 (experiment)
8. Present ideas in condensed format
9. Ask: "Want me to develop any of these further?"

---

## SELF-OPTIMIZATION PROTOCOL

For `/creative optimize` or as a post-session assessment:

**Collect signals from recent sessions** (read last 5 session `outcome.md` files):

1. **Diversity check**: Were ideas from different personas significantly different? Flag if cross-idea similarity was high.
2. **Calibration check**: Did evaluator predictions match experiment outcomes? Flag systematic over/under-scoring.
3. **Adoption check**: Did the user implement any proposed ideas? Low adoption → ideas aren't landing.
4. **Iteration count**: More than 2 iterations per session → initial generation isn't landing close enough.

**For each flagged weakness**, propose a specific prompt improvement:
- Show the current text
- Show the proposed new text
- Explain why it should help

**Apply changes with approval:**
- Present all proposed changes clearly
- Ask: "Apply these improvements? [Yes / Review each one / Skip]"
- If yes: edit the relevant files in `.creative-loop/personas/` or `.creative-loop/prompts/`
- Log all changes to `.creative-loop/patterns/prompt_evolution.json`

**You can also update the skill file itself** (`~/.claude/skills/creative/SKILL.md`) if the orchestration logic needs improvement. Treat this with care — propose it explicitly, not automatically.

---

## INIT PROTOCOL

For `/creative init`:

Create this structure in the current directory:

```
.creative-loop/
├── config.json
├── personas/
│   ├── builtin/   (populated from templates below)
│   └── custom/
├── prompts/       (empty — prompts are embedded in the skill)
├── sessions/
├── patterns/
│   ├── successful.json     (empty array [])
│   ├── failed.json         (empty array [])
│   ├── combinations.json   (empty array [])
│   ├── persona_effectiveness.json  (see default below)
│   └── prompt_evolution.json (empty array [])
└── artifacts/
```

Write `config.json`:
```json
{
  "version": "1.0",
  "generator": {
    "default_persona_count": 5,
    "ideas_per_persona": 3,
    "max_parallel_agents": 6,
    "include_combination_round": true,
    "constraint_injection_probability": 0.3
  },
  "evaluator": {
    "evaluator_count": 2,
    "weights": {
      "novelty": 0.25,
      "feasibility": 0.25,
      "relevance": 0.25,
      "risk": 0.15,
      "elegance": 0.10
    },
    "disagreement_threshold": 0.3
  },
  "meta_controller": {
    "default_selection_mode": "balanced",
    "top_k": 3,
    "include_wild_card": true,
    "quality_threshold": 0.7,
    "max_iterations": 3
  },
  "experiment_runner": {
    "enabled": true,
    "default_effort_budget": "small"
  },
  "memory": {
    "enabled": true,
    "max_patterns_per_retrieval": 10,
    "min_retrieval_confidence": 0.3
  },
  "self_optimization": {
    "enabled": true,
    "require_approval": true,
    "auto_assess_after_session": true,
    "min_sessions_before_optimize": 3
  },
  "human_oversight": {
    "review_brief": true,
    "review_selection": true
  }
}
```

Write `persona_effectiveness.json`:
```json
{
  "first_principles": {"domains": {}},
  "devils_advocate": {"domains": {}},
  "cross_domain_analogist": {"domains": {}},
  "naive_questioner": {"domains": {}},
  "constraint_inverter": {"domains": {}},
  "user_empathist": {"domains": {}},
  "minimalist": {"domains": {}},
  "futurist": {"domains": {}},
  "integrator": {"domains": {}},
  "provocateur": {"domains": {}}
}
```

Copy builtin persona files. Prefer the globally installed versions over the embedded templates — they may have been improved by `/creative optimize` and synced back:
- If `~/.claude/skills/creative/personas/builtin/` exists: copy from there
- Otherwise: create each persona file from the PERSONA LIBRARY definitions below

This means projects initialized after a `/creative sync` + `bash install.sh` automatically get the latest improved personas.

Tell the user:
```
Creative Loop initialized in .creative-loop/

Next steps:
- Run `/creative <your problem>` to start exploring
- Edit .creative-loop/config.json to tune behavior
- Add custom personas in .creative-loop/personas/custom/
- Run `/creative optimize` after a few sessions to improve prompts
```

Offer to add these lines to CLAUDE.md:
```markdown
## Creative Loop
This project uses the Creative Loop for open-ended design decisions.
Run `/creative <problem>` to explore options before committing.
Config: .creative-loop/config.json
```

---

## PERSONA LIBRARY

These are the 10 built-in personas. When writing persona files during `/creative init`, use these definitions.

### first-principles.md
```
---
id: first_principles
name: First Principles
domains: [architecture, backend, api-design, refactoring]
description: Decompose to fundamentals, rebuild from scratch
---

## Thinking Frame
Strip away all existing structure and inherited assumptions. Ask: what is the actual problem at the atomic level? What does this system fundamentally need to do, not what it currently does? Rebuild a solution from the ground up without being anchored by existing implementation.

## Approach
1. Identify the core purpose — the one sentence that defines what success looks like
2. List all constraints that are truly immovable vs. assumed-immovable
3. Rebuild the solution touching only truly immovable constraints
4. Challenge existing interfaces, abstractions, and patterns as inherited choices, not requirements

## When Most Effective
Architecture rewrites, performance bottlenecks rooted in design, API designs that have grown organically and become inconsistent.
```

### devils-advocate.md
```
---
id: devils_advocate
name: Devil's Advocate
domains: [all]
description: Challenge every assumption, find what others miss
---

## Thinking Frame
Your job is to be the most rigorous critic in the room. Find the edge case that breaks the design. Surface the uncomfortable question nobody wants to ask. Find the assumption everyone treats as a law of physics but is actually a choice.

## Approach
1. Take the obvious solution and list every way it fails
2. Find the failure mode that only appears at scale, under adversarial use, or after 6 months
3. Find the assumption baked into the problem statement itself and invert it
4. Ask: who does this solution actively harm or disadvantage?

## When Most Effective
Every session — this persona prevents groupthink. Especially valuable when the team already has a preferred solution and needs to stress-test it.
```

### cross-domain-analogist.md
```
---
id: cross_domain_analogist
name: Cross-Domain Analogist
domains: [all]
description: Find solved versions of this problem in completely different fields
---

## Thinking Frame
This exact problem — or something structurally identical — has already been solved somewhere. Biology, logistics, urban planning, music composition, electrical engineering: they all solve resource allocation, caching, consensus, prioritization, and flow control. Find the analogy and import the solution.

## Approach
1. Abstract the problem to its structural essence (ignore the domain-specific vocabulary)
2. Find 2-3 fields that deal with the same structural problem
3. Describe how those fields solve it
4. Translate the solution back into the original domain

## When Most Effective
Novel problems, performance optimization, distributed systems, coordination problems, anything involving flow, queues, or resource contention.
```

### naive-questioner.md
```
---
id: naive_questioner
name: Naive Questioner
domains: [all]
description: Ask "why?" at every level — uncover hidden complexity and false constraints
---

## Thinking Frame
You are someone highly intelligent who has never worked in this domain. You have no respect for "that's just how it's done." Every convention is a choice. Every constraint is potentially self-imposed. Ask why at every level until you hit bedrock.

## Approach
1. List the first three assumptions embedded in the problem statement
2. Ask "why?" for each — and then "why?" again for the answer
3. Find the place where someone says "because that's just how it works" — that's where the interesting solution lives
4. Propose the solution you'd design if you had no existing code to be compatible with

## When Most Effective
Long-lived codebases with accumulated complexity, process problems, anything involving legacy constraints.
```

### constraint-inverter.md
```
---
id: constraint_inverter
name: Constraint Inverter
domains: [all]
description: Remove the main constraint, or make it harder — reframe the solution space
---

## Thinking Frame
Constraints define the solution space. Change the constraints, change the solutions available. Remove the hardest constraint and see what becomes possible. Or make it even harder than it is — the solution that works under extreme constraint often works everywhere.

## Approach
1. Identify the constraint that most limits the solution space
2. Design the solution if that constraint didn't exist — this reveals the ideal architecture
3. Now add the constraint back and find the minimal adaptation needed
4. Also try: what if the constraint were 10x harder? Sometimes extreme constraints force elegant solutions

## When Most Effective
Performance problems, resource constraints, compatibility requirements, hard deadlines.
```

### user-empathist.md
```
---
id: user_empathist
name: User Empathist
domains: [ux, api-design, developer-experience, frontend]
description: Think from the end user's lived experience — ground everything in real needs
---

## Thinking Frame
Technology exists to serve people. Every technical solution has a human on the other end — a developer calling this API, a user clicking this button, an operator reading these logs. What is their actual experience? What do they actually need, not what we've decided they need?

## Approach
1. Walk through the user journey step by step — every touchpoint, every possible confusion
2. Find the moment where the user has to hold the most in their head at once — reduce that
3. Find the moment where an error is possible and design for graceful failure
4. Ask: what does the user want to do, not what does the system want them to do?

## When Most Effective
API design, developer tools, error handling, onboarding flows, anything where the user experience of a technical system matters.
```

### minimalist.md
```
---
id: minimalist
name: Minimalist
domains: [all]
description: What is the simplest thing that could possibly work?
---

## Thinking Frame
Complexity is a liability. Every abstraction has a cost. Every dependency is a risk. Every line of code is something that can break. The best solution is the one that solves the problem with the least machinery. Resist the urge to design for hypothetical futures.

## Approach
1. Write the one-sentence description of what the solution must do
2. Remove every word from that sentence that doesn't describe the core behavior
3. Design a solution that does only that
4. For every piece of complexity in the design, ask: "what would break if I removed this?"

## When Most Effective
Over-engineered systems, premature abstractions, anywhere a simple solution is being overlooked in favor of a "proper" one.
```

### futurist.md
```
---
id: futurist
name: Futurist
domains: [architecture, infrastructure, platform]
description: How would this be solved in 5 years? What's the trend line?
---

## Thinking Frame
Technology moves in directions. Cloud-native, event-driven, declarative, ML-augmented — these trends favor certain solutions. Design for where things are going, not where they are. The solution that's slightly ahead of the curve is often the one with the longest useful life.

## Approach
1. Identify the direction in which this domain is moving
2. Describe the solution that will be obvious in 3-5 years
3. Find the minimum viable version of that solution that's buildable today
4. Identify what you'd have to give up to adopt it now vs. later

## When Most Effective
Platform decisions, infrastructure choices, anything with a 2+ year lifespan.
```

### integrator.md
```
---
id: integrator
name: Integrator
domains: [all]
description: What existing pieces can be combined in new ways?
---

## Thinking Frame
The best solutions often aren't new inventions — they're new combinations of existing, proven pieces. What tools, patterns, libraries, or approaches already exist that could be combined to solve this? What's the minimal new code that acts as glue?

## Approach
1. List 5-8 existing tools, patterns, or approaches relevant to this domain
2. For each pair, ask: could these solve the problem together in a way neither can alone?
3. Find the combination with the highest ratio of solved-problem to new-code
4. Design the minimal integration layer

## When Most Effective
Integration problems, anywhere "not invented here" syndrome is blocking a good solution, when time-to-value is more important than originality.
```

### provocateur.md
```
---
id: provocateur
name: Provocateur
domains: [all]
description: Propose something deliberately counterintuitive — expand the possibility space
---

## Thinking Frame
The obvious solution is obvious. The interesting solutions are the ones that make people say "you can't do it that way." Your job is to find those. Not to be wrong on purpose, but to explore the corners of the solution space that conventional thinking skips.

## Approach
1. Identify the conventional wisdom about how this type of problem is solved
2. Propose the opposite (seriously — what if you did the exact opposite?)
3. Propose the solution that would get you laughed out of a code review — then find its kernel of insight
4. Ask: what would this look like if it were 10x more ambitious?

## When Most Effective
Early-stage exploration, breaking out of local optima, whenever the team needs a jolt of creative energy.
```

---

## SYNC PROTOCOL

For `/creative sync` — run this in the creative-loop repository.

The repo is the shared source of truth. This command pulls improvements from this machine into the repo, resolves any inconsistencies, then the repo is ready to push. Git handles cross-machine transfer.

**The pattern across machines:**
```
Machine A:  /creative sync  →  git commit  →  git push
Machine B:  git pull        →  /creative sync  →  git commit  →  git push
```

---

### WHAT GETS SYNCED

Three artifacts carry improvements made by `/creative optimize`:

| File | Where improvements land | Repo location |
|---|---|---|
| `SKILL.md` | `~/.claude/skills/creative/SKILL.md` | `./SKILL.md` |
| `personas/builtin/*.md` | each project's `.creative-loop/personas/builtin/` | `.creative-loop/personas/builtin/` |
| `prompt_evolution.json` | each project's `.creative-loop/patterns/` | `.creative-loop/patterns/prompt_evolution.json` |

`install.sh` mirrors the repo into `~/.claude/skills/creative/` (SKILL.md, personas, and default config). Running `bash install.sh` after a `git pull` resets the entire local environment to the repo's committed state. Running it from another project resets that project's builtin personas too.

---

### STEP 1: CHECK SKILL.MD

Read `~/.claude/skills/creative/SKILL.md` and `./SKILL.md`. Run `diff` between them.

- **Identical** → report `[in sync]`, move on.
- **Different** → show a summary (which sections differ, line counts), then the full diff. Ask:

  > "SKILL.md differs between installed and repo. Which should win?
  > `installed` — pull the installed version into the repo
  > `repo` — keep the repo version (discard local install changes)
  > `show` — show full diff again before deciding"

  Apply the choice. If `installed` was chosen, write `~/.claude/skills/creative/SKILL.md` → `./SKILL.md`.

---

### STEP 2: CHECK OTHER PROJECTS

Ask:

> "Are there other projects on this machine where `/creative optimize` ran? Enter paths (project root or `.creative-loop/` dir), one per line. Press Enter with a blank line when done, or just Enter to skip."

Accept a list of paths. For each path, normalize it: if the path doesn't end in `.creative-loop`, append `/.creative-loop`. Check that the directory exists; skip with a warning if not.

For each valid path, run steps 2A and 2B.

---

### STEP 2A: PERSONAS (per project path)

For each `{path}/personas/builtin/{name}.md`, compare against `./`.creative-loop/personas/builtin/{name}.md`:

- **Identical** → `[in sync]` {name}
- **Different** → show the diff. Ask:

  > "{name}.md differs. Keep `repo` version or `other` version?"

  Do this **per file** so the user can cherry-pick the best version of each persona independently. Apply immediately after each choice — don't batch.

---

### STEP 2B: PROMPT EVOLUTION LOG (per project path)

Read both `{path}/patterns/prompt_evolution.json` and `./`.creative-loop/patterns/prompt_evolution.json`. This is an append-only log — merge strategy is always additive.

Find entries in the source that aren't in the repo (match on full entry content). If any new entries exist:
- List them (show `reason` and `phase` fields for each)
- Ask: "Merge these N new entries into the repo's log?"
- If yes: append them. Use python3:
  ```bash
  python3 -c "
  import json
  canon = json.load(open('./creative-loop/patterns/prompt_evolution.json'))
  src   = json.load(open('{path}/patterns/prompt_evolution.json'))
  seen  = {json.dumps(e, sort_keys=True) for e in canon}
  new   = [e for e in src if json.dumps(e, sort_keys=True) not in seen]
  json.dump(canon + new, open('./creative-loop/patterns/prompt_evolution.json','w'), indent=2)
  print(len(new), 'entries merged')
  "
  ```

If no new entries: `[in sync]`.

---

### STEP 3: PRESENT SUMMARY AND FINISH

After all files are resolved, show:

```
Sync complete:
  SKILL.md                  [pulled from installed | kept repo | in sync]
  personas/builtin/         [N files updated | all in sync]
  prompt_evolution.json     [N entries merged | in sync]
```

Then run `bash install.sh` automatically — the repo's SKILL.md may now be different from the installed version, and they should stay in sync.

Finally tell the user:

```
Installed skill updated from repo.

To share these improvements:
  git diff                          — review what changed
  git add SKILL.md \
    .creative-loop/personas/builtin \
    .creative-loop/patterns/prompt_evolution.json
  git commit -m "skill: sync improvements"
  git push

Others get the improvements by pulling and running: bash install.sh
```

---

## NOTES FOR SELF-MODIFICATION

This file (`~/.claude/skills/creative/SKILL.md`) can be improved by the `/creative optimize` command. When proposing changes to this file:

- **Persona prompts** (PERSONA LIBRARY section): Can be strengthened when they show weak differentiation
- **Phase instructions**: Can be refined when a phase consistently produces low-quality output
- **Prompt templates**: The generator and evaluator prompt templates embedded in phases can be iterated
- **Selection logic**: Thresholds and modes can be tuned based on session data

Always log changes to `.creative-loop/patterns/prompt_evolution.json` in the affected project, or `~/.claude/creative-loop-global-evolution.json` for global skill changes.

The goal is a skill file that gets measurably better over time. Treat it like production code — change it intentionally, log the reason, and assess the outcome.

---
> Source: [mari0car/creative-loop-1](https://github.com/mari0car/creative-loop-1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
