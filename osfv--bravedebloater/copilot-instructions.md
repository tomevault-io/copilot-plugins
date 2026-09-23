## bravedebloater

> BraveDebloater is a PowerShell 5.1-compatible Brave Browser policy tool.

# AGENTS.md

BraveDebloater is a PowerShell 5.1-compatible Brave Browser policy tool.

PowerShell is the cross-platform runtime. Keep platform-specific writes native:

- Windows: Brave registry policy keys.
- macOS: `defaults` or managed plist policy payloads.
- Linux: managed JSON policy files.
- Android/iOS: export MDM payloads only; do not add local device writes.

## Code Discovery

Use codebase-memory-mcp before text search for code structure:

1. `search_graph`
2. `trace_path`
3. `get_code_snippet`
4. `query_graph`
5. `search_code`

Use `rg` for docs, configs, literal strings, and when the graph is not enough.

## Safety Rules

Do not add policies that disable Brave updates, disable Brave Shields, add Shield allowlists, weaken Safe Browsing, or remove extensions.

Keep dry-run as the default. `-Apply` must be required before writes.

Keep backup and restore validation strict. Restores must stay limited to Brave policy targets and selected Brave profile `Preferences` files.

Profile preference cleanup must not run while Brave is open.

## Implementation Notes

Keep `Invoke-BraveDebloat.ps1` as the thin entrypoint. Put shared behavior in `src/*.ps1`.

Use existing helpers before adding new ones.

Keep user-facing messages short, concrete, and action-oriented. Say what happened, whether anything changed, and what to do next.

When adding or changing policies, update:

- `config/policies.json`
- `docs/debloatable-validation.md`
- `scripts/Test-PolicyManifest.ps1` or `scripts/Test-LatestPolicyTemplates.ps1` if validation rules change

## Checks

Run the local checks before handing off:

```powershell
pwsh -NoProfile -File scripts/Test-PolicyManifest.ps1
pwsh -NoProfile -File scripts/Test-Behavior.ps1
```

When a Brave policy template zip is available, also run:

```powershell
pwsh -NoProfile -File scripts/Test-LatestPolicyTemplates.ps1 -TemplateZipPath /tmp/brave-policy-templates.zip
```

---
> Source: [osfv/BraveDebloater](https://github.com/osfv/BraveDebloater) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
