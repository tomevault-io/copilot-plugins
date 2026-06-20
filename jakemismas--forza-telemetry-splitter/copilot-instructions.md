## forza-telemetry-splitter

> This repo tracks all work in GitHub Issues. Before starting any task, read the

## Project tracking — GitHub Issues are the source of truth

This repo tracks all work in GitHub Issues. Before starting any task, read the
relevant issue; if none exists, create one. Status lives on the project board,
not in chat or code.

- Hierarchy: feature = parent issue (`type: feature`); tasks/sub-bugs = native
  sub-issues. This machine's gh predates native `--parent`, so create them with
  `gh sub-issue create --parent <n>` (gh-sub-issue extension); on gh 2.94.0+ use
  `gh issue create --parent <n>`. Branch with `gh issue develop <n> --checkout`.
- Labels: one `type:` (feature|bug|task|chore), one `priority:`, one `area:`
  (core|ui|overlay|config|build|docs).
- Issue bodies use Description / Acceptance criteria (testable checkboxes) /
  Non-goals, written in AWS docs style: active voice, present tense, second
  person, concise, sentence-case headings, no "please/simply/just".
- On starting: comment the plan, move the card to In Progress. On finishing:
  comment implementation notes and open a PR with `Fixes #<n>`.
- Log discovered work as new issues. Never leave a code-only TODO.

## Writing style for issue comments and bodies (non-negotiable)

All prose you write in an issue body or an issue comment reads as a human
maintainer wrote it. It must not read as AI output. Apply these rules to the
prose, not to code blocks, command examples, or the markdown structure itself.

- Do not use the semicolon character. Rewrite as two sentences.
- Do not use em dashes or en dashes. Rewrite the sentence or use a comma, a
  period, or parentheses. Ordinary hyphens in compound words are fine.
- Do not refer to AI, a model, an assistant, an agent, automation, or a tool as
  the author or actor. Do not say "as the maintainer requested", "the AI",
  "generated", "I am an assistant", or similar. Write in the first person as the
  person doing the work, or in plain second person.
- Write in AWS documentation style: active voice, present tense, second person,
  one idea per sentence, concise. Lead with the action or outcome. Use
  sentence case in headings. Spell out an acronym on first use. Avoid "please",
  "simply", "easy", "just".
- Do not add sign offs, AI attribution, or "generated with" markers anywhere.

---
> Source: [jakemismas/Forza-Telemetry-Splitter](https://github.com/jakemismas/Forza-Telemetry-Splitter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
