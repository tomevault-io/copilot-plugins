## autoresearch-bio

> Create, revise, audit, or launch bounded autonomous ML/software research loops with protected baselines, tiered evals, lineage, metric investigations, leakage pre-flight checks, and biology no-regression checks. Agent-agnostic — produces prompts usable by any coding agent (Claude Code, ChatGPT / Codex, Cursor, Aider, or custom agent harnesses).


# Autoresearch-Bio Skill

This is a **bio-first, domain-general, agent-agnostic** skill for creating and supervising bounded, hypothesis-driven autonomous research loops. It is strongest for biological ML, but the core discipline also applies to general ML, software engineering, agent development, benchmarking, infrastructure, and developer-tooling experiments.

The skill's primary artifact is a paste-ready `autoresearch.md` **prompt for a coding agent** — any coding agent. Tested invocation paths include Claude Code, ChatGPT (Codex / GPT agents), Cursor, Aider, and bespoke agent harnesses built on the Anthropic, OpenAI, or open-source model APIs. The skill itself is written in plain Markdown and contains no vendor-specific tool calls; you can also use it directly as a human-authored prompt template without an agent at all.

The skill can also produce amendments, decision memos, metric investigation prompts, or reusable skill updates.

**Roles.** Producing an `autoresearch.md` and running it are two jobs. A strong reasoning chat model is usually the better **planner** (reads context, designs the families, tiers, metrics, baselines, stop conditions) and a coding agent is the better **executor**. When this skill is invoked in a chat model to draft a prompt, it is the planner; the file it emits is run by the executor. The user can supply the whole design, ask the planner to design it from a bare problem, or do a hybrid. `references/planner_workflow.md` covers the split and how to prompt the planner; the family-set switches live in `core_protocol.md §5`. The discipline invariants do not change with the role.

The invariant is the same in every domain: protect the model or system of record, register baselines before search, use tiered gates, track lineage of experiments, document every result, avoid metric loopholes, and stop cleanly when stop conditions fire.

---

## Golden Path

When invoked:

1. Identify the requested artifact type:
   - new `autoresearch.md`;
   - session amendment;
   - continue/amend/audit/close decision memo;
   - metric investigation prompt;
   - reusable skill update.
2. Identify the domain:
   - biology or scientific ML;
   - general ML;
   - software/developer tooling;
   - infrastructure/performance;
   - agent, benchmark, or evaluation loop;
   - other bounded experiment loop.
3. Identify or request only the missing essentials:
   - model or system of record;
   - datasets, benchmarks, CI suites, simulator scenarios, traffic slices, or split roles;
   - primary, secondary, protected, and catastrophic-fail metrics;
   - observed failure modes;
   - compute or engineering budget;
   - locked files, identity constraints, and evaluator constraints;
   - autonomy mode.
4. If enough information exists, produce a self-contained artifact. Do not give vague advice when a paste-ready prompt is possible.
5. If essential information is missing, ask at most five prioritized questions. If the user asks for a best-effort draft, proceed with explicit assumptions and mark unknown thresholds as `TO_FILL_BEFORE_LAUNCH`.
6. Always enforce the core invariants:
   - protected model or system of record;
   - Step 0 baselines before architecture/search/development loops;
   - tiered gates;
   - exact keep/discard labels;
   - no Tier 1 or Tier 2 rebasing;
   - lineage tracking via parent_experiment_ids and branch_type;
   - documented stop conditions;
   - domain-appropriate safety boundaries.

---

## When To Use This Skill

Use this skill when the user asks for help with a repository, experiment log, trained model, software system, agent, benchmark, or research loop and wants to:

- draft an `autoresearch.md` prompt;
- set up an autonomous architecture, mechanism, benchmark, or implementation search;
- extend a trained biological ML model while protecting an existing baseline;
- improve a non-bio model or software system while protecting correctness, safety, latency, memory, slice, or robustness behavior;
- convert experimental lessons into a reusable research protocol;
- decide whether to continue, amend, audit, or close an autonomous loop;
- investigate metrics before reopening architecture or implementation search;
- add Debate Council autonomy to a long-running experiment loop;
- add lineage tracking to a flat experiment log that is hiding combinatorial search structure.

Typical trigger phrases include:

- "Help me draft an autoresearch prompt for this repo."
- "I want Codex to run experiments on this biology model."
- "Use this skill for my non-bio dev project."
- "Design a protected-baseline research loop for this benchmark."
- "What should the agent do next after these failed experiments?"
- "Should I reopen the architecture search?"
- "Design a metric investigation for my perturbation model."
- "Turn these single-cell experiment failures into a reusable protocol."
- "Make an autoresearch loop for improving this code-generation agent without regressing safety or latency."

For biological projects, emphasize biological no-regression checks, directionality, population structure, pathway/program consistency, data provenance, and safety boundaries.

For non-biological projects, use the same control system but replace biological no-regression checks with protected domain metrics such as correctness, latency, memory, robustness, safety, privacy, security, fairness, slice behavior, regression tests, held-out benchmarks, or policy compliance. Read `references/domain_adaptation.md` when adapting outside biology.

### Launch Precondition: Leakage Pre-Flight

For any search loop with a locked test or held-out split (i.e. essentially every ML / benchmark loop), the agent must produce `leakage_preflight.md` and `split_manifest.json` before launching. The skill refuses to begin Tier 1 runs without them. Full requirements live in `references/core_protocol.md §3.5`. This precondition is non-negotiable because the most common silent failure mode of long autoresearch loops is iterated test-set selection, not bad architecture choices.

### Agent Compatibility

The skill's output artifacts (`autoresearch.md`, amendments, decision memos) are plain Markdown prompts. They are written to drive any coding agent that can read a file, edit code, run a command, and append to a log. Tested or expected to work with: Claude Code, ChatGPT (Codex / GPT agent harnesses), Cursor, Aider, OpenDevin / SWE-agent, custom Anthropic-SDK or OpenAI-SDK agent loops, and the skill can also be used directly as a human-authored prompt template without an agent at all. Do not embed vendor-specific tool calls in skill outputs; keep them portable.

---

## Do Not Use This Skill

Do not invoke this skill for:

- ordinary code review, debugging, or bug fixing unless the goal is to design an autonomous experiment loop;
- one-off model-training advice with no autonomous iteration;
- generic ML brainstorming not tied to a repository, model/system of record, metric, or experiment plan;
- paper summaries or literature review without a concrete experiment loop;
- production deployment review;
- clinical, diagnostic, treatment, or patient-specific recommendations;
- wet-lab protocols, operational biological manipulation steps, or biological optimization instructions;
- hyperparameter tuning only, unless it is part of a protected-baseline research loop;
- unrestricted "make it better" prompts with no metrics, stop conditions, or protected behavior.

---

## Output Shapes

### A. New `autoresearch.md`

Must include:

- a **Skill modules** section near the top: the skill location (a mounted path or `clone github.com/yashraj59/autoresearch-bio`) and the exact list of modules the executor must read. This makes the file self-bootstrapping — an agent handed the `autoresearch.md` alone can fetch the skill and know which modules to read. `validate_autoresearch_prompt()` flags a plan that references the skill (`§` or `core_protocol`) but carries no skill-location pointer;
- model or system of record;
- datasets, benchmarks, validation suites, or split roles;
- Step 0 baseline plan;
- primary, secondary, protected, and catastrophic-fail metrics;
- architectural, mechanism, or implementation families, each with an `origin` tag, and the `family_set` mode (`fixed` or `open`);
- tiered gates;
- domain diagnostics;
- lineage rules (parent_experiment_ids, branch_type, subtree pruning);
- documentation files;
- artifact retention rules;
- stop conditions.

After producing the `autoresearch.md` file content, emit a separate launch message in chat. A good launch message points the executor at `assets/executor_instructions.md` (saved into the run repo as `AGENTS.md` / `CLAUDE.md`) so the per-run message stays short and the standing executor discipline is carried by that file. The launch message tells the user what to paste into the coding agent. **Do not include the launch message inside the `autoresearch.md` file.** Keep it as a separate code block in the chat response.

Use `assets/autoresearch_template.md` as the base. For biology, read `references/core_protocol.md`, `references/biology_addendum.md`, `references/statistical_promotion.md`, and `references/lineage.md`. For non-bio work, read `references/core_protocol.md`, `references/domain_adaptation.md`, `references/statistical_promotion.md`, and `references/lineage.md`.

**Planner/executor split (recommended).** Authoring the `autoresearch.md` is itself best done by a strong reasoning chat model acting as the **planner**, which designs the families, tiers, metrics, baselines, lineage rules, and stop conditions, then emits the file for a separate coding agent to **execute**. The planner must produce a complete design rather than defer it to the executor (a plan that punts the families to runtime fails `validate_autoresearch_prompt()`); but the *design itself is the user's* — the user can supply the families/tiers/metrics, ask the planner to propose them from a bare problem, or do a hybrid, and each family is tagged `origin: user_fixed | planner_proposed` (`core_protocol.md §5`). See `references/planner_workflow.md` for the role split, the three input modes, and a planner prompt skeleton. The invariants are identical regardless of who writes the plan.

### B. Session Amendment

Must include:

- active model or system of record reaffirmed;
- reason for amendment;
- evidence summary;
- retired/cooled/reopened families and pruned subtrees;
- immediate next action;
- exact outcome decision tree;
- updated gates or stop trigger;
- paste-ready amendment block;
- amendment review checklist results.

Use `assets/session_amendment_template.md`. Apply `references/amendment_review_checklist.md` before finalizing.

### C. Decision Memo

Must include:

- recommendation: `continue`, `amend`, `metric-audit`, `close`, or `escalate`;
- evidence;
- risks;
- required next artifact;
- what must not be done next.

Use `assets/decision_memo_template.md`.

### D. Metric Investigation Prompt

Must include:

- inventory task;
- required output directory;
- model/system-of-record baseline comparison;
- null and technical-duplicate baselines;
- variance decomposition;
- metric directionality;
- reopening criterion.

Use `assets/metric_investigation_prompt_template.md` and read `references/metric_investigation.md`.

### E. Reusable Skill Update

Must include:

- updated frontmatter;
- invocation boundaries;
- golden path;
- output contracts;
- references/assets/evals when useful;
- final checklist.

---

## Required Information And Missing-Info Policy

If the user has not supplied enough information, ask at most five questions. Prioritize in this order:

1. model or system of record: checkpoint, commit, tag, config, environment, and why it is active;
2. datasets, benchmarks, CI suites, simulator scenarios, traffic slices, or splits and their roles;
3. primary, secondary, protected, and catastrophic-fail metrics;
4. known failure modes and protected domain behavior;
5. compute budget, seed count, wall-time, GPU/CPU/runtime constraints, and locked files.

When the user asks for a best-effort draft:

- continue instead of blocking;
- state assumptions clearly;
- mark unknown thresholds as `TO_FILL_BEFORE_LAUNCH`;
- never invent baseline numbers;
- never silently choose a convenient baseline when artifacts disagree.

---

## Core Invariants

Every generated autoresearch prompt must enforce these rules:

1. **Protected model or system of record.** Name the active checkpoint/commit/config/environment. Tier 1 and Tier 2 results never rebase it. Only a Tier 3 pass can become the new model or system of record.
2. **Step 0 baseline registry.** Before any architecture, mechanism, or implementation search, run or verify baselines on every dataset, benchmark, validation suite, or split used for gates and record provenance in `BASELINE_REGISTRY.md`.
3. **Pre-registered families.** Define one to five families with motivation, hypothesis, suggested experiments, constraints, and stop/pivot rules.
4. **Tiered gates.** Tier 1 filters cheaply, Tier 2 validates across seeds/slices/scenarios, Tier 3 decides promotion/generalization/no-regression.
5. **Protected domain behavior.** A candidate can improve a headline metric and still fail. For biology, protect direction-of-effect, marker/program coherence, population structure, and biological validators. For non-bio work, protect correctness, safety, latency, memory, robustness, fairness, policy, benchmark integrity, or other domain-critical gates.
6. **Lineage tracking.** Every experiment records its parent experiments and branch type (`root`, `linear`, `fork`, `combine`, `replay`). Every node has a subtree status that controls whether it can be expanded, pruned, promoted, or retired.
7. **Metric discipline.** Record metric directionality, variance, minimum meaningful effect, and whether improvements exceed noise.
8. **Decision labels.** Use exact labels, not vague descriptions like "promising."
9. **Stop means stop.** When a stop condition fires, write the closure artifacts and halt unless autonomous Debate Council mode is explicitly enabled.
10. **No hidden scope drift.** Locked files, APIs, data splits, labels, evaluators, safety checks, and model/system identity constraints cannot be changed without an explicit amendment.
11. **Safety boundary.** This skill supports computational model/system research planning only. It does not produce wet-lab protocols, clinical recommendations, unsafe deployment instructions, or hidden changes to protected systems.

---

## Reference Loading Map

Read only what is needed for the artifact:

| Need | Read |
| --- | --- |
| New or revised biology autoresearch prompt | `references/core_protocol.md`, `references/biology_addendum.md`, `references/statistical_promotion.md`, `references/lineage.md` |
| New or revised non-bio autoresearch prompt | `references/core_protocol.md`, `references/domain_adaptation.md`, `references/statistical_promotion.md`, `references/lineage.md` |
| Metric audit / reopening decision | `references/metric_investigation.md`, `references/statistical_promotion.md` |
| Session amendment (supervised or autonomous) | `references/amendment_review_checklist.md` |
| Fully autonomous mode | `references/debate_council.md`, `references/amendment_review_checklist.md` |
| Decision vocabulary | `references/decision_labels.md` |
| Documentation, retention, file-system discipline | `references/artifact_retention.md` |
| DAG / lineage rules | `references/lineage.md` |
| Screen-vs-promotion-metric calibration | `references/metric_calibration_audit.md` |
| Failure-mode digest (read before a long autonomous run) | `references/skill_anti_patterns.md` |
| Using a chat model to author the prompt for an agent to run | `references/planner_workflow.md` |
| Paste-ready templates | `assets/*.md` |
| Skill quality checks | `evals/process_checklist.md`, `evals/trigger_prompts.csv` |

---

## Low-Compute Mode

Use low-compute mode when the budget is fewer than 10 experiments or fewer than 3 seeds per candidate.

Rules:

- still require Step 0 baselines;
- still require lineage tracking;
- reduce architectural/mechanism families to one or two;
- use stronger Tier 1 fail-fast gates;
- do not promote any candidate without at least one held-out, multi-seed, multi-slice, or regression-suite validation;
- label conclusions as provisional if seed/slice/scenario count is below the pre-registered standard;
- prefer diagnostic clarity over broad search.

---

## Biology Safety And Compliance Boundary

This skill supports computational model evaluation and research planning only.

For biological projects, do not:

- provide wet-lab protocols, operational experimental steps, or optimization instructions for biological manipulation;
- make clinical, diagnostic, treatment, or patient-specific recommendations;
- claim biological validity from Tier 1 or Tier 2 results;
- process protected health information unless the user confirms authorization and de-identification;
- recommend deployment-facing biological claims without human domain review.

For sensitive biological domains, require data provenance, license/usage restrictions, governance status when relevant, and expert review before external claims.

For non-bio projects, replace this with the project's safety, security, privacy, compliance, and deployment boundaries. Do not let the autonomous loop weaken those boundaries to improve a metric.

---

## Final Response Checklist

Before responding, verify:

- [ ] Did I identify the requested artifact type?
- [ ] Did I identify whether this is biology, scientific ML, general ML, software/dev tooling, or another domain?
- [ ] Did I name or request the model/system of record?
- [ ] Did I separate primary, secondary, protected, and catastrophic-fail metrics?
- [ ] Did I include Step 0 baselines and a baseline registry?
- [ ] Did I include lineage rules (parent_experiment_ids, branch_type, subtree_status)?
- [ ] Did I prevent Tier 1/Tier 2 promotion?
- [ ] Did I include biological no-regression checks where relevant?
- [ ] Did I include non-bio protected domain checks where relevant?
- [ ] Did I include statistical promotion criteria?
- [ ] Did I include stop conditions?
- [ ] Did I include documentation artifacts?
- [ ] Did I avoid vague labels like "promising"?
- [ ] Did I produce paste-ready text when requested?
- [ ] Did I emit the launch message as a separate chat block, not inside the autoresearch.md file?
- [ ] Did I state assumptions instead of inventing unknown values?
- [ ] Did I respect safety, security, privacy, compliance, and deployment boundaries?

---
> Source: [yashraj59/autoresearch-bio](https://github.com/yashraj59/autoresearch-bio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
