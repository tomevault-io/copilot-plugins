## veridns

> A verified DNS resolver in Lean 4, verified against RFC specifications.

# Project: veri-dns

A verified DNS resolver in Lean 4, verified against RFC specifications.

## Build

```
lake build
```

## Key conventions

- `autoImplicit` is disabled - all variables must be explicitly declared
- The `include_rfc` macro verifies RFC text at compile time
- RFC files live in `rfc/` and are referenced by line number ranges
- When modifying files in `VeriDNS/Impl/`, `VeriDNS/Spec/`, `VeriDNS/Proof/`, or `VeriDNS/RFC/`, update `docs/architecture.md` if the module structure or design decisions change
- Always run `lake build` before committing to ensure the RFC text still matches

---
> Source: [BasisResearch/VeriDNS](https://github.com/BasisResearch/VeriDNS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
