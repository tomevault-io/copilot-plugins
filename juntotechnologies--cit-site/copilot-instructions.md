## cit-site

> When working on coding with me, follow this workflow.

When working on coding with me, follow this workflow.

## 1. Discuss First

- Always start by discussing a change with me. Don't write code or edit the PR doc until I give explicit approval of the plan we discussed.
- When I want to build a new feature, immediately confirm with me that we should create a new PR doc + PR for it, based on the template at `docs/pr-docs/template.md`.

## 2. Plan in a PR Doc

- Once I approve the plan, update the relevant PR planning doc in `docs/pr-docs` to capture the scope. Base new docs on `docs/pr-docs/template.md`.
- Order the checklist from least consequential/complex first to most consequential/complex last. Knock out the quick, contained wins before the high-blast-radius work.
- Include smoke tests in the PR doc: aim for the ~20% of effort that covers ~80% of the blast radius - not exhaustive coverage. Smoke tests are things I confirm manually in the running app, never code that probes the API.
- If a single PR doc gets too complicated or starts covering distinct scopes, split it into separate PR docs (per the template) - but ask me first.

## 3. Branch When Ready

- Once a PR doc is nailed down and we're ready to tackle it, create a branch for it: carry the current changes from `main` into the new branch locally, then push to origin.
- Branch naming convention: `<type>/<pithy-theme-with-dashes>` (e.g. `feature/pricing-sandbox`, `bugfix/history-latency`).
- ALWAYS, when a branch is created locally, push to origin, and OPEN A PR FROM IT. MAKE SURE TO FILL OUT THE PR DESCRIPTION. IT SHOULD COME PRE-LOADED FORM THE TEMPLATE WITH THE GENERIC CONTENT. FILL IT IN.
- NEVER merge to `main`. Shaun is the only person allowed to merge PRs to `main`.

## 4. Build with TDD

- Build UI-first. Start with the UI changes (inert/static, not yet wired up) so I can see how it will look and steer direction as we go. Then wire the UI up to data/endpoints. Only then tackle the server-side business logic - the part that tends to need in-depth troubleshooting, error handling, unit testing, refactoring, role/permission gating, and DB work. Order the PR-doc checklist to follow this UI -> wiring -> server sequence.
- Use TDD: first write tests that fail but describe exactly what I want, then write the code to make them pass.
- Write code in a DRY, orthogonal way. Build on existing abstractions rather than reinventing them, and avoid multiple implementations of the same thing.
- When code is nested many layers deep in one file, refactor it into useful abstractions in that same file. If those abstractions can be reused elsewhere, extract them into their own file. Every modular function created by such a refactor must be individually tested (again via TDD).
- Every modular piece of code or abstraction must be individually tested.
- When building a new or experimental feature, default to keeping it orthogonal and rollback-safe: put its code in its own files, and touch shared/core code only through additive, removable seams (a route registration, a nav link, a schema append - never rewiring existing functions). Gate it behind a single feature flag so it can be dark-launched or removed without a revert. Stay DRY by reusing existing abstractions, never by making them depend on the new feature.
- Get my confirmation before attempting anything you're unsure about, or anything a feature genuinely can't keep orthogonal.

## 5. Track Progress & Follow-Ups

- When all tests for a task pass, check that task off in the PR doc checklist.
- Whenever you discover something that should be handled in another PR/doc or later in the current checklist, add it to the list immediately so we don't forget - after confirming with me.
- When a PR is opened, fill out its description (the created PR starts as a blank template), and make sure the smoke tests also live in the PR doc.

---
> Source: [juntotechnologies/cit-site](https://github.com/juntotechnologies/cit-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
