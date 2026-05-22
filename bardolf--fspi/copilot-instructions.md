## fspi

> **fspi** (Fedora Sway Post-Install) is an idempotent shell script framework for

# AGENTS.md - Coding Agent Instructions for fspi

## Project Overview

**fspi** (Fedora Sway Post-Install) is an idempotent shell script framework for
provisioning a Fedora Sway desktop environment. Pure Bash — no compiled languages,
no package managers, no build tools beyond shell.

## Repository Structure

```
fspi/
├── install.sh          # Entry point — runs all steps in order
├── lib/
│   ├── logging.sh      # log_info, log_warn, log_error, log_debug
│   └── utils.sh        # Idempotent helpers (ensure_package, ensure_symlink, etc.)
├── steps/              # Numbered scripts (00-30) executed sequentially
├── config/             # Dotfiles deployed to ~/.config/
├── scripts/            # User scripts deployed to ~/scripts/
└── files/              # Static files (desktop entries, icons)
```

## Running

```bash
./install.sh            # Run all steps sequentially
DEBUG=1 ./install.sh    # Run with verbose debug output
bash steps/02_packages.sh   # Run a single step directly
```

Each step in `steps/` is a standalone script and can be run independently.
Steps are executed in glob-sorted (alphabetical/numerical) order by `install.sh`.

## Tests

There is no test framework. There are no unit tests, integration tests, or CI.
Scripts are validated by running them directly on a Fedora Sway system.
To verify a single step, run it directly: `bash steps/XX_name.sh`.

## Linting

No linting tools are configured. If you want to check shell scripts, use
`shellcheck` manually:

```bash
shellcheck steps/*.sh lib/*.sh install.sh
```

## Code Style Guidelines

### Shebang and Safety

Every script MUST start with:

```bash
#!/usr/bin/env bash
set -euo pipefail
```

This ensures: exit on error (`-e`), undefined variable errors (`-u`), and
pipe failure propagation (`-o pipefail`).

### Script Bootstrapping

Every step script MUST resolve the project root and source libraries:

```bash
SCRIPT_DIR=$(cd "$(dirname "${BASH_SOURCE[0]}")/.." && pwd)
source "$SCRIPT_DIR/lib/logging.sh"
source "$SCRIPT_DIR/lib/utils.sh"
```

Source `logging.sh` before `utils.sh` — utils depends on logging functions.
If a step only needs logging (no utility helpers), sourcing `utils.sh` can
be omitted.

### Naming Conventions

- **Step files**: `NN_descriptive_name.sh` — two-digit prefix controls order.
  Ranges: `00-09` base system, `10-19` tools from source, `20-29` config, `30+` scripts.
- **Functions**: `snake_case` — e.g., `ensure_package`, `config_step_copy`
- **Constants/config vars**: `UPPER_SNAKE_CASE` — e.g., `SCRIPT_DIR`, `PACKAGES`, `BUILD_DIR`
- **Local/temporary vars**: `lower_snake_case` — e.g., `pkg`, `target`, `timestamp`
- **Use `local`** for variables inside functions: `local pkg="$1"`

### Logging

Use the logging functions from `lib/logging.sh`:

- `log_info "message"` — significant actions (always visible)
- `log_debug "message"` — skip/already-done messages (visible only with `DEBUG=1`)
- `log_warn "message"` — non-fatal issues
- `log_error "message"` — fatal errors (typically followed by `exit 1`)

Every step should start and end with `log_info` describing what it does.
Prefer `log_debug` for "already done, skipping" messages to keep default output clean.

### Idempotency (Critical)

ALL steps MUST be idempotent — safe to run multiple times with identical results.
The standard pattern is check-then-act:

```bash
if [[ <already done condition> ]]; then
    log_debug "Already done, skipping"
else
    log_info "Doing the thing..."
    <actual work>
fi
```

Use the helpers from `lib/utils.sh` which implement this pattern:
- `ensure_package "pkg"` — installs only if `rpm -q` fails
- `ensure_symlink "target" "dest"` — creates only if not already correct
- `ensure_file_copy "src" "dest"` — copies only if `cmp -s` finds differences
- `ensure_repo "name" "url"` — adds repo only if not in `dnf repolist`
- `ensure_user_in_group "user" "group"` — adds only if not already member
- `check_command "cmd"` — returns 0/1, does not exit
- `ensure_command "cmd"` — exits with error if command missing

### Error Handling

- `set -euo pipefail` handles most errors automatically
- For critical operations, use explicit handling:
  `command || { log_error "Failed to..."; exit 1; }`
- For non-critical failures, use:
  `command || log_warn "Could not..."` or `command || true`
- Use `run_sudo "cmd"` to run commands with sudo when not root

### Arrays and Iteration

Package lists and similar collections use Bash arrays:

```bash
PACKAGES=(
  vim
  htop
  btop
)

for pkg in "${PACKAGES[@]}"; do
  ensure_package "$pkg"
done
```

Associative arrays are used for key-value mappings (e.g., aliases).

### Config Deployment

Files from `config/` are deployed to `~/.config/` by `steps/20_config.sh`.
Use `config_step_copy_collection` for batch deployment (takes source/target pairs).
The function handles: missing source (skip with warning), missing target (copy),
identical content (skip), different content (timestamped backup then copy).

Never overwrite files containing user credentials — check first and skip if
the target already exists.

### Comments

- Comments may be in Czech or English (the codebase mixes both)
- Use `# --- Section Name ---` for section separators
- Log messages and user-facing output should be in English

### Formatting

- Indent with 2 spaces (consistent across all scripts)
- Use `[[ ]]` for conditionals (not `[ ]`)
- Use `$(command)` for command substitution (not backticks)
- Quote all variable expansions: `"$var"`, `"${array[@]}"`
- Use `"$@"` to pass through arguments

## Key Utility Functions (lib/utils.sh)

| Function | Purpose |
|---|---|
| `ensure_package "pkg"` | Install DNF package if missing |
| `ensure_repo "name" "url"` | Add DNF repo if not present |
| `ensure_symlink "target" "dest"` | Create symlink idempotently |
| `ensure_file_copy "src" "dest"` | Copy file only if changed |
| `ensure_user_in_group "user" "group"` | Add user to group if needed |
| `ensure_command "cmd"` | Assert command exists, exit if not |
| `check_command "cmd"` | Check if command exists (bool) |
| `run_sudo "cmd"` | Run with sudo if not already root |

## Adding a New Step

Create `steps/XX_name.sh` using this template:

```bash
#!/usr/bin/env bash
set -euo pipefail

SCRIPT_DIR=$(cd "$(dirname "${BASH_SOURCE[0]}")/.." && pwd)
source "$SCRIPT_DIR/lib/logging.sh"
source "$SCRIPT_DIR/lib/utils.sh"

log_info "Step XX: Description of what this step does"

# Check-then-act pattern for idempotency
if [[ <already done> ]]; then
    log_debug "Already configured, skipping"
else
    log_info "Configuring..."
    # ... actual work ...
fi

log_info "Step XX complete"
```

---
> Source: [bardolf/fspi](https://github.com/bardolf/fspi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
