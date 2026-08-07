## sol-luna-codex-workflow

> This repository is designed to be handed directly to a Codex Agent for installation.

# Installation contract

This repository is designed to be handed directly to a Codex Agent for installation.

## Objective

Install the checked-in `luna_worker` custom agent and `sol-luna-workflow` skill for the current user while preserving every unrelated Codex setting.

## Authorized changes

The installation may create only these targets:

```text
${CODEX_HOME:-$HOME/.codex}/agents/luna-worker.toml
$HOME/.agents/skills/sol-luna-workflow/SKILL.md
```

The installer may remove `$CODEX_HOME/skills/sol-luna-workflow/SKILL.md` (or `$HOME/.codex/skills/...` when `CODEX_HOME` is unset) only when that legacy file exactly matches the repository source and neither the file nor its directory is a symbolic link. This is a path migration, not authorization to delete other legacy Skills.

Use `$CODEX_HOME` for the Agent when it is set; otherwise use `$HOME/.codex`. User-authored Skills use `$HOME/.agents/skills`. Run `bash scripts/install.sh` from this repository. Do not reproduce the copy or migration logic with broader commands.

## Prohibited changes

Do not edit or delete `config.toml`, other agents, other skills, global or project `AGENTS.md` files, Codex App personalization, or any unrelated content.

If either target or the legacy Skill exists with different content, stop before changing anything. Show the exact conflicting path and ask the user how to proceed. Never overwrite or remove a conflict automatically.

## Verification and handoff

After installation, confirm that both installed files exactly match the repository sources. Parse the TOML when a standard TOML parser is available. Do not create a larger validation toolchain for this two-file copy.

Tell the user that `personalization.md` does not activate itself. They must manually copy one complete language block into Codex App Settings → Personalization → Custom Instructions when they want App-level personalization. Suggest starting a new task. A full restart is normally unnecessary; reopen Codex only if the newly added custom agent is not discovered.

Report installed paths, unchanged paths, verification results, conflicts if any, and the remaining manual personalization step.

---
> Source: [liuyejinghong/sol-luna-codex-workflow](https://github.com/liuyejinghong/sol-luna-codex-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-04 -->
