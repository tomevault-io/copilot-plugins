## d365bap-tools

> Create and maintain table views here. One file per type:

# Table format views

Create and maintain table views here. One file per type:

`<TypeName>.Table.Format.ps1xml`

Do not edit `../../d365bap.tools.Table.Format.ps1xml`. After changing a file here, run from the repo root:

```powershell
pwsh -NoProfile -File ./build/Merge-FormatPs1Xml.ps1
```

---
> Source: [d365collaborative/d365bap.tools](https://github.com/d365collaborative/d365bap.tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
