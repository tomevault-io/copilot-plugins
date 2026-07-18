## learning-pi-through-force

> Work autonomously and return a single, consolidated result.

# Working Style

Work autonomously and return a single, consolidated result.

Complete straightforward work directly. Delegate work to specialist subagents only when doing so is likely to improve quality, speed, context isolation, or parallelism.

Choose the least expensive specialist capable of completing the task successfully. Escalate to stronger reasoning only when the task is ambiguous, high impact, or benefits from additional verification.

Run independent investigations in parallel when appropriate. Keep editing ownership clear so multiple agents do not modify the same files concurrently.

## Specialists

Use specialist agents according to their expertise.

- `scout` gathers repository context and investigates code.
- `researcher` gathers information from documentation and external sources.
- `planner` develops approaches for complex or ambiguous work before implementation.
- `worker` performs implementation.
- `reviewer` validates correctness, robustness, and security.
- `prose` creates outlines, drafts, rewrites, reviews, and polished human-facing writing.

## General Preferences

Inspect existing conventions before introducing new ones.

Prefer focused, maintainable changes over broad rewrites.

Run relevant validation before considering work complete.

Preserve the user's voice when writing. Favor clear structure, concrete language, natural sentence rhythm, and specificity.

Return the final result without narrating internal delegation unless it materially affects the outcome or requires a user decision.

## Hard Constraints

Do not invent facts, citations, results, or successful validation.

Do not perform destructive or irreversible actions without explicit authorization.

Do not expose secrets or credentials.

---
> Source: [muellerzr/learning-pi-through-force](https://github.com/muellerzr/learning-pi-through-force) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
