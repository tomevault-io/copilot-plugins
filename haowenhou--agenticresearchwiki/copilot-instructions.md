## agenticresearchwiki

> This repo is a documentation repo that records everything about this research project: background, goals, methods, experiments, paths, data, etc.

## Reading docs
This repo is a documentation repo that records everything about this research project: background, goals, methods, experiments, paths, data, etc.
Links between docs use wiki-style `[[ ]]` syntax.
When you need to answer questions, make changes, or update project docs for this research project, first gather the necessary information from this wiki.

## Reading order
1. [[Overview]] — project background, goals, methods, task navigation, experimental side tracks
2. Pick based on task type:
   - Data task -> [[DataOverview]]
   - Training task -> [[TrainingOverview]]
   - Eval task -> [[EvalOverview]]
Read the docs selectively and hierarchically based on my task.

## Do NOT read by default
- Historical docs under `**/Archived/`
- Abandoned track docs under `ExperimentalTracks/abandoned/`, unless the task explicitly asks to "review / post-mortem an abandoned plan"
- Long experiment logs, unless the current task explicitly requires tracing back
- `ResearchNotes/` holds background research, comparison, and notes-style docs. These are primarily for human understanding and decision-making and are NOT default task entry points. Only when the task explicitly asks for "research / comparison / reviewing the decision rationale", enter via [[ResearchNotesOverview]] and read on demand.

## Template placeholders
`{{...}}` in docs are unfilled template fields (the text inside the braces is a fill hint). **Do not** treat them as real content or base any judgment on them. When you see one:
- If the task needs the real value of that field: ask the user first, then fill it in and remove the braces after they reply;
- If the task is to fill in the doc: generate content following the hint inside the braces, replacing the entire `{{...}}`;
- When writing records back / tidying docs: do not keep or add new `{{...}}`, unless it's deliberately left as a placeholder for the user.

## Doc-update rules
Invoke the corresponding doc-update skill.

---
> Source: [HaowenHou/AgenticResearchWiki](https://github.com/HaowenHou/AgenticResearchWiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
