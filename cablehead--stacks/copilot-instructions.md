## stacks

> To bump the current version:

## Version Bumping

To bump the current version:

1. Update version in `src-tauri/Cargo.toml` (line 3)
2. Update version in `src-tauri/tauri.conf.json` (line 4)
3. Run `cargo check` to update Cargo.lock
4. Both files must have matching versions

Example: `0.15.14-dev` → `0.15.15-dev`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cablehead) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
