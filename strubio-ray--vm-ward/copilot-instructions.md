## vm-ward

> Auto-halt daemon for forgotten Vagrant VMs. Pure bash — no build system.

# vm-ward

Auto-halt daemon for forgotten Vagrant VMs. Pure bash — no build system.

## Project Structure

```
bin/vmw              # Entry point — resolves symlinks, detects platform, dispatches subcommands
lib/vmw-common.sh    # Shared utilities (logging, JSON helpers, duration parsing)
lib/vmw-host.sh      # macOS host daemon (Vagrant/VBox integration, leases, sweep)
lib/vmw-update.sh    # Copier template update module (lazy-sourced by `vmw update`)
tui/                 # Interactive TUI dashboard (Go/bubbletea, built to bin/vmw-tui)
share/vm-ward/       # launchd plist template
```

## Key Concepts

- **Vagrant machine IDs** (hex hashes) ≠ **VirtualBox UUIDs** (dashed format). Use `resolve_vbox_uuid()` to bridge them.
- **Leases** track how long a VM is allowed to run. Stored in `~/.local/state/vm-ward/leases.json`. Lease modes: `standard` (timed), `indefinite`, `exempt`, `halted` (post-halt with `halted_at` timestamp).
- **Sweep** runs every 5 min via launchd — warns at T1 (50%) and T2 (87.5%), halts on expiry. Activity detection uses `VBoxManage metrics query` (host-side CPU%). First sweep after VM start returns "idle" (metrics need one sampling period to populate). Writes epoch to `last-sweep` after each run. Cleans up halted leases >24h old and expired standard leases for poweroff VMs.
- **Event log**: `~/.local/state/vm-ward/events.jsonl` — structured JSONL log of lease/halt events, auto-trimmed to 500 lines.
- **Version placeholder**: `bin/vmw` contains `VMW_VERSION="%%VERSION%%"` — injected by Homebrew formula at install time.
- **Interactive dashboard**: `vmw` or `vmw status` launches the TUI dashboard (Go/bubbletea). `vmw status --json` returns machine-readable JSON for scripting. The TUI supports keybindings for extend, halt, destroy, exempt, update template (`u`), update all templates (`U`), and peek inside VMs (`p`).
- **Peek**: `vmw peek` SSHes into a running VM to show terminal session output (from `script` auto-logging) and top processes. Requires VMs to have terminal logging enabled (provisioned by copier-agent-sandbox template). The TUI renders raw terminal output through a virtual terminal emulator (`charmbracelet/x/vt`) for full color fidelity.
- **Status JSON schema**: `vmw status --json` returns `{daemon, last_sweep, recent_events, vms}` wrapper object (not a bare array). Each VM includes `section` (`active`|`halted`), `duration`, `halted_at`, and `template_version` fields.
- **Template tracking**: `vmw update` runs `copier update` across projects created from copier templates. Template version (from `.vm/.copier-answers.yml`) is shown in the TEMPLATE column of the dashboard.

## Release Flow

1. Commit with conventional commit format (`feat:`, `fix:`, etc.)
2. Run `cog bump --patch` (or `--minor`/`--major`)
3. Cocogitto creates the version tag and post-bump hooks push tag + commits to origin
4. GitHub Actions (`bump-homebrew.yml`) detects the `v*` tag push
5. `mislav/bump-homebrew-formula-action` updates the formula in `strubio-ray/homebrew-tap`
6. Users get the update via `brew update && brew upgrade vm-ward` (`brew update` refreshes the tap index first)

## Dependencies

Runtime: `jq`, `vagrant`, `VBoxManage`
Optional: `copier` (for `vmw update`)

## Useful Commands

```bash
bash -n lib/vmw-host.sh          # Syntax check
vmw                              # Launch interactive dashboard
vmw status --json                # JSON output for scripting
VBoxManage list runningvms       # Cross-check running VMs
vmw update .                     # Update current project's copier template
vmw update --all                 # Update all copier-managed projects
vmw update --all --provision     # Update all and reload running VMs
vmw peek .                       # Peek inside current project's VM
cog bump --patch                 # Release a patch version
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/strubio-ray) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
