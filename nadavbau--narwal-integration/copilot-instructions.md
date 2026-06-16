## version-and-release

> Version bumping and release workflow


# Version & Release

## Always Bump Version

When making functional changes (not just docs), bump the patch version in
`custom_components/narwal/manifest.json`. HACS detects updates by comparing
the manifest version against the GitHub release tag.

## Push Command

The `GH_TOKEN` env var may be set to a different GitHub account.
Always push with it unset:

```bash
GH_TOKEN="" GITHUB_TOKEN="" git push origin main
```

## Do NOT Create Tags or Releases

A GitHub Actions workflow automatically creates the git tag and GitHub
release when changes are pushed to `main`. Never manually run `git tag`
or `gh release create` — it will conflict with the automated workflow.

## No Personal Data

Never commit device IDs, user UUIDs, access tokens, or email addresses.
Use placeholders like `abcdef1234567890abcdef1234567890` in docs/README.

---
> Source: [nadavbau/narwal-integration](https://github.com/nadavbau/narwal-integration) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
