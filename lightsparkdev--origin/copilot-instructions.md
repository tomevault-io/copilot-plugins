## publishing

> Origin publishes to GitHub Packages on release.

# Publishing

Origin publishes to GitHub Packages on release.

## When to Publish

- After component additions/changes ready for consumption
- After token updates
- After bug fixes affecting consumers

## How to Publish

```bash
npm version patch  # or minor/major
git push && git push --tags
gh release create vX.Y.Z --title "vX.Y.Z" --generate-notes
```

## Before Publishing

- Verify build passes: `npm run build`
- Verify tests pass: `npm run test:unit`
- Update the consuming product's dependency version after publishing

## Version Guidelines

- **patch** (0.0.X): Bug fixes, minor style tweaks
- **minor** (0.X.0): New components, new features, non-breaking changes
- **major** (X.0.0): Breaking API changes, major refactors

## After Publishing

In the consuming product, update to the new version:

```bash
npm install @lightsparkdev/origin@latest
```

---
> Source: [lightsparkdev/origin](https://github.com/lightsparkdev/origin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
