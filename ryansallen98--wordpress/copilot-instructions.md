## wordpress

> README, gitignore, git init, naming, authorship, version bump, changelog.txt


# Scaffold, git, naming, versioning (Sections C, D, E)

## New plugin or theme package

**README.md** at package root: purpose, requirements, Composer/npm install, build/test, link to `readme.txt` if applicable, author/repo, relation to Bedrock. Do not replace WordPress.org `readme.txt` when both apply.

**`.gitignore`:** at minimum `node_modules/`, `.phpunit.cache/`, `.env` / `.env.*`, OS/IDE noise. `vendor/`: ignore only when deploy installs deps and `vendor/` is not committed; if monorepo commits `vendor/` for a plugin, omit `vendor/` from the package ignore.

**`git init`:** standalone package → `git init` at package root, first commit README + `.gitignore`. **Monorepo:** never `git init` inside `web/app/plugins/{slug}/` or `themes/{slug}/`. Standalone + Bedrock copy: subtree/submodule/mirror — document in README.

## Naming

Directory / main file / text domain / Composer name: describe behavior (e.g. `logical-media-folders`), not client brand, unless asked. Namespace, hooks, options, REST, handles: short neutral prefix (e.g. `VirtualMediaFolders`, `vmf_`, `vmf/v1`).

## Authorship (this template)

New first-party plugin or theme:

- **Ryan Allen** — https://rallendev.com — https://github.com/ryansallen98

Use in WP headers, `readme.txt`, `composer.json` authors, `package.json` author.

## Git: first-party must track

If Bedrock `.gitignore` uses `web/app/plugins/*`, add:

```gitignore
!web/app/plugins/{folder}/
```

If themes are blanket-ignored: `!web/app/themes/{folder}/`. New plugins: agent scaffold per `04-wordpress-plugins.mdc`; run `git check-ignore` on the package.

## Version bump + `changelog.txt`

Bump on every substantive change (fixes, features, security, shipped PHP/Blade/TS/CSS/ACF JSON, etc.). Skip only non-shipped edits (comment typo, agent-only doc). When in doubt: patch. Semver in spirit: patch / minor / major.

**Where:** plugin — headers, constants, `readme.txt` Stable + Changelog. Theme — `style.css` Version. Keep aligned per release.

**`changelog.txt`** at package root: plugin `web/app/plugins/{slug}/changelog.txt`, theme `web/app/themes/{THEME_SLUG}/changelog.txt`. Newest first, `## x.y.z`, optional date, bullets — same commit as bump + code.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ryansallen98) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
