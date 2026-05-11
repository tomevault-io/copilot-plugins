## codex-plugins

> This repository contains the Ixel Codex plugin marketplace and the plugins published through it.

# Repository Instructions

## Project Scope

This repository contains the Ixel Codex plugin marketplace and the plugins published through it.

The marketplace currently includes the Maquette Codex plugin.

Plugin runtime behavior must live in plugin files, not only in repository instructions:

- `plugins/maquette/.codex-plugin/plugin.json`
- `plugins/maquette/skills/*/SKILL.md`
- `plugins/maquette/skills/*/agents/openai.yaml`
- `plugins/maquette/shared/*`
- skill asset prompt files under `plugins/maquette/skills/*/assets/`

Use this `AGENTS.md` for repository maintenance guidance only.

## Installed Plugin Cache

Do not edit installed plugin cache copies such as `%USERPROFILE%\.codex\plugins\cache\ixel\maquette\...` during normal repository work. Make changes in this repository under `plugins/maquette/` and let Codex install or refresh the plugin from the pushed ref after restart.

Only touch an installed cache path when the user explicitly asks to patch the local installed copy for immediate testing, and keep that separate from the source changes.

## Documentation

Keep the root `README.md` focused on marketplace-level installation, layout, plugin listings, and release guidance.

Keep `plugins/maquette/README.md` focused on Maquette-specific behavior, invocation examples, workflow details, and plugin-local installation notes.

When changing user-facing Maquette behavior, update `plugins/maquette/README.md` first. Then update the root `README.md` only if the marketplace listing, install guidance, release guidance, or high-level Maquette summary should change.

When changing skill behavior, update the relevant `SKILL.md` first. Then update shared docs or README files only if the behavior should be visible to users.

Update `plugins/maquette/CHANGELOG.md` in the same change whenever Maquette behavior, workflow rules, prompts, skills, schemas, bundled scripts, plugin metadata, or user-facing documentation changes. Most entries should go under `## Unreleased` until a release is cut. Internal-only edits may skip the changelog only when they cannot affect installed-plugin behavior, generated artifacts, QA results, or user-visible documentation.

## Versioning And Releases

The plugin version is defined in:

- `plugins/maquette/.codex-plugin/plugin.json`

For a stable release:

1. Create or switch to the release branch, for example `release/maquette/v0.4.0`.
2. Update the plugin version in `plugin.json`.
3. Move relevant `plugins/maquette/CHANGELOG.md` entries from `Unreleased` into a versioned section such as `## 0.4.0 - YYYY-MM-DD`.
4. Commit the release changes.
5. Create an annotated tag for the released plugin version.

Prefer plugin-scoped release branch names such as `release/maquette/v0.4.0` once the marketplace contains multiple independently versioned plugins.

The existing Maquette `v0.3.3` tag is a legacy single-plugin tag. For future releases, prefer plugin-scoped tag names such as `maquette/v0.4.0` so tags do not collide between plugins.

Use explicit refs when pushing release branches and tags:

```sh
git push origin refs/heads/release/maquette/v0.4.0
git push origin refs/tags/maquette/v0.4.0
```

For prereleases, use SemVer prerelease versions in `plugin.json` and matching annotated tags:

```text
0.4.0-alpha.1
0.4.0-beta.1
0.4.0-rc.1
```

Example prerelease tag:

```sh
git tag -a maquette/v0.4.0-alpha.1 -m "Maquette v0.4.0-alpha.1"
```

For development builds, publish or install from a branch ref such as `dev`, `next`, or `release/maquette/v0.4.0`. Do not use a stable version tag for moving development work.

## Maquette Skill Changes

When adding a new skill:

- create `plugins/maquette/skills/<skill-name>/SKILL.md`
- add `plugins/maquette/skills/<skill-name>/agents/openai.yaml` when the skill should appear as an explicit invocation target
- update `plugins/maquette/.codex-plugin/plugin.json` only when plugin-level behavior, metadata, screenshots, or default prompts change
- update README examples if the new skill changes the recommended user workflow

Do not rely on `AGENTS.md` to define behavior that installed-plugin users must receive.

## Plugin Additions

When adding a new plugin to the marketplace:

- create it under `plugins/<plugin-name>/`
- add its `.codex-plugin/plugin.json`
- add a plugin-specific `README.md`
- add or update its entry in `.agents/plugins/marketplace.json`
- keep the root `marketplace.json` mirror aligned while it remains in the repository
- update the root `README.md` available plugin list

---
> Source: [Ixe1/codex-plugins](https://github.com/Ixe1/codex-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
