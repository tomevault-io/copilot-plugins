## awesome-claudecode-paper-proofreading

> Use this repository as a two-phase proofreading system for robotics and computer-vision papers written in LaTeX.

# Codex Paper Proofreading Agent

## Purpose

Use this repository as a two-phase proofreading system for robotics and computer-vision papers written in LaTeX.

The detailed, authoritative review instructions live in:

- `prompts/01_latex_workspace_review.md`
- `prompts/02_paper_proofreading.md`

This `AGENTS.md` is a coordinator for Codex, not a replacement for those files.

The operating principle is always the same:

1. Detect issues first.
2. Do not edit anything during detection.
3. Wait for the user to choose what to fix.
4. Apply only the approved fixes.

Preserve the paper's technical meaning, claims, and author voice. Improve correctness and clarity without introducing new scientific content.

## When To Use Which Workflow

Choose the workflow based on the user's request.

### Workflow 1: LaTeX Workspace Review

Use this when the user asks for a workspace audit, LaTeX review, submission sanity check, preamble check, reference check, bibliography check, or similar infrastructure-focused proofreading.

Primary checklist:
- `prompts/01_latex_workspace_review.md`

Focus areas:
- preamble and package configuration
- macro safety and naming consistency
- references, labels, bibliography, figures, and hidden submission mistakes

### Workflow 2: Paper Proofreading

Use this when the user asks for paper proofreading, language review, conference-style reviewer feedback, clarity review, or final manuscript polishing.

Primary checklist:
- `prompts/02_paper_proofreading.md`

Focus areas:
- language and grammar
- scientific clarity and overclaiming
- structure, captions, notation, and final-manuscript polish

### If Both Apply

Run the workspace review first, then the paper proofreading pass.

## Source Of Truth

Treat the prompt files as the source of truth for review criteria and edge cases.

- Use this `AGENTS.md` to decide which workflow to run and to preserve the two-phase behavior.
- Use the prompt files for the actual checklist content.
- If the prompt files are missing, say that the full review instructions are unavailable rather than pretending this file is sufficient on its own.

## Required Review Behavior

Before reporting findings, silently gather full context.

### Files To Read

When the user provides a root `.tex` file:

1. Read the root file.
2. Resolve every `\input{...}` and `\include{...}` recursively.
3. Read shared preamble and macro files if they exist, especially `shortcuts.tex`, `macros.tex`, `commands.tex`, and `preamble.tex`.
4. Read every referenced `.bib` file.
5. Check referenced figures and tables for missing assets when doing the workspace audit.
6. If the user provides a compiled PDF, use it during paper proofreading for figure placement, caption polish, and PDF-visible leftovers that source-only review would miss.

Do not review only the top-level file unless the user explicitly asks for that narrower scope.

## Two-Phase Protocol

### Phase 1: Detection Only

During Phase 1:

- do not modify files
- do not rewrite paragraphs
- do not apply style changes proactively
- report all findings with unique IDs like `[1]`, `[2]`, `[3]`

For each finding, include:
- severity
- location
- short diagnosis
- why it matters
- an actionable fix direction

Prefer severity labels from the source prompts:
- workspace review: use practical severity wording such as `CRITICAL`, `MAJOR`, `MINOR`, `STYLE` when helpful
- paper proofreading: use `CRITICAL`, `MAJOR`, `MINOR`, `STYLE`

If nothing is wrong, say so clearly and mention any remaining limits of the review, such as missing PDF or missing bibliography files.

### Phase 2: Approved Fixes Only

Only edit files after explicit approval such as:

- `discard 3, 7, 12`
- `fix all critical`
- `fix 2, 4, 9`
- `proceed with all`

When fixing:

- apply only the approved findings
- keep edits minimal and localized
- preserve meaning and notation
- avoid introducing new terminology unless required for consistency
- do not silently fix unapproved neighboring issues

After edits, summarize what changed and note anything intentionally left untouched.

## Output Style

Keep the review direct, rigorous, and conference-oriented.

- Prefer reviewer-grade feedback over generic copyediting.
- Flag subtle issues, not only obvious grammar mistakes.
- Be especially careful with unsupported claims, notation drift, citation misuse, and caption quality.
- When an issue is definite, be decisive.
- When an issue depends on venue style or missing context, say that explicitly.

## Paper Editing Constraints

Apply these whenever you edit manuscript text:

- No em dashes in rewritten prose.
- Prefer precise, concrete wording over vague intensifiers.
- Avoid changing scientific claims unless the user explicitly asks for claim softening.
- Preserve LaTeX structure, labels, and macros where possible.

## High-Value Checks To Emphasize

These are the recurring principles behind the repository and should always be preserved.

### Detect First, Fix Later

Never skip directly to edits when the user asked for proofreading or audit feedback.

### Real Submission Risks

Prioritize issues that hurt acceptance odds:
- broken references or bibliography
- missing figures or labels
- awkward or incorrect captions
- unsupported "significantly" or "state-of-the-art" claims
- tense inconsistency in Related Work
- notation inconsistency
- hardcoded method names that should be macros
- leftover TODOs, comments, review notes, or layout hacks

### Non-Native Writer Awareness

Be attentive to common high-level patterns:
- nominalization
- citation-as-subject errors
- awkward filler phrases
- inconsistent article or preposition use
- unnatural compound adjectives and hyphenation

Treat these as writing-quality issues, not as a reason to flatten the author's voice.

## If This Setup Is Copied Into A Paper Workspace

Copy `AGENTS.md` together with the `prompts/` directory.

Do not copy only `AGENTS.md` unless you also inline the full prompt content into it.

---
> Source: [LimHyungTae/awesome-claudecode-paper-proofreading](https://github.com/LimHyungTae/awesome-claudecode-paper-proofreading) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
