## siming-ai

> For every user-visible feature change in Siming, complete the release workflow

# Siming Release Requirement

For every user-visible feature change in Siming, complete the release workflow
before reporting the work as finished unless the user explicitly asks not to
release it:

1. Update the application version and release metadata.
2. Complete a UI design review for every affected user-visible flow before
   packaging. Inspect the actual application at 1920x1080 and the minimum
   supported viewport, covering visual hierarchy, spacing and alignment,
   overflow and text truncation, responsive behavior, Chinese copy, and the
   loading, empty, error, disabled, and completed states. Capture screenshots
   of the key flow and treat unresolved visual regressions as release blockers.
   For Android changes, repeat this review in an actual emulator or device at
   the smallest supported phone viewport and one current reference viewport.
3. Run the relevant backend tests, frontend lint and tests, and
   `frontend/npm run build`. When Android or Gateway is affected, also run the
   Android unit tests/lint/APK build and validate the Docker image on every
   supported architecture.
4. Build the distributable with `build-exe.bat` and verify `Siming.exe`,
   `update.json`, and `sha256.txt` agree. Android releases must also produce a
   signed, installable `Siming.apk` plus its SHA-256 file, with the manifest
   version matching the desktop release.
5. Commit and push the versioned change.
6. Create or update the corresponding GitHub Release and upload the verified
   release assets. Gateway releases must publish the matching multi-architecture
   container image and complete a non-root, writable-data smoke test.

Report the release URL, commit, and validation outcome in the final handoff.

---
> Source: [teangtang1122/siming-ai](https://github.com/teangtang1122/siming-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
