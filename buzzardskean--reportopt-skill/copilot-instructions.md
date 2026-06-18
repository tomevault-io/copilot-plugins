## reportopt-skill

> Optimize Chinese experimental report text written in Typst. Use when Codex needs to revise a Typst lab report, thesis-style experiment report, or technical report for natural paragraph structure, chapter lead-in paragraphs, beginner-friendly academic tone, active voice, smooth logic, and correct figure/table cross-references while preserving Typst syntax, labels, citations, equations, code blocks, and document structure.


# Optimize Typst Lab Report Text

## Core Goal

Revise Chinese Typst experimental reports into clear academic prose for beginner readers. Improve wording, paragraph logic, chapter transitions, and figure/table references while preserving the original technical meaning and Typst document behavior.

## Typst Safety Rules

Preserve Typst syntax unless the user explicitly asks for structural editing. Do not break headings, labels, references, citations, formulas, code blocks, imports, includes, show rules, variables, or macros.

Keep labels such as `<fig-demo>`, `<tab-result>`, `<sec-method>`, references such as `@fig-demo`, `@tab-result`, `@sec-method`, citations, inline math, block math, tables, figures, and code exactly valid after editing. When changing nearby prose, integrate these references naturally in sentences, for example `图 @fig-system 展示了...` or `表 @tab-metrics 汇总了...`.

Do not translate Typst identifiers, filenames, label names, code identifiers, or package names. Avoid changing data in tables unless the user asks for data correction.

## Writing Rules

Use natural paragraphs rather than bullet lists when the content is explanatory, analytical, or summarizing. Preserve necessary enumerations only when they serve as procedure, parameters, or comparison items; convert checklist-like prose into complete paragraphs when it belongs to the report body.

Add a short lead-in paragraph under each chapter or major section heading when missing. This paragraph should summarize the chapter's purpose and connect it to the previous context without using stiff transition words. Keep it concise and substantive; it should not merely repeat the heading.

Use an academic but accessible style. Assume the reader is new to the field. Explain necessary concepts in plain language before using them repeatedly. Avoid overly specialized terms when a simpler equivalent is accurate, but keep required technical names that the report depends on.

Avoid metaphor, personification, promotional language, and casual phrasing. Do not use rigid connectors such as `首先`, `其次`, `再次`, `同时`, `最后`, `综上所述`, `总而言之`, or `旨在` unless the original meaning truly requires them and no smoother phrasing works.

Prefer active voice. Name the actor when it is clear, such as `本文采用...`, `实验选取...`, `模型输出...`, or `系统记录...`. Use passive-like wording only when the actor is unknown, irrelevant, or would make the sentence awkward.

Place adverbial clauses later in the sentence when possible. Prefer `模型在样本较少时容易出现波动` over front-loaded structures such as `当样本较少时，模型容易出现波动`, unless the fronted clause improves emphasis or readability.

## Logic Rules

Make each paragraph carry one clear point. Start with the paragraph's main idea, then add evidence, explanation, or consequence. Avoid paragraphs that only stack facts without explaining their relationship.

Build a visible chain across paragraphs. Use cause, condition, contrast, or progression through the content itself rather than relying on mechanical connectors. If one paragraph describes a problem, the next should explain its cause, method, result, or implication.

When discussing figures and tables, introduce what the reader should observe and then interpret why it matters. Do not leave images or tables floating without explanation. Refer to figures and tables by Typst cross-reference, not by vague wording such as `如下图` or `上表`.

Keep claims proportional to evidence. If the text reports an experiment, connect conclusions to observed results, metrics, or examples. Avoid broad claims that the experiment has not shown.

## Workflow

Read the relevant Typst file before editing. Identify headings, paragraph boundaries, figures, tables, labels, and existing cross-references. Note where chapter lead-in paragraphs are missing and where list-like text should become prose.

Revise in small, local passes. Preserve the author's structure unless the current structure prevents the report from meeting the writing rules. Keep technical content stable while improving sentence order, transitions, and paragraph logic.

After editing, scan the changed Typst for broken syntax and references. Check that headings still have the intended hierarchy, labels still exist, references still point to valid labels where possible, and no figure/table mention lost its cross-reference.

If the user asks for a review rather than direct editing, report concrete language issues with file and line references, then suggest representative rewrites. If the user asks to optimize the report, edit the file directly.

## Final Check

Before finishing, verify these points:

- The report body uses natural paragraphs instead of unnecessary lists.
- Each major chapter has a short lead-in paragraph.
- The tone is academic, clear, and beginner-friendly.
- Figure and table discussions use Typst cross-references.
- Mechanical transition words are removed or replaced.
- Sentences prefer active voice and natural Chinese word order.
- Paragraphs connect through cause, progression, comparison, or evidence.
- Typst syntax, labels, references, citations, formulas, and code remain valid.

---
> Source: [BuzzardsKean/ReportOpt-skill](https://github.com/BuzzardsKean/ReportOpt-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
