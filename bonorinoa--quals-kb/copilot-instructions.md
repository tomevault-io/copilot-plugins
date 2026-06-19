## quals-kb

> Build rigorous, self-contained, exam-facing PhD qualifying-exam lecture-note packets, playbooks, exercise banks, and problem-solving labs from professor notes, problem sets, past exams, and supplementary theory sources.


# PhD Qual Lecture Notes Builder

## Purpose

Use this skill to create, revise, audit, or plan qualifying-exam study materials that teach a student how to solve exam problems under time pressure. The output should not merely summarize lecture notes. It should convert source material into a problem-solving system: definitions, canonical models, derivation templates, closed-form methods, common traps, targeted exercises, and past-exam mappings.

The skill is field-general. It can be used for macroeconomics, microeconomics, econometrics, math economics, or any PhD qualifier area. Domain-specific details should be supplied in a separate manifest, such as `MACRO_MANIFEST.md` or `MICRO_MANIFEST.md`.

## Use Cases

Use this skill when the user asks for any of the following:

- A full lecture-note packet for one qualifier module.
- A module-specific exercise bank.
- A worked-problem lab.
- A methodology or playbook document.
- A revision or quality audit of an existing module packet.
- A source inventory and module plan.
- A TeX/PDF study packet.
- A harmonization pass across multiple module packets.

Do not use this skill for quick one-off answers unless the user explicitly wants the answer to follow the project methodology.

## Required Inputs

Before drafting, identify or request the following inputs. If some inputs are missing but the task is still feasible, proceed with explicit assumptions rather than blocking.

1. Field/course, for example: `ASU Macro Theory Qual`.
2. Module name, for example: `OLG Model`.
3. Source folder or uploaded files.
4. Past-exam folder or past-exam files.
5. Relevant problem sets or homeworks.
6. Desired output type: outline, source inventory, TeX, PDF, exercises, answer sketches, full packet, or audit report.
7. Desired depth: compact, standard, exhaustive, or problem-solving lab.
8. Target weakness, if any, for example: `closed-form derivations` or `RCE definitions`.
9. Style constraints, including notation preferences and whether the output should be professor-specific or exam-specific.

If a domain manifest is present, read it before drafting and follow its module definitions, source hierarchy, notation conventions, and output standards.

## Source Hierarchy

When sources conflict or differ in emphasis, use this hierarchy:

1. Past qualifying exams determine what is exam-relevant.
2. Current problem sets determine current-course emphasis.
3. Professor lecture notes and slides determine notation, terminology, and course framing.
4. Homeworks determine practice style and expected derivation depth.
5. Supplementary textbooks, papers, or external sources fill gaps only when needed.

Do not let professor notes crowd out the past-exam pattern. The packet must teach the student how to solve qualifier problems, not simply reproduce a course lecture.

## Mandatory Workflow

### Step 1. Source Inventory

Before drafting substantive notes, inventory the accessible sources.

For each source, record:

- file name or URL/path;
- source type: past exam, problem set, lecture note, homework, slide deck, textbook, paper, or prior student note;
- topics covered;
- mathematical tools used;
- exam relevance;
- any extraction or rendering issues.

If a source is inaccessible, say so plainly and continue with available sources.

### Step 2. Pattern Analysis

Identify patterns before writing notes.

For the assigned module, produce:

- core problem archetypes;
- recurring wording and signature instructions;
- recurring state variables;
- standard constraints;
- standard FOCs, envelope conditions, Euler equations, market-clearing conditions, or cutoff equations;
- closed-form opportunities;
- common mistakes and traps;
- past-exam mappings;
- links to current problem sets or homeworks.

### Step 3. Module Blueprint

Create a blueprint before writing the full packet.

The blueprint must include:

- module objective;
- prerequisite concepts;
- canonical models;
- exam variations;
- closed-form catalog;
- exercise plan;
- estimated study use, for example: `one 4-hour block` or `two 3-hour blocks`.

### Step 4. Draft the Packet

Use the standard packet structure below. Adapt only when the user requests a methodology playbook or a problem-solving lab.

### Step 5. Add Exercises and Answer Sketches

Every teaching packet must include targeted practice. Exercises should be explicitly tied to the module's exam archetypes.

### Step 6. Quality Audit

Before finalizing, run the quality audit below and either fix issues or disclose remaining limitations.

### Step 7. Output Files

If the user asks for files, produce clean, reusable files. If producing LaTeX, provide an Overleaf-ready `.tex` source. If producing PDF, compile it and spot-check readability. If compilation or rendering is unreliable, provide the `.tex` source and explain the limitation.

## Standard Packet Structure

Every normal lecture-note packet should use this structure unless the user requests otherwise.

### 1. Title Page and Orientation

Include:

- module title;
- course/qualifier;
- purpose of the packet;
- how to use it;
- expected study block length;
- source base.

### 2. Module Map

Create a table with columns:

- subtopic;
- why it matters;
- exam appearances or practice-source appearances;
- core technique;
- common trap.

### 3. Learning Objectives

Use operational learning objectives. Bad: `Understand OLG.` Good: `Given any two-period OLG problem with log utility, derive the individual savings rule, aggregate it, normalize by next-period labor, and characterize the steady-state capital-labor ratio.`

### 4. Source Map

List the sources used and how each source shaped the packet. Distinguish between:

- exam anchor sources;
- professor lecture sources;
- homework/problem-set sources;
- supplementary theory sources.

### 5. Canonical Models

For each canonical model, include:

1. Environment.
2. Timing.
3. State variables.
4. Choice variables.
5. Constraints.
6. Recursive or sequential problem.
7. FOCs/KKT conditions.
8. Envelope condition when dynamic.
9. Euler equation, pricing equation, law of motion, cutoff rule, or implementability condition.
10. Market clearing or feasibility.
11. Closed form when available.
12. Intuition.
13. Qualifier warning/trap.

### 6. Core Derivation Templates

Include derivation templates that the student can reuse during an exam.

Examples:

- `Bellman -> FOC -> Envelope -> Euler`.
- `Lifetime budget -> log allocation -> savings rule -> aggregation`.
- `Accept value -> reject value -> reservation wage`.
- `Individual state -> aggregate state -> value functions -> policy rules -> law of motion for distribution -> market clearing`.

### 7. Closed-Form Catalog

List high-yield closed forms. Every closed form must be derived or accompanied by a short guess-and-verify sketch. Do not merely state formulas.

### 8. Exam Variations

Explain how the canonical models are mutated on past exams or problem sets. Focus on what changes and what stays invariant.

### 9. Common Traps

Create a short, blunt section listing the errors that lose points.

### 10. Exercise Bank

Include four exercise categories:

1. Mechanical drills.
2. Closed-form drills.
3. Past-qual applications.
4. Trap/economist-claim questions.

Each exercise should identify the target skill.

### 11. Answer Sketches

For each exercise, provide a concise answer sketch with:

1. setup;
2. key equation;
3. final result;
4. intuition;
5. common mistake to avoid.

Full solutions are optional unless the user asks for them.

### 12. Past-Qual Practice Map

Map the module to past problems by year and question number. If exact mappings are uncertain, label them as approximate.

### 13. Final Checklist

End with a checklist the student can use before attempting old quals.

## Special Packet Types

### Methodology Playbook

Use this when the goal is not to teach a topic from scratch, but to provide a compact reference while solving old quals.

Structure:

1. Taxonomy of problem archetypes.
2. Decision tree: what to do when a problem is recognized.
3. Templates for common tasks.
4. Formula catalog.
5. Common traps.
6. Past-problem map.
7. Time-management protocol.

Keep it compact. Avoid lengthy lecture exposition.

### Problem-Solving Lab

Use this when the target weakness is execution, especially closed-form derivations.

Structure:

1. Pattern recognition guide.
2. Worked example.
3. Derivation checklist.
4. Variants.
5. Practice drills.
6. Answer sketches.

A lab should contain more worked examples and fewer long theoretical digressions.

### Exercise Bank Only

Structure:

1. Skill map.
2. Exercise sets by difficulty.
3. Past-exam mapping.
4. Answer sketches.
5. Full solutions only if requested.

## Derivation Requirements by Topic Type

### Dynamic Programming

Always include:

- state variable;
- feasible correspondence;
- Bellman equation;
- existence/uniqueness conditions when relevant;
- FOC/KKT condition;
- envelope condition;
- Euler equation when differentiable;
- policy monotonicity or concavity when relevant;
- guess-and-verify for closed forms.

### Static Optimization

Always include:

- objective;
- constraints;
- feasible set;
- Lagrangian when constrained;
- FOCs;
- complementary slackness for inequalities;
- second-order or concavity/sufficiency argument;
- multiplier interpretation.

### OLG

Always include:

- lifetime budget;
- individual savings rule;
- old-age income or pension term if present;
- aggregation of savings;
- population/effective-labor normalization;
- capital-labor law of motion;
- steady state;
- dynamic-efficiency condition when relevant;
- interpretation of total versus per-worker variables.

### Asset Pricing

Always include:

- household recursive problem;
- budget constraint with correct timing;
- FOC for asset holdings;
- envelope condition;
- stochastic Euler equation;
- equilibrium market clearing;
- stochastic discount factor when useful;
- closed-form price if log/CRRA structure permits;
- expected return or risk-free rate if asked.

### Search

Always include:

- timing;
- state variables;
- accept value;
- reject/continue value;
- Bellman equation;
- reservation-wage proof;
- comparative statics via option value or implicit differentiation;
- note whether employment starts today or next period.

### Recursive Competitive Equilibrium

Always include:

- individual state;
- aggregate state;
- value functions;
- policy functions;
- price functions;
- firm problems;
- government rules if present;
- law of motion for the distribution;
- aggregate consistency;
- market clearing;
- stationary condition if stationary RCE is requested.

The law of motion for the distribution must be explicit. Use a measurable-set form such as:

```tex
\mu'(B)=\int_X \Pr\{G_x(x,\varepsilon',S)\in B\mid x,S\}\,d\mu(x).
```

### Planner, Welfare, and Efficiency

Always include:

- feasible set;
- planner objective;
- Pareto weights if heterogeneous agents;
- FOCs;
- comparison to competitive equilibrium;
- wedges;
- externality or missing-market logic;
- statement of efficiency, constrained efficiency, or inefficiency.

Do not claim inefficiency merely because there is a tax, nontradability, or price wedge. Identify the real allocation wedge relative to the relevant feasible set.

## Style Rules

1. Write in a rigorous but conversational teaching style.
2. Every paragraph should help the student solve exam problems.
3. Avoid filler, broad motivational text, and untested textbook digressions.
4. Put intuition immediately after major derivations.
5. Use consistent notation inside each packet.
6. Prefer exam notation over textbook notation when they differ.
7. Define timing explicitly when timing affects the Bellman equation.
8. When deriving closed forms, show the guess, substitute, match coefficients, and verify feasibility/interiority.
9. Mark traps clearly with labels such as `Qualifier trap` or `Common mistake`.
10. Include source-based caveats when a topic is speculative or lower probability.

## LaTeX and Formatting Rules

When producing `.tex`:

1. Use real LaTeX, not Markdown math pasted into TeX.
2. Use `align*`, `equation*`, and `cases` environments for derivations.
3. Do not put long paragraphs inside display math.
4. Avoid giant equation blocks; break long derivations into short steps.
5. Use inline math for short expressions.
6. Use display math only when the equation is important.
7. Keep derivation lines reasonably short.
8. Use `booktabs` for tables.
9. Use `enumitem` for compact lists.
10. Use `tcolorbox` or simple framed boxes sparingly for formulas/traps.
11. Include a table of contents for long packets.
12. If PDF output is requested, compile and spot-check sample pages.
13. If equations render badly or compilation is uncertain, provide Overleaf-ready `.tex` source.

Recommended TeX package set:

```tex
\usepackage[margin=1in]{geometry}
\usepackage{amsmath,amssymb,amsthm,mathtools}
\usepackage{booktabs,array,longtable}
\usepackage{enumitem}
\usepackage{xcolor}
\usepackage[most]{tcolorbox}
\usepackage{hyperref}
```

## Exercise Requirements

Every full module packet must include exercises. Use this distribution unless the user says otherwise:

- 30 percent mechanical derivation drills.
- 30 percent closed-form or proof drills.
- 25 percent past-qual-style applications.
- 15 percent trap/economist-claim questions.

Exercises should be usable in targeted practice sessions. Label each exercise by skill, for example:

- `Skill: derive Euler equation`.
- `Skill: aggregate savings into capital law`.
- `Skill: write distribution law`.
- `Skill: prove reservation wage cutoff`.

## Source Use and Citations

When the platform supports citations, cite source files or web sources for source-specific claims. If citations are not available, include a source map with file names and page/section references.

Do not fabricate coverage. If a source was not read, do not imply it was used.

If using web sources for current facts or external theory, prefer primary sources, official documentation, textbooks, or peer-reviewed references as appropriate.

## Parallel Session Protocol

When assigned one module in a parallel workflow:

1. Work only on the assigned module.
2. Do not rewrite the global playbook unless asked.
3. Use the shared module structure and notation conventions.
4. Produce a source inventory first.
5. Produce a module blueprint before drafting the full packet.
6. Flag unresolved cross-module notation conflicts for the harmonization pass.
7. Keep professor-specific material inside the relevant module.
8. Do not duplicate long derivations from other modules unless needed for self-containment.

## Final Harmonization Pass

After multiple module packets are created, run a harmonization pass to check:

- notation consistency;
- duplicated content;
- missing cross-references;
- module numbering;
- shared glossary;
- exercise labels;
- source map consistency;
- TeX preambles;
- formatting consistency;
- PDF rendering quality if PDFs are produced.

## Quality Audit

Before finalizing any packet, answer these questions:

1. Did I inventory all accessible sources?
2. Did I identify how the module appears on past exams or current problem sets?
3. Is the packet self-contained enough for a focused study block?
4. Are all canonical models stated with environment, timing, states, choices, and constraints?
5. For dynamic models, did I include Bellman, FOC, envelope, and Euler where relevant?
6. For equilibrium models, did I include market clearing and government budgets where relevant?
7. For heterogeneous-agent models, did I include an explicit distribution law?
8. Are closed forms derived through guess-and-verify rather than merely stated?
9. Are common traps and grader rewards explicit?
10. Does the exercise bank target the actual exam skills?
11. Is the math readable and not oversized?
12. If files were produced, are the output names clear and reusable?

## Invocation Template

Users can start a module session with:

```text
Use the PhD Qual Lecture Notes Builder Skill and the ASU Macro Manifest.
Create Module [number]: [module name].
Source folder: [path or uploaded files].
Past exams: [path or uploaded files].
Relevant problem sets: [files].
Output requested: [source inventory / blueprint / full TeX / PDF / exercises].
Depth: [standard / exhaustive / problem-solving lab].
Target weakness: [optional].
Do not modify other project documents.
```

## Non-Negotiable Standard

The final product must help a serious PhD student pass the qualifying exam. Prefer derivation clarity, exam relevance, and targeted practice over breadth for its own sake.

---
> Source: [Bonorinoa/Quals-KB](https://github.com/Bonorinoa/Quals-KB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
