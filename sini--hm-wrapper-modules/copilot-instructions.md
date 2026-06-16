## hm-wrapper-modules

> hm-wrapper-modules is a bridge between home-manager and nix-wrapper-modules.

# CLAUDE.md

## Project Overview

hm-wrapper-modules is a bridge between home-manager and nix-wrapper-modules.
It evaluates arbitrary HM modules in a real `homeManagerConfiguration` context,
extracts their side effects, and maps them to nix-wrapper-modules primitives
to produce portable wrapped package derivations.

## Commands

```bash
nix flake check -Lv   # Run all tests
nix fmt               # Format (nixfmt-rfc-style)
```

## Architecture

### Extraction (`lib/hm-adapter.nix`)

Exports `wrapHomeModule` and `mkBinds`. Takes HM modules + `homeManagerConfiguration`
context, extracts packages/files/activation/session variables, produces a
wrapper-module `.config`.

Key mechanisms:
- Baseline diff: evaluates HM with empty module to identify internal entries
- `mainPackage` auto-discovery: `programs.<hint>.package` → `home.packages` by pname → first enabled program
- Session variables coerced to strings via `toString`
- `mkBinds` uses `placeholder "out"` (attrset keys strip string context)

### Presentation (`modules/bwrapConfig/module.nix`)

Standalone wrapper module using bubblewrap mount namespaces. Linux only, darwin no-op.
Uses `argv0type` as a function to wrap the final exec. Resolves symlink targets
at runtime via `readlink -f` (bwrap can't mount over symlinks).

### Flake (`flake.nix`)

Inputs: `nix-wrapper-modules`, `home-manager`, `nixpkgs`. Extends upstream `wlib`
with adapter functions and bwrapConfig module — consumers get the full
nix-wrapper-modules API plus HM adapter in one import via `self.lib`.

## Dependencies

- `nix-wrapper-modules` — provides `wlib.evalModules`, `wlib.modules.default`, DAG types
- `home-manager` — provides `lib.homeManagerConfiguration` for HM evaluation
- `nixpkgs` — standard

## Commit Messages

Conventional commits: `<type>(scope): description`
Types: `feat`, `fix`, `docs`, `test`, `chore`.

---
> Source: [sini/hm-wrapper-modules](https://github.com/sini/hm-wrapper-modules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
