## dotcube

> dotcube is a dotfiles management system that provisions ZSH shell configurations and system environments using Ansible. It supports macOS, Ubuntu/Debian, Raspbian, and WSL.

# AGENTS.md - dotcube

## Project Overview

dotcube is a dotfiles management system that provisions ZSH shell configurations and system environments using Ansible. It supports macOS, Ubuntu/Debian, Raspbian, and WSL.

## Repository Structure

```
bin/                    # Executable shell scripts
  dotcube               # Main bootstrap and provisioning script
  dotcube-remote        # Remote machine provisioning via SSH
  dotcube-custom        # Machine-specific custom config deployer
  spinner               # UI spinner utility for long operations

ansible/                # Ansible playbook infrastructure
  main.yml              # Local provisioning playbook (hosts: localhost)
  remote.yml            # Remote provisioning playbook (hosts: all)
  ansible.cfg           # Ansible configuration
  requirements.yml      # Ansible Galaxy dependencies
  group_vars/all/       # Default role list and variables
  pre_tasks/            # Environment detection (WSL, user)
  callback_plugins/     # Custom output formatter
  roles/
    system/             # Base OS setup (first-run only)
    zsh/                # ZSH plugins, prompt, and config (every run)
    extra-tools/        # Convenience utilities (lsd, gh, jq, htop, etc.)
    netops/             # Networking tools (optional, -t netops)
    custom/             # Machine-specific configs from external repo
    upgrade/            # OS package upgrades
    rollback/           # Revert shell to original state

dotfiles/               # Configuration files deployed to the system
  zsh/
    zshrc               # Main zshrc (symlinked to ~/.zshrc)
    zsh-config/         # Sourced config files (exports, aliases, functions, prompt)
  starship/
    starship.toml       # Starship prompt configuration
  terminals/            # Terminal emulator configs
```

## Execution Flow

1. `bin/dotcube` bootstraps dependencies (git, python3, ansible) and clones/updates the repo
2. It runs `ansible-playbook` against `ansible/main.yml` (local) or `ansible/remote.yml` (remote)
3. Ansible pre-tasks detect WSL and current user
4. Roles execute based on tags (`-t`) or defaults defined in `group_vars/all/roles.yml`
5. The first-run marker at `~/.config/.dotcube.run` controls one-time-only tasks
6. ZSH config is symlinked: `~/.zshrc` -> `~/.dotcube/dotfiles/zsh/zshrc`

## Development Workflow

### Branch Strategy

- `main` is the default and production branch
- All changes (bug fixes, features, improvements) go through pull requests
- Each fix or feature gets its own branch off `main`

### Making Changes

For each fix or feature:

1. Create a branch off `main` with a descriptive name:
   - Bug fixes: `fix/<short-description>` (e.g. `fix/rollback-restore-typo`)
   - Features: `feat/<short-description>` (e.g. `feat/add-tmux-role`)
   - Improvements: `improve/<short-description>` (e.g. `improve/quote-shell-vars`)
2. Make the code changes on that branch
3. Commit with a message that explains **what** changed and **why** (not just "fixed bug")
4. Push the branch to origin
5. Create a pull request against `main` with:
   - A clear title (under 70 characters)
   - A summary section with bullet points describing the changes
   - A test plan section with steps to verify the fix/feature
6. After PR is created, delete the local branch (`git branch -d <branch>`)

### Multiple Fixes

When working on multiple independent fixes:
- Create each on a separate branch off `main`
- Each branch gets its own commit, push, and PR
- Clean up all local branches after PRs are created

## Code Change Guidelines

### Shell Scripts (`bin/`)

- All scripts use `#!/bin/bash` and `set -e`
- Color variables and emoji codes are defined at the top of each script that uses them
- The `_task` / `_cmd` / `_task_done` pattern in `bin/dotcube` handles progress display and error logging
- `_cmd` uses `eval` - avoid passing untrusted input
- Use `"$@"` (not `$*`) when forwarding arguments to preserve quoting

### Ansible Roles (`ansible/roles/`)

- Each role has a `tasks/main.yml` that dispatches to OS-specific files (`ubuntu.yml`, `osx.yml`) or a `common.yml`
- OS detection uses `ansible_os_family`: `Debian` for Linux, `Darwin` for macOS
- First-run detection: stat `{{ ansible_user_dir }}/.config/.dotcube.run` - the marker is created by the `dotcube` shell script, not by Ansible
- Default variables go in `defaults/main.yml` within each role
- Tasks use `become: true` when root privileges are needed
- Use `community.general.homebrew` for macOS packages, `ansible.builtin.apt` for Debian/Ubuntu

### ZSH Configuration (`dotfiles/zsh/`)

- `zshrc` sources everything through helper functions defined in `zsh-config/zsh-functions`
- `zsh_add_file` sources from the repo (`$ZDOTDIR_REPO`), `zsh_add_file_local` from `$ZDOTDIR` (~/.config/zsh)
- Plugins are cloned to `~/.config/zsh/plugins/` by both Ansible (initial install) and `zsh_add_plugin` (runtime fallback)
- The prompt is controlled by `MYPROMPT` variable in `zsh-config/zsh-options` (starship or pure)
- Machine-specific ZSH overrides go in `~/.config/zsh/zsh-custom` (not tracked in this repo)

### Adding a New Ansible Role

1. Create `ansible/roles/<name>/tasks/main.yml`
2. Add OS-specific task files if needed (`ubuntu.yml`, `osx.yml`)
3. Add default variables in `ansible/roles/<name>/defaults/main.yml`
4. To include in default runs, add the role name to `ansible/group_vars/all/roles.yml`
5. The role is always available via `dotcube -t <name>` regardless of the default list

### Testing Changes

- Run `dotcube -t <role> -vv` to test a specific role with verbose output
- Ansible tasks are idempotent - safe to run repeatedly
- Use `dotcube -t rollback` to revert shell changes
- For remote testing: `dotcube-remote <host> -t <role> -vv`

### Conventions

- Ansible task names follow the pattern: `ROLE | PLATFORM | Description`
- Shell scripts use `snake_case` functions prefixed with `_` for internal helpers
- Color output: define variables at script top, use `printf` with color codes
- Commit messages should describe what changed and why

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/onexodus)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/onexodus)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
