## pi-ssh-remote

> All commits must follow this format:

# Repository Instructions

## Commit convention

All commits must follow this format:

```text
<type>(<scope>): <description>
```

Examples:

```text
feat(remote): add connection retry support
fix(auth): handle expired SSH agent credentials
docs(changelog): record pending release changes
refactor(config): simplify endpoint loading
test(exec): cover command timeout behavior
chore(deps): update dependencies
```

Allowed types include `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `build`, `ci`, `perf`, `style`, and `revert`.

- Use a concise, lowercase, imperative description.
- Choose a short scope that identifies the affected component.
- Do not create commits whose subject does not follow this convention.

## Versioning

- Keep the package on the `0.1.x` line for small, backward-compatible features, behavior refinements, bug fixes, and documentation corrections.
- Increment the patch version for those release units, for example `0.1.4` to `0.1.5`.
- Use a new minor line such as `0.2.0` only for a substantial feature set or compatibility change that clearly warrants it.
- After `1.0.0`, follow standard Semantic Versioning: patch for fixes, minor for backward-compatible features, and major for breaking changes.
- Never create a version-only release. Every version bump must correspond to a tested, user-visible release unit.

## Changelog maintenance

`CHANGELOG.md` is the canonical and complete release history. It follows Keep a Changelog and must be maintained as part of every user-visible change.

- Record every user-visible addition, behavior change, deprecation, removal, bug fix, or security fix under `[Unreleased]` in the same commit as the change.
- Use the relevant `Added`, `Changed`, `Deprecated`, `Removed`, `Fixed`, and `Security` categories.
- Keep `Added`, `Changed`, and `Fixed` in every published version entry; write `None` when one of these required categories has no entries.
- Record npm publication dates in `YYYY-MM-DD` format. Do not use commit dates as release dates.
- At release time, move the pending entries from `[Unreleased]` into the new version and recreate an empty `[Unreleased]` section.
- Keep comparison links at the bottom of `CHANGELOG.md` aligned with the newest tag.
- Never claim a runtime change when a release only changed package or release metadata.

## README release index

`README.md` and `README.zh-CN.md` provide concise release indexes; they are not the canonical full history.

- Update both README release tables in the same release commit.
- Keep the latest release link and the most useful recent releases in each table.
- Ensure each table links to the matching `CHANGELOG.md` version entry.
- Keep the English and Chinese tables semantically aligned.
- Do not duplicate the complete changelog in either README.

## Release alignment

For every release, keep all public release state aligned:

1. Finalize the version entry and publication date in `CHANGELOG.md`.
2. Update the release indexes in `README.md` and `README.zh-CN.md`.
3. Set the same version in `package.json` and `package-lock.json`.
4. Run extension-load validation, relevant tests, `git diff --check`, and `npm pack --dry-run`.
5. Commit the complete release unit using the commit convention.
6. Create an annotated Git tag named `v<version>` at that commit.
7. Push the release commit and tag to GitHub over SSH.
8. Publish the exact same version to npm and verify the registry artifact matches the repository.
9. Create a GitHub Release from the same tag using the matching changelog entry as release notes.
10. Verify the npm version, Git commit, Git tag, GitHub Release, and changelog all describe the same artifact.

## Release safety

- Do not publish to npm, push release commits/tags, or create GitHub Releases unless the user explicitly requests a release.
- Confirm the working tree contains only intended changes before committing or publishing.
- Do not reuse, overwrite, unpublish, or retag an existing released version to correct metadata; publish an appropriate subsequent version instead.
- Never put access tokens, passwords, or other credentials in repository files, shell history, release notes, or chat messages.

---
> Source: [petrichor20211/pi-ssh-remote](https://github.com/petrichor20211/pi-ssh-remote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
