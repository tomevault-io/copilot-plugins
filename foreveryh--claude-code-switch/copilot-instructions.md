## claude-code-switch

> The core CLI logic lives in `ccm.sh`, a Bash script that manages provider detection, environment exports, and status output. `install.sh` and `uninstall.sh` wrap macOS/Linux setup and teardown of the `ccm` shell function without touching unrelated shell rc entries. Documentation and change history are tracked in `README.md`, `README_CN.md`, and `CHANGELOG.md`. Runtime configuration is stored in the user-scoped `~/.ccm_config`; keep sample data light inside the repo and rely on environment overrides during development.

# Repository Guidelines

## Project Structure & Module Organization
The core CLI logic lives in `ccm.sh`, a Bash script that manages provider detection, environment exports, and status output. `install.sh` and `uninstall.sh` wrap macOS/Linux setup and teardown of the `ccm` shell function without touching unrelated shell rc entries. Documentation and change history are tracked in `README.md`, `README_CN.md`, and `CHANGELOG.md`. Runtime configuration is stored in the user-scoped `~/.ccm_config`; keep sample data light inside the repo and rely on environment overrides during development.

## Build, Test, and Development Commands
Use `chmod +x ccm.sh install.sh uninstall.sh` before first run to ensure scripts stay executable. `./install.sh` installs the local development build and refreshes shell hooks for manual testing. Run `./ccm.sh status` to confirm environment precedence and newly added providers. Lint scripts with `shellcheck ccm.sh install.sh uninstall.sh` and follow up with `bash -n ccm.sh` to catch syntax regressions early.

### ⚠️ IMPORTANT: Reinstall After Code Changes
**When you modify `ccm.sh` during development, you MUST reinstall to update the installed version:**
```bash
./install.sh
source ~/.zshrc  # or source ~/.bashrc
```

**Why?** The `ccm` shell function uses the installed script at `~/.local/share/ccm/ccm.sh`, NOT the development version in your working directory. If you skip reinstalling:
- ❌ Your code changes won't take effect when using `ccm` command
- ❌ New features (like new models) will cause "command not found" or parsing errors
- ❌ You'll be testing the OLD version, leading to confusing results

**Development workflow:**
1. Edit `ccm.sh` in your working directory
2. Run `./install.sh` to copy changes to `~/.local/share/ccm/`
3. Run `source ~/.zshrc` to reload shell functions
4. Test with `ccm <command>` to verify your changes

**Quick verification:** Run `ccm status` to confirm you're using the latest version. If new commands fail with parsing errors like `bad pattern: ^[[0` or `command not found`, you forgot to reinstall.

## Coding Style & Naming Conventions
Stick to POSIX-friendly Bash 4 features, four-space indentation, and double quotes around variable expansions. Functions follow `snake_case` naming, and user-facing commands stay lowercase (for example `ccm fallback`). Prefer `[[ ]]` tests, `local` variables inside functions, and guard secrets by masking values on output. Reuse the existing color constants and message icons to keep status output consistent.

## Testing Guidelines
We do not yet ship automated integration tests, so rely on linting plus smoke checks. Populate temporary environment variables (`export DEEPSEEK_API_KEY=dummy`) before running `./ccm.sh status` to watch the masking logic, and verify fallback behavior by unsetting keys. When adding providers, document minimal happy-path commands in the README and confirm uninstall leaves no residual rc entries.

## Commit & Pull Request Guidelines
Commits follow Conventional Commit prefixes (`feat:`, `fix:`, `docs:`) as seen in `git log`. Keep messages imperative and scoped to a single concern; mention the touched provider if relevant. Pull requests should summarize behavior changes, list manual verification commands, and link any tracked issues. Include screenshots or terminal transcripts when altering output formatting so reviewers can validate masking and color usage.

## Security & Configuration Tips
Never commit real API keys or expanded configs—use placeholders matching the existing `sk-your-...` convention. Highlight any new environment variables or config keys in both README files, and remind contributors to `chmod 600 ~/.ccm_config` when documenting workflow changes. If a change impacts API routing, call out required base URLs and whether the fallback path still respects precedence rules.

---
> Source: [foreveryh/claude-code-switch](https://github.com/foreveryh/claude-code-switch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
