## spawndev-ilgpu

> ILGPU uses T4 templates (.tt files) that regenerate .cs files on rebuild. Always check for .tt sources before editing generated .cs files.


# ILGPU T4 Template Code Generation

ILGPU uses **T4 templates** (`.tt` files) to generate many `.cs` files at build time. Edits to generated `.cs` files will be silently overwritten on the next build.

## Before editing any `.cs` file in the ILGPU/ directory:

1. Check if a matching `.tt` file exists (same name, e.g. `MemoryBuffers.tt` generates `MemoryBuffers.cs`)
2. If a `.tt` file exists, edit the **`.tt` template** instead of the `.cs` output
3. The generated `.cs` will be regenerated on rebuild

## Known T4-generated files (non-exhaustive):

- `ILGPU/Runtime/MemoryBuffers.tt` → `MemoryBuffers.cs`
- Many files under `ILGPU/IR/`, `ILGPU/Runtime/`, `ILGPU/Backends/`
- Check `ILGPU/Static/TypeInformation.ttinclude` for shared type definitions

## Quick check command:

Search for `.tt` files near any ILGPU `.cs` file you plan to edit:
```
Get-ChildItem -Path ILGPU -Recurse -Filter "*.tt"
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LostBeard) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
