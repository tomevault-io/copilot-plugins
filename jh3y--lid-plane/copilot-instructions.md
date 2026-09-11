## lid-plane

> Standalone macOS SwiftPM menu bar app. Work from this directory using SwiftPM. No third-party dependencies.

# Working on Lid Plane

Standalone macOS SwiftPM menu bar app. Work from this directory using SwiftPM. No third-party dependencies.

## Commands

- Build without launching: `./script/build_and_run.sh --build`.
- Build and launch the bundle: `./script/build_and_run.sh`.
- Motion checks: `swift run LidPlaneChecks` (custom executable, not XCTest).
- Metal preview: `./script/build_and_run.sh --preview`.
- Visible-pixel regression and hotkey dispatch: `./script/build_and_run.sh --window-check`. Briefly shows generated artwork and exits; tell the user before running it.
- Package without changing the everyday app: `./script/package_release.sh --experimental`.
- Export a fresh standalone folder: `./script/export_standalone.sh [new-destination]`.

## Preserve these behaviours

- Start off, as a menu bar app, without stealing focus.
- Keep the overlay click-through and non-key; normal input passes through.
- Keep sensing on an independent timer while the overlay is hidden.
- Set the Metal layer's `contentsScale` to the display backing scale when sizing it. Zero scale produces invisible content even when GPU commands succeed. Keep the visible-pixel regression check.
- Soften warped image coverage with the progressive blur; do not return a hard background colour outside UV bounds before blurring the boundary. Keep `RenderChecks` in the preview command: it verifies blur outside both side edges, tighter falloff near the hinge and blur-off behaviour.
- Retain capture buffers until GPU work completes; exclude this app from capture.
- Keep desktop frames in memory. No disk recording, networking, audio, camera, login service or broad input monitor without explicit authorization. Diagnostic PNGs must contain generated artwork only.
- Preserve ordinary lid-close sleep and graceful sensor/capture failure handling.
- Register only the toggle hotkey. Report conflicts without adding Accessibility or Input Monitoring requirements.

## Shipping and permissions

- Read `README.md`, `DEVELOPMENT.md` and `DISTRIBUTION.md` before changing onboarding or packaging.
- Only distributable ZIPs, `dist/SHA256SUMS.txt` and `dist/README.md` belong in Git. Exclude caches, loose bundles, screenshots and exported copies.
- Keep binary versions, download links, signatures and checksums consistent.
- Ad-hoc builds are experimental, not notarized. Never imply otherwise or advise disabling Gatekeeper.
- Rebuilding can invalidate Screen Recording approval. Avoid unnecessarily rebuilding the working bundle; permission repair requires authorization and must target only `dev.jhey.lidplane`.
- Do not create a public repo, choose a source license, upload a release, change permissions or install signing identities without authorization.
- Report what was actually verified. Local tests do not establish compatibility with every MacBook or Gatekeeper acceptance of a downloaded app.

---
> Source: [jh3y/lid-plane](https://github.com/jh3y/lid-plane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-10 -->
