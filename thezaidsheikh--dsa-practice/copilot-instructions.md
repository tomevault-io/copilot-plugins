## dsa-practice

> This file is the working guide for any coding or documentation agent operating in this repository.

# AGENTS.md

This file is the working guide for any coding or documentation agent operating in this repository.

## Repository Purpose

This is a DSA practice repository made up almost entirely of Markdown notes.

The repo is not a runnable application. It is a study system for:

- documenting algorithm patterns
- storing problem-specific solutions
- comparing brute-force, better, and optimal approaches
- keeping implementation examples, mostly in Java
- preserving reasoning, intuition, and complexity analysis for interview prep

The current README describes the repo as a curated DSA practice collection focused on clear explanations, complexity analysis, and clean solutions.

## What Actually Exists Here

At the time of writing, the repo contains `81` Markdown files and no normal source-code package structure such as `src/`, `tests/`, `package.json`, `pom.xml`, or `requirements.txt`.

Key top-level items:

- `README.md`: short repository description
- `CLAUDE.md`: existing AI guidance, but incomplete and slightly generic
- `AGENTS.md`: this file
- topic notes at the root:
  - `two-pointers.md`
  - `sliding-window.md`
  - `prefix-sum.md`
  - `kadanes-algorithm.md`
- problem-set folders:
  - `Algorithms-Patterns/`
  - `Leetcode-150/`
  - `Striver/`
  - `RisingBrain/`
  - `padho-with-pratyush/`

There are also editor/project folders like `.idea/` and `.vscode/`, but they are not part of the learning content.

## Content Map

### 1. Root-Level Pattern Notes

The root contains broad algorithm-pattern explainers such as:

- `two-pointers.md`
- `sliding-window.md`
- `prefix-sum.md`
- `kadanes-algorithm.md`

These are generally more structured than the problem notes. They often include:

- a definition
- when to use the pattern
- why it works
- common variations
- a simple example
- differences from related techniques
- complexity summary

When editing these files, preserve their more educational and concept-driven style.

### 2. `Algorithms-Patterns/`

This folder is a second pattern library and includes files like:

- `Algorithms-Patterns/graph.md`
- `Algorithms-Patterns/heap.md`
- `Algorithms-Patterns/two-pointers.md`
- `Algorithms-Patterns/sliding-window.md`
- `Algorithms-Patterns/prefix-sum.md`
- `Algorithms-Patterns/kadanes-algorithm.md`
- `Algorithms-Patterns/two-sum.md`
- `Algorithms-Patterns/Tree/BinaryTree.md`
- `Algorithms-Patterns/Tree/levelOrder.md`

This area leans toward conceptual explanations and pattern-first thinking rather than only one problem at a time.

Use this folder when the content is about:

- a reusable technique
- a core data structure pattern such as heap / priority queue
- a general traversal pattern
- a problem family
- a template or recognition framework

Do not add single LeetCode problem notes here unless the note is explicitly teaching a pattern through that example.

### 3. `Leetcode-150/`

This folder contains compact notes for Top Interview 150 style problems, such as:

- `RemoveElement.md`
- `MergeSortedArray.md`
- `majority-element.md`
- `rotate-array.md`
- `best-time-to-buy-and-sell-stock.md`

These notes are typically short and direct:

- problem link
- one or more named solutions
- numbered steps
- Java code
- time and space complexity

This folder is suitable for concise problem-by-problem documentation.

### 4. `Striver/`

This folder currently focuses on Striver-style array content, for example:

- `Striver/Array/rotate-array.md`
- `Striver/Array/spiral-matrix.md`
- `Striver/Array/set-matrix-zeros.md`
- `Striver/Array/majority-element-ii.md`
- `Striver/Array/longestConsecutiveSeq.md`

Use this area for notes aligned with Striver’s sequencing or naming conventions. Preserve existing naming even when filenames are inconsistent with strict slug formatting.

### 5. `RisingBrain/`

This currently includes array/two-pointer material such as:

- `RisingBrain/Array/Two-Pointers/3sum.md`
- `RisingBrain/Array/Two-Pointers/container-with-most-water.md`
- `RisingBrain/Array/Two-Pointers/trapping-rain-water.md`
- `RisingBrain/Array/Two-Pointers/two-sum-ii-input-array-is-sorted.md`

Use this folder when content is tied to that learning track or grouping.

### 6. `padho-with-pratyush/`

This is one of the largest content sections and includes:

- `Binary-Search/`
- `Heap/`
- `Prefix-Sum/`
- `Recursion/`
- `Tree/`

Representative files:

- `padho-with-pratyush/Binary-Search/KokoEatingBanana.md`
- `padho-with-pratyush/Heap/task-scheduler.md`
- `padho-with-pratyush/Recursion/generate-parentheses.md`
- `padho-with-pratyush/Tree/validate-binary-search-tree.md`

This folder usually contains fuller explanations than the most compact notes, but the style still varies file to file.

## Important Reality About Style

This repository is useful, but not fully standardized.

An agent must work with that reality instead of pretending it is already normalized.

Observed style traits:

- Many files begin with `Prob:` and a direct LeetCode link.
- Many solutions are labeled like `Sol 1: Brute Force`, `Sol 2: Optimal`, or similar.
- Java is the dominant implementation language in problem notes.
- Some pattern files use polished Markdown headings and narrative sections.
- Some files use informal phrasing.
- Some complexity explanations are terse; some are verbose.
- Some filenames are clean slugs, while others contain spaces or inconsistent capitalization.

Do not mass-reformat the repo just to impose consistency.

Only standardize the file you are actively editing, and only to a reasonable degree.

## Primary Agent Responsibilities

When working in this repository, an agent should optimize for:

- accuracy of algorithm explanation
- correctness of complexity analysis
- preservation of the study-oriented format
- minimal disruption to neighboring notes
- better readability without unnecessary rewriting

An agent is expected to:

- read existing nearby files before adding a new one
- place new notes in the correct learning-track folder
- keep explanations practical and interview-oriented
- prefer Java examples unless the local context strongly suggests otherwise
- clearly separate brute-force, better, and optimal approaches where relevant
- explain why an optimization works, not just what the final code is

## What Not To Do

Do not:

- turn the repository into a runnable software project
- add package managers, build tooling, or test harnesses unless explicitly asked
- rewrite dozens of existing notes to force uniform formatting
- remove older explanation styles just because they are less polished
- silently change the language of examples from Java to another language in existing problem-note folders
- invent problem statements or complexity claims without checking them

## File Placement Rules

Before creating a file, decide whether it belongs to:

- a root-level pattern note
- `Algorithms-Patterns/` for concept-first material
- a curated track folder like `Leetcode-150/`, `Striver/`, `RisingBrain/`, or `padho-with-pratyush/`

Use these rules:

- If the content teaches a reusable pattern, place it under `Algorithms-Patterns/` or update an existing root pattern note.
- If the content is a single interview problem tied to a known study track, place it in that track’s folder.
- If a very similar note already exists, update it instead of creating a duplicate.
- If both a root-level pattern file and a folder-level pattern file exist on the same topic, preserve both unless the user explicitly asks to consolidate.

## Naming Rules

Because the repo already has mixed conventions, use the nearest local convention rather than a global cleanup rule.

Examples already present:

- kebab-case: `majority-element.md`
- mixed camel/caps: `BuyAndSellStock.md`
- spaces in filenames: `Find Minimum in Rotated Sorted Array.md`

Agent rule:

- match the style of sibling files in the same folder
- keep names descriptive and problem-identifiable
- avoid renaming existing files unless explicitly requested

## Preferred Structure For New Problem Notes

When creating a new problem note, prefer this structure unless sibling files strongly suggest a shorter format:

````md
Prob: <problem link>

## Problem Summary
<1-3 lines>

## Sol 1: Brute Force
1. ...
2. ...

```java
// code
```

Time Complexity: ...
Space Complexity: ...

## Sol 2: Better
1. ...

```java
// code
```

Time Complexity: ...
Space Complexity: ...

## Sol 3: Optimal
1. ...
2. ...

Why this works:
- ...

```java
// code
```

Time Complexity: ...
Space Complexity: ...

## Key Takeaways
- ...
````

This does not mean every file must have all three solutions. If only one worthwhile solution exists, one solution section is enough.

## Preferred Structure For Pattern Notes

For pattern or concept documents, prefer a structure like:

- title
- definition
- when to use
- why it works
- common variations
- example problem
- template or pseudocode/code outline
- differences from nearby patterns
- complexity
- recognition cues / interview heuristics

The existing root pattern files and `Algorithms-Patterns/*.md` files are the best references for this style.

## Language and Code Conventions

### Default Language

For problem solutions, default to Java because that is what most existing problem notes use.

Only switch languages if:

- the target file already uses another language
- the user explicitly requests another language
- the surrounding folder clearly uses another language convention

### Code Expectations

Code should be:

- correct
- interview-appropriate
- readable without external dependencies
- formatted cleanly inside fenced Markdown blocks

Prefer:

- `class Solution` for LeetCode-style snippets
- short helper methods when they make the solution clearer
- variable names that reflect the algorithm, such as `left`, `right`, `prev`, `prefix`, `low`, `high`

Avoid:

- adding imports unless they clarify needed Java collections usage
- overly clever code golf
- framework-specific or runnable project boilerplate

## Complexity Guidance

Complexity claims must be treated carefully.

An agent should verify:

- time complexity
- auxiliary space complexity
- whether output storage is being counted or excluded
- whether recursion stack space should be mentioned separately

If the existing file is slightly informal, improve accuracy without turning the note into a textbook.

## Editing Existing Notes

When updating an existing Markdown note:

1. Read the whole file first.
2. Preserve the original intent and learning flow.
3. Fix factual mistakes before polishing wording.
4. Keep the file as compact as the folder’s local style expects.
5. Avoid unnecessary heading churn if the file is already simple and readable.

Good edits include:

- fixing a wrong explanation
- correcting a broken algorithm
- tightening complexity statements
- adding missing edge-case notes
- improving numbered steps so the logic is easier to follow

Bad edits include:

- replacing a short useful note with a long generic tutorial
- changing the voice of the repo everywhere
- adding sections that don’t help with solving or remembering the problem

## Quality Bar For New Content

A new or revised note should answer these questions clearly:

- What is the problem asking?
- What is the brute-force idea, if relevant?
- What observation leads to the optimized approach?
- Why does the optimized approach work?
- What are the time and space costs?
- What edge cases matter?
- What pattern should the reader remember next time?

If a note fails to teach recognition and reasoning, it is incomplete even if the code is correct.

## Practical Workflow For Agents

Recommended workflow:

1. Inspect sibling files in the target folder.
2. Identify whether the note is pattern-first or problem-first.
3. Match local naming and formatting conventions.
4. Draft or revise the explanation.
5. Add or correct Java code blocks.
6. Verify complexity statements.
7. Keep the final file concise enough for revision use.

## Repository-Specific Observations To Preserve

These are real patterns in the repo and should influence edits:

- Root pattern notes are more polished and educational.
- `Algorithms-Patterns/` is the best home for reusable topics like heap, prefix sum, sliding window, and two pointers.
- Folder-specific problem notes are often compact and direct.
- `Leetcode-150/` favors short problem-centric writeups.
- `padho-with-pratyush/` often includes intuition and deeper explanation.
- Some files intentionally compare multiple solution tiers.
- The repo values study utility over stylistic perfection.

An agent should preserve this character.

## No Build/Test Assumptions

There is no established automated test suite for this repository.

That means:

- do not claim to have run application tests
- validate changes by reading and reasoning about the Markdown content
- if code snippets are changed, mentally verify logic and complexity

If the user later asks for tooling to validate snippets, that is a separate task.

## Suggested Standard For Future Additions

Without rewriting the whole repo, future additions should gradually move toward:

- clear headings
- one problem link near the top
- explicit solution labels
- one concise intuition section for non-obvious approaches
- fenced code blocks with language tags
- separate time and space complexity lines
- concise key takeaway or pattern reminder

This is the direction of improvement, not a forced migration rule.

## Final Instruction To Agents

Act like a careful maintainer of a personal study repository, not like a framework engineer.

The correct default behavior is:

- read first
- match local style
- improve correctness
- improve clarity
- avoid unnecessary churn

If you add value while preserving the repo’s study-first character, you are doing the right thing.

---
> Source: [thezaidsheikh/dsa-practice](https://github.com/thezaidsheikh/dsa-practice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
