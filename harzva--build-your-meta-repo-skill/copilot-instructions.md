## build-your-meta-repo-skill

> Create auto-updating GitHub meta repositories for a user or organization, including a project atlas, release hub, GitHub Pages hub, and skills hub. Use when the user wants a "repo of repos", repository index, portfolio atlas, release dashboard, pages gallery, skill registry, or GitHub Action that refreshes repository metadata as repos are added or removed.


# Build Your Meta Repo

## Quick Start

Use the scaffold script to generate the standard four-repo set:

```powershell
python C:\Users\harzva\.codex\skills\build-your-meta-repo\scripts\build_meta_repo.py --owner Harzva --output D:\work\meta-repos
```

This creates:

- `<owner>-project-atlas`: the top-level download map for APK, EXE, package, archive, and release asset links.
- `<owner>-release-hub`: repositories with latest GitHub releases.
- `<owner>-pages-hub`: visual GitHub Pages gallery with a deployable `docs/index.html`.
- `<owner>-skills-hub`: Codex skills, workflow skills, and agent recipes.

Each generated repository includes:

- `scripts/update_meta.py` for live GitHub metadata refresh.
- `.github/workflows/update-meta.yml` for daily and manual updates.
- `data/*.json` for downstream automation.
- GitHub Pages deployment workflow for every generated hub.
- A README designed as a GitHub-rendered homepage.

## Privacy Defaults

Default to public repositories only. Do not publish private repository names, descriptions, local paths, or private metadata into a public meta repository.

If the user explicitly wants private repo coverage, make the meta repository private first, then set:

```powershell
$env:META_INCLUDE_PRIVATE = "true"
$env:META_GITHUB_TOKEN = "<token in a secret or local environment>"
```

For GitHub Actions, store the token in `META_GITHUB_TOKEN` and keep `META_INCLUDE_PRIVATE` in repository variables or workflow env.

## Recommended Workflow

1. Identify the target owner and account route.
2. Generate the meta repositories locally with `build_meta_repo.py`.
3. Review README first screens and `docs/index.html` for pages-focused repos.
4. Publish with the correct GitHub account.
5. Verify Actions are present and Pages workflows are queued or triggerable.
6. For public repos, mention private repositories are intentionally omitted.

Read `references/meta_repo_blueprint.md` when customizing artifact categories, hub naming, or automation policy.

---
> Source: [Harzva/build-your-meta-repo-skill](https://github.com/Harzva/build-your-meta-repo-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
