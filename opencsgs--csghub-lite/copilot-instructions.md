## platform-sensitive-files

> Multi-platform requirements for Go and install script changes


# Multi-Platform Support

- Changes in Go, shell, or PowerShell must work on macOS, Linux, and Windows.
- Never hardcode path separators, home directories, or binary suffixes; use `filepath.Join()`, `os.UserHomeDir()`, `exec.LookPath()`, `runtime.GOOS`, and `os.PathListSeparator`.
- Do not assume `bash`, `sudo`, `/usr/local/bin`, or Unix-only commands exist on Windows.
- If install or uninstall behavior changes, update both `scripts/install.sh` and `scripts/install.ps1`.
- If the UI or docs show install commands, either provide platform-specific variants or clearly label them as previews or OS-specific examples.

```go
// ❌ BAD
path := home + "/.csghub-lite/models"

// ✅ GOOD
path := filepath.Join(home, ".csghub-lite", "models")
```

---
> Source: [OpenCSGs/csghub-lite](https://github.com/OpenCSGs/csghub-lite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
