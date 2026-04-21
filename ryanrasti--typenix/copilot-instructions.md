## typenix

> 1. **Reproduce** — write a failing test first (in `internal/nixparser/lsptest/` or `internal/nixparser/nixtest/`)

# tsgo nix type checker — project status

## How to fix bugs
1. **Reproduce** — write a failing test first (in `internal/nixparser/lsptest/` or `internal/nixparser/nixtest/`)
2. **Fix** — make the test pass
3. **Verify** — run the broader test suites to check for regressions

## What works today
- **Zero type errors** on all 42,298 `.nix` files in nixpkgs in **13 seconds**
- Full type inference flowing through imports, `callLibs`, `makeExtensible`
- Hover, go-to-definition, and completions in the LSP across `.nix` files
- Class transform for fixed-point patterns (`makeExtensible`, `fix`, etc.) — maps nix's self-referential attrsets to TypeScript classes so the checker understands them
- Minimal annotations — most of nixpkgs is unannotated and just works
- Found real nixpkgs bugs during development: `ledfx` (`pyproject.pkgs` → `python3.pkgs`), `lib/cli.nix` (`lib.throw` → `builtins.throw`), dead code after `throw` in 3 files

## Key design decisions

### `__import` phantom type brand
Path expressions like `./foo.nix` carry a phantom type brand: `string & { readonly __import: typeof import("./foo.nix")["default"] }`. When passed through `builtins.import`, a conditional type extracts the module's exported type. This threads module types through nix's import system without runtime cost — the brand is erased, but the checker sees full type information flow through variables, `callLibs`, and `callPackage`.

### Reserved word handling
Instead of sanitizing nix identifiers that clash with JS reserved words (`throw` → `_throw`), we suppress the relevant binder/checker errors (TS1214, TS1215, TS2730) for `.nix` files. JS reserved words are valid as property names, and since we build the AST programmatically, the parser never rejects them. Bare `throw`/`import` references that aren't in scope are rewritten to `builtins.throw`/`builtins.import`.

### Class transform for fixed-point patterns
`makeExtensible (self: { foo = 1; bar = self.foo + 1; })` becomes a TypeScript class where `self` → `this`. This lets TypeScript's `this`-typing handle the self-referential nature of nix's fixed-point combinators. Let bindings in the lambda become `#private` class fields to avoid name conflicts.

---
> Source: [ryanrasti/typenix](https://github.com/ryanrasti/typenix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
