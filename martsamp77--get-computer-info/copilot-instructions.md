## get-computer-info

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Read-only diagnostic scripts that map a machine — OS, users/groups, filesystem layout, services, scheduled tasks, packages, containers, databases, hardware, GPU, network, security posture, health — into one **Markdown** report to hand an AI assistant as context about the box you're on. **One script per platform, same report structure, same security model.** No build system, manifest, or test suite. Everything must stay **generic** — no machine-, site-, or stack-specific values.

## Platforms

| Platform | Script | Runtime | Status |
|----------|--------|---------|--------|
| Linux   | `server-audit.sh`       | Bash 4+          | ✅ Available (reference implementation) |
| macOS   | `server-audit-macos.sh` | Bash 3.2-safe / zsh | 🚧 To build |
| Windows | `server-audit.ps1`      | PowerShell 5.1/7 | 🚧 To build |

## Start here

1. These rules are **always loaded** and apply to every platform — follow them no matter what you're doing:
   - `.claude/rules/security.md` — the non-negotiable secret-safety model (the whole point).
   - `.claude/rules/report-structure.md` — the canonical section order every script must produce.
2. Then read the playbook for the platform you're working on (also auto-loads when you open that script):
   - Linux → `.claude/rules/linux.md`
   - macOS → `.claude/rules/macos.md`
   - Windows → `.claude/rules/windows.md`
3. **Building a new platform variant?** `server-audit.sh` is the reference — mirror its helper structure, section order, and three security layers, translating probes to native tooling per the platform playbook. When done, flip that row to ✅ in `README.md` and fill in its usage block.

## Always-true constraints (details in security.md)

- **Never leak secrets.** (1) Sensitive paths are reported existence-only, never read. (2) Everything read is piped through a redactor (passwords/tokens/keys/credential-URLs/PEM blocks/serials). (3) No credentialed access — detect services by port+process+version only; environment variables by **name only**, never values.
- **Read-only.** Safe on production — no writes/installs/restarts/state changes (sole exception: writing the report file).
- **Degrade gracefully.** Guard every external tool; a missing tool prints a note, never aborts the run.
- **Keep network identifiers.** IPs, MACs, hostname are intentionally included for context (the report is sensitive — don't publish it; generated `*_context_*.md` are gitignored). A `--mask-net` mode / wizard option masks them for a shareable report.
- **Interactive wizard, non-blocking.** With no flags on a TTY, the script runs a wizard (destination/format/scope/mask/clipboard); piped/cron/SSH runs and `--no-wizard` must fall back to defaults without ever blocking on a prompt. Details in `report-structure.md`.

## Linting

- Bash (`*.sh`): `shellcheck --severity=warning <script>` must pass. Info/style notes (SC2016 on Markdown `printf` formats, SC2012 for names-only `ls`) are intentionally allowed. (This shellcheck build ignores `.shellcheckrc`, so the flag is explicit.)
- PowerShell (`*.ps1`): `Invoke-ScriptAnalyzer -Severity Warning,Error`.
- The PostToolUse hook in `.claude/settings.json` lints both automatically on edit (no-ops if the linter isn't installed).

## Adding a section

Use the `/add-audit-section` skill — it covers placement, the shared helpers, the summary table, linting, and the secret-safety red-team check.

---
> Source: [martsamp77/get-computer-info](https://github.com/martsamp77/get-computer-info) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
