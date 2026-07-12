## web-cap

> When asked to publish a beta version:

# Agent Notes

## Beta Release Flow

When asked to publish a beta version:

1. Do not change `package.json` version on the current branch.
2. Determine `BASE_VERSION` from the latest npm beta dist-tag, unless the user explicitly pins a different base version. Do not derive it from the current branch `package.json`.
   - npm beta: `npm_config_cache=/private/tmp/web-cap-npm-cache npm view web-capability@beta version`
   - Example: if npm beta is `0.0.7-beta.3`, then `BASE_VERSION=0.0.7` and the next beta is `0.0.7-beta.4`.
3. Check existing beta versions from both npm and the private git remote, then choose the next beta number for that `BASE_VERSION`.
   - npm: `npm_config_cache=/private/tmp/web-cap-npm-cache npm view web-capability versions --json`
   - private tags: `git ls-remote --tags private "v${BASE_VERSION}-beta.*"`
4. Commit any requested code changes on the current branch first, without changing the package version.
5. Create a temporary release worktree from the current `HEAD`.
   - Example: `git worktree add /private/tmp/web-cap-tag-${BASE_VERSION}-beta.N HEAD`
6. In that temporary worktree only, update `package.json` to the beta version.
7. Commit the release version change in the temporary worktree.
   - Example: `chore: release ${BASE_VERSION}-beta.N`
8. Create `v${BASE_VERSION}-beta.N` on that temporary release commit.
9. Push only the tag to the `private` remote.
   - Example: `git push private v${BASE_VERSION}-beta.N`
10. Verify:
   - `git show v${BASE_VERSION}-beta.N:package.json` reports the beta version.
   - The current branch `package.json` still has its original version.
   - The current branch worktree is clean.

---
> Source: [edgestorage/web-cap](https://github.com/edgestorage/web-cap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
