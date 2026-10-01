## publish-uefn-ducky-store

> >-


# Publish desktop plugins to UEFN Ducky Store

The **app** is this repo. Each **plugin** is
`C:\Users\tas13\Documents\GitHub\uefn-plugins\uefn-plugin-<id>\`.

```bash
cd /c/Users/tas13/Documents/GitHub/uefn-plugins/uefn-plugin-<id>
py -3 scripts/release.py --publish --changelog "vN: …"
```

`--publish` commits + pushes the clone first (every shippable file). It refuses
to upload if anything real is still dirty. Do not zip or `uds_release` a dirty tree.

Needs `DUCKYOS_API_KEY` (or `~/.cursor/mcp.json` Bearer).

Do not bundle plugins into the EXE. Do not commit `*.ducky-plugin.zip` / `deploy/`.

---
> Source: [UEFN-Ducky/UEFN-Ducky](https://github.com/UEFN-Ducky/UEFN-Ducky) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
