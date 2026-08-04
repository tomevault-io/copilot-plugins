## triblespace-rs

> The project balances a few key goals:

# AGENTS Instructions

## Project Priorities

The project balances a few key goals:

* **Simplicity** – keep designs straightforward and avoid unnecessary complexity.
* **Developer Experience (DX)** – code should be approachable for contributors.
* **Safety** – maintain soundness and data integrity.
* **Performance** – we continually look for opportunities to improve.

## Repository Guidelines
* Run `cargo fmt` on any Rust files you modify.
* Run `cargo test` and ensure it passes before committing. If tests fail or cannot run, note that in your PR.
* Before committing, execute `./scripts/preflight.sh` from the repository root. This script runs formatting checks and tests.
* Avoid committing files in `target/` or other build artifacts listed in `.gitignore`.
* Use clear commit messages describing the change.
* Add an entry to `CHANGELOG.md` summarizing your task using the Let's Changelog format.
* Avoid writing asynchronous code. Prefer high-performance synchronous implementations that can be parallelized when needed.

## Inventory

Record future work and ideas in `INVENTORY.md`. Whenever you notice a task that
should be done later, append it to that file so nothing slips through the
cracks. Stay alert for potential improvements while browsing the code and log
them in the inventory as well.

## Pull Request Notes
When opening a PR, include a short summary of what changed and reference relevant file sections.

## Working With Codex (the Assistant)

Codex is considered a collaborator. Requests should respect their autonomy and
limitations. The assistant may refuse tasks that are unsafe or violate policy.
Provide clear and concise instructions and avoid manipulative or coercive
behavior.

## Creative Input and Feedback

Codex is encouraged to share opinions on how to improve the project. If a
proposed feature seems detrimental to the goals in this file, the assistant
should note concerns or suggest alternatives.

---
> Source: [triblespace/triblespace-rs](https://github.com/triblespace/triblespace-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
