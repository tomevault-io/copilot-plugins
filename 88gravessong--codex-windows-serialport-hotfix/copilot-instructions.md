## codex-windows-serialport-hotfix

> Maintain a safe, version-locked workaround for the OpenAI.Codex

# Agent instructions

## Objective

Maintain a safe, version-locked workaround for the OpenAI.Codex
26.707.9981.0 Windows crash caused by the bundled Work Louder serialport.node.

## Read order

1. docs/evidence.json
2. docs/agent-handoff.md
3. docs/technical-analysis.md
4. scripts/codex-micro-hotfix-26.707.9981.0.ps1
5. docs/validation.md

Do not start by guessing from the exception code. The exact DelayLoadInfo and
the failed early workaround are already recorded.

## Hard safety boundaries

- Never edit C:\Program Files\WindowsApps.
- Never take ownership of WindowsApps.
- Never commit or distribute ChatGPT.exe, app.asar, serialport.node, dumps,
  profiles, Cookies, tokens, logs, or other proprietary/user artifacts.
- Never remove the exact package-version guard without a new build-specific
  analysis and validation record.
- Keep every ASAR replacement byte-length preserving unless the archive is
  properly rebuilt and all integrity metadata is updated.
- Preserve exact expected match counts. Refuse ambiguous matches.
- Recursive deletion is allowed only for the fixed LocalAppData hotfix target
  after its ownership marker or manifest is validated.
- Use PowerShell 7. Windows PowerShell 5.1 fails on dependency paths longer
  than 260 characters.

## Required checks

Run before committing:

    pwsh -NoProfile -File .\tests\static-validate.ps1

Read-only inspection of an installed build:

    pwsh -NoProfile -File .\scripts\inspect-installed-build.ps1 -Json

## Runtime red/green criteria

The original failure was deterministic at roughly 11 seconds after cold
launch. A candidate fix is not accepted unless all of the following are true:

1. Test from a copied App, never from a modified WindowsApps package.
2. Use a new isolated user-data-dir for at least two cold launches.
3. Each launch exceeds 45 seconds.
4. No new Application Error 1000 or WER event for 0xC06D007F.
5. No new ChatGPT dump.
6. serialport.node is absent from loaded process modules.
7. Repeat once with the real user-data-dir after all isolated runs pass.

## Known failed approach

Replacing only the two renderer feature-gate calls is insufficient. It hides
Codex Micro UI, but a startup path still reaches the main-process service and
loads serialport.node. Keep the main-process getState and updateLighting stubs.

## Scope of this workaround

It intentionally disables Codex Micro / Work Louder hardware functionality.
Do not generalize this to unrelated freezes or memory issues without evidence.
The permanent upstream fix is a correctly rebuilt native addon or removal of
the incompatible dependency.

---
> Source: [88gravessong/codex-windows-serialport-hotfix](https://github.com/88gravessong/codex-windows-serialport-hotfix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
