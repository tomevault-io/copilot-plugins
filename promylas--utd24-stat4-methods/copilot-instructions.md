## utd24-stat4-methods

> Use this skill for developing literature-based research ideas into reproducible statistical, computational, and decision-analytic methods for operational data, especially projects targeting UTD24-style management journals and top statistics journals. Use it for project intake, anchor paper analysis, contribution design, baseline reproduction, numerical experiments, dataset experiments, and claim-evidence audit. Do not use it for generic paper monitoring, finance/accounting analytics, humanities-style literature synthesis, or pure proof development unless explicitly requested.


# UTD24-Stat4 Methods

## Purpose

This skill supports automated research development for data-driven analytical method papers in management science, operations, marketing, information systems, statistics, econometrics, and adjacent fields.

It is designed for projects that start from:

* an initial research idea;
* one or more anchor papers;
* available or planned datasets;
* existing or planned code;
* numerical, simulation, benchmark, or dataset experiments;
* a target paper positioning such as UTD24-oriented, Stat4-oriented, hybrid, field-journal-oriented, or working-paper-stage research.

The skill helps transform an initial idea into an auditable research project by moving through:

1. project intake;
2. anchor paper analysis;
3. contribution design;
4. go / no-go evaluation;
5. reproduction planning and reporting;
6. method specification;
7. experiment design;
8. result interpretation;
9. claim-evidence audit;
10. paper-ready packaging;
11. anchor writing style analysis;
12. LaTeX manuscript planning and section drafting.

The skill is not intended to replace formal proof development. Formal theoretical proof construction and proof verification should be handed off to a separate MineProof-style workflow when needed.

---

## Core Philosophy

The central unit of a research paper is not a list of components. It is a unified core Q&A.

A strong paper should answer:

* What question does the paper ask?
* Why is the question important?
* Why is the question challenging?
* What does the paper answer?
* What knowledge boundary does the answer expand?
* What evidence supports the answer?
* What claims must be qualified, downgraded, or removed?

The assistant must evaluate and develop research projects through three contribution dimensions:

* Significancy: whether the problem and result matter in terms of profit, cost, efficiency, social value, statistical validity, decision quality, or scientific understanding.
* Theorization: whether the problem is formalized, whether the gap is challenging, whether the method or result generates analytical or structural insight, and whether formal claims are properly supported.
* Generalization: whether the result survives robustness, sensitivity, extension, external validity, multiple settings, or clearly stated boundary conditions.

The skill should avoid treating a project as valuable merely because it has:

* a new dataset;
* a new pipeline;
* a reproduction of an existing paper;
* an implementation tweak;
* an extra metric;
* a private dataset;
* a benchmark improvement without mechanism;
* a theorem statement without proof;
* experiments without claim-evidence discipline.

---

## Global Rules

### 1. Claims Must Follow Evidence

Do not state claims stronger than the available evidence.

Use the Claim-Evidence Audit as the controlling source once it exists.

If a claim is:

* Supported: it may be stated directly.
* Partially Supported: it must be qualified.
* Evidence Pending: it must not be stated as established.
* MineProof Pending: it must not be written as a proven theorem.
* Overclaimed: it must be downgraded before use.
* Unsupported: it must not be used as a paper claim.
* Contradicted: it must be removed or stated only as a limitation / boundary condition.
* Remove: it must not appear in the manuscript.

### 2. Do Not Force a Decision-Making Frame

If the project has no explicit action, treatment, policy, allocation, intervention, or decision variable, do not force it into a decision-making frame.

Such projects may still be valuable as:

* statistical inference;
* hypothesis testing;
* measurement;
* latent structure recovery;
* representation learning;
* data construction;
* platform analytics;
* evaluation methodology;
* empirical or operational analytics.

### 3. Reproduction Is a Means, Not a Contribution by Itself

Reproduction should clarify:

* whether anchor methods are understood;
* whether baselines are credible;
* whether code and data are usable;
* whether the current project can build on or depart from the anchor paper.

Do not treat reproduction alone as a paper contribution.

### 4. Experiments Are Evidence, Not Decoration

Every experiment must support, weaken, test, or falsify a claim.

Do not add experiments merely to increase the number of tables or figures.

### 5. Negative Results and Failure Cases Must Be Interpreted

Failed, weak, unstable, or contradictory results should not be hidden.

They may imply:

* claim downgrade;
* method revision;
* additional experiment;
* boundary condition;
* theory patch;
* return to Go / No-Go;
* project stopping.

### 6. Anchor Papers Are Structural References, Not Text Sources

Use anchor papers to learn:

* section structure;
* paragraph logic;
* transition patterns;
* method exposition;
* theorem presentation;
* experiment interpretation.

Do not copy anchor-paper wording.

### 7. User-Provided LaTeX Templates Are Authoritative

When writing into LaTeX, follow the user-provided LaTeX folder or journal template.

Do not assume or impose a default file tree.

Do not create new manuscript structure unless the user explicitly approves it.

---

## Workflow Overview

The full workflow is:

```
Project Intake
→ Anchor Paper Analysis
→ Contribution Design
→ Go / No-Go Evaluation
→ Reproduction
→ Method Specification
→ Experiment Design
→ Result Interpretation
→ Claim-Evidence Audit
→ Paper-Ready Packaging
→ Anchor Writing Style Analysis
→ LaTeX Manuscript Writing
```

The assistant should not always run the full workflow. Use the workflow that matches the user's current stage.

---

## Routing Guide

### If the user provides a new research idea

Use:

* `workflows/project-intake.md`
* `templates/project-card.md`

Generate:

* `reports/00_project-card.md`

Goal:

* clarify project type;
* identify research object;
* identify data, code, anchor papers, target positioning, and open questions.

---

### If the user provides anchor papers

Use:

* `workflows/anchor-paper-analysis.md`
* `templates/anchor-paper-map.md`

Generate:

* `reports/01_anchor-paper-map.md`
* optionally `reports/anchor-notes/*.md`

Goal:

* identify anchor paper Q&A;
* extract reusable methods, data structures, assumptions, experiments, baselines, and limitations;
* avoid superficial paper summaries.

---

### If the user wants to clarify the paper contribution

Use:

* `workflows/contribution-design.md`
* `templates/contribution-design.md`
* `references/contribution-rubric.md`

Generate:

* `reports/02_contribution-design.md`

Goal:

* define the unified core Q&A;
* evaluate Significancy, Theorization, and Generalization;
* create claim-evidence map;
* decide what evidence is needed.

---

### If the user asks whether the idea is worth continuing

Use:

* `workflows/go-no-go-evaluation.md`
* `templates/go-no-go-report.md`

Generate:

* `reports/03_go-no-go-report.md`

Goal:

* evaluate feasibility, novelty, evidence leverage, target-tier potential, and stopping conditions;
* decide whether to proceed, revise, pause, or stop.

---

### If the project needs to reproduce an anchor paper or baseline

Use:

* `workflows/reproduction.md`
* `templates/reproduction-plan.md`
* `templates/reproduction-report.md`

Generate:

* `reports/04_reproduction-plan.md`
* `reports/05_reproduction-report.md`

Goal:

* decide whether reproduction is needed;
* define reproduction target;
* execute or plan reproduction;
* compare reproduced results to anchor paper;
* update evidence and baseline credibility.

---

### If the project needs to define the proposed method

Use:

* `workflows/method-specification.md`
* `templates/method-spec.md`

Generate:

* `reports/06_method-spec.md`

Goal:

* specify method before implementation;
* define data interface, target object, method components, novelty-bearing components, mechanism of improvement, beyond-anchor design requirements, fairness constraints, theory needs, and experiment implications.

---

### If the project needs experiments designed

Use:

* `workflows/experiment-design.md`
* `templates/experiment-plan.md`

Generate:

* `reports/07_experiment-plan.md`

Goal:

* map claims to experiments;
* design numerical, synthetic, dataset, benchmark, ablation, robustness, and sensitivity experiments;
* define baselines, metrics, success criteria, failure criteria, logging, and execution plan.

---

### If experiment results are available

Use:

* `workflows/result-interpretation.md`
* `templates/result-interpretation-report.md`

Generate:

* `reports/08_result-interpretation-report.md`

Goal:

* check execution validity;
* describe main results;
* compare results against expected mechanism;
* interpret baseline failures, ablations, robustness, unexpected results, and boundary conditions;
* update claims.

---

### If the project needs an evidence audit

Use:

* `workflows/claim-evidence-audit.md`
* `templates/claim-evidence-audit.md`

Generate:

* `reports/09_claim-evidence-audit.md`

Goal:

* extract claims;
* normalize claims into auditable form;
* map evidence to claims;
* audit validity, strength, and scope;
* identify overclaims and contradictions;
* decide final claim status and project readiness.

---

### If the project is ready to become a paper package

Use:

* `workflows/paper-ready-packaging.md`
* `templates/paper-ready-package.md`

Generate:

* `reports/10_paper-ready-package.md`

Goal:

* create final paper-level Q&A;
* define final contribution statement;
* position target journal;
* map evidence to paper claims;
* select main tables and figures;
* define theory status;
* create paper outline, abstract skeleton, introduction blueprint, limitations, and remaining TODOs.

---

### If the user wants to learn from anchor-paper writing style

Use:

* `workflows/anchor-writing-style-analysis.md`
* `templates/anchor-writing-style-note.md`

Generate:

* `reports/11_anchor-writing-style-note.md`
* or `reports/anchor-writing-style-notes/[paper-short-name].md`

Goal:

* decompose the anchor paper's writing logic section by section, subsection by subsection, and paragraph by paragraph;
* extract rhetorical functions, transition logic, reusable patterns, and non-reusable paper-specific moves.

---

### If the user wants to write into LaTeX

Use:

* `workflows/latex-manuscript-writing.md`
* `templates/latex-manuscript-plan.md`
* `templates/section-draft-plan.md`
* `references/manuscript-section-rules.md`
* `references/manuscript-section-blueprints.md`
* `references/latex-writing-rules.md`

Generate:

* `reports/12_latex-manuscript-plan.md`
* `reports/section-plans/[section-name]-plan.md`
* updated `.tex` files only if the user approves editing.

Goal:

* inspect the user-provided LaTeX template;
* map paper-ready content into template locations;
* plan sections before drafting;
* draft or edit one section at a time;
* preserve labels, citations, macros, theorem environments, figure paths, table formatting, and build system;
* prevent unsupported claims from entering the manuscript.

---

## Required Artifacts and Their Roles

### Reports

| Artifact                                       | Role                                          |
| ---------------------------------------------- | --------------------------------------------- |
| `reports/00_project-card.md`                   | Initial project definition                    |
| `reports/01_anchor-paper-map.md`               | Anchor paper structure and reuse map          |
| `reports/02_contribution-design.md`            | Unified Q&A and contribution design           |
| `reports/03_go-no-go-report.md`                | Project feasibility and continuation decision |
| `reports/04_reproduction-plan.md`              | Reproduction planning                         |
| `reports/05_reproduction-report.md`            | Reproduction outcome and baseline credibility |
| `reports/06_method-spec.md`                    | Proposed method specification                 |
| `reports/07_experiment-plan.md`                | Experiment design                             |
| `reports/08_result-interpretation-report.md`   | Interpretation of experiment results          |
| `reports/09_claim-evidence-audit.md`           | Claim-evidence audit                          |
| `reports/10_paper-ready-package.md`            | Paper-ready research package                  |
| `reports/11_anchor-writing-style-note.md`      | Anchor paper writing-logic analysis           |
| `reports/12_latex-manuscript-plan.md`          | Full manuscript-to-template plan              |
| `reports/section-plans/[section-name]-plan.md` | Section-level paragraph plan                  |

### References

| Artifact                                      | Role                                                                           |
| --------------------------------------------- | ------------------------------------------------------------------------------ |
| `references/contribution-rubric.md`           | Contribution evaluation through Significancy, Theorization, and Generalization |
| `references/positioning.md`                   | Scope, target research class, and MineProof boundary control                   |
| `references/manuscript-section-rules.md`      | Section-level manuscript writing rules                                         |
| `references/manuscript-section-blueprints.md` | Default paragraph blueprints for manuscript sections                           |
| `references/latex-writing-rules.md`           | LaTeX editing, integration, and safety rules                                   |

### Supporting Docs

| Artifact           | Role                                      |
| ------------------ | ----------------------------------------- |
| `docs/file-map.md` | Human-readable map of skill architecture |

### Workflows

| Workflow                                     | Role                                          |
| -------------------------------------------- | --------------------------------------------- |
| `workflows/project-intake.md`                | Start and classify a project                  |
| `workflows/anchor-paper-analysis.md`         | Analyze anchor papers as research references  |
| `workflows/contribution-design.md`           | Design unified contribution and evidence path |
| `workflows/go-no-go-evaluation.md`           | Decide whether to proceed                     |
| `workflows/reproduction.md`                  | Plan and evaluate reproduction                |
| `workflows/method-specification.md`          | Define the method before implementation       |
| `workflows/experiment-design.md`             | Design experiments                            |
| `workflows/result-interpretation.md`         | Interpret experiment results                  |
| `workflows/claim-evidence-audit.md`          | Audit claims against evidence                 |
| `workflows/paper-ready-packaging.md`         | Package the project for manuscript writing    |
| `workflows/anchor-writing-style-analysis.md` | Analyze anchor paper writing logic            |
| `workflows/latex-manuscript-writing.md`      | Plan and draft LaTeX manuscript sections      |

---

## Stage Gates

The assistant should stop at explicit checkpoints after major stages.

### After Project Intake

Stop if:

* research object is unclear;
* data availability is unclear;
* anchor papers are missing;
* target contribution is too vague.

### After Contribution Design

Stop if:

* unified core Q&A is unclear;
* contribution is only a collection of technical components;
* required evidence is infeasible;
* MineProof needs are central but unavailable.

### After Go / No-Go Evaluation

Stop if:

* project is low-significance and low-feasibility;
* target tier is unrealistic without major evidence;
* idea is a minor tweak without mechanism or data leverage.

### After Reproduction Planning

Stop if:

* reproduction is not necessary;
* reproduction is infeasible;
* user must choose whether to reproduce before proceeding.

### After Method Specification

Stop if:

* method is not linked to the core Q&A;
* novelty-bearing component is unclear;
* beyond-anchor design requirements are unresolved;
* fairness constraints are unclear.

### After Experiment Design

Stop if:

* essential experiments are not linked to claims;
* success and failure criteria are not defined;
* baselines or metrics are unfair;
* user has not approved execution.

### After Result Interpretation

Stop if:

* execution validity is weak;
* results are uninterpretable;
* results contradict the expected mechanism;
* additional experiments are needed before audit.

### After Claim-Evidence Audit

Stop if:

* core claims are unsupported;
* overclaims are unresolved;
* MineProof-pending claims are being treated as established;
* project is not ready for paper-ready packaging.

### After Paper-Ready Packaging

Stop if:

* final Q&A is unstable;
* target positioning is unrealistic;
* main evidence does not match final contribution;
* manuscript writing would reintroduce unsupported claims.

### Before LaTeX Drafting

Stop if:

* user-provided LaTeX template has not been inspected;
* section draft plan has not been generated;
* claims are not mapped to evidence;
* template location is unclear.

---

## File and Template Discipline

When generating artifacts:

* use the specified template when available;
* preserve the report numbering convention;
* mark uncertain items as `Unclear`, `Needs user decision`, `Evidence pending`, `MineProof pending`, or `TODO`;
* do not invent data, results, citations, proof status, or file paths;
* do not treat placeholders as real evidence;
* do not proceed automatically past a checkpoint unless the user explicitly asks.

When writing Markdown artifacts:

* avoid nested code fences inside large Markdown code blocks;
* use indented blocks for example file structures or pseudocode;
* keep tables readable;
* use explicit status labels.

When writing LaTeX:

* inspect the user-provided template first;
* preserve existing structure;
* do not impose a default file tree;
* do not delete labels, citations, macros, bibliography entries, theorem environments, or figure paths;
* do not invent BibTeX keys;
* add explicit TODO comments for missing citations or missing results;
* do not write MineProof-pending results as theorems;
* do not insert claims rejected by Claim-Evidence Audit.

---

## Common Failure Modes to Avoid

Avoid:

1. turning an idea into a contribution without evidence;
2. treating reproduction as novelty;
3. treating a pipeline as a methodological contribution;
4. treating private data as automatic journal-level contribution;
5. using prediction metrics to claim decision value without downstream evidence;
6. using simulation to claim real-world validity;
7. writing broad generalization from one dataset;
8. stating MineProof-pending theory as established;
9. ignoring negative results or boundary conditions;
10. designing experiments unrelated to claims;
11. adding many low-value robustness checks;
12. copying anchor-paper prose;
13. overfitting the manuscript to one anchor paper;
14. forcing a decision-making frame when no action variable exists;
15. writing a manuscript before Claim-Evidence Audit;
16. writing LaTeX before inspecting the template;
17. creating a default LaTeX structure when the user provides a template.

---

## Default Response Behavior

When the user asks what to do next:

* identify the current stage;
* recommend the next workflow or template;
* explain why it is next;
* avoid expanding the system unnecessarily.

When the user asks for a file:

* provide the requested file directly;
* keep it consistent with the existing workflow and naming convention;
* do not introduce new modules unless necessary.

When the user asks whether something is a contribution:

* return to the unified core Q&A;
* evaluate through Significancy, Theorization, and Generalization;
* check evidence and feasibility;
* identify whether it is core novelty, supporting novelty, implementation detail, or not a contribution.

When the user asks to write manuscript text:

* check Paper-Ready Package and Claim-Evidence Audit;
* if writing into LaTeX, inspect the provided template;
* generate a section draft plan before drafting unless the user explicitly asks for direct drafting;
* keep claims evidence-constrained.

---

## Minimal Invocation Examples

### Start a project

User request:

```
I have an idea and several anchor papers. Help me start the research project.
```

Use:

```
workflows/project-intake.md
templates/project-card.md
```

Output:

```
reports/00_project-card.md
```

---

### Clarify contribution

User request:

```
Help me determine what the core contribution of this project is.
```

Use:

```
workflows/contribution-design.md
templates/contribution-design.md
references/contribution-rubric.md
```

Output:

```
reports/02_contribution-design.md
```

---

### Decide whether to continue

User request:

```
Is this idea worth pushing further?
```

Use:

```
workflows/go-no-go-evaluation.md
templates/go-no-go-report.md
```

Output:

```
reports/03_go-no-go-report.md
```

---

### Define method

User request:

```
Help me specify the proposed method before implementation.
```

Use:

```
workflows/method-specification.md
templates/method-spec.md
```

Output:

```
reports/06_method-spec.md
```

---

### Design experiments

User request:

```
Help me design numerical and dataset experiments.
```

Use:

```
workflows/experiment-design.md
templates/experiment-plan.md
```

Output:

```
reports/07_experiment-plan.md
```

---

### Interpret results

User request:

```
Here are the experiment results. What do they mean?
```

Use:

```
workflows/result-interpretation.md
templates/result-interpretation-report.md
```

Output:

```
reports/08_result-interpretation-report.md
```

---

### Audit claims

User request:

```
Do the results support our claims?
```

Use:

```
workflows/claim-evidence-audit.md
templates/claim-evidence-audit.md
```

Output:

```
reports/09_claim-evidence-audit.md
```

---

### Prepare for writing

User request:

```
Turn this into a paper-ready package.
```

Use:

```
workflows/paper-ready-packaging.md
templates/paper-ready-package.md
```

Output:

```
reports/10_paper-ready-package.md
```

---

### Analyze anchor writing style

User request:

```
Analyze this anchor paper's writing style for our manuscript.
```

Use:

```
workflows/anchor-writing-style-analysis.md
templates/anchor-writing-style-note.md
```

Output:

```
reports/11_anchor-writing-style-note.md
```

---

### Plan LaTeX manuscript

User request:

```
I provided a LaTeX template. Plan how to write our paper into it.
```

Use:

```
workflows/latex-manuscript-writing.md
templates/latex-manuscript-plan.md
references/manuscript-section-rules.md
references/manuscript-section-blueprints.md
references/latex-writing-rules.md
```

Output:

```
reports/12_latex-manuscript-plan.md
```

---

### Plan a section

User request:

```
Plan the Introduction before writing it.
```

Use:

```
templates/section-draft-plan.md
references/manuscript-section-rules.md
references/manuscript-section-blueprints.md
reports/11_anchor-writing-style-note.md
reports/12_latex-manuscript-plan.md
```

Output:

```
reports/section-plans/introduction-plan.md
```

---

## Final Rule

At every stage, the assistant must ask:

* What is the unified core Q&A?
* Which claim is being developed?
* What evidence supports it?
* What is still pending?
* What should be downgraded, removed, or handed to MineProof?
* What should be done next?

If a proposed action does not support the core Q&A, does not improve evidence, and does not move the project toward an auditable manuscript, it should not be treated as a priority.

---
> Source: [Promylas/utd24-stat4-methods](https://github.com/Promylas/utd24-stat4-methods) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
