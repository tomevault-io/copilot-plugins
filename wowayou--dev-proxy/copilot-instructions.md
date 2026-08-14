## dev-proxy

> This directory is a standalone ops tool. Keep it decoupled from the `cc-switch` repository.

# Dev Proxy Agent Guide

This directory is a standalone ops tool. Keep it decoupled from the `cc-switch` repository.

## Boundaries

- Do not edit CC Switch databases or source files from this tool.
- Do not edit Claude, Codex, OpenCode, or other provider config files.
- Do not read, write, transform, log, or validate API keys.
- Do not add automatic elevation. WinHTTP and `.wslconfig` changes should remain explicit user actions.
- Do not change proxy-client configuration. The tool may tell the user when LAN or non-loopback listening is required.

## Safe Edit Rules

- Keep `dev-proxy.ps1`, `config.json`, `templates/wsl-proxy-env.sh`, and local docs self-contained.
- Preserve `enableWslMirrored` as a visible preference, not a hidden compatibility field.
- Preserve WSL commands: `proxy_status`, `proxy_refresh`, and `proxy_off`.
- Preserve dynamic NAT host detection. Use `DEV_PROXY_HOST_OVERRIDE` only as an intentional fixed-host override.
- Suppress noisy localized `netsh` output and WSL PATH translation warnings in tool output.
- Avoid coupling validation to live provider credentials. HTTP `401`, `403`, or `404` can still be a connectivity pass.

## Mirrored Vs NAT Reasoning

Mirrored networking lets WSL use the Windows proxy listener at `127.0.0.1:<port>`. NAT mode cannot rely on Windows localhost, so the WSL profile falls back to the default-route gateway, often similar to `172.17.0.1`.

When NAT fallback is active, the Windows proxy client must accept non-loopback connections. That usually means LAN access, a `0.0.0.0` listener, and compatible firewall rules.

## Validation Commands

Run from `C:\Users\Public\ops-tools\dev-proxy` in PowerShell unless noted.

PowerShell parser check:

```powershell
$tokens = $null
$errors = $null
[System.Management.Automation.Language.Parser]::ParseFile((Resolve-Path .\dev-proxy.ps1), [ref]$tokens, [ref]$errors) | Out-Null
if ($errors) { $errors; exit 1 }
```

JSON parse check:

```powershell
Get-Content .\config.json -Raw | ConvertFrom-Json | Out-Null
```

WSL template syntax check:

```powershell
wsl.exe -- bash -lc "bash -n /mnt/c/Users/Public/ops-tools/dev-proxy/templates/wsl-proxy-env.sh"
```

Dry-run with the saved mirrored preference:

```powershell
.\dev-proxy.ps1 -NonInteractive -DryRun
```

Dry-run with mirrored disabled, then restore the original value:

```powershell
$config = Get-Content .\config.json -Raw | ConvertFrom-Json
$original = $config.enableWslMirrored
$config.enableWslMirrored = $false
$config | ConvertTo-Json -Depth 4 | Set-Content .\config.json -Encoding UTF8
.\dev-proxy.ps1 -NonInteractive -DryRun
$config.enableWslMirrored = $original
$config | ConvertTo-Json -Depth 4 | Set-Content .\config.json -Encoding UTF8
```

Interactive smoke checks:

- Option 1 saves the proxy target.
- Option 4 explains mirrored networking and NAT fallback, then persists `enableWslMirrored`.
- Option 5 reports Windows and WSL verification without raw localized `netsh` noise.
- Option 6 prints CC Switch paths only; it does not edit CC Switch.

## Static Checks

Before finishing, scan for temporary notes, debug-only output, progress UI calls, and markdown formatting mistakes. Keep the docs themselves free of those marker strings so the scan remains actionable.

---
> Source: [wowayou/dev-proxy](https://github.com/wowayou/dev-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
