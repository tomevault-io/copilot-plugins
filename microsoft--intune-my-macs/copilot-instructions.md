## intune-my-macs

> <!-- Copyright (c) Microsoft Corporation. -->


<!-- Copyright (c) Microsoft Corporation. -->
<!-- Licensed under the MIT License. -->

# AGENTS.md — intune-my-macs

Canonical entry point for humans and coding agents working in this repository.
Start here, then follow the links below to the detailed docs and standards.

> **What this repo is:** a **proof-of-concept** toolkit that deploys a Microsoft
> Intune device configuration (policies, compliance, scripts, apps, optional
> Microsoft Defender for Endpoint) into a tenant from a single script. It is
> **sample/evaluation code, not a hardened production baseline** — see
> [README.md](README.md). Classification: **non-production**.

## Authority Map

Trust calibration: what to treat as today's truth vs. historical/context-only.

| Path | Authority | Read it as |
|---|---|---|
| `AGENTS.md` | canon | Source of truth for how this repo works *today* |
| `agency.toml` | canon (sidecar) | Machine-readable runtime config (MCP, security, readiness tier) |
| `.github/copilot-instructions.md` | canon (pointer) | Thin pointer to `AGENTS.md` |
| `.github/instructions/telemetry.instructions.md` | canon (scoped) | Verbatim org-wide PR + ADO work-item telemetry rules (`applyTo: "**"`) |
| `docs/architecture.md` | canon | Architecture, data flow, and invariants |
| `docs/conventions.md` | canon | Day-to-day code/content/structure conventions |
| `docs/testing-patterns.md` | canon | Validation / testing patterns |
| `standards/policy-naming-standard.prd` | canon | Reference-ID and display-name standard |
| `standards/manifest-standard.prd` | canon | Sibling-manifest schema standard |
| `.github/CODEOWNERS` | canon | Ownership routing for reviews |
| `.github/prompts/ship.prompt.md` | canon | Release checklist prompt |
| `.github/prompts/pr-review-reply.prompt.md` | canon | PR review-reply prompt |
| `INTUNE-MY-MACS-DOCUMENTATION.md` | canon | Generated payload catalog — regenerate, never hand-edit |
| `mainScript.ps1`, `Start-IntuneMyMacs.ps1`, `macOS/**`, `tools/**`, `scripts/**` | code | Deployment engine, artifacts, and tooling — the docs above are the agent context for them |
| `README.md`, `CUSTOMIZATION-GUIDE.md`, `BLOG-INTUNE-MY-MACS.md` | human | Human-facing overviews; not maintained as agent context |
| `CONTRIBUTING.md`, `SECURITY.md`, `SUPPORT.md`, `CODE_OF_CONDUCT.md`, `CHANGELOG.md` | human | Human-facing project and policy docs |
| `*-oss-compliance-report.md` | transient | Local advisory audit output (gitignored) — context only, never canon |

> **Trust calibration.** Treat *canon* as truth about today. Read *transient* /
> human rows for context only. When canon and the code disagree, canon is stale
> and must be fixed in the same PR — not ignored.

## Repository map

Platform artifact content lives under a per-platform folder (today: `macOS/`);
the deployment engine and developer tooling live at the repository root.

| Path | What lives here |
| --- | --- |
| `mainScript.ps1` | The deployment engine (PowerShell 7+). Selects a platform folder via `--platform` (default `macOS`), discovers that folder's artifact manifests, and creates/updates/deletes Intune objects via Microsoft Graph. |
| `Start-IntuneMyMacs.ps1` | macOS-only SwiftDialog GUI frontend for `mainScript.ps1`. |
| `macOS/` | All macOS artifact content (the deployable trees below). |
| `macOS/configurations/` | Settings Catalog policies (`.json`), custom configuration profiles (`.mobileconfig`), and compliance policies, grouped by area (`entra/`, `intune/`, `office/`, `Secure Enterprise Browser/`). |
| `macOS/apps/` | Application deployment manifests and helper scripts. |
| `macOS/custom attributes/` | macOS custom attribute scripts (`.zsh`/`.sh`) and their manifests. |
| `macOS/scripts/intune/` | Device shell scripts deployed via Intune (install/config). |
| `macOS/mde/` | Microsoft Defender for Endpoint onboarding, settings, and install script (opt-in via `--mde`). |
| `macOS/resources/` | Shared assets (e.g. wallpaper). |
| `tools/` | Local developer tooling (export, dedupe, doc generation, assignment reporting, fork-sync). See [tools/README.md](tools/README.md). |
| `standards/` | The naming and manifest standards every artifact must follow. |
| `scripts/verify.ps1` | One-command PowerShell validation loop — also runs `scripts/check-context.ps1` (run before pushing). |
| `.github/prompts/ship.prompt.md` | The release checklist (validate, regenerate docs, changelog, commit, push). |

## The manifest model (read this first)

Every deployable artifact (a `.json`, `.mobileconfig`, script, or `.pkg`) is
paired with a **sibling `.xml` manifest** containing a `<MacIntuneManifest>`
element. `mainScript.ps1` discovers artifacts by **scanning the selected
platform folder (e.g. `macOS/`) for these XML manifests** — a file without a
manifest is never deployed. Each manifest carries a unique `<ReferenceId>`, a
`<SourceFile>` that must resolve to a real file, and a `<SettingsCount>`.
Reference IDs follow the `TYPE-CATEGORY-NUMBER` standard (e.g. `POL-SEC-001`).

See [standards/policy-naming-standard.prd](standards/policy-naming-standard.prd)
and [standards/manifest-standard.prd](standards/manifest-standard.prd) for the
full rules, and [docs/conventions.md](docs/conventions.md) for the day-to-day
conventions an agent must follow.

## Build / run / validate loop

There is no compiled build. The fast local loop is:

```bash
# 1. Validate every artifact parses and scripts are syntactically sound:
pwsh ./scripts/verify.ps1

# 2. Preview a deployment (dry-run is the default — nothing is created):
pwsh ./mainScript.ps1 --assign-group "Intune Mac Pilot"

# 3. Apply for real (only against an evaluation tenant):
pwsh ./mainScript.ps1 --assign-group "Intune Mac Pilot" --apply
```

`--platform` defaults to `macOS`; see [README.md](README.md) for the full flag
list. Run `pwsh ./scripts/verify.ps1` before every push. When artifacts under `macOS/`
change, regenerate the catalog with
`python3 tools/Generate-ConfigurationDocumentation.py` (never hand-edit
`INTUNE-MY-MACS-DOCUMENTATION.md` — it is generated).

## Conventions an agent must follow

- Follow the reference-ID and display-name standards for any new artifact, and
  add a matching `.xml` manifest with a **unique** `<ReferenceId>`.
- Keep Settings Catalog `settings[]` IDs **contiguous and 0-based** (`"0"`,
  `"1"`, `"2"`, …).
- Add the Microsoft copyright header to every new source file
  (`# Copyright (c) Microsoft Corporation.` / `# Licensed under the MIT
  License.`). Never remove a pre-existing third-party copyright notice.
- Keep changes small and in logical units; update `CHANGELOG.md` (newest first).
- Full detail: [docs/conventions.md](docs/conventions.md).

## Key references

- [README.md](README.md) — quick start and what gets deployed.
- [CUSTOMIZATION-GUIDE.md](CUSTOMIZATION-GUIDE.md) — how to adapt the toolkit.
- [INTUNE-MY-MACS-DOCUMENTATION.md](INTUNE-MY-MACS-DOCUMENTATION.md) — generated payload catalog.
- [CONTRIBUTING.md](CONTRIBUTING.md) · [SECURITY.md](SECURITY.md) · [SUPPORT.md](SUPPORT.md).
- [.github/prompts/ship.prompt.md](.github/prompts/ship.prompt.md) — the release checklist.
- [.github/instructions/telemetry.instructions.md](.github/instructions/telemetry.instructions.md) — **mandatory** PR + ADO work-item telemetry tagging (applies on every turn).

---
> Source: [microsoft/intune-my-macs](https://github.com/microsoft/intune-my-macs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
