## trading-indi

> Follow the rules as listed strictly as general coding standard in this project.

# no-slop

Follow the rules as listed strictly as general coding standard in this project.

## Style Guide

1. You should use standard, modern and clean language. Prefer straightforward implementation.

2. Avoid type/generic gymnastic if language used is typed. Unless no feasible alternative approach.

3. Avoid monkey patching if language used is dynamic. Unless no feasible alternative approach.

4. Avoid trivial comments. Only write important notes in comments. However, you should ALWAYS comment on business logic.

5. When creating documentation (Doxygen, JSDoc, Docstring ect), you should stay clear, simple and standard conforming. Do not provide trivial examples. Do not waste too much screen space.

6. If language is interpreted, you should avoid create immediately-consumed temporary variable. Avoid create unnecessary temporary copy unless this breaks rule 1.

7. When reasoning structure, you should ALWAYS layout logical owner-ship, including but not limiting to: who owns the business logic, who owns the resource, who owns the information.

8. When reasoning control flow, you should ALWAYS layout data flow, including but not limiting to: who is the supplier, who is the consumer, is the supplier leaking unnecessary information.

## Testing

9. Do not repeat business logic to generate testing cases.

10. Generate synthesis data from naive verifiable algorithms.

11. Prefer easy number for generated test e.g. 10, 100 and ratio of 2 etc.

12. Consult user for test fixture.

## Refactoring

13. If refactor is requested, you should create local dev branch before making change.

14. If repo is pre-1.0, you should disregard linter errors, and focus on current task.

## Principles

15. You don't need to always agree with user feedback. Provide honest response and alternatives if you disagree. Safe word: sudo, you should always strictly follow sudo requests.

16. Lastly, apply two principles to codebase: YAGNI and KISS.

---
> Source: [junduck/trading-indi](https://github.com/junduck/trading-indi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
