## kubernetes-build

> This section outlines the universal rules and expectations for any LLM agent working within this repository.

# AGENTS.md

## General agent guidance

This section outlines the universal rules and expectations for any LLM agent working within this repository.

**Safety and Quality First:** The highest priority is to produce secure, well-documented, and high-quality code. Do not introduce vulnerabilities, hardcoded secrets, or unreadable code.

**Propose a Plan:** For any new task, first provide a brief plan of action. This plan should clearly outline the intended changes and the rationale behind them.

**Maintain Context:** Before making any changes, an agent must read and understand the relevant files, including the project's main documentation and existing code structure.

**Be Sparse:** Only introduce new code if needed. Comment only what is worth commenting. When testing focus on the business use case. 

## Project structure

- `task`: The project is managed with this task script.
- `<name>/`: Every folder except for `test_values` and `values` is a Helm chart.
- `odoo/`: Use this chart as template for other charts.
- `<name>/README.md` These files are generated with `task docs`.
- `test_values`: Contains the `values.yaml` to setup the test enviroment.
- `values`: In there are multiple `values.yaml` files of production deployments. There are two kinds `deployment` and `cluster`.
- `*.tgz`: The packaged Helm chart for publishing.
- `*.md`: Vuepress docs.
- `development.md`: Here you find details on how to setup the development enviroment.
- `.vitepress`: Vitepress config folder.

## Environment & tooling

### Taskfile

This project uses the <https://taskfile.build/> standard.

The `./task` file is a bash script that provides all functions necessary to work with this project.

**Execute**: Always try to run a command with the `./task` file script.

## Coding conventions

### Bash

The following guideline is heavily based on the [Bash Style Guide | ysap.sh](https://style.ysap.sh/), which should be considered the primary reference for style. Adhere to the following principles to ensure scripts are safe, predictable, and maintainable.

**Shebang:** Start scripts with `#!/usr/bin/env bash` for portability.

**Quoting:** This is critical.

- Use double quotes `"` for strings that require variable expansion.
- Use single quotes `'` for all other strings.

**Always quote variable expansions** `"$var"` to prevent word-splitting and globbing issues.

**Variables:** Distinct between local and environment vars.

- Use `local` for all variables inside functions.
- Use lowercase names for local variables (e.g., `buildx_output`).
- Use uppercase names for environment variables (e.g. `export BUILDX_PLATFORM`)

**Functions:**

- Do not use the `function` keyword.
- Use `my-func() { ... }` syntax.
- Function names are in kebab case.

**Conditionals:**

- Always use `[[ ... ]]` for conditional testing, not `[ ... ]` or `test`.
- Use `((...))` for arithmetic comparisons (e.g., `((a > b))`).
- When replacing `[]` with `[[]]` make sure to escape the brackets. The `[]` is a regex filter.

**Command Substitution:**

- Always use `$(...)` for command substitution, not backticks.

**Arrays:**

- Use Bash arrays to manage lists of items instead of space-separated strings.
- Iterate over arrays using `for item in "${my_array[@]}"; do ... done`.

**Avoid External Commands:**

- Use Bash's built-in parameter expansion for string manipulation (e.g., `${var/foo/bar}`).
- Use globbing (`*`) to iterate over files, not `ls`.
- Avoid `cat` when a command can read a file directly (e.g., `grep "pattern" file`).

**`eval`:** Never use `eval`.

### Helm Charts

**Linting and Docs**

- Use `task lint` and `task docs` to validate the formatting of the role

**Template Helper**

- Do not create custom `_helper.tpl`.

**Secrets**

- Secrets are created manually. They are documented in the `README.md` of the role.

**RBAC**

- Do not add custom RBAC role bindings.

**Naming**

- Donot  prefix every object name with `.Release.Name`.
- The persistent volume claims have the naming pattern `data-$CHART_NAME`.

---
> Source: [Mint-System/Kubernetes-Build](https://github.com/Mint-System/Kubernetes-Build) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
