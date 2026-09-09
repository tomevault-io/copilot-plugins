## gha-remove-artifacts

> cp -n .env.example .env

@README.md

## Development

### Setup

```sh
mise install
cp -n .env.example .env
pnpm install
pnpm run lint
pnpm run typecheck
pnpm test
pnpm run build
```

### Release

Consumers reference the action by tag: `@v1` (moving major tag) or `@v1.x.y`.

1. Bump `version` in `package.json`, commit and push to `main`. Wait for CI to pass.
2. Tag the release and move the major tag to the same commit:
   ```sh
   git tag v1.x.y
   git tag -f v1
   git push origin v1.x.y
   git push -f origin v1
   ```
3. Create the GitHub release from the tag: `gh release create v1.x.y --generate-notes`.

---
> Source: [c-hive/gha-remove-artifacts](https://github.com/c-hive/gha-remove-artifacts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
