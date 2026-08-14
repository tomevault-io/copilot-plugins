## zsh

> - Full checks: `./test.sh`

# Agent Guidelines for Zsh Configuration

## Verify changes

- Full checks: `./test.sh`
- Focused checks: `./test.sh syntax|environment|modules|installer|update`
- Targeted syntax: `zsh -n <file>`
- ShellCheck runs in GitHub Actions for supported shell scripts.

`./check-project.sh`, standalone status/validation/performance scripts, and category names not listed above are not supported commands.

## Project shape

- `zshenv` establishes XDG and Zsh paths.
- `zshrc` loads modules from `modules/`, optional local environment overrides, and `local.zsh`.
- `themes/prompt.zsh` configures the prompt.
- `plugins/core.list` is the plugin registry parsed by `modules/plugins.zsh`.
- `VERSION` is the canonical version source.

## Changes

- Use `#!/usr/bin/env zsh` for `.zsh` files and `#!/usr/bin/env bash` for `.sh` files.
- Keep shell functions focused, validate arguments, and return meaningful exit codes.
- Use `$ZSH_CONFIG_DIR` and XDG paths rather than hard-coded user paths.
- Do not add scripts or documentation for capabilities that are not implemented and tested.
- Run the applicable test command after editing.

## Agent skills

### Issue tracker

Issues 在 Linear 中跟踪，通过 Linear MCP 创建和更新。See `docs/agents/issue-tracker.md`.

### Triage labels

默认 triage 标签（needs-triage、needs-info、ready-for-agent、ready-for-human、wontfix）。See `docs/agents/triage-labels.md`.

### Domain docs

Single-context：仓库根目录一个 `CONTEXT.md` + `docs/adr/`。See `docs/agents/domain.md`.

---
> Source: [windyboy/zsh](https://github.com/windyboy/zsh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
