## claude-auto-patch

> Configurable auto-patch system for Claude Code. Applies binary patches before Claude starts via a shell wrapper.

# claude-auto-patch

Configurable auto-patch system for Claude Code. Applies binary patches before Claude starts via a shell wrapper.

## Project Structure

```
install.py              - Shell wrapper installer/uninstaller
auto_patch.py           - Patch engine (called by shell wrapper)
auto-patch-config.json  - Toggle individual patches on/off
skills/
  create-patch/         - Skill for creating new patch definitions
    SKILL.md            - Skill instructions
    scripts/            - Helper scripts for code analysis
    references/         - Patching best practices guide
```

## Key Concepts

- **Shell wrapper**: `install.py` injects a `claude()` function into shell profiles that runs patches before launching claude
- **Equal-length replacement**: Patches must be exactly the same byte length as original code
- **Three-state detection**: Each patch has `target_re` (unpatched), `patched_re` (patched), and implicit "unknown"
- **Cache mechanism**: Uses file mtime to skip re-scanning unchanged binaries
- **Platform handling**: Windows rename-swap for running exe, macOS codesign

---
> Source: [Cedriccmh/claude-auto-patch](https://github.com/Cedriccmh/claude-auto-patch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
