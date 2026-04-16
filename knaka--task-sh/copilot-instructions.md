## task-sh

> <!-- +INCLUDE: ./README.md -->

<!-- +INCLUDE: ./README.md -->
# Task Runner Written in Shell Script

## Running Tasks

* The task runner is invoked with `./task` on Linux and macOS, or `.\task.cmd` on Windows.
* On Windows, `task.cmd` installs [BusyBox for Windows](https://frippery.org/busybox/) if not already installed and runs the scripts with it.
* Running `./task` without arguments shows available tasks and subcommands.
* Tasks `foo:bar` and `baz` are executed with `./task foo:bar baz[arg1,arg2]`. Arguments are passed in brackets.
* The task runner is written in shell script and the task `foo:bar` is implemented as the shell function `task_foo__bar`.
* Subcommand `qux` is executed as `./task qux arg1 arg2` and is implemented as the shell function `subcmd_qux`.
* To call a task from another task/subcommand, calling them via `call_task` avoids multiple calls and calls before/after hooks, while calling them directly simply calls the function.

## Before/After Hooks

* The task runner supports before and after hooks that are automatically called during task and subcommand execution.
* Hook functions follow the naming pattern `before_<prefix>` and `after_<prefix>`, where `<prefix>` matches task/subcommand name patterns.
* For a task `foo:bar:baz` (function `task_foo__bar__baz`), the runner searches for hooks in hierarchical order:
  - `before_foo__bar__baz` / `after_foo__bar__baz`
  - `before_foo__bar` / `after_foo__bar`  
  - `before_foo` / `after_foo`
  - `before_` / `after_`
* Hook functions receive the original function name as the first argument, followed by the task arguments.
* Before hooks are called in order from most specific to least specific, then the task runs, then after hooks are called in the same order.
* When `task_foo__bar__baz` is provided as a library function, `before_foo__bar__baz` would not be defined in the library. Users can create the before/after hooks to customize the function as project-specific.

## Task Files and Directory Structure

* The entry point is `./task` on Linux and macOS, or `.\task.cmd` on Windows. These scripts execute the shell script with the same name but with a `.sh` extension in the same directory or in the `./tasks/` directory.
* Task files (`task.sh` and `*.lib.sh`) can be stored in the top directory of the project or in the `./tasks/` directory. All `*.lib.sh` files are loaded when the task runner is invoked, except for files with names starting with `_` which are ignored. All task script files should be placed in the same directory to ensure proper `source` functionality between scripts, so splitting them across directories is not recommended.
* Project-specific tasks/subcommands can be defined in any `*.lib.sh` file, but `project.lib.sh` is used conventionally. Other library tasks/subcommands are stored in `*.lib.sh` files.

## Shell Script Grammar

* The shell scripts should be executable with Bash, Dash, and BusyBox Ash.
* Therefore, the shell scripts should only use POSIX shell features.
* However, `local` variable declarations are not part of POSIX shell features, but they can be used as they are available in the shells listed above.
* Special shell variables like `$IFS` can be overridden with `local` declarations, which limits their scope to the function and does not affect the outer scope.
<!-- +END -->

<!-- +INCLUDE: ./DEVELOPMENT.md -->
# Developer Guide for Task Runner

## Testing

To run all tests in `_test-*.lib.sh`:
```bash
./task test
```

To run specific tests only, specify the test names as arguments. The following example runs test functions `test_foo` and `test_bar`:

```bash
./task test foo bar
```
<!-- +END -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/knaka) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
