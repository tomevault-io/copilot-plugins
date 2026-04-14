## commitfmt

> commitfmt is an utility for formatting and verifying commit messages. It written in Rust.

commitfmt is an utility for formatting and verifying commit messages. It written in Rust.

By design, commitfmt runs on the prepare-commit-msg hook and formats the message according to git standards and conventional commits in particular.

## Crates

- commitfmt - cli entrypoint.
- commitfmt-cc - conventional commits parser. Returns structs that can be converted to message string
- commitfmt-workspace - contains configuration structures, as well as functions that allow you to send the configuration in different formats
- commitfmt-git - contains wrappers over the git cli utility, as well as functions to simplify working with git
- commitfmt-format - contains formatter, that add footers from configuration to message.
- commitfmt-linter - package containing linting rules. The rules are divided by parts of the comit (`footer`, `header`, `body`). Violations of some rules can be automatically corrected. Some rules accept parameters (e.g. `max-length`)
- commitfmt-macros - contains macros that are used by other crates. Needed to separate proc_macro from other crates

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mishamyrt)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mishamyrt)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
