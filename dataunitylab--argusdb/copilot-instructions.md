## argusdb

> - After cloning the repo, make sure that `prek install` has been run if `.git/hooks/pre-commit` does not exist

# Project: ArgusDB - A JSON Document-oriented Database using smart compression and log-structured merge trees

## General Instructions

- After cloning the repo, make sure that `prek install` has been run if `.git/hooks/pre-commit` does not exist
- Always start by examining the current status of the project in @ROADMAP.md
- Then implement the next checklist item in the roadmap, checking off completed items
- Items can be added or removed to the roadmap as necessary
- Items previously completed on the roadmap can be revisited as necessary to fix bugs or add missing features
- Specifications of different components can be found in the `specs` directory
- Make a detailed plan before writing any code, refining the existing specifications if needed
- Ask questions if any part of the specification is unclear
- Write small amounts of well-tested code at a time and  commit after each change
- Whenever a configuration option is needed or removed, make sure to update CONFIGURATION.md

## Coding style

- Follow the default Rust style and use `cargo fmt` to reformat code before committing
- Avoid the use of `unsafe` code where possible
- Add comments to every function added and ensure comments are up-to-date with any changed code
- Use short, simple documentation tests where possible
- Always add tests for new features and ensure all tests pass before committing code
- Create small logical commits for every change to the project
- Each commit should have a clear and concise description and tests that run successfully (follow the [conventional commits](https://www.conventionalcommits.org/en/v1.0.0/) guidelines)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dataunitylab) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
