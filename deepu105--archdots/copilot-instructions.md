## archdots

> This repository contains sanitized public dotfiles for an Arch Linux, niri, DankMaterialShell, OpenCode, and llama.cpp setup.

# Agent Instructions

This repository contains sanitized public dotfiles for an Arch Linux, niri, DankMaterialShell, OpenCode, and llama.cpp setup.

## Safety Rules

- Treat this repo as public.
- Do not add secrets, tokens, API keys, shell history, private hostnames, Wi-Fi names, account data, SSH keys, GPG keys, certificates, or machine IDs.
- Keep paths generic and based on `$HOME` unless a placeholder is clearly documented.
- Do not copy files from private dotfiles wholesale. Extract only reusable configuration.
- Prefer small, focused changes.
- Do not run package installs, service enables, Stow, or system-changing commands unless explicitly asked.

## Scope

- Keep the repo focused on Arch, niri, DMS, OpenCode, and llama.cpp.
- Avoid adding unrelated desktop state from KDE, GNOME, browsers, editors, cloud CLIs, or chat apps.
- Keep examples runnable but generic.

## Validation

Use the narrowest validation that matches the change:

```bash
niri validate -c .config/niri/config.kdl
bash -n scripts/llama_server.sh
bash -n scripts/update_llama_cpp.sh
zsh -n .profile
zsh -n .zshenv
zsh -n .config/zsh/.zshrc
```

If validation cannot run because a tool is missing, state that clearly.

---
> Source: [deepu105/archdots](https://github.com/deepu105/archdots) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
