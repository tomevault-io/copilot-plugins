## signaturemap

> Generate and use signatures.json as a lightweight declaration index for fast, low-cost code navigation.


# Signature Map

Use this skill to navigate declarations before broad repo scans.

## Runtime files

- `scripts/sigmap`
- `scripts/generate-signatures.sh`
- `scripts/sigmapgen/main.go`

## Recommended workflow

1. Refresh once:

```bash
./scripts/sigmap refresh --root <repo_root>
```

2. Query declarations:

```bash
./scripts/sigmap name <SymbolName> --root <repo_root> --no-refresh
./scripts/sigmap search "<regex>" --field all --icase --root <repo_root> --no-refresh
./scripts/sigmap open "<relative/file::symbol>" --root <repo_root> --context 60 --no-refresh
```

3. Diagnose when needed:

```bash
./scripts/sigmap doctor --root <repo_root>
```

Use `rg` for call-site search; use sigmap for declaration lookup.

---
> Source: [Xopoko/SignatureMap](https://github.com/Xopoko/SignatureMap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
