## autocad-installer-build

> AutoCAD MSI build rules for installer files


# AutoCAD Installer Rules

- Treat `installers/msi/Build-Installer.ps1` as the source of truth for MSI builds.
- Do not run `dotnet build installers/msi/DeepBimMCP.AutoCAD.Installer.wixproj` directly for normal MSI builds. That bypasses version-specific framework selection and generated `PackageContents.xml`.
- Do not proactively build MSI packages just because installer files changed. Only build when the user explicitly asks, or when verification is necessary for the requested task.
- When you do need an MSI build, run it from the repo root with:
  - `.\installers\msi\Build-Installer.ps1 -AutoCADVersion 2024`
  - `.\installers\msi\Build-Installer.ps1 -AutoCADVersion 2026`
  - `.\installers\msi\Build-Installer.ps1 -AutoCADVersion 2027`
- Framework mapping is fixed:
  - AutoCAD `2018-2024` -> `net48`
  - AutoCAD `2025-2026` -> `net8.0-windows`
  - AutoCAD `2027` -> `net10.0-windows`
- `PackageContents.xml` is generated per target version. Do not hand-edit files under `installers/msi/obj/generated/**`; edit `PackageContents.Legacy.xml`, `PackageContents.Modern.xml`, or the generator logic in `Build-Installer.ps1`.
- For installed MSI flows, the expected behavior is bundle autoload on AutoCAD startup. `NETLOAD` is a fallback for workstation-specific issues, not the primary installer flow.
- If the task only needs packaging from existing plugin/server outputs, prefer:
  - `.\installers\msi\Build-Installer.ps1 -AutoCADVersion <year> -SkipBuild -SkipNpmPrune`

---
> Source: [nguyenngocdue/DeepBIM-MCP-Autocad-Plugin](https://github.com/nguyenngocdue/DeepBIM-MCP-Autocad-Plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
