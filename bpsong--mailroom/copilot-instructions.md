## mailroom

> - Workspace root: [`d:/python_code/mailroom`](AGENTS.md)

# Workspace Agent Notes

## Environment

- Workspace root: [`d:/python_code/mailroom`](AGENTS.md)
- Operating system: Windows
- Default shell in this workspace is commonly [`cmd.exe`](AGENTS.md), but PowerShell is preferred for reliable test execution.

## Python and Pytest Execution

- Prefer explicit interpreter invocation instead of bare `pytest`.
- Recommended command pattern:
  - `C:\Python313\python.exe -m pytest tests\unit\test_config_settings.py -v`
- Do not rely on bare `pytest`, because interpreter resolution may differ from `python` on this machine.

## Known Windows PATH Issue

- This workspace has experienced pytest hangs/crashes caused by shell `PATH` contamination.
- The main issue is Git Unix tooling in `d:\Program Files\Git\usr\bin` interfering with Python/pytest runtime behavior.
- Symptom observed: pytest appears to hang or crashes before normal output.

## Recommended Test Shell Setup

Use PowerShell and reset `PATH` before running pytest:

```powershell
$env:PATH='C:\Python313;C:\Python313\Scripts;C:\Windows\System32;C:\Windows;C:\Windows\System32\Wbem'
$env:SECRET_KEY='test-secret-key'
$env:APP_ENV='testing'
C:\Python313\python.exe -m pytest tests\unit\test_config_settings.py tests\unit\test_auth_service.py tests\test_security_fixes.py -v
```

## Playwright Usage

- Playwright is available in this workspace via [`@playwright/test`](package.json:1).
- Chromium can be installed with:
  - `npx playwright install chromium`
- Use PowerShell with a minimal Node/Windows `PATH` when invoking Playwright commands.

### Recommended Playwright Setup

```powershell
$env:PATH='D:\Program Files\nodejs;C:\Windows\System32;C:\Windows;C:\Windows\System32\Wbem'
npx playwright install chromium
```

### Running the App for Browser Automation

Start the FastAPI app in a separate PowerShell session with a clean Python `PATH`:

```powershell
$env:PATH='C:\Python313;C:\Python313\Scripts;C:\Windows\System32;C:\Windows;C:\Windows\System32\Wbem'
$env:SECRET_KEY='test-secret-key'
$env:APP_ENV='development'
$env:LOG_LEVEL='INFO'
C:\Python313\python.exe -m app.main
```

### Login Smoke Test

- A reusable Playwright login smoke script exists at [`scripts/playwright_login_smoke.js`](scripts/playwright_login_smoke.js).
- It expects the local app to be running at `http://127.0.0.1:8000`.
- Current smoke-test account used for validation:
  - username: `playwright_user`
  - password: `Playwright123!`

Run it with:

```powershell
$env:PATH='D:\Program Files\nodejs;C:\Windows\System32;C:\Windows;C:\Windows\System32\Wbem'
node .\scripts\playwright_login_smoke.js
```

Expected success output:

```text
LOGIN_OK title=Dashboard - Mailroom Tracker url=http://127.0.0.1:8000/dashboard
```

## Notes for Future Agents

- If pytest hangs, suspect environment/toolchain issues before suspecting application code such as [`app/config.py`](app/config.py) or [`tests/conftest.py`](tests/conftest.py).
- A virtual environment is optional for this workspace, not mandatory, as long as PowerShell uses a clean `PATH` and pytest is invoked via explicit Python.
- Pydantic deprecation warnings currently appear from model files and are unrelated to the pytest environment issue.
- For browser automation, keep Node commands isolated from Git Unix tooling in `PATH` just like Python test commands.

---
> Source: [bpsong/mailroom](https://github.com/bpsong/mailroom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
