## rebuild-and-run-app

> After code changes, stop the old app process, rebuild, and launch the new build


# Rebuild and replace running Media Notification

Whenever you change anything that affects the built app (source, project, XAML, assets), finish by **replacing the running instance** with a fresh build. Do not leave a stale `MediaNotification.exe` running.

## Steps (run in PowerShell from the repo root)

1. **Stop the old process** so the file can be overwritten and only one instance runs:
   - `Stop-Process -Name MediaNotification -ErrorAction SilentlyContinue`
   - If that fails (different name), use `Get-Process MediaNotification` and stop by id, or `taskkill /IM MediaNotification.exe /F`

2. **Build** the current sources:
   - `dotnet build MediaNotification.sln -c Release`  
   - Fix any errors before launching.

3. **Run the new build** (path matches TFM in the `.csproj`, e.g. `net8.0-windows10.0.19041.0`):
   - `Start-Process -FilePath ".\src\MediaNotification\bin\Release\<TFM>\MediaNotification.exe" -WorkingDirectory (Get-Location)`

Or in one line after `cd` to the repo:

`Stop-Process -Name MediaNotification -ErrorAction SilentlyContinue; dotnet build MediaNotification.sln -c Release; if ($?) { Start-Process ".\src\MediaNotification\bin\Release\net8.0-windows10.0.19041.0\MediaNotification.exe" }`

Adjust the `net8.0-windows10.0.*` folder if the project TFM changes—read `TargetFramework` from `src/MediaNotification/MediaNotification.csproj` if unsure.

## Notes

- Skip kill/start only if the change cannot affect the binary (e.g. docs-only) or the user explicitly asked not to run the app.
- Prefer **Release** for parity with how you last shipped the exe unless debugging requires Debug.

---
> Source: [justAleks0/Media-Notification](https://github.com/justAleks0/Media-Notification) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
