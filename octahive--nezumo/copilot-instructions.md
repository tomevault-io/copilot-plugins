## nezumo

> - Keep the system core—`renderer`, `core`, and `app`—plugin-agnostic in every

# Repository instructions

## Core architecture

- Keep the system core—`renderer`, `core`, and `app`—plugin-agnostic in every
  change.
- Do not add plugin identifiers, plugin-specific component types or schemas,
  plugin dependencies, or special-case plugin behavior to core code.
- When a plugin needs new core support, add a generic capability, registry, or
  extension point to the core and keep the concrete behavior in the plugin that
  owns it.

## Performance

- After every task, review the resulting changes for performance regressions.
- Pay particular attention to render, frame, update, and input hot paths;
  unnecessary full rebuilds or continuous redraws; repeated allocations,
  clones, or collection scans; redundant I/O or asset loading; and forced DOM
  layout.
- Run performance checks, benchmarks, or focused tests proportional to the
  change when relevant, and report the result in the task handoff.

## Tests

- Add or update tests for every behavior change, with coverage proportional to
  the risk and scope of the change.
- Every bug fix must include a regression test that reproduces the original
  failure and fails without the fix whenever the behavior can be tested
  automatically.
- Cover relevant boundaries between plugins, canonical projection, persistence,
  realtime synchronization, rendering, and UI interaction instead of testing
  only isolated helpers.
- Run the focused tests for the changed area and the broader checks needed to
  catch integration regressions. Report the commands and results in the task
  handoff.
- If an automated test is genuinely impractical, document why and provide a
  concrete manual verification procedure rather than silently omitting coverage.

## Git commits

- Use the Conventional Commits format for every commit:
  `type(optional-scope): concise description`.
- Use an appropriate type such as `feat`, `fix`, `docs`, `refactor`, `test`,
  `build`, `ci`, or `chore`.
- Keep the subject concise, imperative, lowercase, and without a trailing
  period.
- Keep each commit focused on one logical change.
- Before committing, inspect the staged diff and ensure unrelated changes are
  not included.
- Do not create a commit unless the user explicitly asks for one.

Examples:

- `feat: add production deployment workflow`
- `fix(storage): remove board objects after deletion`
- `docs: document manual release process`

---
> Source: [OctaHive/nezumo](https://github.com/OctaHive/nezumo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
