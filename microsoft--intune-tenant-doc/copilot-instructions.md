## intune-tenant-doc

> Start with [`AGENTS.md`](../AGENTS.md) for the repository map, conventions, and verification steps. This file adds Copilot-specific guardrails.

# Copilot instructions — intune-tenant-doc

Start with [`AGENTS.md`](../AGENTS.md) for the repository map, conventions, and verification steps. This file adds Copilot-specific guardrails.

## What this repo is

A PowerShell 7 Intune tenant configuration exporter. The main script uses Microsoft Graph read-only permissions to collect Intune and Conditional Access configuration and generate Markdown documentation.

## Working here

- Keep `Export-IntuneTenantDoc.ps1` read-only against tenant data; do not add write, update, or delete Graph operations.
- Preserve PowerShell 7 compatibility and the existing `CmdletBinding()` parameter style.
- Prefer small helper functions for Graph calls, classification, Markdown rendering, and output-file assembly.
- Keep generated output as Markdown; update `README.md` and `PROJECT.md` when parameters, required scopes, or output files change.
- Validate edits with `pwsh scripts/verify.ps1` before opening a PR.

## PR guardrails

- Keep PRs small and single-purpose — prefer one behavior, output category, or documentation change per PR.
- Do not commit credentials, tenant data, exported customer content, or generated tenant documentation.
- Apply the PR labels and description footer required by
  [`.github/instructions/telemetry.instructions.md`](instructions/telemetry.instructions.md).

---
> Source: [microsoft/intune-tenant-doc](https://github.com/microsoft/intune-tenant-doc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
