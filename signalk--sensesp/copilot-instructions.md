## sensesp

> Guidance for AI coding agents working in this repository. Human contributor

# AGENTS.md

Guidance for AI coding agents working in this repository. Human contributor
docs live in [`docs/pages/contributing_code/index.md`](docs/pages/contributing_code/index.md);
the human release guide is [`RELEASE_PROCESS.md`](RELEASE_PROCESS.md). This file
currently covers the release workflow, where the agent-specific mechanics differ
enough from the human steps to be worth writing down.

> **Building firmware _with_ SensESP, rather than developing the framework?**
> The [SensESP Workspace](https://github.com/hatlabs/SensESP-workspace) is a more
> comprehensive Claude Code environment for that: it bundles the framework and
> add-on libraries as reference code, per-board hardware docs and pin maps,
> project scaffolding, and a guided requirements-to-flashing workflow. This
> `AGENTS.md`, by contrast, is for agents working on the SensESP framework itself.

## Build and test

This is a library, not an app — `src/*.cpp` has no `setup()`/`loop()`, so a bare
`pio run` at the root does not link. The library is built against an example:

```bash
# Build the library against one example (what CI does per example)
PLATFORMIO_CI_SRC=examples/minimal_app.cpp ci/run-ci.sh

pio test -e native                               # host tests (test/native/*)
pio test -e pioarduino_esp32 -f "system/<name>"  # on-target tests (test/system/*)
```

`test/native` only holds tests over self-contained headers (no `sensesp.h`
include). Anything pulling in `sensesp.h` — the SK client, WiFi, most of the
framework — needs Arduino/ReactESP/FreeRTOS and builds only on-target, so those
tests live in `test/system/*` and run in CI, not on the host. To compile-check a
header-only change without a device, add a throwaway `test/system/` test that
includes and uses it, then build it with
`pio test -e pioarduino_esp32 -f "system/<name>" --without-testing --without-uploading`
(`[PASSED]` on such a run means it compiled and linked; nothing ran on a device).

## Release process

This mirrors [`RELEASE_PROCESS.md`](RELEASE_PROCESS.md) step for step; the notes
below are the agent-specific mechanics and pitfalls. Prerequisites, all expected
to be present in the maintainer's environment: `bump2version`, `pio`, an
authenticated `pio account` (for the publish), and `npm`/`terser` (only if the
web-UI bundle needs rebuilding).

1. **Choose the version (semver).** Read `git log <latest vX.Y.Z tag>..origin/main --no-merges`.
   Any `feat` → minor bump; any `BREAKING CHANGE:` footer or `type!:` subject →
   major; otherwise patch. Do **not** read the target off `.bumpversion.cfg`: its
   version is an `-alpha` placeholder auto-set after the previous release and it
   usually undershoots (a patch-alpha even when features have landed).

2. **Work in a fresh worktree** off `origin/main` (`git worktree add -b release_X.Y.Z ...`)
   so the maintainer's checkout is left alone.

3. **Bump.** `bump2version --new-version X.Y.Z minor` (the part argument is
   ignored when `--new-version` is given). It rewrites all six version-bearing
   files — `VERSION`, `library.json`, `library.properties`, `Doxyfile`,
   `src/sensesp/sensesp_version.h`, `.bumpversion.cfg` — and auto-commits
   `Bump version: A → B`. Editing by hand misses some of these, so use the tool.

4. **Web-UI bundle.** `scripts/update_autogen.sh` regenerates
   `src/sensesp/net/web/autogen/frontend_files.h` and commits only if it changed.
   First check whether it is even needed:

   ```bash
   git log <last "rebuild embedded frontend bundle" commit>..HEAD \
       -- frontend/src frontend/scss frontend/index.html frontend/public
   ```

   If that is empty, no frontend source changed since the last rebuild, the
   bundle is current, and the script is a no-op — skip it. Re-running the build
   can emit a spurious diff from a different `terser`/`pnpm` version, so only run
   it when frontend source actually changed.

5. **Open the release PR.** Push the branch. `gh pr create` is blocked by a hook
   that reveals a sanctioned helper path at the moment of use — use that path,
   and never copy it into any durable file. Merging is separately gated: it
   requires a real maintainer message naming the PR ("merge NNNN"); an agent
   cannot self-authorize the merge.

6. **After merge, draft the GitHub release.**

   ```bash
   gh release create vX.Y.Z --draft --target <merge-commit-sha> \
       --title "Version X.Y.Z" --notes-file <notes>
   ```

   A `--draft` release creates no git tag until it is published, so it is safe to
   prepare and hand to the maintainer to review and publish. Build the notes from
   `git log <lasttag>..HEAD --no-merges` grouped by conventional-commit type.

7. **PlatformIO publish — mind the version.**
   `pio package publish --owner SignalK --no-interactive .` packages whatever
   version is in the **current worktree's** `library.json`. Run it from a clean
   `vX.Y.Z` **tag** checkout (`git worktree add --detach ../pub vX.Y.Z`), never
   from `main`: by publish time `main` has already been bumped to the next
   `-alpha` (step 8), and publishing from there would push a dev-alpha as the
   release. `library.json`'s `export.include` keeps the package to source files.
   The registry accepts the upload asynchronously and emails the maintainer when
   it finishes.

8. **Post-release bump.** New worktree/branch off `origin/main`,
   `bump2version patch` → `X.Y.(Z+1)-alpha`, then PR and merge. Its order
   relative to step 7 does not matter as long as step 7 publishes from the tag.

---
> Source: [SignalK/SensESP](https://github.com/SignalK/SensESP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
