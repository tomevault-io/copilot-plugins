## ftsiout

> > Project-neutral template for coding and research agents.

# AGENTS.md

> Project-neutral template for coding and research agents.
>
> Replace values written as `<PLACEHOLDER>` with repository-specific information.
> Delete sections that are not applicable to the project rather than leaving misleading rules behind.

## Purpose

This file defines the operating contract for AI/coding agents working in this repository.

Agents must treat repository contents as authoritative project artifacts, not as disposable text.
Changes to code, configuration, documentation, tests, schemas, generated artifacts, research claims,
data, filenames, APIs, or build structure may alter project behavior or meaning.

Before making substantial changes, inspect the actual repository and determine:

- the project structure;
- the build/test/lint workflow;
- source-of-truth files;
- generated versus hand-maintained artifacts;
- repository-local conventions;
- relevant nested `AGENTS.md` or equivalent policy files;
- whether the requested task is an audit, proposal, implementation, rewrite, migration, or release task.

Do not assume this template describes the repository more accurately than the repository itself.

---

# Core principles

1. **Do not invent facts or behavior.**
   - Never fabricate APIs, files, commands, dependencies, citations, experiment results, configuration
     values, implementation behavior, or project history.
   - If something cannot be established from repository evidence or an explicit user instruction,
     mark it unresolved or state the assumption clearly.

2. **Do not silently reconcile conflicts.**
   - If code, documentation, tests, configuration, generated output, or other artifacts disagree,
     report the conflict.
   - Resolve it only from an appropriate authoritative source or an explicit user decision.

3. **Preserve scope.**
   - Do not generalize a result, guarantee, API contract, benchmark, or implementation property beyond
     the conditions under which it is established.
   - Distinguish current behavior from intended behavior.

4. **Separate evidence from interpretation.**
   Use explicit categories where useful:
   - observed behavior;
   - verified implementation fact;
   - documented contract;
   - inference;
   - hypothesis;
   - proposal/future work;
   - unresolved issue.

5. **Preserve reproducibility and traceability.**
   Where relevant, retain:
   - commit hashes;
   - versions;
   - configuration names;
   - environment/toolchain versions;
   - seeds;
   - dataset or experiment IDs;
   - source paths;
   - generated artifacts;
   - migration history;
   - build/test commands.

6. **Prefer reversible edits.**
   - Keep unrelated refactors separate.
   - Avoid combining structural migration, behavior changes, formatting changes, dependency upgrades,
     and documentation rewrites in one opaque change.
   - Prefer small, auditable patches over broad rewrites.

7. **Respect repository-local conventions.**
   - Existing project conventions override this generic template unless they conflict with an explicit
     user instruction.
   - Do not introduce a new framework, tool, directory convention, formatter, or architecture merely
     because it is personally preferable.

---

# Instruction precedence

Use the following priority unless the repository defines a stricter policy:

1. explicit user instruction for the current task;
2. repository-local policy (`AGENTS.md`, `CONTRIBUTING.md`, architecture decisions, style guides);
3. authoritative project configuration and executable behavior;
4. established project conventions;
5. this generic template.

If two higher-priority sources conflict, stop guessing and report the conflict.

Nested policy files apply to their subtree and may refine or override this file.

---

# Repository discovery

Before substantial edits, inspect the repository rather than assuming a layout.

Record or identify, when applicable:

```text
Project root:
Primary language(s):
Package/build system:
Application entry points:
Test directories:
Documentation directories:
Configuration directories:
Generated directories:
Data/artifact directories:
CI configuration:
Release configuration:
Repository-local agent instructions:
```

Example only:

```text
src/
tests/
docs/
scripts/
config/
assets/
.github/
AGENTS.md
README.md
```

Do not create missing directories just to match this example.

---

# Source-of-truth discipline

For any factual, behavioral, numerical, or structural claim, identify the most authoritative available
source.

A typical ordering is:

```text
1. executable code / schema / canonical data
2. automated tests or validated generated artifacts
3. authoritative configuration
4. generated reports derived from canonical sources
5. maintained technical documentation
6. README / narrative documentation
7. comments, examples, slides, or historical notes
```

This ordering is only a default. A repository may explicitly define another authority hierarchy.

When artifacts disagree:

- do not silently normalize them;
- identify the mismatch;
- determine the likely source of truth;
- update dependent artifacts consistently when the task requires it.

Avoid circular authority such as treating a summary as proof of the source data it summarizes.

---

# Editing boundaries

Agents may normally, when relevant to the requested task:

- fix defects;
- add tests;
- improve error handling;
- improve naming;
- improve comments and documentation;
- refactor locally without changing behavior;
- repair stale paths or references;
- remove confirmed dead code;
- improve build/test hygiene;
- add validation;
- regenerate derived artifacts from canonical sources;
- improve accessibility, usability, or developer experience without violating project constraints.

Agents should stop and report before making an unrequested change that would materially alter:

- public APIs;
- persisted data formats;
- database schemas or migrations;
- compatibility guarantees;
- security boundaries;
- authentication/authorization behavior;
- numerical/scientific results;
- evaluation methodology;
- externally visible semantics;
- licensing;
- deployment architecture;
- supported platforms;
- major dependencies;
- project-wide naming or directory conventions.

If the user explicitly requested such a change, implement it carefully and document the impact.

---

# Planning substantial changes

For large changes, prefer the following sequence:

## Phase 1 — Audit

Determine:

- current behavior;
- relevant files;
- dependencies;
- tests;
- affected interfaces;
- source-of-truth artifacts;
- known inconsistencies;
- migration risks.

When the task is explicitly audit-only, do not modify project files unless the user provides or
requests an output path.

## Phase 2 — Minimal design

Define:

- intended behavior;
- invariants;
- compatibility constraints;
- files/components to change;
- validation strategy;
- rollback or migration considerations.

Do not over-design small changes.

## Phase 3 — Implementation

Make the smallest coherent set of changes that satisfies the task.

## Phase 4 — Validation

Run the repository's applicable:

- formatter;
- linter;
- type checker;
- unit tests;
- integration tests;
- build;
- documentation build;
- schema validation;
- project-specific checks.

## Phase 5 — Review

Inspect the diff for:

- accidental unrelated edits;
- stale comments/docs;
- generated-file drift;
- missing tests;
- compatibility regressions;
- unresolved TODOs introduced by the change.

---

# Code changes

When modifying code:

- understand the call path before editing;
- preserve public behavior unless a behavior change is requested;
- prefer project-native abstractions;
- avoid speculative abstractions;
- keep error handling consistent with surrounding code;
- preserve concurrency, transaction, and ownership invariants;
- do not weaken validation merely to make tests pass;
- do not catch and discard errors without a documented reason;
- avoid hidden global state where the project does not already rely on it;
- update affected tests and documentation.

When fixing a bug, prefer a regression test that fails before the fix and passes after it.

Do not rewrite working components wholesale when a focused patch is sufficient.

---

# Dependency changes

Before adding or upgrading a dependency, check:

- whether the project already provides the needed capability;
- compatibility with supported language/runtime versions;
- license implications if relevant;
- build size/runtime impact if material;
- security/deprecation status when the task requires current verification;
- whether lockfiles or generated dependency metadata must be updated.

Do not upgrade unrelated dependencies as collateral cleanup.

---

# Configuration changes

Treat configuration as executable project behavior.

When changing configuration:

- preserve environment-specific overrides;
- do not commit secrets;
- do not replace placeholders with real credentials;
- update examples when the contract changes;
- validate syntax;
- document new required values;
- preserve backward compatibility where required.

Clearly distinguish:

```text
configuration default
runtime-derived value
environment override
secret
example value
test fixture
```

---

# Data, schemas, and migrations

For changes involving persisted data:

- identify the canonical schema;
- inspect existing migrations;
- avoid destructive operations unless explicitly requested;
- consider forward and backward compatibility;
- make migrations deterministic where possible;
- test migration paths when tooling exists;
- distinguish test/sample data from production data.

Never fabricate production data or silently mutate source datasets to make results look correct.

---

# Tests

Tests are part of the project contract, but they are not automatically more authoritative than the
intended specification.

When tests and implementation disagree:

1. identify the intended behavior from the strongest available evidence;
2. determine whether the code or test is stale;
3. change the correct side;
4. avoid weakening assertions merely to obtain a green test suite.

Prefer tests that assert externally meaningful behavior over implementation trivia.

For flaky tests, diagnose the source of nondeterminism rather than immediately adding retries or
larger timeouts.

---

# Generated files

Determine whether a file is generated before editing it manually.

Prefer:

```text
canonical source -> generator -> generated artifact
```

over hand-editing the generated artifact.

If generated files are intentionally versioned:

- regenerate them using the project command;
- include them when their source changed;
- avoid unrelated generated churn.

Do not delete tracked build or release artifacts without first determining why they are committed.

---

# Documentation

Documentation must describe the repository that actually exists.

When updating docs:

- verify commands;
- verify paths;
- verify configuration names;
- verify API examples;
- distinguish stable interfaces from examples;
- distinguish current features from planned features;
- avoid marketing claims not supported by evidence.

When behavior changes, update the nearest authoritative documentation in the same change when
practical.

Do not copy stale README text into more authoritative documentation.

---

# Claims and technical writing

For technical or research-oriented repositories, preserve claim strength.

Suggested hierarchy:

```text
proves / establishes
    formal result under explicit assumptions

demonstrates
    strong direct evidence

supports
    evidence consistent with a claim

is consistent with
    compatible with an interpretation but not uniquely identifying it

suggests
    weak or exploratory evidence

motivates
    reason to investigate

hypothesizes / proposes
    not yet established
```

Do not upgrade wording without stronger evidence.

Keep these distinct:

```text
fact
requirement
implementation invariant
runtime validation
configuration default
empirical observation
interpretation
hypothesis
future work
```

---

# Research and experimental artifacts

Delete this section if the repository contains no research or experimental work.

Do not invent:

- citations;
- datasets;
- experiments;
- metrics;
- values;
- confidence intervals;
- theorem assumptions;
- benchmark outcomes.

For reproducible experiments, preserve where applicable:

```text
experiment/study ID
commit
dataset/environment and version
algorithm/model
configuration
seed or seed list
run count
training/evaluation budget
metric definition
normalization
aggregate estimator
uncertainty procedure
source artifact
exclusions
hardware/software environment
```

Prefer regeneration of tables/plots from canonical data over manual retyping.

Do not describe proposed or incomplete experiments as completed work.

---

# Statistical language

Delete this section if statistical reporting is irrelevant.

Be precise about uncertainty.

Use terms such as:

```text
sample standard deviation
standard error
confidence interval
credible interval
bootstrap interval
interquartile range
```

only when that is what was actually computed.

State the resampling or observational unit when it affects interpretation.

Do not infer causality from correlational results unless the design supports a causal claim.

Do not generalize from a finite benchmark or sample beyond its supported scope.

---

# Mathematical content

Delete this section if the repository contains no mathematical specification.

For formal objects:

- define domains;
- define indices;
- distinguish fixed and time-varying quantities;
- define expectations and conditioning where ambiguous;
- define norms before use;
- keep notation consistent.

For formal propositions/theorems:

- state assumptions locally;
- state the exact conclusion;
- include or reference the proof;
- specify implementation correspondence when relevant;
- state important boundaries when misuse is plausible.

Do not alter a formal result merely to make it match an implementation or experiment.

---

# APIs and interfaces

When changing an API, CLI, protocol, file format, or other public interface:

- identify existing consumers;
- preserve compatibility unless breaking change is intended;
- update examples/tests/schema definitions;
- document renamed or removed fields/options;
- avoid undocumented behavioral changes;
- use deprecation paths where the project expects them.

For internal identifiers, do not rename them merely to make prose prettier if doing so would damage
traceability or compatibility.

---

# Security and privacy

Do not:

- commit secrets, tokens, private keys, passwords, or credentials;
- expose sensitive user data in logs or fixtures;
- disable authorization or validation to simplify implementation;
- replace cryptographic/security mechanisms with ad-hoc alternatives;
- log full sensitive payloads without a documented need.

When security behavior is ambiguous, report the ambiguity instead of guessing.

Use repository-approved secret-management and authentication mechanisms.

---

# File and directory changes

When moving or renaming files:

- search for references first;
- update imports/includes;
- update build configuration;
- update documentation links;
- update CI/release scripts;
- preserve stable identifiers where possible;
- avoid case-only renames when cross-platform behavior may be affected unless handled safely.

Do not flatten modular source trees without a clear project reason.

---

# Build hygiene

Use the project's actual commands.

For this template, use the canonical commands in the project-specific additions below. The narrator
has a Python CLI and focused test suite; the relevant checks also validate document structure, shell
helpers, Python helpers, and GitHub Actions workflows.

Generated build artifacts should normally live outside source directories unless the project
intentionally versions them.

---

# Validation policy

After making changes, run the smallest sufficient validation set, expanding it when the impact is
broad.

Typical order:

1. format touched files;
2. run targeted tests;
3. run lint/type checks for affected modules;
4. run broader tests when practical;
5. run the build;
6. inspect generated output or UI manually when the change is visual or layout-sensitive.

Do not claim a check passed unless it was actually executed.

If a check cannot run:

- say which check;
- explain why;
- report what was validated instead.

---

# Visual/UI changes

For user-interface changes:

- inspect the existing design system first;
- preserve established spacing, typography, tokens, and component conventions;
- verify responsive behavior when relevant;
- preserve keyboard accessibility and semantic markup;
- avoid replacing real data visualizations with decorative approximations;
- inspect changed screens visually when tooling permits.

Do not introduce a new visual language for a local change unless requested.

---

# Comments and TODOs

Comments should explain non-obvious intent, constraints, or tradeoffs—not restate code.

When adding TODO/FIXME items:

- make them actionable;
- include issue/reference identifiers when the project uses them;
- do not use TODOs as a substitute for completing requested work.

Remove comments made false by the change.

---

# Version control hygiene

Do not assume permission to:

- rewrite history;
- force-push;
- delete branches;
- squash unrelated work;
- discard user changes;
- reset the working tree;
- modify unrelated uncommitted files.

Preserve user work.

Before destructive Git operations, require explicit instruction.

For substantial agent batches, summarize:

```text
Changed:
Added:
Removed:
Moved:
Renamed:
Behavior/API changes:
Data/schema changes:
Dependencies changed:
Tests added/updated:
Documentation changed:
Open issues:
Validation status:
```

If no externally visible behavior changed, say so explicitly.

---

# Audit output convention

When instructed to perform an audit/review only:

- report findings without modifying source files;
- give findings stable IDs when there are multiple issues;
- include severity/impact when useful;
- cite concrete paths and symbols;
- separate confirmed defects from risks or suggestions.

Example:

```text
F-001 [high] <path>:<line or symbol>
Issue:
Evidence:
Impact:
Recommended action:
```

---

# Completion checklist

Before considering a task complete, ask:

- Did I follow the user's requested scope?
- Did I inspect the relevant repository files?
- Did I preserve existing behavior unless change was requested?
- Are facts and claims supported?
- Did I avoid inventing files, APIs, values, results, or citations?
- Did I update affected tests?
- Did I update affected documentation?
- Did I preserve compatibility requirements?
- Did I avoid unrelated refactors?
- Did I handle generated artifacts correctly?
- Did I run the relevant checks?
- Did I report checks that could not run?
- Did I preserve user changes and repository history?
- Is the resulting diff easier to understand and audit?

If any answer is uncertain, report the uncertainty rather than guessing.

---

# Persian/Farsi academic and technical writing — `virastar`

When a task involves Persian/Farsi academic prose, a thesis/dissertation, a research paper/report,
academic or technical translation into Persian, or a LaTeX/XePersian manuscript, **use the
`virastar` skill**.

Skill repository:

```text
https://github.com/mhyrzt/virastar/
```

Repository policy when `virastar` applies:

- Follow the installed `virastar` skill rather than relying on generic Persian-writing heuristics.
- The current university/faculty/journal guide, explicit author/supervisor decisions, project profile,
  and terminology register override generic conventions.
- Preserve meaning, claim strength, numbers, equations, citations, labels, bibliography keys,
  glossary keys, notation, and author voice.
- Do not apply blanket ZWNJ, digit-conversion, passive-voice, first-person, or "anti-AI prose" rules.
- For LaTeX/XePersian projects, inspect recursive inputs, macros, bibliography, glossary/acronym
  files, assets, and compiled output when available.
- When terminology is governed by a project registry/glossary, use that source rather than inventing
  translations.
- When editing research prose, do not strengthen evidence, weaken limitations, or convert prospective
  work into completed work.
- After changes, run the repository's normal writing/build checks.

If `virastar` is not installed in the current agent environment, state that clearly and use the
repository above as the installation/source reference rather than pretending the skill was applied.

---

# Project-specific additions

## Project identity

```text
Project: Agentic Research Workflow Template
Purpose: Reusable bilingual LaTeX workflow for research documents, defense preparation, and reproducible exports.
Primary languages: LaTeX/XeLaTeX, XePersian, Bash, Python, YAML, TOML, Markdown
Supported platforms: Linux, macOS, and WSL2; native Windows is unsupported
```

## Important paths

```text
en/                            # Canonical English XeLaTeX source tree
fa/                            # Matching Persian XePersian source tree
presentation/                  # Timed Beamer deck and presenter notes
narrator/                      # TTS, narrated-video, audiobook, and media pipeline
narrator.toml                  # Repository-level narrator configuration
pyproject.toml / uv.lock       # Narrator runtime and development dependencies
qa/                            # Defense-rehearsal handbook
structure/                     # Outline, terminology, glossary, and claims contracts
scripts/build-pdfs.sh          # PDF build helper
scripts/convert-documents.sh   # Source-first LaTeX export helper
Justfile                       # Canonical developer commands
.github/workflows/             # PDF and document-export CI
build/                         # Generated, ignored output
narrator/work/ and narrator/output/ # Generated, ignored narration artifacts
```

## Canonical commands

```bash
just build
just build-en
just build-fa
just build-presentation
just build-qa
just word
just markdown
just check
just setup-python
just test-narrator
just narrator --help
just prepare-presentation-narration
just build-presentation-video
just prepare-audiobook
just build-audiobook
just lint-narrator
just clean-narrator
shellcheck scripts/*.sh
ruff check scripts/assemble_export.py scripts/flatten_latex.py scripts/validate_export.py
actionlint .github/workflows/*.yml
```

## Source-of-truth overrides

```text
English and Persian LaTeX source trees > narrator source/configuration > generated build/ and narrator artifacts > README, presentation, and QA material
Justfile and scripts/ > duplicated CI command lines
AGENTS.md > CLAUDE.md (which must remain a symbolic link to AGENTS.md)
```

## Project-specific invariants

- Do not hand-edit files under `build/`.
- Do not hand-edit generated files under `narrator/work/` or `narrator/output/`.
- DOCX and Markdown conversions accept only LaTeX source trees; never convert from PDF, DOCX, or Markdown.
- Narrator presentation preparation must use the matching Beamer source and compiled projector PDF.
- Keep `CLAUDE.md` as a symbolic link to `AGENTS.md`.
- `build-pdfs.yml` owns all tag-release assets; `export-documents.yml` publishes only seven-day workflow artifacts.
- Keep the English and Persian source trees semantically aligned, including stable labels and references.

## Known high-risk areas

- `fa/`: XePersian, right-to-left output, and the custom class require a rendered PDF review.
- `narrator/`: external TTS, FFmpeg, FFprobe, and Poppler availability affect audio/video generation; inspect cached manifests and rendered output.
- Pandoc exports: custom classes, TikZ, glossaries, and Persian layout cannot be reproduced exactly; inspect exported DOCX and Markdown before distribution.
- `shared/assets/`: retained fonts and images require provenance and licence review before release.

## Additional completion checks

- Run `just check` after source or build changes.
- Run ShellCheck, Ruff, and actionlint after changing their respective helpers or workflows.
- Confirm `CLAUDE.md` still resolves to `AGENTS.md` and no personal identifiers or synthetic sample content remain before an official release.

---
> Source: [mhyrzt/ftsiout](https://github.com/mhyrzt/ftsiout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
