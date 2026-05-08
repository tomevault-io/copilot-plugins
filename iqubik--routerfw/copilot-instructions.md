## file-header

> File header convention — every builder-owned script must start with a file path comment


# File Header Convention

Every file that belongs to the routerFW build system must have a **file path comment** near the top.

## Format by file type

| File type | Header format | Position |
|---|---|---|
| `.sh` | `# file: relative/path/to/file.sh` | line 2, after `#!/bin/bash` or `#!/bin/sh` |
| `.bat` | `rem file: relative\path\to\file.bat` | line 2, after `@echo off` |
| `.ps1` | `# file: relative/path/to/file.ps1` | line 1, or inside `<# ... #>` block |
| `.env` | `# file: relative/path/to/file.env` | line 1 |
| `.yaml` | `# file: relative/path/to/file.yaml` | line 1 |

## Rules

- Path is **relative to repository root** (forward slashes on all platforms)
- No version suffix required (unlike ib_builder.sh / src_builder.sh which carry `v1.x`)
- Do NOT add a file header to: generated output, user profile `.conf` files, `firmware_output/`

## Examples

```bash
#!/bin/bash
# file: system/create_profile.sh
```

```batch
@echo off
rem file: _Builder.bat
```

```
# file: system/lang/ru.env
```

---
> Source: [iqubik/routerFW](https://github.com/iqubik/routerFW) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
