## opensearch

> Use OpenSearch for broad, evidence-backed investigation across session history, the web, and public code when a user asks to search, research, compare sources, or gather official docs and examples.


# OpenSearch Skill

Use this skill when the task is primarily about finding, comparing, and synthesizing evidence rather than editing code immediately.

## When to Use It

- The user says `search`, `look up`, `research`, `investigate`, or `find examples`
- The answer should combine project context, official docs, and public code patterns
- The task needs contradictory-source checking before implementation
- The user wants official references, best practices, or GitHub examples

## When Not to Use It

- A local `read`, `grep`, or `glob` is enough to answer the question
- The task is a straightforward edit in a known file
- The user explicitly wants a pure code change with no research step

## Default Workflow

1. Call `opensearch` first for broad or ambiguous research questions.
2. Prefer `depth: "thorough"` for broad or ambiguous research; keep `quick` for focused lookups.
3. Use all sources when external context matters; narrow sources only when the user asks for repo-local or session-local evidence.
4. Summarize what the tool found, highlight contradictions or gaps, then continue with implementation or recommendation.

## Query Patterns

- Official docs and examples:
  - `React Server Components caching behavior official docs GitHub examples`
- Repo plus external evidence:
  - `OpenCode plugin loading order project behavior official docs`
- Contradiction check:
  - `semantic-release trusted publishing npm GitHub Actions official guidance 2026`

## Source Selection Guide

- `session` for prior decisions, earlier runs, and local conversation history (OpenCode only)
- `web` for official docs, changelogs, and current vendor guidance
- `code` for public implementation examples and real usage patterns

## Output Expectations

- Return the strongest evidence first
- Call out missing or unavailable sources explicitly
- Keep citations grounded in the tool response instead of guessing

---
> Source: [kagan-sh/opensearch](https://github.com/kagan-sh/opensearch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
