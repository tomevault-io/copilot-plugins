## dshidmini

> Use the repo NUKE scripts for every build


# Use NUKE to build

When a build, rebuild, compile, or target run is required, use the provided NUKE
scripts. Do not invent an alternative (`dotnet build` / `dotnet msbuild` on the
solution or projects, raw `MSBuild.exe`, Visual Studio, CMake, or ad-hoc `cl`).

- Windows: `.\build.cmd` (or `.\build.ps1`)
- Unix: `./build.sh`

Pass NUKE targets and parameters as script arguments, for example:

```
.\build.cmd Compile
.\build.cmd Compile --target-platform x64
.\build.cmd PublishControlApp
```

Default with no arguments is `Compile` (which depends on `BuildDmf`). Inspect
targets in `build/Build.cs` if unsure; do not bypass that entry point.

## Exception: the setup installer project

`setup/DsHidMini.Installer.csproj` is listed in `dshidmini.sln` with `ActiveCfg`
entries only (no `Build.0`), so it is excluded from the solution build and no NUKE
target compiles it. The setup workflow is otherwise the first thing that ever
compiles it, which turns a typo into a failed dispatch.

Before pushing changes under `setup/`, compile that project directly:

```
dotnet build -c Release .\setup\DsHidMini.Installer.csproj
```

This stays a plain compile: the `PostBuild` target that runs the installer to emit
the MSI only fires with `-p:GenerateMsi=true`, which remains CI-only.

---
> Source: [nefarius/DsHidMini](https://github.com/nefarius/DsHidMini) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
