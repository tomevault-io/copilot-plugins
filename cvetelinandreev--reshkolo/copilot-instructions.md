## project-node-for-cli

> Before Node-related CLI commands, use the version from .nvmrc (via repo scripts or nvm).


# Node: read `.nvmrc` before running commands

This repo pins Node in **`.nvmrc`** (and **`.node-version`**; keep them in sync). Automated shells often use a **different** global default, which breaks Wasp and other tooling.

**Before running any terminal command in this project that uses Node** — including `node`, `npm`, `npx`, `wasp`, `prisma`, `vite`, `tsc`, or project `package.json` scripts that invoke those — do this:

1. **Read** `.nvmrc` from the **project root** so you know the required major/minor/patch version.
2. **Run the command with that Node on `PATH`**, in this order of preference:
   - **`npm run …`** scripts in `package.json` that already wrap with **`bash scripts/with-project-node.sh`** (e.g. `npm run wasp -- …`, `npm run db:migrate:deploy`), or  
   - **`bash scripts/with-project-node.sh <command> [args...]`** for anything else (the script reads `.node-version` / repo root and fixes `PATH`).

**Fallback** if a one-off cannot go through the wrapper: from the repo root, after `source "$HOME/.nvm/nvm.sh"`, run **`nvm use`** (no version argument — nvm reads `.nvmrc`), then run the command.

Do **not** assume an interactive shell has already switched Node; verify with `node -v` if something still fails.

---
> Source: [cvetelinandreev/reShkolo](https://github.com/cvetelinandreev/reShkolo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
