## shell

> All terminal commands must use devenv shell


**MANDATORY:** Wrap ALL terminal commands with `devenv shell --`

```bash
# ✅ DO
devenv shell -- git status
devenv shell -- nix flake check
devenv shell -- cargo build

# ❌ DON'T
git status
nix flake check
cargo build
```

**Applies to:** All terminal commands (git, nix, cargo, formatting, testing, etc.)

---
> Source: [Industrial/id_effect](https://github.com/Industrial/id_effect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
