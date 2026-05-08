## dist-on-changes

> Run product build (dist) after code changes and always when prepping a new version


# Run dist after changes

After implementing code or version changes to Noggin, run:

```bash
npm run dist
```

That updates `dist/noggin/` and `dist/noggin.zip` with the latest build and version so the dist folder is never stale.

**When prepping a new version:** always run `npm run dist` as the final step. Prepping a version (bump in package.json, changelog/upgrade notice in readme.txt, then `npm run version:sync`) is not complete until dist has been run—otherwise the dist folder and zip stay at the previous release.

---
> Source: [RegionallyFamous/noggin](https://github.com/RegionallyFamous/noggin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
