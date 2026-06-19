## gh-release-notes-skill

> Draft and publish GitHub release notes from actual git diffs and tags using gh, or turn the same release evidence into docs-backed article pages. Use when Codex needs to create, revise, or verify release notes for a GitHub release, especially when the notes must be based on real code changes, commit ranges, touched files, validation results, or a first release with no previous tag. Also use it when the user wants release articles written from GitHub release material or repository changes. When the target repository already has a docs surface, treat docs-backed release notes as mandatory, create a companion walkthrough article in docs by default for release publishing unless the user explicitly narrows the scope, and do not call the task complete until README and primary operator docs have been truth-synced or explicitly waived with code-backed rationale and a validated QA inventory artifact exists.


# GitHub Release Notes

## Overview

Create release notes from repository evidence, not commit subjects alone.

Use this skill to:

- draft release notes before publishing a tag
- rewrite thin or inaccurate existing GitHub release bodies
- create or update GitHub releases with `gh release create` or `gh release edit`
- handle first releases where there is no previous tag
- draft docs-backed article pages from the same release evidence when the user wants a blog or announcement post
- create a companion walkthrough article by default when publishing a release from a repository that already has a docs surface
- derive a new versioned release header SVG when the repository already has an earlier release header asset
- derive a new versioned release header SVG by default when the repository already ships suitable reusable SVG branding such as `assets/icon.svg`, `assets/logo.svg`, or a branded `assets/social-card.svg`, the release would benefit from a hero image, and there is no earlier versioned release header asset yet
- validate every candidate SVG before reusing existing header art or deriving new release header assets from repo branding
- mirror release notes into repository docs by default when the target repository already publishes docs
- keep release collateral, README, and primary operator docs aligned with code-backed shipped behavior
- require a release QA inventory artifact and validate it before closing the task
- keep release notes grounded in actual shipped behavior

## Prerequisites

- access to a local clone of the target repository
- `git` available in the shell
- `gh` installed and authenticated if publishing is requested
- permission to inspect diffs, tags, and release state

## Default Workflow

1. Read the target repository `README` if one exists and inspect the tag or release state.
   - Run `gh auth status` if the user expects you to publish or edit a GitHub release.
   - Fetch tags first when the local clone may be stale.
2. Determine the comparison range.
   - If the target tag has a previous tag, compare from that previous tag to the target tag.
   - If there is no previous tag, treat the release as an initial release and cover the full shipped history explicitly.
   - Use an explicit base tag when the release line is non-linear or when backfilled tags make auto-detection ambiguous.
3. Run the bundled collector first.
   - `powershell -ExecutionPolicy Bypass -File ./scripts/collect-release-context.ps1 -Tag v0.1.0`
   - If the tag exists only on GitHub and not in the local clone yet, fetch tags first or use `-Target <commit-or-branch>`.
   - Use `-BaseTag <tag>` to override the automatically detected previous tag.
   - Read the output, then inspect the actual diffs for the high-impact files and commits.
4. Review implementation diffs, not just summaries.
   - Read the changed file list and diff stats first.
   - Always inspect new or heavily changed scripts, workflows, fixtures, docs, and user-facing assets.
   - Search for existing release header assets such as `assets/release-header-v0.2.0.svg`, `docs/public/.../release-header-v0.2.0.svg`, or similar versioned SVGs before deciding whether to create a new header image.
   - If there is no versioned release header asset yet, search for reusable SVG branding such as `assets/icon.svg`, `assets/logo.svg`, `assets/social-card.svg`, or equivalent repo branding and treat that as the default seed for a new `release-header-v*.svg` only when the branding is suitable for a release hero image.
   - Validate every candidate SVG you might reuse with `powershell -ExecutionPolicy Bypass -File ./scripts/verify-svg-assets.ps1 -RepoPath . -Path <svg-paths>` before treating it as a reusable seed. Broken XML, missing `<svg>` roots, unresolved internal id references, or SVGs without scalable dimensions are not valid release-header inputs.
   - Use `git show` on major commits and touched files until you can name concrete capabilities added.
   - For implementation-sensitive claims such as model selection, retry/backoff, routing, defaults, environment variables, telemetry surfaces, or command behavior, inspect the actual code paths and tests that implement them.
   - When those claims depend on configuration or runtime wiring, read the config readers, command entrypoints, runtime call sites, and operator-visible output formatting instead of relying on one file in isolation.
   - If a behavior applies only to one command, service, embed, or deployment mode, keep the wording scoped to that exact surface instead of generalizing it.
5. Draft the requested output in the user's requested language.
   - Open with release scope and whether it is an initial release.
   - Group the material by user-visible capabilities and implementation areas, not by raw commit count.
   - Mention validation only if you actually ran it.
   - Use [references/release-note-template.md](./references/release-note-template.md) when you want a release-note scaffold.
   - If the user wants GitHub release notes, draft a publishable release body.
   - If the repository has a docs surface and the user is publishing a release, create both docs-backed release notes and a companion walkthrough article by default unless the user explicitly asks for release-notes-only.
   - If the user wants article output, draft docs-backed article pages instead of a detached local draft whenever the repository has a docs surface.
   - Do not stop at "I will draft the article next" when article output is in scope. Create the article pages in the same task unless the user explicitly paused the work.
   - Do not treat the release-note page as a substitute for the companion walkthrough article when the repository already publishes docs and the release is being published.
6. If you are drafting article output, or if you are publishing a release in a repository with a docs surface, inspect the repository docs surface and place the article in docs by default.
   - When the repository already has bilingual English and Japanese docs, create both `docs/guide/articles/<slug>.md` and `docs/ja/guide/articles/<slug>.md`.
   - When the repository has only one docs locale, use the matching existing docs structure.
   - Keep the body free of Zenn or Qiita frontmatter.
   - Use a structure such as short introduction, key points, main features, workflow impact, validation, and links.
   - Reuse public screenshots, animated assets, and docs links when they help readers understand the release.
   - Update article index pages such as `docs/guide/articles.md` and `docs/ja/guide/articles.md` when they already exist.
   - Add a related article link from release summary pages when the docs structure already has release pages.
7. If the repository already has versioned release header SVG assets, create a new header image for the target version and reuse it everywhere the release appears.
   - Use the nearest existing asset, such as `assets/release-header-v0.2.0.svg`, as the visual base instead of inventing a totally new style.
   - Update the version text and adjust the header copy or visual emphasis to match the current release scope.
   - Save the new asset using the repository's established naming and folder pattern.
   - When the seed asset is outside the published docs asset surface, also create or mirror a published copy so docs pages and GitHub releases can reference it.
   - Place the header image near the top of the GitHub release body, the docs release page, and any docs article page created for the same release.
   - In the GitHub release body, use a published URL such as the docs site URL or a raw GitHub asset URL rather than a local relative path.
   - If the repository does not already have versioned release header art but does have reusable SVG branding such as `assets/icon.svg`, `assets/logo.svg`, or a branded `assets/social-card.svg`, treat header generation as the default behavior only when the release would benefit from a hero image, the SVG is suitable for reuse, and the user did not narrow scope away from visual collateral.
   - Run [scripts/verify-svg-assets.ps1](./scripts/verify-svg-assets.ps1) on the source SVGs before reuse, then run it again on the generated `release-header-v*.svg` before you publish or reference that asset anywhere.
   - Derive the new `release-header-v*.svg` from that existing SVG branding so the release inherits the repo's established visual language rather than shipping without a hero image.
   - If the available SVG is icon-only, too low-detail, stylistically mismatched, invalid, broken, or otherwise unsuitable for a release hero image, explicitly skip header generation or mark it as review-required instead of forcing a weak result.
8. Inspect the repository docs surface before publishing and treat docs-backed release notes plus a companion docs-backed walkthrough article as the default path.
   - Reuse the existing docs framework, locale structure, and navigation style instead of inventing a parallel format.
   - Create or update the matching docs page in every language already supported by the repository, unless the user narrowed the request.
   - Treat the release-note page and companion walkthrough article as release collateral, not as an automatic substitute for steady-state docs such as `README`, overview, CLI, setup, deployment, smoke-test, or operator runbooks.
   - Extract the operator-facing claims introduced or emphasized by the release docs, then review the primary steady-state docs to decide which of those surfaces must also be updated.
   - When the release changes operator expectations, configuration, routing, retry behavior, outputs, or verification flow, update the matching steady-state docs in the same task instead of leaving the detail only in release collateral.
   - Update stale "latest release" links, overview pointers, and docs navigation when the repository exposes them.
   - If the GitHub release body should link into docs, commit and push the docs changes first, wait for docs deployment, and only then publish or edit the final GitHub release body so it can point at live URLs.
   - Prefer badge-style links at the top of the GitHub release body so readers can jump to the docs pages.
   - In the GitHub release body, render top docs links as shields.io badge images inside a centered HTML paragraph, not as plain Markdown links or a punctuation-separated link list.
   - Keep the badge paragraph centered with `<p align="center">...</p>` and include one badge per important destination such as English docs, Japanese docs, walkthrough, and Japanese walkthrough when those pages exist.
   - Prefer linking both the docs-backed release notes page and the companion walkthrough article from the GitHub release body when both exist.
   - Skip the companion article only when the repository clearly has no docs publishing surface or the user explicitly asks for release-notes-only.
9. Materialize the QA inventory as a file in the target repository before calling the work complete.
   - Copy [references/release-qa-inventory-template.md](./references/release-qa-inventory-template.md) into the standard task artifact path `tmp/release-qa-<tag>.md` in the target repository, for example `tmp/release-qa-v1.2.3.md`.
   - Fill in the claim matrix, steady-state docs review table, and QA inventory rows with concrete evidence from the current task.
   - Validate the filled artifact with [scripts/verify-release-qa-inventory.ps1](./scripts/verify-release-qa-inventory.ps1) by passing the target repo path and tag, for example `powershell -ExecutionPolicy Bypass -File <skill-root>\scripts\verify-release-qa-inventory.ps1 -RepoPath . -Tag v1.2.3`.
   - Do not close the task while the validator reports any failure.
10. Publish or update the release with `gh` when GitHub publication is part of the task.
   - Ensure every docs page and asset referenced by the release body is already committed before creating the release tag.
   - Create and push the release tag only after the release docs and article pages are committed if those artifacts are part of the shipped release collateral.
   - Use `gh release create <tag> --title ... --notes-file ...` when the release does not exist.
   - Use `gh release edit <tag> --notes-file ...` when the release already exists or needs a rewrite.
11. Verify the published body when you published or edited the GitHub release.
   - Run `gh release view <tag> --json url,name,body` and confirm the text matches what you intended.
   - Confirm the GitHub release body still has centered badge links at the top when docs or walkthrough links exist.
   - Confirm every major `##` section heading in the GitHub release body starts with an appropriate emoji, for example `## ✨ Highlights`, `## 🛡️ Reliability`, `## 📚 Docs`, `## ✅ Validation`, or `## 🧩 Follow-Up`.
   - If you created docs pages, verify those URLs resolve and that the release body points at the published docs routes.
   - If you created a companion walkthrough article, verify that URL too and confirm the release body links to it when expected.
   - If you added a release header image, verify that the image URL resolves and renders from the GitHub release body and the docs pages.
   - Confirm the QA inventory artifact path and the validator result before you call the work done.

## Evidence Standard

- Do not rely on `git log --oneline` alone.
- Do not rely on `gh release create --generate-notes` alone when the user asks for diff-based notes.
- Do not summarize a large initial release with only generic bullets.
- Do not claim support for a feature unless you can point to the file or diff that introduced it.
- Do not claim checks passed unless you ran them in the current repo.
- Do not claim routing, model, retry, default, environment, or telemetry behavior without inspecting the implementing code path or a test that covers it.
- Do not broaden a behavior from one surface to all surfaces without code evidence.
- If the earlier note is thin, deepen it by reading the code diffs and rewriting the release body.
- Do not turn a docs article into vague marketing copy detached from the diff evidence.
- Do not treat a release-note page or walkthrough article as proof that README and steady-state operator docs are already current.

## Inspection Priorities

Inspect these categories whenever they appear in the diff:

- new or rewritten scripts under `scripts/`
- CI or automation under `.github/workflows/`
- fixtures, test data, or regression helpers
- README, `SKILL.md`, docs, references, or public assets
- versioned release header assets such as `release-header-v*.svg`
- reusable SVG branding such as `assets/icon.svg`, `assets/logo.svg`, or branded `assets/social-card.svg` when no versioned release header exists yet
- packaging or version metadata such as `package.json`, `pyproject.toml`, or release config

For detailed drafting rules and anti-patterns, read [references/release-note-checklist.md](./references/release-note-checklist.md).

## Drafting Rules

- Prefer sections like `Highlights`, `Tooling`, `Validation`, `Docs And Assets`, or equivalent based on the diff.
- In GitHub release bodies, prefix major `##` section headings with concise emojis that match the section purpose. Keep docs pages more conservative if the existing docs style avoids emoji, but do not publish a GitHub release body with bare `## Highlights` / `## Validation` style headings unless the user explicitly asks for plain headings.
- Lead with the biggest shipped change, not the easiest file to summarize.
- Mention docs and visuals after the product or tooling changes unless the release is docs-only.
- For initial releases, say explicitly that the notes cover the full history shipped in that tag.
- When a script adds real behavior, name the behavior, not just the filename.
- When a workflow or fixture materially protects the release, explain what it validates.
- Keep the GitHub release body readable on its own and use badges or short links to point at the fuller docs pages.
- When using docs or article links in the GitHub release body, make them shields.io badges wrapped in a centered paragraph near the top; plain inline text links are a fallback only when badges cannot render.
- Treat docs-backed release notes as the standard outcome whenever the repository already has a published docs surface.
- If a release claim affects operator understanding outside release week, sync the matching steady-state docs instead of leaving the claim only in release collateral.
- Narrow the wording when a feature applies only to one path, such as `drive watch`, upload-test, long-running services, or a specific embed surface.
- When a versioned release header SVG already exists in the repository, create and include the updated header instead of leaving the release without a hero image.
- When there is no versioned release header yet but the repository already ships suitable reusable SVG branding, derive and include a new `release-header-v*.svg` by default instead of omitting the hero image, unless the user narrowed scope or the branding is not suitable for a release hero image.
- Do not reuse or publish any SVG asset until it passes [scripts/verify-svg-assets.ps1](./scripts/verify-svg-assets.ps1).
- Keep the final note dense with evidence but still readable.

## Docs Article Mode

When the user wants article output instead of, or in addition to, a GitHub release body, or when you are publishing a release in a repository that already has a docs surface:

- write natural prose, not translated commit history
- explain reader impact before internal implementation detail
- create docs article pages in the repository instead of saving a detached draft elsewhere
- finish the docs article pages in the same turn instead of leaving a follow-up promise
- when the docs site already supports both English and Japanese, write both locale pages
- place the release header image near the top when a versioned header SVG exists or when a suitable new one can be derived from existing SVG branding such as an icon or logo
- include a final title, stable lead paragraph, main sections, and explicit links
- treat the docs article as the canonical source that can later be handed to `oasis-skill` for Zenn and Qiita distribution

## Docs Truth-Sync

When a release ships with docs collateral, run an explicit truth-sync pass before you call the work complete:

- extract the operator-facing claims from the GitHub release body, docs-backed release notes, and companion walkthrough article
- mark which claims are implementation-sensitive and verify them against code paths or tests
- review the repository's steady-state docs surfaces such as `README`, overview, CLI, setup, deployment, smoke-test, and operator runbooks
- update every surface that should reflect the shipped behavior, or explicitly record why a surface does not need changes
- tighten wording for path-specific behaviors so the docs do not over-claim beyond the implementation

If the repository exposes "latest release" links or overview pointers, update them during the same pass.

Do not leave this pass as an undocumented mental checklist. Write the reviewed surfaces and evidence into a QA inventory artifact so the release task leaves an auditable trail.

## QA Inventory Gate

Before calling the release task done, produce and internally check a QA inventory with criterion status for at least these items:

- comparison range resolved from actual tags or an explicitly justified commitish override
- release claims backed by inspected diffs, files, or commits
- docs-backed release notes created or explicitly skipped with user-approved rationale
- companion walkthrough article created or explicitly skipped with user-approved rationale
- operator-facing claims extracted from the release body and docs collateral
- implementation-sensitive claims verified against code paths, tests, or both
- README and primary operator docs reviewed for truth-sync updates, with changed files or explicit no-change rationale
- claim scope kept precise when behavior applies only to a subset of commands, services, embeds, or deployment modes
- stale latest-release links or overview pointers updated where the repository exposes them
- companion walkthrough article treated as insufficient while any required steady-state docs truth-sync item is `fail` or `blocked`
- SVG assets used as release-header inputs or outputs validated with `verify-svg-assets.ps1`, or explicitly marked `not_applicable`
- docs pages and assets referenced by the release body committed before tag creation
- docs deployment completed and live URLs verified before the final release body links to them
- GitHub release top docs/article links rendered as centered badge images when such links exist, or a rationale recorded when badges were not possible
- GitHub release major `##` section headings use purpose-matched emoji prefixes, or a user-approved rationale recorded for plain headings
- release tag created locally and pushed remotely
- GitHub release published or updated and the final body verified with `gh release view`
- validation commands actually run, or clearly marked as not run
- hardcoded publish dates sourced from the actual tag or release timing, or omitted until the release exists

Write that inventory to `tmp/release-qa-<tag>.md` in the target repository and validate it with [scripts/verify-release-qa-inventory.ps1](./scripts/verify-release-qa-inventory.ps1) by passing the target repo path and tag. A release task is not complete while the inventory file is missing, incomplete, or failing validation.

Do not mark the work complete while any required item is `fail` or `blocked`.

## Windows Notes File Handling

When you need a temporary notes file on Windows, write UTF-8 without BOM before calling `gh`:

```powershell
$notesPath = Join-Path $env:TEMP "release-notes-v0.1.0.md"
$body = @"
# Highlights

- Replace this with evidence-based release notes.
"@
$utf8NoBom = New-Object System.Text.UTF8Encoding($false)
[System.IO.File]::WriteAllText($notesPath, $body, $utf8NoBom)
```

## Verification

- If you did not publish the release, say so clearly.
- If you did not run validation in the target repository, say so clearly.
- If there is no previous tag, verify that the drafted scope really covers the full shipped history.
- If the release already existed, confirm the final published body matches the rewritten note.
- If you created docs-backed release notes, confirm the docs build or deployment path succeeded before you call the work done.
- If the repository has a docs surface and you published a release, verify the live docs-backed release-note URL and the live companion walkthrough-article URL unless the user explicitly opted out of the article.
- If the release body includes docs or article links, confirm they are badge-style and centered in the published GitHub release body.
- Confirm the published GitHub release body uses emoji-prefixed major section headings.
- If the release introduced or emphasized operator-facing claims, report whether `README` and primary operator docs were reviewed for truth-sync and list the files you updated or explicitly found unchanged.
- If the release body links to docs, confirm those docs pages were committed, pushed, and deployed before the final release body was published.
- If you created a release header image, report where the SVG was saved, which existing SVG branding it was derived from when applicable, which `verify-svg-assets.ps1` commands you ran, and where it is referenced.
- If you skipped header generation despite existing SVG branding, report whether the SVG validator failed, why the branding was unsuitable, or why the scope excluded visual collateral.
- Do not hardcode a `Published on ...` date before the release exists. After publishing, align the date with the actual release or tag timing, or state the exact source of the date.
- Confirm the QA inventory artifact path, the validator command you ran, and whether the validator passed.
- If you drafted only docs article pages and did not publish a GitHub release, say that clearly and report the saved docs paths.

## Publishing With gh

Common commands:

```powershell
gh release view v0.1.0
gh release create v0.1.0 --target main --title "v0.1.0" --notes-file $notesPath
gh release edit v0.1.0 --notes-file $notesPath
gh release view v0.1.0 --json url,name,body
```

## Resources

- Use [scripts/collect-release-context.ps1](./scripts/collect-release-context.ps1) to gather tags, commit range, diff stats, and changed files.
- Use [scripts/verify-release-qa-inventory.ps1](./scripts/verify-release-qa-inventory.ps1) to validate a filled release QA inventory artifact before you close the task.
- Use [scripts/verify-svg-assets.ps1](./scripts/verify-svg-assets.ps1) to validate candidate SVG branding and generated release-header SVGs before reuse or publication.
- Use [references/release-note-checklist.md](./references/release-note-checklist.md) when the repo is large or when you need a second pass on release-note depth and accuracy.
- Use [references/release-qa-inventory-template.md](./references/release-qa-inventory-template.md) to record the claim matrix, steady-state docs review, and QA gate evidence.
- Use [references/release-note-template.md](./references/release-note-template.md) for a drafting scaffold before publishing.

---
> Source: [Sunwood-ai-labs/gh-release-notes-skill](https://github.com/Sunwood-ai-labs/gh-release-notes-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
