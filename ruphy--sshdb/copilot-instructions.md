## sshdb

> `sshdb` releases are tag-driven by `.github/workflows/release.yml`. The release workflow builds the platform archives, publishes the GitHub release using the matching `CHANGELOG.md` section as the release body, updates the Homebrew tap when `HOMEBREW_TAP_TOKEN` is available, and then commits `Cargo.toml` back to `main` first as the release version and then as `<version>-dev`.

# AGENTS

## Release Process

`sshdb` releases are tag-driven by `.github/workflows/release.yml`. The release workflow builds the platform archives, publishes the GitHub release using the matching `CHANGELOG.md` section as the release body, updates the Homebrew tap when `HOMEBREW_TAP_TOKEN` is available, and then commits `Cargo.toml` back to `main` first as the release version and then as `<version>-dev`.

Use this sequence for every release:

1. Start from `main` with a clean worktree.
2. Land the actual product changes first.
3. Update `Cargo.toml` to the release version and update `CHANGELOG.md` for the same version.
4. Run `cargo fmt --check` and `cargo test`.
5. Commit the release prep itself as `chore: update version to <version> for release`.
6. Create the tag on that exact commit: `git tag -a v<version> -m "v<version>"`.
7. Verify the tag points at `HEAD` before pushing:
   `test "$(git rev-parse HEAD)" = "$(git rev-list -n 1 v<version>)"`
8. Push `main`, then push the tag: `git push origin main` and `git push origin v<version>`.
9. Watch the `Release` GitHub Actions workflow until it finishes successfully.

Do not repeat the `0.17.0` mistake: never tag an older commit and assume the workflow will pick up newer release-prep changes. The pushed tag is the release.

---
> Source: [ruphy/sshdb](https://github.com/ruphy/sshdb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
