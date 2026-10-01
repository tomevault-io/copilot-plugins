## spaceo

> SpaceO gives AI agents a headless macOS display so they can drive real applications without

# Repository Guide

## Purpose

SpaceO gives AI agents a headless macOS display so they can drive real applications without
occupying the user's visible desktop, moving the physical pointer, or deliberately taking the
frontmost application. It isolates attention, not security: agent applications still run as the
logged-in user and retain that user's files, network, credentials, and application sessions.

The design depends on runtime-discovered private macOS APIs. Availability is not compatibility;
preserve fail-closed capability checks, explicit partial/unknown isolation results, and the release
qualification requirements.

## Project map

- `Sources/SpaceOKit/` — display lifecycle, placement, input, capture, isolation, daemon transport,
  ownership, and recovery.
- `Sources/SpaceOPrivate/` — the only target that directly resolves or calls private APIs.
- `Sources/spaceo/` — CLI, daemon, setup, doctor, and demo entry points.
- `Sources/SpaceOMCP/` — MCP stdio server and tool schemas.
- `Sources/SpaceOViewer/` — SwiftUI/AppKit display viewer and human-control surface.
- `Tests/SpaceOKitTests/` — deterministic and live XCTest coverage.
- `scripts/` and `Makefile` — supported build, test, Viewer, and release workflows.
- `docs/` — setup, live-test, support, recovery, installation, and release policy.
- `RELEASE_AUDIT.md`, `TICKETS.md`, `PRODUCT_BACKLOG.md` — safety history and open work.

## Working commands

- Build: `make build`
- Optimized build: `make release`
- Source install: `make install` (writes `~/.local/bin/spaceo` by default)
- Guided host setup: `spaceo setup` (requests TCC access, starts the daemon, self-tests, and prints
  MCP configuration; run only when the user asked to configure the host)
- Safe tests: `make test`
- Full deterministic release check: `make verify-release`
- One XCTest or suite while iterating: `swift test --filter <TestName>`
- Build an ad-hoc Viewer: `SPACEO_CODESIGN_IDENTITY=- make viewer`
- See every Viewer screen without touching the desktop: `scripts/viewer-snapshots.sh [scenario…]`
  (each `--preview` scenario in `ViewerPreview.swift`, launched with `--background` inside its own
  SpaceO session, one PNG per screen). Never launch the Viewer without `--background` from an
  agent: its normal launch activates it and takes the person's focus.
- Check the host without creating a session: `./.build/release/spaceo doctor`
- Improvement loop from real use: `spaceo logging enable`, then
  `node scripts/journal-report.mjs --daemon-log ~/Library/Logs/SpaceO/daemon.log`
  (see `docs/IMPROVEMENT_LOOP.md`; never paste `full` journal content into commits or tickets)
- Validate release configuration without publishing: `make release-check` and
  `make release-dry-run`

The Swift package uses tools version 5.9 and targets macOS 14 or later. Public packaging is
currently `arm64` only; CI and release workflows pin their own Xcode and Swift versions.

## Conventions

- Use GitHub noreply email addresses for public commits and tags. For GitHub CLI merges, pass
  `--author-email` with the intended author's GitHub noreply address explicitly; local Git
  configuration does not control GitHub's generated merge metadata. Check the resulting remote
  author and committer metadata without printing personal values.
- Prefer Accessibility actions over coordinate input. Treat unconfirmed delivery as unconfirmed;
  never turn an attempted action into a success claim without evidence.
- Keep CLI, daemon protocol, MCP schemas, limits, lease handling, and help text aligned when adding
  or changing a command.
- Preserve controller leases and operator scope. They coordinate same-user clients but are not a
  security boundary.
- Bound input, framing, collection sizes, waits, screenshots, and filesystem reads at every public
  boundary. Return structured errors instead of trapping or silently downgrading behavior.
- Keep safe tests deterministic and free of WindowServer, application, input, and general
  pasteboard mutation.
- Update `CHANGELOG.md` for user-visible behavior and the relevant audit/backlog record when closing
  a documented finding.

## Verification

- Run the narrowest relevant test while iterating.
- Before handoff, run `git diff --check` and `make verify-release`.
- For Viewer changes, build the bundle and run
  `codesign --verify --deep --strict --verbose=2 ".build/SpaceO Viewer.app"`.
- For release-policy or workflow changes, also run `bash Tests/ReleaseSecurityTests.sh`,
  `bash Tests/LiveTestGateTests.sh`, and `swift build -c release -Xswiftc -warnings-as-errors`.
- Do not count skipped live tests as release evidence. Qualification requires `make test-live-full`
  and `make computer-use-check-full` on an eligible host with retained results.

## Safety

- Never call or restore the incompatible private key/typing-focus getters. Never introduce
  `SLPSSetFrontProcessWithOptions`, production display-origin mutation, pointer warping, or a cursor
  fence; the audit documents display loss and memory corruption in these paths.
- Do not run `make test-live`, `make test-live-full`, `spaceo demo`, or the computer-use matrix on
  an actively used desktop. They create virtual displays, launch applications, and synthesize
  input in the current graphical login. Follow `docs/LIVE_TESTS.md` first.
- Treat `make install`, `make install-viewer`, and `spaceo setup` as host changes. Do not run them
  merely to verify a source edit.
- Keep SIP enabled. Do not suggest bypassing Gatekeeper or weakening TCC to make a test pass.
- Do not print, copy, rotate, or commit signing/notarization credentials, controller leases,
  screenshots, Accessibility content, or user data.
- Do not create or move a release tag, upload artifacts, or describe `1.0.0` as released unless all
  gates in `docs/RELEASE_POLICY.md` and `docs/RELEASE_HANDOFF_1.0.0.md` are satisfied and the release
  owner explicitly records GO.

---
> Source: [ParthJadhav/SpaceO](https://github.com/ParthJadhav/SpaceO) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
