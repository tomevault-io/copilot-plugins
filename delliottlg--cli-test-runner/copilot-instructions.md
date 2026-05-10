## cli-test-runner

> Minimal CLI tool for running Lingraphica Unity tests on Android devices. Single Python file, zero dependencies.

# cli-test-runner

Minimal CLI tool for running Lingraphica Unity tests on Android devices. Single Python file, zero dependencies.

## Quick Start

```bash
# List connected devices
python3 run.py --list-devices

# List test categories
python3 run.py --list-categories

# Run Sanity tests (builds APK, deploys, runs, pulls results)
python3 run.py --device R5GYB3421PN --category Sanity --yes

# Skip build (reuse existing APK)
python3 run.py --device R5GYB3421PN --category Sanity --skip-build --yes

# Deploy and run only (no build)
python3 run.py --device R5GYB3421PN --deploy-only --yes

# Multiple categories
python3 run.py -d R5GYB3421PN -c Login_Section -c Dashboard_Section -y
```

## What It Does (6 steps)

1. **Build** — Calls Unity batch mode to build test APK with baked-in category
2. **Uninstall** — Removes old APK from device
3. **Install** — Pushes new APK to device
4. **Permissions** — Grants storage, camera, audio (uses `appops` for Android 16+)
5. **Launch** — Starts test activity on device
6. **Poll & Pull** — Watches for XML results, pulls when done, parses and prints summary

## Exit Codes
- `0` — All tests passed
- `1` — Failures or errors
- `130` — Interrupted (Ctrl+C)

## Output
Plain text to stdout. Results are NUnit XML saved to `results/YYYY-MM-DD/`.

## Prerequisites
- Python 3.9+
- ADB (`brew install android-platform-tools`)
- Unity 6000.3.2f1 at `/Applications/Unity/Hub/Editor/6000.3.2f1/`
- Unity project at `/Users/delliott/Documents/GitHub/lingraphica-app`
- Device with USB debugging enabled, test_account.txt and azure.txt on device

## Config
All constants are at the top of `run.py`. Edit UNITY, PROJECT, PKG paths there.

## TODO: QA Reports Upload

**Status**: Not yet implemented. Next step is adding the 3-step upload flow to run.py.

### API (https://qa-reports.vercel.app)

After tests complete and XML is parsed, upload results in 3 steps:

1. **Create Suite** — `POST /api/test-suite/create`
   ```json
   {
     "name": "Sanity Run",
     "device_info": {"model": "SM-X230"},
     "is_scheduled": false
   }
   ```
   Returns `suite_id` (e.g. "TS-1"). Save it.

2. **Add Run** (one per category) — `POST /api/test-suite/{suite_id}/add-run`
   ```json
   {
     "execution_id": "unique-uuid",
     "test_category": "Sanity",
     "device_info": {"model": "SM-X230"},
     "is_scheduled": false,
     "timestamp": "2026-02-24T02:30:00+00:00",
     "summary": {"total_tests": 18, "passed": 17, "failed": 1, "duration": 245.3},
     "test_results": [
       {"test_name": "LoginTest.Verify", "result": "passed", "duration": 12.5, "error_message": ""},
       {"test_name": "LoginTest.Invalid", "result": "failed", "duration": 8.2, "error_message": "Expected error dialog"}
     ]
   }
   ```

3. **Complete Suite** — `POST /api/test-suite/{suite_id}/complete` (no body needed)

### Implementation Notes

- Use `urllib.request` (stdlib) to keep zero-dependency constraint
- Add `--scheduled` flag — sets `is_scheduled: true` so results appear on main dashboard with charts. Default is `false` (ad hoc, shows on Test Log only).
- Add `--no-upload` flag to skip uploading if needed
- Parse XML test cases into the `test_results` array format (test_name, result, duration, error_message)
- `device_info.model` comes from `adb shell getprop ro.product.model`

### Scheduling Decision

**Do NOT add scheduling to run.py.** Keep it a dumb single-shot CLI tool.

Scheduling should be built into qa-reports (the web app) instead — it already has a `GET /api/schedules` endpoint. Add a UI there where Jyotsna/Troy can pick device + categories + time. Then a tiny daemon or cron on the test lab machine polls `/api/schedules` and kicks off `run.py` when it's time.

This keeps run.py simple and puts the scheduling UI where non-technical users can access it (the web app they already use).

### Reference: qa_testrunner (the big version)

The full GUI version lives at `~/Documents/GitHub/qa_testrunner`. It uses PyQt6, APScheduler, Slack notifications, S3 uploads — ~2500+ lines across many files. This CLI tool is the minimal replacement. Key files for reference:
- `qa_testrunner/src/core/api_client.py` — API upload logic
- `qa_testrunner/src/utils/constants.py` — endpoints and constants
- `qa_testrunner/src/ui/tabs/scheduling_tab.py` — how scheduling worked in the GUI

---
> Source: [delliottlg/cli-test-runner](https://github.com/delliottlg/cli-test-runner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-27 -->
