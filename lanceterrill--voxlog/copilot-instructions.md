## voxlog

> > Nebraska Department of Insurance — NITC 8-609 Compliant Voicemail Transcription Pipeline

# VoxLog Skill v1.2.1
> Nebraska Department of Insurance — NITC 8-609 Compliant Voicemail Transcription Pipeline

---

## Project Identity
- **Repo:** `lanceterrill/VoxLog` (GitHub) + `\\stnnas01.stone.ne.gov\DOIhome$\lance.terrill\Profile\Documents\GitHub\VoxLog`
- **Version:** `1.2.1` (always check `config.json` → `app.version`)
- **Commit convention:** All commits must start with `[v1.2.1]` matching `config.json` version
- **Environment:** Nebraska GovCloud / M365 / OCIO NITC 8-609
- **Tenant:** `usgovtexas` (Power Automate GovCloud)
- **SharePoint:** `stateofne.sharepoint.com/sites/DOI.SHIP`
- **Mailbox:** `DOI.SHIP@nebraska.gov`
- **Machine 1 (deployed):** `BB05260001` — `lance.terrill@stone.ne.gov`
- **Machines remaining:** 4

---

## Pipeline Architecture

```
Voicemail (uc.allovmail.com)
  → DOI.SHIP inbox
  → Flow A (Power Automate)
      ParseCaller / ParsePhone / TodayDate (yyyy-MM-dd)
      For each attachment:
        Create file → SharePoint VoxLog/Voicemails/Pending/
        Send email (V2) → DOI.SHIP with WAV + file path in body
      Subject: "VoxLog Ingest | CALLER | PHONE | DATE"
  → voxlog_transcribe.py (Outlook COM, polls every 15s)
      Downloads WAV → C:\VoxLog\Temp\
      Transcribes via faster-whisper (127.0.0.1:5001)
      Builds filename: YYYYMMDD_HHMMSS_CallerName_PhoneNumber.wav
      Sends plain-text result email (BodyFormat=1)
      Subject: "VoxLog Result | [filename]"
  → Flow B (Power Automate)
      Filename / CallerName / Phone / CallDate / CallTime (h:mm tt)
      WAVLink: concat(SP_BASE, outputs('Filename'))
      CleanBody: triggerOutputs()?['body/body'] (full plain text)
      Create item → VoxLog MS List
      Delete email (V2)
      Post Teams message → VoxLog channel
```

---

## Key Files

| File | Purpose |
|------|---------|
| `voxlog_tray.py` | Tray app — single instance mutex, manages Whisper + Transcribe subprocesses |
| `voxlog_transcribe.py` | Outlook COM polling, faster-whisper transcription, result email |
| `whisper_server.py` | Local Flask server wrapping faster-whisper |
| `install_voxlog.ps1` | Full installer — downloads from GitHub, PyInstaller compile, Task Scheduler |
| `voxlog.ico` | NDOI navy/gold bold V icon |
| `config.json` | Version, settings, commit convention |
| `start_voxlog.bat` | Legacy launcher (Task Scheduler now points to exe) |

---

## Local Paths (Each Workstation)

```
C:\VoxLog\
├── App\                ← Installed files + voxlog_tray.exe
├── Processed\          ← WAVs after transcription
├── Temp\               ← WAVs during processing
└── Logs\
    ├── transcribe.log
    └── pyinstaller_build.log
```

---

## MS List Schema

| Column | Type | Source |
|--------|------|--------|
| CallDate | Date | Flow B — from filename |
| CallTime | Text | Flow B — `h:mm tt` format |
| PhoneNumber | Text | Flow B — from filename |
| CallerName | Text | Flow B — from filename |
| Transcription | Multi-line | Flow B — full email body |
| WAVLink | Hyperlink | Flow B — constructed URL |
| OriginalFilename | Text | Flow B |
| Status | Choice | Default: Pending |
| StaffName | Person | Staff selects from directory |
| ReturnedCallDate | Date | Staff |
| ReturnedCallTime | Text | Staff |
| Notes | Multi-line | Staff |

**WAVLink base URL:**
`https://stateofne.sharepoint.com/sites/DOI.SHIP/Shared%20Documents/VoxLog/Voicemails/Pending/`

**WAVLink column formatting:** 🔊 Listen (JSON in README)

---

## Flow B — Key Expressions

```
Filename:   trim(last(split(triggerOutputs()?['body/subject'], '| ')))
CallerName: trim(split(outputs('Filename'), '_')[2])
Phone:      trim(replace(split(outputs('Filename'), '_')[3], '.wav', ''))
CallDate:   concat(sub(0,4),'-',sub(4,2),'-',sub(6,2)) from filename
CallTime:   formatDateTime(reconstructed ISO from filename, 'h:mm tt')
WAVLink:    concat('https://stateofne.sharepoint.com/sites/DOI.SHIP/Shared%20Documents/VoxLog/Voicemails/Pending/', outputs('Filename'))
CleanBody:  triggerOutputs()?['body/body']
```

---

## Critical Fixes Applied (v1.2.1)

| Fix | File | Detail |
|-----|------|--------|
| Single instance mutex | `voxlog_tray.py` | `ctypes` Windows named mutex `Global\VoxLog_SingleInstance`, `os._exit(0)` |
| PYTHON_EXE for subprocesses | `voxlog_tray.py` | `shutil.which("python")` when `sys.frozen` — prevents PyInstaller exe launching itself |
| APP_DIR frozen path | `voxlog_tray.py` | `sys.executable` path when frozen, `__file__` otherwise |
| Plain text result email | `voxlog_transcribe.py` | `mail.BodyFormat = 1` before `mail.Body` |
| RPC reconnect logic | `voxlog_transcribe.py` | Catches `hresult == -2147023174`, exponential backoff, re-dispatches COM |
| Flow B rebuilt | Power Automate | Create item (not update) — Flow A no longer creates MS List rows |
| bodyPreview → body | Flow B CleanBody | Full transcript, not truncated 255-char preview |
| TodayDate format | Flow A | `formatDateTime(utcNow(), 'yyyy-MM-dd')` |
| CallTime format | Flow B | `formatDateTime(..., 'h:mm tt')` for AM/PM |
| WAVLink URL | Flow B | Built from filename, not passed through email |

---

## Installer Behavior

```powershell
# Standard install
powershell -ExecutionPolicy Bypass -File "\\stnnas01...\install_voxlog.ps1"

# Force exe rebuild
Remove-Item "C:\VoxLog\App\voxlog_tray.exe" -Force
# then run installer

# Kill + restart
Get-Process | Where-Object { $_.Name -match "voxlog" } | Stop-Process -Force
Start-Process "C:\VoxLog\App\voxlog_tray.exe"
```

The installer:
1. Creates folders
2. Downloads latest from GitHub (not local repo)
3. Checks Python deps
4. PyInstaller compile with `--windowed --icon=voxlog.ico` (skips if exe newer than source)
5. Registers Task Scheduler — `LogonType Interactive`, `RunLevel Limited`
6. Verifies all files

---

## Emergency Shutdown

```powershell
Stop-ScheduledTask -TaskName "VoxLog Local"
Get-Process | Where-Object { $_.Name -match "voxlog|pythonw|python" } | Stop-Process -Force
```

---

## Known Issues / Watch List

- **Duplicate rows** — caused by multiple VoxLog instances; mutex fix resolves on fresh start
- **Caller disclosure** — NITC 8-609 gap; callers not notified of AI transcription (AI Security rating: B+)
- **Task Scheduler logon** — exe sometimes crashes at logon before Outlook is ready; use `Start-Process` for manual launches
- **WAVLink on older rows** — rows created before WAVLink fix have 404 links; leave as-is

---

## Rollout Status

| Machine | Status | Notes |
|---------|--------|-------|
| BB05260001 | ✅ Deployed | lance.terrill — production confirmed |
| TBD | ⏳ Pending | |
| TBD | ⏳ Pending | |
| TBD | ⏳ Pending | |
| TBD | ⏳ Pending | |

---

## Rules for Claude

1. Always check `config.json` version before suggesting commit messages
2. Commit format: `[v1.2.1] type: description`
3. All files deploy via GitHub → installer, not direct copy
4. Never suggest Graph API or external AI APIs — NITC 8-609 compliance
5. Standard M365 connectors only in Power Automate (no premium)
6. Keep overhead low — no folder choreography, status lives in MS List only
7. WAVs stay in `Pending/` folder permanently — no moving based on status
8. When patching Python files, always push to GitHub first, then reinstall
9. Single instance is enforced by mutex — if multiple processes appear, kill all and start fresh
10. `bodyPreview` = 255 char truncated; use `body` for full transcript

---
> Source: [lanceterrill/VoxLog](https://github.com/lanceterrill/VoxLog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
