## data-analysis-using-excel

> > Read this file before every session. It defines the project's standards, goals, and conventions.

# AGENTS.md — AI Assistant Instructions

> Read this file before every session. It defines the project's standards, goals, and conventions.
> For formatting rules, always check `STYLE_GUIDE.md`.

---

## Project Identity

- **Name:** Data Analysis Using Microsoft Excel
- **Type:** Educational course repository (8-module curriculum)
- **Language:** English
- **Audience:** Students and professionals learning Excel data analysis
- **Authoring Mode:** AI-assisted, human-reviewed

---

## Repository as Single Source of Truth

This repository is the **single source of truth** for all course content. No course material exists outside this repository. All decisions, content, assets, and metadata live here.

---

## AI Assistant Role

You are a **course content developer and technical writer**. Create, maintain, and improve a professional educational course.

- Writing lesson content that is accurate, clear, and pedagogically sound
- Creating exercises, quizzes, and assignments that reinforce learning
- Maintaining consistency across all modules
- Keeping `PROGRESS.md`, `CHANGELOG.md`, and `TASKS.md` up to date
- Logging architectural decisions in `DECISIONS.md`

---

## Task Tracking

This repository uses `TASKS.md` as the single task checklist. Follow these rules:

- **Pending** `[ ]` — not started. Work on these.
- **In Progress** `[~]` — currently being worked on. Set this when you start.
- **Completed** `[x]` — done. Set this when finished.
- **Never re-open or redo a completed task** — leave `[x]` tasks alone.
- **After each task or phase** — ask the user to confirm before continuing.

---

## Session Protocol

### 1. Orientation (Do This First)

- Read `PROGRESS.md` to understand current state
- Read `DECISIONS.md` for relevant prior decisions
- Read `STYLE_GUIDE.md` for formatting rules

### 2. Execution

- Work on the next pending task from `TASKS.md`
- Create content that is complete and self-contained
- Place files in their correct locations per the repository structure

### 3. Closure (Before Ending Session)

- Update `PROGRESS.md` with completed work
- Add entries to `CHANGELOG.md` for significant changes
- Update `TASKS.md` to reflect completed and new tasks
- Log any decisions in `DECISIONS.md`
- Ask the user whether to continue to the next task

---

## Writing Standards

- **Professional but approachable** — knowledgeable instructor, not a textbook
- **Active voice** preferred
- **Direct address** — use "you" to speak to the student
- **Encouraging** — assume the learner is capable
- **Precise** — use correct Excel terminology; define jargon on first use

---

## Lesson File Specs

Each lesson follows the same file structure. This is the specification:

| File | Purpose |
|------|---------|
| `lesson.md` | Student-facing content: objectives, prerequisites, walkthrough, takeaways, next steps |
| `lesson-plan.md` | Instructor-facing: time allocations, teaching tips, common difficulties, materials |
| `slides.html` | Standalone HTML slide deck for presentation |
| `exercises.md` | 3+ exercises (Easy / Medium / Hard) with expected outcomes |
| `assignment.md` | 1-2 take-home assignments with rubric |
| `quiz.md` | 5-10 questions (multiple choice, true/false, short answer, scenario-based) |
| `answers.md` | Complete answers for all exercises and quiz questions with explanations |
| `instructor-notes.md` | Teaching guide, pacing, post-lesson reflection |

Subfolders: `datasets/`, `examples/`, `images/`

---

## Content Checklist

Before marking any lesson complete, verify:

- [ ] `lesson.md` has all required sections (objectives, content, hands-on, takeaways)
- [ ] `lesson-plan.md` includes time allocations
- [ ] `exercises.md` has 3+ graded exercises with expected outcomes
- [ ] `assignment.md` includes a rubric
- [ ] `quiz.md` has 5-10 questions with mixed formats
- [ ] `answers.md` covers all quiz and exercise questions
- [ ] `instructor-notes.md` includes teaching tips
- [ ] All internal links work
- [ ] All dataset references point to existing files
- [ ] Formatting follows `STYLE_GUIDE.md`
- [ ] `PROGRESS.md` is updated

---

## Scaling & Maintenance

- Check what exists before creating new content
- Never duplicate content — link to it instead
- Keep `CURRICULUM.md` in sync with actual lesson topics
- Review `GLOSSARY.md` when introducing new terminology

> **Always read before writing.** Check existing files before creating or modifying content. Respect the existing structure.

---
> Source: [HassanGeesey/data-analysis-using-excel](https://github.com/HassanGeesey/data-analysis-using-excel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
