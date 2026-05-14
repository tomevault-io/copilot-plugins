## agent-yes

> **TypeScript changes:**

# agent-yes

## After making changes, always rebuild and relink

**TypeScript changes:**

```bash
bun run build && bun link
```

This compiles `ts/` → `dist/` via tsdown and registers the binaries globally.

**Rust changes (`rs/` directory):**

```bash
bun run build:rs && bun run build && bun link
```

`build:rs` runs `cargo install --path rs` (release build, installs to `~/.cargo/bin/agent-yes`).
Must be done whenever any `.rs` file changes, otherwise the old binary stays in place.

---
> Source: [snomiao/agent-yes](https://github.com/snomiao/agent-yes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
