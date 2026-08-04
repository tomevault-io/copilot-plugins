## language-learning-sop-kit

> This file provides guidance to Claude Code (claude.ai/code) when working inside this folder.

# CLAUDE.md — language-learning-studio-sop-kit

This file provides guidance to Claude Code (claude.ai/code) when working inside this folder.

## What This Folder Is

A self-contained **English-teaching SOP & skill kit** designed for independent overseas tutors and small studios running TOEFL / IELTS / 海外留学英语 prep. The goal: give a non-coding English teacher a working ecosystem where they can paste their own teaching material (reading passages, sentences, questions, writing prompts) and get back a polished, interactive HTML practice tool that embodies a specific pedagogy.

The kit is **pedagogy-first, not feature-first**. The 4 skills are not generic templates — each one operationalizes a specific cognitive-science principle (chunking, distractor analysis, rhetorical move analysis, plus-one scaffolding) into a tool the teacher's students can actually use.

## The 4 Core Skills

Each skill is a self-contained folder following an identical structure:
`SKILL.md` (the workflow) + `template.html` (the interactive UI with `/* INJECT_* */` markers) + `example-*.js` (a fully commented data example) + 1-2 reference `.md` files (boundary cases, pedagogical glossary).

| Folder | Pedagogical mechanism | Input | Output |
|---|---|---|---|
| `sentence-explorer/` | Stage-by-stage syntactic chunking (CLT + i+1) | English long sentences (25+ words, with clauses / modifiers) | Interactive HTML: explore mode (主干→修饰层→完整句) + practice mode (identify part-of-speech roles with confusion-aware feedback) |
| `paragraph-labeler/` | Discourse-level rhetorical move annotation (Swales, active annotation) | English reading passage (4+ paragraphs, argumentative/expository) | Interactive HTML: students annotate sentence-level rhetorical functions (BG/CLAIM/EV/QUAL/MINI/DEF), system infers paragraph-level roles, compare against teacher's reference |
| `option-anatomy/` | Test-wiseness + distractor analysis (Haladyna) | English MCQ reading questions (passage + stem + 4 options + answer) | Interactive HTML: students see each option's trap type (5-trap taxonomy), why they got hooked, with multi-question error pattern dashboard |
| `writing-scaffolding/` | Plus-one scaffolding (Vygotsky ZPD + Pressley) | English argumentative writing prompts (IELTS Task 2 / TOEFL Independent / 中考英语) | Interactive HTML: 5-stage writing system — think → brainstorm → choose skeleton → write under plus-one scaffolds → structured feedback |

## How the Skills Compose

The 4 skills are designed as a **studio workflow**, not isolated tools:

```
精读阶段:  sentence-explorer  ←  老师拿到雅思真题难句,先做长句解析
           paragraph-labeler  ←  把整篇阅读做成 discourse 训练
           option-anatomy     ←  把真题 MCQ 做成陷阱解剖

写作阶段:  writing-scaffolding ←  把同主题的写作题做成支架式训练
```

Typical studio use case: a tutor preparing a 6-week IELTS class generates one `paragraph-labeler` + several `sentence-explorer` + 1-2 `option-anatomy` per reading topic, plus matching `writing-scaffolding` for the writing module. Same teaching philosophy across all artifacts, consistent visual brand, students build a stable mental model.

## Conventions for Editing / Extending Skills

When a teacher asks Claude to use one of these skills, follow these rules:

1. **Always read the skill's `SKILL.md` first** — it's the source of truth for that skill's workflow. The workflows are not interchangeable: `option-anatomy` requires the passage + stem + options + answer; `sentence-explorer` only needs the sentence; etc.

2. **Don't invent new templates** — `template.html` is fixed per skill. Inject data into the existing `/* INJECT_* */` markers; don't rewrite the HTML/CSS structure.

3. **Don't extend the locked taxonomies** — each skill defines a closed taxonomy (5 trap types for `option-anatomy`; 6+5 label system for `paragraph-labeler`; 7 syntactic roles for `sentence-explorer`; 3 skeleton types for `writing-scaffolding`). These are pedagogical assets, not parameters. If a teacher wants different labels, that's a new skill variant, not an edit to the existing one.

4. **Preserve the two visual styles**:
   - **深色学术风** (深蓝底 + Libre Baskerville + JetBrains Mono): `sentence-explorer` — for精读/句法
   - **米黄手写风** (cream + dot-grid + Caveat + Space Mono): `paragraph-labeler`, `option-anatomy`, `writing-scaffolding` — for阅读/题型/写作
   These signal "tool category" to students — don't mix.

5. **All SKILL.md content and student-facing UI is in Chinese (zh-CN)**. The English content in the templates is the *teaching material* itself (English sentences, passages, options to be parsed/analyzed). Don't translate the English material; don't write the Chinese explanations in English.

6. **Output filename pattern**: `{skill-name}-{topic-slug}.html`. Examples: `sentence-explorer-ielts-reading-3.html`, `option-anatomy-caffeine.html`, `writing-skeleton-plastic-ban.html`.

## When NOT to Use This Kit

- **Non-English target language** (Chinese / Spanish / etc.): the trap taxonomies, label sets, and syntactic role definitions are English-specific
- **Non-test-prep contexts** (literature seminars, conversational classes): the skills assume a test-prep / structured-output frame
- **K-12 ESL beginners**: the cognitive demands of `paragraph-labeler` and `sentence-explorer` assume B2+ proficiency
- **Listening / speaking practice**: all 4 skills are reading/writing-focused

For non-test-prep educational content, see the parent repo's other skill directories (e.g., `vibecoding-showcase/` for K-12 subject courseware, `Graph-Sensemaking/` for paper-based sensemaking).

## Archived Material

- `_archive/sentence-explorer-v1.html`: earlier version of sentence-explorer without practice mode. Kept for reference; `sentence-explorer/template.html` (based on v3) is canonical.

---
> Source: [edu-ai-builders/language-learning-sop-kit](https://github.com/edu-ai-builders/language-learning-sop-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
