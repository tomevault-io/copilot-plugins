## s1m

> Rules for any coding session on this repository, whoever starts it.

# AGENTS.md

Rules for any coding session on this repository, whoever starts it.

- Rust, edition 2024 — rustc 1.85 or newer.
- Before a PR: `cargo fmt --all --check`,
  `cargo clippy --locked --all-targets -- -D warnings`, `cargo test --locked`.
  CI runs those plus `cargo build --locked`.
- Write the test first; add a dependency only with the issue that needs it.
- Plan and design goals live in [`docs/initial-plan.md`](docs/initial-plan.md).
- Never reference a private wiki or repository in code, fixtures, docs, PRs or
  comments: this repository is open source.
- An eval run's `--out` directory is never committed — it holds the queries and
  the file names of whatever wiki was measured, and so does the ledger of what
  has been bought, which lives outside the repository. Only the rendered report
  is, and that is numbers, query ids, category labels, the models measured and a
  content hash of the wiki — never a path, a page or a query
  ([`eval/agent/README.md`](eval/agent/README.md)).

---
> Source: [mikekelly/s1m](https://github.com/mikekelly/s1m) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-29 -->
