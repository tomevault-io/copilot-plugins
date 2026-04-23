## nix-config

> nixos-rebuild build --flake .#desktop    # test build only

# Nix Config

## Build Commands

**NixOS (desktop):**

```bash
nixos-rebuild build --flake .#desktop    # test build only
sudo nixos-rebuild switch --flake .#desktop  # build and activate
```

**Darwin (personal):**

```bash
darwin-rebuild build --flake .#Shanes-MacBook-Pro
darwin-rebuild switch --flake .#Shanes-MacBook-Pro
```

**Darwin (work):**

```bash
darwin-rebuild build --flake .#Shanes-Work-MacBook-Pro
darwin-rebuild switch --flake .#Shanes-Work-MacBook-Pro
```

## Secrets Management

Secrets are managed with [agenix](https://github.com/ryantm/agenix). Encrypted `.age` files live in `secrets/`. Public keys and secret declarations are in:

- `secrets/secrets.nix` — maps `.age` files to authorised public keys
- `home/shane/modules/common/age.nix` — declares secrets for home-manager to decrypt at runtime

When adding a new secret: create the `.age` file with `agenix -e secrets/<name>.age`, add the public key entry to `secrets/secrets.nix`, and declare it in `age.nix`.

When removing a secret: remove from all three locations (`.age` file, `secrets.nix`, `age.nix`) and any references in nix modules.

## NixVim

Neovim is configured declaratively via [NixVim](https://nix-community.github.io/nixvim/) at `home/shane/modules/common/nixvim/`.

- `default.nix` — main entry point, enables nixvim with aliases and imports
- `plugins/default.nix` — aggregator that imports all individual plugin modules
- Each plugin is a self-contained module in `plugins/<name>.nix`

**Platform-specific packages:** Some plugins reference Darwin-only packages (e.g. `xcbeautify`, `swiftformat`, `swiftlint`, `sourcekit`). Guard these with `lib.mkIf pkgs.stdenv.isDarwin` or `lib.optionals pkgs.stdenv.isDarwin` to prevent Linux build failures.

## AI Modules

AI tool configs live under `home/shane/modules/common/ai/`:

- `mcp/` — shared MCP server definitions (neovim, obsidian, posthog), imported by Claude Code, Claude Desktop, and Gemini
- `cc/` — Claude Code: settings, permissions, hooks, tweakcc theme
- `cdesktop/` — shared Claude Desktop MCP selection; platform-specific wrappers remain in `modules/linux/claude-desktop.nix` and `modules/macos/claude.nix`
- `gemini/` — Gemini CLI settings and theme

## Updating tweakcc

tweakcc (Vex theme for Claude Code TUI) lives at `home/shane/modules/common/ai/cc/tweakcc.nix`. The upstream repo is `Piebald-AI/tweakcc`.

Update steps:

1. Get the latest commit: `curl -s https://api.github.com/repos/Piebald-AI/tweakcc/commits/main | jq -r '.sha'`
2. Update `rev` in `tweakcc.nix`, set `hash` to empty string `""`, build to get the correct hash from the error output.
3. **Regenerate the lockfile** — upstream frequently adds deps to `package.json` without committing an updated `package-lock.json`. Download the source, run `npm install --package-lock-only --legacy-peer-deps --ignore-scripts` (ignore-scripts dodges their husky hook), and copy the result to `tweakcc-package-lock.json`.
4. Set `npmDepsHash` to empty string `""`, build to get the correct hash, then set it.
5. Test build with `nixos-rebuild build --flake .#desktop` — the build will fail with "patches are stale" if the tweakcc version doesn't match the Claude Code CLI version in nixpkgs.

The package uses `npmDepsFetcherVersion = 2` for reliable dependency resolution.

## Git Hygiene

When creating new files (secrets, modules, configs), always `git add` them before building. Nix flakes only see files tracked by git — untracked files are invisible to the build and will cause confusing errors.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shaneplunkett) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
