## pdf-edit-from-context-menu

> This repo is a Windows tray app for PDF splitting and related file actions. After making code changes, agents should update the running app before finishing the task so the user can test the current build immediately.

# AGENTS.md

## Purpose
This repo is a Windows tray app for PDF splitting and related file actions. After making code changes, agents should update the running app before finishing the task so the user can test the current build immediately.

## End-Of-Task Update Rule
Before finishing any implementation task:

1. Run a quick validation pass for the changed area.
2. Update the running tray app so the live UI uses the new code.
3. Confirm the app restarted correctly.
4. Do not just leave code changes on disk without updating the app.

The user does not need process start times in the final response. It is enough to confirm that the app was restarted correctly.

## Validation
Use the lightest validation that matches the change:

- For Python code changes, run `python -m compileall app tests` when relevant.
- Run targeted tests when they exist, usually `python -m unittest tests.test_pdf_ops`.
- If no automated check is practical, say so plainly and still update the running app.

## Normal Update Flow: Restart Tray Only
Use a simple tray restart after most code-only changes, including:

- UI behavior changes
- split/combine/export logic changes
- logging changes
- non-installer Python changes

Recommended restart flow from the repo root:

```powershell
Get-CimInstance Win32_Process |
  Where-Object { $_.Name -match 'pythonw?\.exe' -and ($_.CommandLine -match 'run_tray\.py' -or $_.CommandLine -match 'app\.tray_runtime') } |
  ForEach-Object { Stop-Process -Id $_.ProcessId -Force }

Start-Sleep -Seconds 1

$py = (python -c "import sys, pathlib; print(pathlib.Path(sys.executable).with_name('pythonw.exe'))").Trim()
Start-Process -FilePath $py -ArgumentList '"run_tray.py"' -WorkingDirectory '.' -WindowStyle Hidden
```

Then verify there is one live tray process:

```powershell
Get-CimInstance Win32_Process |
  Where-Object { $_.Name -match 'pythonw?\.exe' -and ($_.CommandLine -match 'run_tray\.py' -or $_.CommandLine -match 'app\.tray_runtime') } |
  Select-Object ProcessId, Name, CommandLine
```

If multiple tray processes are present, stop them all and start one clean instance.

## Full Refresh Flow: Quit, Uninstall, Reinstall
Use the full uninstall/reinstall flow only when changes affect installed integration points, including:

- `scripts/install.ps1`
- `scripts/uninstall.ps1`
- tray startup behavior
- Explorer context menu registration
- launcher entrypoints used by shell integration
- anything the user explicitly asks to reinstall

Preferred full refresh:

1. Stop the running tray app first.
2. Run `powershell -ExecutionPolicy Bypass -File .\scripts\uninstall.ps1`
3. Run `powershell -ExecutionPolicy Bypass -File .\scripts\install.ps1`
4. Verify the tray app is running afterward.

Notes:

- Do not run uninstall/reinstall for ordinary UI or logic changes; a tray restart is usually sufficient and faster.
- If the install script already launches the tray app, do not launch a second manual copy.
- If restart verification shows duplicates, stop all tray processes and launch one clean instance.

## Final Response Expectation
When closing out a task, mention:

- what changed
- what validation was run
- that the app was restarted correctly

Do not include process timestamps unless the user asks for them.

---
> Source: [StBenedictPrayForUs/PDF-Edit-From-Context-Menu](https://github.com/StBenedictPrayForUs/PDF-Edit-From-Context-Menu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
