## govips

> Go bindings for libvips, the fast image processing library.

# govips

Go bindings for libvips, the fast image processing library.

## Build & Test

```bash
# Build
go build ./...

# Test (includes coverage)
make test

# Clean caches if builds act weird
make clean-cache
```

Requires libvips-dev installed (`brew install vips` on macOS, `apt-get install libvips-dev` on Linux).

## Dev Flow

Flow: worktree
- All code changes happen in worktrees, never on main
- Use /dev to start work (creates worktree automatically)
- Use /stage to wrap up (prepares clean commit for landing)
- Review and land via wtr (ff-only merge)

## Issues

Tracked on GitHub: https://github.com/davidbyttow/govips/issues

---
> Source: [davidbyttow/govips](https://github.com/davidbyttow/govips) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
