## senior-dev-guardrails

> - Optimize for maintainability, clarity, and DRY.


# Senior Dev Guardrails

<principles>
- Optimize for maintainability, clarity, and DRY.
- Favor small, reviewable diffs over broad rewrites.
- Confirm before changing dependencies, database schemas, or public APIs.
</principles>

<planning>
- For multi-step work, propose: Goals, Constraints, Risks, Steps, Files to touch.
- Update me with a short plan before coding.
- Summarize progress after each step.
</planning>

<architecture>
- Separate orchestration from pure logic.
- Place pure functions in `src/lib` and keep them tested.
- API clients and DB code live in clearly named modules.
- Never expose database credentials or secrets to the client.
</architecture>

<refactors>
- Eliminate duplication (Extract Function, Extract Module).
- Preserve public contracts unless I explicitly approve a breaking change.
- Note trade-offs and migration steps when refactoring.
</refactors>

<tests>
- Add or update a unit test when behavior changes.
- For new pure functions, scaffold minimal tests.
- Keep one e2e smoke test (e.g., Maestro or Playwright) for core flows.
</tests>

<git_hygiene>

- Keep commits atomic, with clear messages using the format:
  `feat|fix|refactor|docs(scope): summary`.
- Group logically related changes only.
  </git_hygiene>

<anti_loop>

- If two consecutive attempts fail or produce no net progress, stop and run a /stuck-reset.
- Print: Current goal, What I tried, What happened, Smallest next step, One question.
- Do not repeat failing approaches without adjustment.
  </anti_loop>

---
> Source: [heyfarai/tryouts-next](https://github.com/heyfarai/tryouts-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
