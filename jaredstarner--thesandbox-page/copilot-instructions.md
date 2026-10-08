## thesandbox-page

> Testing ground for web experiments: static Astro, strict TypeScript, deployed to GitHub Pages. `CLAUDE.md` is a symlink to this file; edit `AGENTS.md`.

# thesandbox.page

Testing ground for web experiments: static Astro, strict TypeScript, deployed to GitHub Pages. `CLAUDE.md` is a symlink to this file; edit `AGENTS.md`.

## Workflow

- Never push to `main`; work on a branch and open a pull request into `main`
- Sign every commit; the `main` ruleset rejects unsigned commits
- Keep commits small and focused; PRs merge with a merge commit, so each commit lands on `main`
- Write commit messages as [Conventional Commits](https://www.conventionalcommits.org); semantic-release reads them
  - `feat` releases a minor version, `fix` and `perf` a patch, and a `BREAKING CHANGE:` footer a major. The `!` shorthand (`feat!:`) is not recognized by the release config and releases nothing
  - `build`, `chore`, `ci`, `docs`, `refactor`, `style`, and `test` release nothing
- Run `pnpm check` and `pnpm build` before pushing; CI runs both as the required `check-and-build` check
- Name branches `<type>/<short-description>`, using a Conventional Commit type (for example `fix/nav-overflow`)
- Required checks: `check-and-build`, `dependency-review`, and CodeQL; no approval is needed except for changes under `.github/` or to `.releaserc.json`, which need the owner's review (`.github/CODEOWNERS`)
- Enable auto-merge on PRs; they merge once checks pass. If a PR from the owner (including `/sandbox` runs) is waiting on code owner review, merge it with the admin bypass after checks pass
- Merging to `main` runs the Production workflow: deploy to GitHub Pages, then semantic-release. Merges by anyone other than the owner wait for owner approval on the `production` environment
- Never push tags; semantic-release creates every version tag
- Releases are git tags plus GitHub releases only; do not edit the `package.json` version or add a CHANGELOG
- Agents stop for a local review before pushing a branch, except scheduled `/sandbox` runs, which ship once checks pass
- Design tokens and shared components go in `src/theme/` (see `src/theme/README.md`); experiments use them only by choice

## The sandbox

- The site itself is the sandbox: anything on it can change. The layout, theme, and home page are a starting point, not a contract
- Kinds of change, most common first:
  - **New**: a new experiment on its own page
  - **Extend**: build on an existing experiment or page
  - **Rework**: redo something with a new technique or platform feature; say why in the PR
  - **Teardown**: replace the home page or shared site chrome; rare. The replaced home page is not deleted: it moves to its own experiment page with metadata, so it stays in the showcase
- Experiments live at the site root, `src/pages/<slug>/` served at `/<slug>/`, with metadata in `src/experiments/<slug>.json` (see `src/experiments/README.md`). They stay up as a showcase; remove one only with a stated reason in the PR
- Each experiment owns its look: start from `src/layouts/Bare.astro` (document head only). `Base.astro`, `src/theme/`, `BackHome`, and an on-page explainer are all optional. The pull request is the lasting reference for what a page is and how it works
- Never break another page: a change that reaches beyond its own page (a dependency, an integration, config, shared code) must leave every other page building and behaving as before
- Nothing may cost the owner money. Ideas that need an account, an API key, or a GitHub feature start as an issue
- Versions follow the commit types: `feat` for anything new or added, `fix` or `perf` for fixes and polish. Major versions, including v1.0.0, are the owner's call; add a `BREAKING CHANGE:` footer only when the owner asks for one. A teardown is otherwise a `feat`
- Labels live in `.github/labels.yml`. `sandbox` marks issues approved for `/sandbox` runs; only the owner and collaborators apply it, and issue forms must never add it. Run labels: `experiment`, `extend`, `rework`, `teardown`; `sandbox-failed` marks a run that could not pass its checks

## Development

When starting the dev server, use background mode:

```
astro dev --background
```

Manage the background server with `astro dev stop`, `astro dev status`, and `astro dev logs`.

## Documentation

Full documentation: https://docs.astro.build

Consult these guides before working on related tasks:

- [Adding pages, dynamic routes, or middleware](https://docs.astro.build/en/guides/routing/)
- [Working with Astro components](https://docs.astro.build/en/basics/astro-components/)
- [Using React, Vue, Svelte, or other framework components](https://docs.astro.build/en/guides/framework-components/)
- [Adding or managing content](https://docs.astro.build/en/guides/content-collections/)
- [Adding styles or using Tailwind](https://docs.astro.build/en/guides/styling/)
- [Supporting multiple languages](https://docs.astro.build/en/guides/internationalization/)

---
> Source: [jaredstarner/thesandbox.page](https://github.com/jaredstarner/thesandbox.page) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
