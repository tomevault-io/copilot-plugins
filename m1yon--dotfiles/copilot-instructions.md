## dotfiles

> - This is a PURE Nix system. NEVER install global/system packages with `brew`, `nix profile`, language-specific package managers, app installers, manual downloads, or any other imperative install path.

- This is a PURE Nix system. NEVER install global/system packages with `brew`, `nix profile`, language-specific package managers, app installers, manual downloads, or any other imperative install path.
- Project-local dependency commands are allowed when they operate inside a repo project and update that project's normal lockfiles/dependency state. Example: running `bun install` inside `scripts/bun/` is allowed; running `bun add -g`, `npm install -g`, or using a language package manager to install global tools is not.
- For package changes, settings changes, services, apps, casks, defaults, or system behavior, edit the Nix flake/modules only. The agent may run Nix eval/check commands, but must not run `task rebuild`, `darwin-rebuild switch`, `nixos-rebuild switch`, `home-manager switch`, or any other Nix switch/rebuild/apply command itself.
- Agents may stage newly added Nix files when needed so flake eval can see imported paths. Do not stage unrelated changes.
- Once the agent is ready for a Nix rebuild, it should ask the user to run the appropriate rebuild command.
- Do not edit system or app settings manually through GUIs, `defaults write`, direct config-file mutation outside the Nix-managed files, or vendor CLIs. Make the desired state declarative in Nix.
- Homebrew is allowed only as a nix-darwin/nix-homebrew backend declared in the flake. Do not call the `brew` CLI to install, remove, upgrade, or mutate packages.

---
> Source: [m1yon/dotfiles](https://github.com/m1yon/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
