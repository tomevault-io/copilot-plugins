## nixvim

> This repository is a [nixvim](https://github.com/nix-community/nixvim) configuration — a Neovim

# AGENTS.md — Contributor Guide for Agentic Coding Agents

This repository is a [nixvim](https://github.com/nix-community/nixvim) configuration — a Neovim
distribution defined entirely in Nix using the nixvim module system. It exposes two profiles
(`full` and `lite`) as Nix flake outputs.

---

## Repository Layout

```
flake.nix               # Flake definition: inputs, packages, checks, devShell, formatter
justfile                # Task runner shortcuts (requires `just`)
config/
  full.nix              # Entry point for the "full" profile
  lite.nix              # Entry point for the "lite" profile
  opts.nix              # Neovim options, extraPackages, custom `nixvim.flavour` option
  keymaps.nix           # All keymaps
  autocmds.nix          # Autocommands
  icons.nix             # Nerd Font icon definitions (pure Nix attribute set)
  icons_lua.nix         # Converts icons.nix → a Lua global via lib.nixvim.toLuaObject
  plugins/              # Plugin configs grouped by category
    ai/ coding/ colorscheme/ custom/ editor/ formatting/
    lang/ linting/ lsp/ treesitter/ ui/
  utils/                # Lua utility libraries embedded in .nix files
    base.nix lsp.nix root.nix ui.nix lualine.nix inject.nix default.nix
```

Each plugin category directory contains:

- `default.nix` — re-exports everything (used by both profiles)
- `full.nix` — full-profile-only imports
- `lite.nix` — lite-profile-only imports (subset, fewer heavy plugins)

Per-language support modules live under `config/plugins/lang/<name>/default.nix` and expose a
`sys.lang.<name>.enable` option toggled from `lite.nix` / `full.nix`.

---

## Build Commands

```bash
# Enter the dev shell (installs `just` and pre-commit hooks)
nix develop

# List available just commands
just

# Validate the full config builds without errors (primary CI check)
nix flake check .

# Build and run the full Neovim profile
nix run .           # or: just run-full  /  just test
nix run .#full

# Build and run the lite Neovim profile
nix run .#lite      # or: just run-lite  /  just run
nix run .#lite-obsidian

# Update all flake inputs
just update         # equivalent to: nix flake update

# Format all Nix files
nix fmt             # runs alejandra
```

---

## Testing

There are no unit test files. The only automated check is the Nix flake evaluation:

```bash
nix flake check .
```

This runs two checks:

1. **`default`** — `nixvimLib.check.mkTestDerivationFromNixvimModule` on the full profile.
   Verifies the entire config evaluates and builds without errors.
2. **`pre-commit-check`** — runs `alejandra` (formatter) and `statix` (linter) on all `.nix` files.

To verify a specific change, run `nix flake check .` after editing. There is no way to run a
single isolated test; the check evaluates the whole config.

---

## Linting & Formatting

Pre-commit hooks run automatically on `git commit` (installed by `nix develop`):

| Tool       | Purpose                              | Scope         |
|------------|--------------------------------------|---------------|
| `alejandra` | Opinionated Nix formatter            | `.nix` files  |
| `statix`   | Nix linter (anti-pattern detection)  | `.nix` files  |

To run them manually:

```bash
# Format all Nix files
nix fmt

# Run statix linter
nix run nixpkgs#statix -- check .
```

The `nix fmt` command is the canonical way to format; `alejandra` is also available in the dev
shell.

---

## Code Style: Nix

All source files are Nix (`.nix`). Style is enforced by **alejandra** — let the formatter decide
indentation and line-wrapping. Never fight the formatter.

**Module structure:**

```nix
# Modules that use lib heavily open with `with lib;`
{ lib, config, pkgs, ... }:
with lib; {
  options.sys.lang.foo = {
    enable = mkEnableOption "foo support";
  };

  config = mkIf config.sys.lang.foo.enable {
    plugins.foo.enable = true;
  };
}
```

**Attribute sets:**

- Semicolons, not commas: `{ desc = "Save File"; }` not `{ desc = "Save File" }`
- Short sets on one line: `{ desc = "Go to Definition"; }`
- Long sets multiline with each attribute on its own line
- `inherit` for attribute passthrough: `inherit pkgs;`, `inherit (icons.git) added modified;`

**Lists:**

- Short lists inline: `extraPackages = with pkgs; [ fd fzf ripgrep ];`
- Long lists multiline, one entry per line, with `[` on the same line as the key

**`let … in` bindings** for local reuse:

```nix
config = let
  inherit (config.sys.lang.rust) enable;
in mkIf enable { … };
```

**`mkOption` / `mkEnableOption` / `mkIf` / `mkDefault` / `mkForce`** are the standard NixOS module
option helpers. Prefer `mkEnableOption` for boolean feature flags.

**`with` scoping:** Use `with lib;` at the top of modules that heavily call lib functions.
Use `with pkgs;` inline for `extraPackages` lists. Avoid deeply nested `with`.

---

## Code Style: Inline Lua

Lua code never exists in standalone `.lua` files — it lives only as multiline Nix strings
embedded in `.nix` files. Mark Lua strings with the `# lua` comment for editor syntax injection:

```nix
callback = {
  __raw =
    # lua
    ''
      function()
        vim.highlight.on_yank()
      end
    '';
};
```

**`__raw`** is the nixvim convention for a raw Lua expression (function reference, function body,
or any Lua value) that must not be serialised as a Nix/Lua string. Always use `__raw` when the
value needs to be evaluated as Lua rather than passed as a string literal.

**Lua style inside strings:**

- 2-space indentation
- `local` variables at the top of each scope
- PascalCase for module/namespace tables: `NixVim`, `NixVim.lsp`, `NixVim.root`
- snake_case for local functions and variables
- `require("module")` for plugin access; `pcall(require, "module")` for optional requires
- `vim.keymap.set(mode, key, action, opts)` style; local helpers (`nmap`, `vmap`) for brevity
- Trailing commas on last table entries
- Standard global utilities available everywhere: `NixVim.notify`, `NixVim.error`,
  `NixVim.info`, `NixVim.warn`; debug helpers `P(v)` (print-inspect) and `DN(v)` (notify)

---

## Adding a New Plugin

1. Create `config/plugins/<category>/<plugin-name>.nix` (or add to an existing file if small).
2. Follow the pattern: `plugins.<name>.enable = true;` with `settings = { … };` for config.
3. Use `__raw = # lua '' … '';` for any Lua callbacks or function references.
4. If the plugin should only appear in the full profile, import it only from
   `config/plugins/<category>/full.nix`. For lite, add it to `lite.nix`.
5. For language-specific plugins, add a `sys.lang.<name>.enable` option and wrap with `mkIf`.
6. Run `nix flake check .` to verify the config evaluates correctly.

---

## Adding a New Language

Create `config/plugins/lang/<name>/default.nix` following this pattern:

```nix
{ lib, config, ... }:
with lib; {
  options.sys.lang.<name> = {
    enable = mkEnableOption "<name> support";
  };

  config = mkIf config.sys.lang.<name>.enable {
    plugins.lsp.servers.<lsp_server>.enable = true;
    # treesitter, formatters, linters, etc.
  };
}
```

Then import the new module from `config/plugins/lang/default.nix` and enable it in the
appropriate profile (`full.nix` or `lite.nix`).

---

## Key Conventions Summary

| Topic              | Convention                                                      |
|--------------------|-----------------------------------------------------------------|
| Formatter          | `alejandra` (run via `nix fmt`)                                 |
| Linter             | `statix` (run via pre-commit or `nix flake check`)              |
| Lua code location  | Inline strings in `.nix` files only; never standalone `.lua`    |
| Lua marker         | `# lua` comment before multiline Lua strings                    |
| Raw Lua values     | Always use `__raw = "…"` or `__raw = # lua '' … '';`            |
| Module options     | `mkEnableOption` for booleans; `mkOption` with type for others  |
| Profile splitting  | Heavy/optional features in `full.nix`; minimal set in `lite.nix`|
| Language features  | Gated behind `sys.lang.<name>.enable` option                    |
| Icons              | Defined in `config/icons.nix`; consumed via `inherit (icons.x)` |
| Lua utilities      | `NixVim.*` namespace; defined in `config/utils/`                |

<!-- bv-agent-instructions-v2 -->

---

## Beads Workflow Integration

This project uses [beads_rust](https://github.com/Dicklesworthstone/beads_rust) (`br`) for issue tracking and [beads_viewer](https://github.com/Dicklesworthstone/beads_viewer) (`bv`) for graph-aware triage. Issues are stored in `.beads/` and tracked in git.

### Using bv as an AI sidecar

bv is a graph-aware triage engine for Beads projects (.beads/beads.jsonl). Instead of parsing JSONL or hallucinating graph traversal, use robot flags for deterministic, dependency-aware outputs with precomputed metrics (PageRank, betweenness, critical path, cycles, HITS, eigenvector, k-core).

**Scope boundary:** bv handles *what to work on* (triage, priority, planning). `br` handles creating, modifying, and closing beads.

**CRITICAL: Use ONLY --robot-* flags. Bare bv launches an interactive TUI that blocks your session.**

#### The Workflow: Start With Triage

**`bv --robot-triage` is your single entry point.** It returns everything you need in one call:
- `quick_ref`: at-a-glance counts + top 3 picks
- `recommendations`: ranked actionable items with scores, reasons, unblock info
- `quick_wins`: low-effort high-impact items
- `blockers_to_clear`: items that unblock the most downstream work
- `project_health`: status/type/priority distributions, graph metrics
- `commands`: copy-paste shell commands for next steps

```bash
bv --robot-triage        # THE MEGA-COMMAND: start here
bv --robot-next          # Minimal: just the single top pick + claim command

# Token-optimized output (TOON) for lower LLM context usage:
bv --robot-triage --format toon
```

#### Other bv Commands

| Command | Returns |
|---------|---------|
| `--robot-plan` | Parallel execution tracks with unblocks lists |
| `--robot-priority` | Priority misalignment detection with confidence |
| `--robot-insights` | Full metrics: PageRank, betweenness, HITS, eigenvector, critical path, cycles, k-core |
| `--robot-alerts` | Stale issues, blocking cascades, priority mismatches |
| `--robot-suggest` | Hygiene: duplicates, missing deps, label suggestions, cycle breaks |
| `--robot-diff --diff-since <ref>` | Changes since ref: new/closed/modified issues |
| `--robot-graph [--graph-format=json\|dot\|mermaid]` | Dependency graph export |

#### Scoping & Filtering

```bash
bv --robot-plan --label backend              # Scope to label's subgraph
bv --robot-insights --as-of HEAD~30          # Historical point-in-time
bv --recipe actionable --robot-plan          # Pre-filter: ready to work (no blockers)
bv --recipe high-impact --robot-triage       # Pre-filter: top PageRank scores
```

### br Commands for Issue Management

```bash
br ready              # Show issues ready to work (no blockers)
br list --status=open # All open issues
br show <id>          # Full issue details with dependencies
br create --title="..." --type=task --priority=2
br update <id> --status=in_progress
br close <id> --reason="Completed"
br close <id1> <id2>  # Close multiple issues at once
br sync --flush-only  # Export DB to JSONL
```

### Workflow Pattern

1. **Triage**: Run `bv --robot-triage` to find the highest-impact actionable work
2. **Claim**: Use `br update <id> --status=in_progress`
3. **Work**: Implement the task
4. **Complete**: Use `br close <id>`
5. **Sync**: Always run `br sync --flush-only` at session end

### Key Concepts

- **Dependencies**: Issues can block other issues. `br ready` shows only unblocked work.
- **Priority**: P0=critical, P1=high, P2=medium, P3=low, P4=backlog (use numbers 0-4, not words)
- **Types**: task, bug, feature, epic, chore, docs, question
- **Blocking**: `br dep add <issue> <depends-on>` to add dependencies

### Session Protocol

```bash
git status              # Check what changed
git add <files>         # Stage code changes
br sync --flush-only    # Export beads changes to JSONL
git commit -m "..."     # Commit everything
git push                # Push to remote
```

<!-- end-bv-agent-instructions -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/magicmonty)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/magicmonty)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
