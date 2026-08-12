## release-publisher

> Create semver Git tags and GitHub releases from actual repository changes, including release-note generation, annotated tags, and GitHub release publishing.


# Release Publisher

Use this rule when preparing, tagging, or publishing a semantic-versioned GitHub release.

Create releases from repository state, not guesses.

## Workflow

1. Inspect the repository before changing anything.
   - Check `git status --short --branch`.
   - If there are uncommitted changes, inspect them with `git diff` and `git diff --staged`.
   - Stage the uncommitted changes and commit them before starting the release flow.
   - Use a concise, meaningful commit message based on the actual diff; do not use a generic release message unless the diff only contains release artifact updates.
   - After committing pending changes, continue the release flow from the new `HEAD`.
   - Find the latest tag with `git tag --sort=-v:refname`.
   - Inspect unreleased commits with `git log <latest-tag>..HEAD --oneline`.
   - If there are no unreleased commits and the user did not provide a version, stop and report that there is nothing to release.

2. Decide the version.
   - If the user explicitly provides a version, use it unless it conflicts with existing tags.
   - Otherwise derive the bump from the actual changes since the latest tag.
   - Use `patch` for fixes, docs-only changes, tooling-only changes, dependency range relaxations, configuration adjustments, test-only changes, or other backward-compatible maintenance.
   - Use `minor` for backward-compatible features, new endpoints, new public fields, new optional behavior, or additive package capabilities.
   - Use `major` for breaking API changes, removed behavior, incompatible payload or response changes, renamed stable public surface, or required migration steps for consumers.
   - If the change set mixes categories, choose the highest required bump.
   - Never bump blindly by `+0.0.1` unless the diff truly merits a patch release or the user explicitly asked for that exact versioning rule.

3. Summarize only real changes.
   - Base release notes on the diff and commits since the last tag.
   - Do not invent features, tests, or docs updates.
   - Format the GitHub release body exactly as:

```markdown
## <tag> - <YYYY-MM-DD>

### What's Changed

<One concise paragraph summarizing the release in plain language.>

### Features

- <Feature bullet.>

### Fixes

- <Fix bullet.>

### Tooling

- <Tooling, documentation, request-example, CI, or developer-experience bullet.>

### Tests

- <Test coverage bullet.>

**Full Changelog**: <canonical compare URL>
```

   - Use the tag text exactly as it will appear on GitHub, including the `v` prefix when the repository uses one.
   - Use the current local date for the release date unless the user explicitly provides another date.
   - Keep the `What's Changed` heading even when the release is small, and write the summary as prose instead of bullets.
   - Include only change category sections that have real supporting changes, but keep supported sections in this order: `Features`, `Fixes`, `Tooling`, `Tests`.
   - Use the exact heading `Tooling` for documentation, request examples, CI, scripts, configuration, and developer-experience changes.
   - End with exactly one `**Full Changelog**: ...` line using the canonical GitHub repository path and a compare range from the previous tag to the new tag.

4. Create a release commit only when there is a meaningful release artifact.
   - Prefer updating an existing `CHANGELOG.md` or equivalent release-tracking file if the repository uses one.
   - Do not create empty release commits just to force a new SHA.
   - If there is nothing to edit for the release itself, release from the existing unreleased commits.

5. Create and publish the tag.
   - Use the repository's existing tag naming convention.
   - Create an annotated tag and use the generated release notes as the tag message.
   - Push the branch first, then push the tag.
   - If GitHub reports the repository has moved, use the canonical destination for compare links and release creation.

6. Create the GitHub release.
   - Use the exact release body format from step 3 for the GitHub release text.
   - Target the same tag and branch that were pushed.
   - Confirm the published release URL in the final response.

## Guardrails

- Treat tags, changelog entries, request and response shapes, and public package APIs as stable surfaces.
- Prefer primary repository evidence: git history, tracked files, remotes, and GitHub API responses.
- Do not assume `gh` is installed; fall back to the GitHub API if needed.
- Do not rewrite or squash unrelated user commits.
- Commit any uncommitted working tree changes with a meaningful message before deriving the release version or notes.
- If the requested version already exists, stop and report the conflict instead of retagging.

## Final Response

Report:

- the version released
- the commit used for pre-release uncommitted changes, if one was created
- the commit used for the release commit, if one was created
- the tag name
- the GitHub release URL
- whether tests were run

---
> Source: [pictastudio/contento](https://github.com/pictastudio/contento) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
