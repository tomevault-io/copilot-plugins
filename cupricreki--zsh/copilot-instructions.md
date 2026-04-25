## zsh

> zsh config migration convention — create migrations for breaking changes


# zsh Config Migrations

When making a breaking change to the zsh config, always create a migration script in `migrations/`.

## What counts as a breaking change

- Removing or renaming a file/directory that existing installs may have cached
- Adding a required new tool not covered by preflight-check
- Renaming an env var, or changing its value in a way that breaks code already depending on the old value (e.g. a path that code outside zshenv reads directly)
- Moving config to a new path (e.g., sheldon config dir)
- Removing a cache file that must be regenerated

## What does NOT require a migration

- **Additive PATH entries** — adding a new `dir_exists ... && path=(...)` line only activates if the directory exists; it cannot break existing installs
- **Removing a dead PATH entry** — if the removed path never existed on real installs (e.g. guarded by `dir_exists` and always false), removing it changes nothing
- **GOPATH / env var value changes that are purely additive** — e.g. prepending a new entry to a colon-separated list while keeping the old value; existing installs lose nothing
- In general: if the migration body would be a no-op on every existing install, skip it — empty migrations are noise

The test is **"does any existing install lose something it had before?"** If no, no migration is needed.

## How to write a migration

1. Find the highest existing number in `migrations/` and increment it
2. Name the file `NNN_short_description.zsh` (zero-padded to 3 digits)
3. Write idempotent zsh — check before acting, use `log info/warning/error`
4. Signal failure with `return 1` — this prevents `run-migrations` from marking the migration as applied, so it retries on the next run
5. Wrap the body in a private function to properly scope `local` variables — `local` is a no-op in sourced files outside of a function

Simple example (no locals needed):
```zsh
# Migration 003: Remove stale dircolors cache after format change
[[ -f "${ZSH_CACHE_DIR}/dircolors.sh" ]] && rm -f "${ZSH_CACHE_DIR}/dircolors.sh"
```

Example with locals and failure handling:
```zsh
# Migration 004: Example with locals and failure signalling
function _migration_004() {
  local target="${ZSH_CACHE_DIR}/some-cache"
  if [[ -f "$target" ]]; then
    rm -f "$target" || { log error "migration 004: failed to remove $target"; return 1; }
    log info "migration 004: removed stale cache"
  fi
}
_migration_004
unset -f _migration_004
```

## Available helpers (from zshenv)

- `log info|warning|error "message"` — coloured output
- `command_exists cmd` — check if a command is on PATH
- `$ZSH_DIR`, `$ZSH_CACHE_DIR`, `$SHELDON_CONFIG_DIR`, `$SHELDON_DATA_DIR`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CupricReki) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
