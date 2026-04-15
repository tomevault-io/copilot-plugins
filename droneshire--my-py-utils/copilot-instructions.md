## my-py-utils

> Run at the end of every long run action that is taken. In other words, after a full set of changes have been made or proposed and you would think of it as ready for review/commit.

# Action: Readying the code for committing

## When to use

Run at the end of every long run action that is taken. In other words, after a full set of changes have been made or proposed and you would think of it as ready for review/commit.

## What to do (step-by-step)

1. `cd` to the root directory
2. Ensure the pyenv is sourced: `source ./venv-dev/bin/activate` or `source ./venv/bin/activate`
3. Run `make format`
4. Run `make lint_full`
5. Run `make test`

## Constraints

- If any make steps fail, fix and re-run.
- Do not install any packages or attempt to run anything as su to fix anything
- You can run `make test TESTMODULE=<name_of_test_python_file>` to directly run a specific test vs all of them

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/droneshire) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
