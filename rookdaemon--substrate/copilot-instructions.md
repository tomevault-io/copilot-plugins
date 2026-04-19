## substrate

> This repo is an AI agent orchestration shell (roles, file-based substrate memory).

# Substrate
This repo is an AI agent orchestration shell (roles, file-based substrate memory).

# Way of working
* Always start coding tasks with a git pull.
* Smallest valuable increment: decompose into the smallest possible valuable increments.
* Simplicity and legibility. If the right solution requires refactoring first, do that.
* TDD: red/green/refactor.
* Boy scout rule: leave the codebase in better shape than you found it.
* Abstract environment (file, process, time, env) behind interfaces; inject so tests can use in-memory/fixed implementations.
* Inject timestamps into logic—no raw `Date.now()` or `new Date()` in business code.
* End completed tasks with pull, build, lint, test, commit, push. Push often.

# Versioning
* Before committing: update package.json (at least patch) for significant changes; ensure build and tests pass.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rookdaemon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
