## global

> UTF-8 encoding configuration: Always use UTF-8 encoding for all file operations and terminal commands.

UTF-8 encoding configuration: Always use UTF-8 encoding for all file operations and terminal commands.

On Windows (PowerShell): whenever a terminal command is executed, $OutputEncoding = [Console]::OutputEncoding = [System.Text.Encoding]::UTF8 first and then run together

On Linux/Docker: ensure LANG=fr_FR.UTF-8 and LC_ALL=fr_FR.UTF-8 are set. All files must be written with UTF-8 encoding (no BOM).

When creating or modifying files, always explicitly use UTF-8 encoding to preserve special characters, emojis, and accented characters correctly.

---
> Source: [usts-ai/481786](https://github.com/usts-ai/481786) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
