## unreal-home-wizard

> This repository distributes the Unreal Home Wizard Codex skill and its small,

# Repository rules

This repository distributes the Unreal Home Wizard Codex skill and its small,
auditable Windows helpers.

- Never commit client photographs, addresses, credentials, paid assets, Unreal
  caches, packaged projects, or generated client scenes.
- Keep the public workflow usable by a beginner. Explain unavoidable user actions
  in plain language.
- Do not claim that Codex can bypass Epic sign-in, license acceptance, installation
  dialogs, hardware limits, or asset licenses.
- Prefer Unreal's built-in Python editor automation. Do not add a third-party MCP
  or plugin unless its version, license, maintenance status, and network exposure
  have been reviewed and documented.
- Treat a successful script run as a prerequisite check, not proof of visual
  fidelity. Matched-camera review and a manual walkthrough remain mandatory.
- Preserve strict reconstruction mode: visible architecture and dominant objects
  must match the supplied references, and critical or major defects block release.

Before publishing a release, run:

```powershell
./tools/verify-public-repo.ps1
./skills/unreal-home-wizard/scripts/preflight.ps1
```

---
> Source: [amirmushichge/unreal-home-wizard](https://github.com/amirmushichge/unreal-home-wizard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
