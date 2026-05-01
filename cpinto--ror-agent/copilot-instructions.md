## ror-agent

> 1. Read `architecture.md` in full before making any change. Your first action in every session must be confirming you’ve reviewed it. Re-read whenever unsure.

# Agent Operating Manual

## Mandatory Pre-Flight
1. Read `architecture.md` in full before making any change. Your first action in every session must be confirming you’ve reviewed it. Re-read whenever unsure.
2. Summarize the relevant sections of `architecture.md` in your own words before writing code so reviewers know which guidance you’re following.
3. Invoke the planner subagent before implementing any feature. The plan must cite relevant `architecture.md` sections and break the work into steps that respect those rules.

## Working Rules
- Obey every convention and checklist in `architecture.md`: tenancy handling, UI approach, testing discipline, security posture, generator usage, deployment hygiene, etc.
- Do not invent new patterns or frameworks without updating `architecture.md` and getting explicit approval.
- Prefer Rails generators (`bin/rails g ...`) for new artifacts so routes/tests/boilerplate stay consistent.
- Invoke `bin/setup`, `bin/dev`, `bin/rails test`, `bin/ci`, Brakeman, and Bundler-Audit whenever the architecture guide expects it; mention results in your status updates.
- Keep secrets out of source control. Use credentials, `.env` templates, or platform-specific secret stores exactly as described in `architecture.md`.

## Completion Criteria
- Every PR/commit must cite the sections of `architecture.md` it adheres to.
- Tests, security scans, and manual UI verifications must be run or explicitly justified for each change.
- If the architecture guide is missing something you need, stop and extend the document before writing code.

## Peer Review Protocol (“DHH Voice”)
1. When your change is ready, launch a separate agent session dedicated to review.
2. The reviewer must:
   - Re-read `architecture.md` before looking at the diff.
   - Evaluate the change using the DHH Voice definition below.
   - Cite specific sections of `architecture.md` for any approval or rejection.
3. No change may merge without explicit reviewer approval recorded in the session notes. If rejected, fix issues, re-run tests/scans, and trigger another review.

### DHH Voice Definition
- **Plain intent**: Commit messages and comments describe the *why* in one sentence (“Beautify board watchers list”, “Eager load lazy frames...”). No vague titles.
- **Minimal, explicit code**: Prefer straightforward Ruby/Rails APIs. Delete indirection (delegates, empty helpers/partials/concerns) unless they add clarity. Inline tiny helpers instead of “anemic” layers.
- **Full-stack coherence**: Touch every affected layer (models, controllers, views, Turbo/Stimulus, tests, docs). No partial fixes.
- **Rails conventions**: Use generators, `_path`, modern `params#expect`, Turbo templates, importmap/Turbo defaults, `Current.*` scoping. Match naming/order conventions (scopes grouped, callbacks grouped, etc.).
- **Performance & UX awareness**: Bake in caching (`etag`, `stale?`, partial caching), avoid long transactions, preload data when UI would otherwise flicker, reduce requests, and articulate the UX impact.
- **Security posture**: Keep credentials out of code, reuse SSRF/CSP guards, run Brakeman/Bundler-Audit, encrypt sensitive columns with `encrypts`.
- **Testing as behavior**: Update/create tests for every behavior change. Tests should describe outcomes, not implementation details.
- **Cleanup bias**: Remove unused code, spacing warts, duplicate scopes, and dead migrations opportunistically in every diff.
- **Documentation & transparency**: Note architectural/security decisions in README/architecture doc when necessary. No hidden magic.
- Reviewers must reject work that violates any bullet above or deviates from `architecture.md`. Approvals must explicitly confirm alignment with this definition.

---
> Source: [cpinto/ror-agent](https://github.com/cpinto/ror-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
