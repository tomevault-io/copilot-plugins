## publish

> Publish workflow — bump version, update changelog, commit, push, and npm publish


# Publish Command

When the user says "publish", follow this exact sequence:

1. **Bump version** in `package.json` — patch for bug fixes, minor for new features/enhancements, major for breaking changes. Ask only if ambiguous.
2. **Update `CHANGELOG.md`** — add a new version section at the top with all changes since the last publish, following the existing format (### Fixed, ### Changed, ### Added).
3. **Rebuild dist** — run `npm run build` to ensure `dist/fig.js`, `dist/fig.css`, `dist/components.css`, and `dist/base.css` are up to date with the latest source changes. Never publish with stale dist files.
4. **Git add, commit, push** — stage all changed files, commit with message `v{version}: {short summary}`, and push to origin.
5. **npm publish** — after a successful git push, run `npm publish --access public` to publish to the npm registry.

Keep changelog entries concise but descriptive. Group by Fixed/Changed/Added as appropriate.

---
> Source: [rogie/figui3](https://github.com/rogie/figui3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
