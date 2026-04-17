## frontend-templates

> description: "Windows-specific command guidelines to ensure proper syntax"

 ---
description: "Windows-specific command guidelines to ensure proper syntax"
globs: ["*"]
tags: ["windows", "powershell"]
priority: 1
---

# Windows Command Guidelines

## Direcctory verification

⚠️ **STOP**: You **MUST** follow instruction in this section **before runing ANY shell commands**

⚠️ **WARNING: It is ABSOLUTELY PROHIBITED to run any shell commands without first executing the *pwd* command to confirm your working directory**

- Execute *pwd*
- Verify you are in the proper working directory
- *If in the wrong directory* Execute *cd* to change to the correct working directory
- Execute your planned command

## Environment Detection
- OS: Windows 10 Pro (version 2009, build 10.0.22621.2506)
- Shell: PowerShell 7.5.0 Core
- Path Format: C:\Users\axelr\OneDrive\Documents\GitHub\frontend-templates
- Architecture: AMD64 (Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz)

## System Information
- Computer Name: SURFACEBOOK2-S6
- Model: Surface Book 2 (Microsoft Corporation)
- Total Physical Memory: 16GB
- Processor: Intel64 Family 6 Model 142 Stepping 10 (2112 MHz)
- Domain: WORKGROUP

## Command Requirements
- Use PowerShell commands and syntax
- Use backslashes for paths (C:\Users\axelr)
- Use semicolons for command chaining
- Use $env: prefix for environment variables (e.g., $env:USERPROFILE is C:\Users\axelr)

## Common Commands
```powershell
# File operations
New-Item -ItemType Directory -Path $env:USERPROFILE\Documents\NewFolder
Copy-Item -Path source.txt -Destination target.txt
Get-Content file.txt

# Process management
Get-Process
Stop-Process -Name "processname"

# System information
Get-ComputerInfo
Get-CimInstance Win32_ComputerSystem
Get-CimInstance Win32_Processor
```

## Incorrect Syntax to Avoid
```bash
# Don't use forward slashes for paths
cd /c/Users/axelr/Documents

# Don't use && for command chaining
mkdir test && cd test

# Don't use Unix-style environment variables
echo $HOME
```

## Environment Variables
Important environment variables on this system:
- USERPROFILE: C:\Users\axelr
- HOMEPATH: \Users\axelr
- HOMEDRIVE: C:
- OneDrive: C:\Users\axelr\OneDrive
- SystemRoot: C:\WINDOWS
- ProgramFiles: C:\Program Files
- TEMP: C:\Users\axelr\AppData\Local\Temp

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ericaxelrod-1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
