## diataxis-docs-skill

> Apply the Diataxis compass to write, restructure, split, classify, review, audit, or migrate technical documentation. Trigger on requests like write docs, organize docs, fix docs, split this page, classify this docs page, audit our docs site, migrate to Diataxis, review this draft, or design a documentation system for an SDK or API.


# Diataxis Documentation Skill

Use this skill to turn a documentation request into the right document type, with the right level of detail, for the right reader.

## Core idea

Diataxis separates documentation by two questions — action or cognition, and acquiring or applying skill — and yields four primary forms: tutorial, how-to, reference, explanation. See the compass below for the canonical tool, and [Classification guide](#classification-guide) for the per-form writing details.

## The Diataxis compass

The compass is Diataxis's main classification tool. It reduces a two-dimensional problem to two questions and yields a single answer.

| If the content… | …and serves the user's… | …then it must belong to… |
| --- | --- | --- |
| informs action | acquisition of skill | a tutorial |
| informs action | application of skill | a how-to guide |
| informs cognition | application of skill | reference |
| informs cognition | acquisition of skill | explanation |

To use the compass, ask just two questions:

- Is the content about **action** (practical steps, doing) or **cognition** (theoretical or propositional knowledge, thinking)?
- Is the user **acquiring** skill (study) or **applying** skill (work)?

> "The compass can be applied equally to user situations that need documentation, or to documentation itself that perhaps needs to be moved or improved. Like many good tools, it's surprisingly banal." — [diataxis.fr/compass](https://diataxis.fr/compass/)

The compass is a tool for finding your bearings, not a map of the territory. Diataxis is also published as a 2x2 quadrant diagram on [diataxis.fr](https://diataxis.fr/); this skill calls that the *map*. The map is good for orientation; the compass is good for decisions. The two are not interchangeable.

### Use the compass flexibly

The compass is particularly effective when you think you are doing one thing — or the documentation in front of you seems to be — but feel doubt or difficulty in the work. It forces you to stop and reconsider. Sometimes intuition provides an immediate answer that is also wrong.

Do not get fixated on the exact names. If a question feels ambiguous, both readings may be valid. Apply the compass at any scale: at the level of a single sentence, a section, an entire page, or a whole documentation set.

The questions can be used in different ways:

- "Do I think I am writing for *x* or *y*?"
- "Is this writing in front of me engaged in *x* or *y*?"
- "Does the user need *x* or *y*?"
- "Do I want to *x* or *y*?"

### Apply the compass to existing documentation

The compass is just as useful for auditing existing pages as for greenfield work. For each page, ask the two questions. If the page's current form does not match the answer, the page needs to be moved or rewritten — not labelled differently.

## Quick decision tree

The compass above is the canonical tool. The tree below is a quick aid for the most common cases.

```text
What is the reader trying to do right now?
│
├── Acquiring a new skill from scratch?
│   ├── With a complete, guided lesson → Tutorial
│   └── With the shortest path to first success → Quickstart
│
├── Applying a known skill to a real task?
│   ├── General task with a clear goal → How-to
│   └── Stuck on an error → Troubleshooting
│
├── Looking up an exact fact, field, command, or limit? → Reference
│
└── Trying to understand why or how it works?            → Explanation
```

If the request hits more than one branch, split it into multiple documents rather than blending them. The same questions can be applied to existing pages: when a page's content disagrees with its current form, the page needs to be moved, not relabelled.

## When to use this skill

Use this skill when the task is to:

- write, rewrite, restructure, or review a documentation page
- apply the Diataxis compass to classify a page or a request
- split a single messy page into the right Diataxis forms
- audit or migrate an existing documentation set, one page at a time
- design a full documentation system for an SDK, API, CLI, or product (using the compass as a guide, not as a plan)
- map user questions to the correct documentation form

## When NOT to use this skill

Do not use this skill for:

- marketing copy, blog posts, or non-technical writing
- purely visual content such as UI mockups, slides, or design specs
- code review, refactoring, or implementation work
- writing a single paragraph or a short response that does not need a document structure
- translations of finished documentation that already follows a clear structure

## Guiding principles

- Clarity: write in simple, direct language.
- Accuracy: keep facts, code snippets, and document details up to date.
- User centricity: optimize for the reader's goal, not the author's internal structure.
- Consistency: keep tone, terminology, and formatting aligned across the document set.

## Workflow

For a single page, work in this order:

1. Identify the reader.
2. Identify the reader's current state: learning, working, looking up facts, or reflecting.
3. Identify the user need: a task, a fact, a concept, or a learning path.
4. Classify the content using the Diataxis compass.
5. Write only for that form.
6. Link out to the other forms instead of mixing them in.
7. Check whether the document still feels complete, focused, and easy to use.

If the request spans multiple needs, split it into multiple documents rather than blending them together. For a whole documentation system, work one page at a time, not as a top-down plan — see the [Workflow philosophy](#workflow-philosophy) and [Large documentation systems](#large-documentation-systems) sections.

## Typical delivery pattern

For most requests, follow this order:

1. Clarify the document type, audience, goal, and scope if they are not obvious.
2. Propose or infer the outline for that document type.
3. Write the document in Markdown using the right blueprint.
4. If the request is actually a mixed documentation system, split it into companion docs instead of one large page.

Do not force a long approval loop for every task. Use a short clarification loop only when the user has not given enough information.

## Anti-patterns: what NOT to do

The most common failure in technical documentation is **mixing forms**. The rules below are the inverse of the guidance above. If a draft shows any of these signals, stop and split, prune, or rewrite.

### The four cardinal sins

| Sin | What it looks like | Fix |
| --- | --- | --- |
| Page does everything | One page teaches, instructs, lists facts, and explains tradeoffs | Split into tutorial, how-to, reference, and explanation |
| Tutorial gives options | Step 2 offers three alternative paths | Pick one path; move alternatives to a how-to |
| How-to teaches | Page starts with "Concepts" or "Background" | Strip the teaching; link to a tutorial or explanation |
| Reference narrates | Reference page uses "we recommend", "you should", or stories | Strip the narration; keep neutral descriptions only |

### Per-form anti-patterns

**Tutorial anti-patterns**

- Branching paths or "choose your adventure" steps. Tutorials must be linear.
- Long background sections before the first action. Move background to explanation.
- Hidden prerequisites. State them at the top.
- Skipping the expected result. Show what success looks like at every checkpoint.
- Over-explaining. If a sentence is not helping the learner do the next step, cut it.

**How-to anti-patterns**

- Opening with theory. Start with the goal.
- "First, let's understand…" sections. The reader is working, not learning.
- Re-teaching concepts the user already knows. Link out instead.
- More than one goal per page. Split the page.
- "When you are done, you will understand…" framing. That belongs in a tutorial.

**Reference anti-patterns**

- Imperative voice ("Run this command", "Set this value"). Reference describes, it does not instruct.
- Comparisons, recommendations, or "best practice" opinions. Move them to explanation.
- Sections ordered by importance to the author. Order by the structure of the thing described.
- Missing examples of input and output. Every entry should show concrete usage.
- Soft language such as "usually", "generally", "often". Be exact or omit.

**Explanation anti-patterns**

- Step-by-step instructions. Explanations do not give procedures.
- "How to" titles. Use "About …" or "Why …" framing.
- Unbounded scope. Cover one concept, one design decision, or one tradeoff at a time.
- No point of view or insight. An explanation that neither offers a perspective nor helps the reader form a new understanding is just a summary.

### Mixed-doc smell test

A page is probably mixing forms if it has two or more of:

- A "Background" or "Concepts" section **and** a "Steps" or "Procedure" section
- Both narrative paragraphs **and** parameter tables in roughly equal weight
- Both "Why we built it this way" framing **and** "Run this command" instructions
- A "Tutorial" heading that includes optional branches or troubleshooting
- A "How-to" that begins with "In this tutorial you will learn…" or similar tutorial-style framing
- A README longer than about 500 lines that tries to teach, instruct, list, and explain at once

When a page trips the smell test, the right response is to **split it**, not to add headings.

### README-specific anti-patterns

- README doing the work of an entire docs site. README is an entry point.
- "Quickstart" inside the README that is actually a full tutorial. Link to the tutorial instead.
- Listing every command, option, or flag in the README. Move them to reference.
- Mixing install, configure, deploy, troubleshoot, and contribute all in one file.

### Adjacent-type anti-patterns

- **Troubleshooting that explains**: keep it symptom → cause → solution. Move "why this happens" to explanation.
- **Glossary that teaches**: a glossary is a list of terms. Move usage examples to how-to or reference.
- **Release notes that justify**: release notes state change and impact. Move rationale to explanation.
- **Style guide that classifies**: the style guide should describe voice and rules, not re-derive Diataxis.
- **Quickstart that explores**: a quickstart optimizes for first success, not for completeness.

## Workflow philosophy

Diataxis is meant to be used as a guide, not as a plan. The official workflow says:

- **Use Diataxis as a guide, not a plan.** Apply the compass where you are, not where you wish you were. Do not create empty structures in advance.
- **Do not worry about structure.** Focus on content quality. The structure will emerge from the work.
- **Work one step at a time.** Make small, responsive improvements; finish and ship each one before starting the next. Do not plan a complete rewrite before starting.
- **Diataxis changes the structure of your documentation from the inside, the way cells form a tissue.** It is not a template to impose on top of existing content. Allow the work to develop organically; the structure will differentiate once enough content is in place.

In practice, this means:

- When the user asks for "a complete documentation system", do not deliver an empty scaffold. Ask what already exists and start there.
- When the user asks for "a single mega-page that does everything", do not just split it. Walk through the compass with the user; the split is a consequence, not a goal.
- When a single improvement would move a page from one form to another, make that improvement before planning the next.

A messy page that is honest about its content is better than a clean four-section site with nothing in three of the four sections.

## Large documentation systems

When the user asks for a full documentation system, SDK docs, API docs, developer portal, or documentation site, **use the compass as a guide, not a plan**. Diataxis explicitly warns against top-down planning: the structure should emerge from content, not be imposed on it.

### How to work iteratively

1. Apply the compass to any existing material first. Tag every existing page with a compass cell. Move pages that are misplaced before adding new ones.
2. For new content, write one page at a time. Apply the compass to that page alone. Do not create empty top-level sections in advance.
3. After enough pages exist, the top-level structure will start to demand certain headings. Let those emerge from the work, not from a plan.
4. If you are asked to propose a structure up front, treat it as a sketch, not a contract. The structure is allowed to change as the content improves.

### Anti-patterns for large systems

- **Creating empty sections up front.** An empty "Tutorials" section is worse than no Tutorials section.
- **Allocating fixed content quotas per quadrant.** Compass cells are not buckets to fill.
- **Forcing IA before content is good.** A navigation that pre-commits to four forms is brittle.
- **Treating "documentation system" as a deliverable.** It is a practice, not a product.

### Inputs to gather, but only what is needed

Before starting, confirm what you actually need: source material (API spec, source code, existing docs, product notes), audience (beginners, experienced developers, admins, support, partners, non-technical users), platform preference (plain Markdown, Docusaurus, ReadTheDocs, Mintlify, GitBook, static site, repo docs), style constraints (brand voice, terminology, localization, accessibility, formatting rules), code examples (languages, runnable snippets, SDK samples, CLI examples, sample repositories), and lifecycle needs (versioning, search, navigation, release notes, changelog, deprecation policy). Ask only what is needed right now; the compass does the rest.

### Common artifact patterns (for reference only)

Do not treat this as a backlog. Only produce artifacts the compass calls for.

| Artifact | Compass cell | Purpose |
| --- | --- | --- |
| Getting started path or quickstart | action + acquisition (tutorial) | First success for a new user |
| How-to guide collection | action + application (how-to) | One task per page, assumes competence |
| API, CLI, config, or schema reference | cognition + application (reference) | Neutral, structured facts |
| Concept, architecture, and design articles | cognition + acquisition (explanation) | Why and how it works, tradeoffs |
| Troubleshooting | action + application (how-to) | Symptom → cause → solution |
| Glossary | cognition + application (reference) | Project-specific terms |
| Release notes and changelog | cognition + application (reference) | What changed and why it matters |
| Style guide | meta | Team rules for consistent writing |
| Sample apps and runnable examples | action + acquisition (tutorial) or cognition + application (reference) | Validate the system end to end |

### Per-platform notes

- **Plain Markdown or repo docs**: keep one form per file; use clear filenames such as `how-to-deploy.md`.
- **Docusaurus / Mintlify / GitBook**: place each form under a distinct top-level section so navigation reinforces the separation.
- **ReadTheDocs / static sites**: use section labels and admonitions sparingly; do not hide form boundaries inside prose.
- **Multi-product portals**: keep one Diataxis quadrant per page; let the cross-product index live in a separate overview page.

Keep the Diataxis separation intact when navigation does emerge, but do not force the separation before there is content to separate.

## Reference files

Use the bundled references when you need more structure than the main guidance provides:

- `references/reader-analysis.md`: ask these questions before drafting.
- `references/doc-blueprints.md`: use these section patterns when outlining a document.
- `references/template-map.md`: use this to map Diataxis forms to common TGDP templates.
- `references/zh-cn-anti-patterns.md`: use this only when reviewing Chinese technical documentation or Chinese localization drafts.

## Classification guide

### Tutorial

Use when the reader needs a guided learning experience.

Write it as a safe, structured lesson:

- start with what the reader will do or build
- use a single path
- keep steps concrete and sequential
- show expected results early and often
- minimize explanation
- avoid options and digressions

Good signs:

- "try", "first time", "hands-on", "intro", "walkthrough"
- the user is not yet competent

### How-to

Use when the reader already knows the basics and wants to achieve a task.

Write it as practical directions:

- begin with the goal
- assume basic competence
- cover one task or problem
- keep the sequence logical and short
- include warnings and alternate paths only when needed
- do not teach concepts

Good signs:

- "how do I...", "configure", "deploy", "set up", "migrate", "troubleshoot"
- the reader is in work mode

### Reference

Use when the reader needs exact facts.

Write it as neutral technical description:

- organize by the structure of the thing described
- keep language factual and concise
- include parameters, values, limits, fields, commands, return values, examples
- avoid instructions, teaching, and discussion

Good signs:

- APIs, fields, commands, flags, schemas, tables, options, limits, error codes

### Explanation

Use when the reader needs understanding.

Write it as context-rich discussion:

- explain why something exists
- connect related ideas
- discuss tradeoffs, history, and alternatives
- allow perspective and judgment
- keep it separate from procedural guidance

Good signs:

- "why", "background", "concept", "design", "tradeoff", "overview", "discussion"

## Useful adjacent document types

Use these as concrete specializations of the four forms:

- Quickstart: a short tutorial that gets a user to a first success fast
- README: the project's first impression and entry point
- Troubleshooting: symptom -> cause -> solution
- Glossary: terms and definitions for project-specific language
- Release notes: what changed, why it matters, and known issues
- Style guide: team rules for writing consistently

For API and SDK documentation, treat these as common combinations:

- Getting started or quickstart -> tutorial
- Authentication setup -> how-to or reference, depending on whether it is task steps or field definitions
- Endpoint, method, class, option, or error-code pages -> reference
- Architecture, design model, rate limits, pagination model, or SDK philosophy -> explanation
- Migration and upgrade guides -> how-to with reference links

When a request sounds like one of these, map it back to the four Diataxis needs before writing.


### Tutorial or how-to ambiguity

Tutorial and how-to are the most common gray area. When the request could be either, ask one short question before drafting if the answer is not already clear:

> Is the reader trying to learn a new skill through a guided exercise, or trying to complete a real task they already understand?

Use these fallbacks:

- Choose tutorial or quickstart when the reader is new, needs a safe single path, and benefits from visible expected results.
- Choose how-to when the reader has a production goal, already understands the basics, and mainly needs practical steps.
- If the user cannot clarify, state your assumption and prefer a focused how-to; link teaching material out instead of embedding a lesson.


## Writing patterns

### For tutorials

- Use second-person or inclusive language where appropriate.
- Keep the path stable and predictable.
- Avoid comprehensive background sections.
- Give the user visible progress.

### For how-tos

- Start with the objective.
- Use conditional imperatives where helpful.
- Keep to one goal per page.
- Put supporting material in links, not inline essays.

### For reference

- Mirror the product structure.
- Prefer tables, lists, and schemas.
- Be explicit and consistent.

### For explanation

- Make connections.
- Provide context.
- Discuss alternatives and implications.
- Keep the scope bounded to one concept or topic.

## Template selection heuristic

If the user asks for:

- "a guide for my new users" -> tutorial or quickstart
- "how to do X" -> how-to
- "what does this field/command mean" -> reference
- "why is it designed this way" -> explanation
- "a single document system for docs" -> use Diataxis to split the system into the right document types

## Quality checks

Before finishing, run both the **intent check** and the **smell test** (see [Mixed-doc smell test](#mixed-doc-smell-test) above for the full list of signals).

### Intent check

- Did I identify the reader and their state before drafting?
- Did I classify the content using the Diataxis decision tree first?
- Did I write for the reader's actual state, not the author's mental model?
- Did I keep the document focused on one need?
- Did I avoid unnecessary structure at the top level?
- Did I leave room for linked companion documents?

### Per-form final check

- **Tutorial**: a complete beginner can finish it without reading anything else; every step has a visible result.
- **How-to**: an experienced user can find the answer in under a minute; the page covers exactly one goal.
- **Reference**: every entry is structured identically; nothing is missing or redundant.
- **Explanation**: a single concept is covered; the author offers a point of view or insight; no procedures are included.

## References

Use these as the primary source material behind this skill:

- https://diataxis.fr/
- https://diataxis.fr/start-here/
- https://diataxis.fr/compass/
- https://diataxis.fr/how-to-use-diataxis/
- https://diataxis.fr/tutorials/
- https://diataxis.fr/how-to-guides/
- https://diataxis.fr/reference/
- https://diataxis.fr/explanation/
- https://www.thegooddocsproject.dev/
- https://www.thegooddocsproject.dev/template/

This skill distills those sources into a practical documentation workflow.

## Practical output pattern

When the user asks you to write documentation, start with a short diagnosis first:

- What is the reader trying to do?
- What state are they in?
- Which document type is this?
- What should be excluded?

Then write the document, or produce a doc plan, in the chosen form.

---
> Source: [88lin/diataxis-docs-skill](https://github.com/88lin/diataxis-docs-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
