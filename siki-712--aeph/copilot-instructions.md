## aeph

> When asked to release a new version:

# Claude Code Instructions

## Release

When asked to release a new version:

```bash
./scripts/bump-version.sh <version>
git add -A
git commit -m "Release v<version>"
git tag v<version>
git push && git push --tags
```

---
> Source: [siki-712/aeph](https://github.com/siki-712/aeph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
