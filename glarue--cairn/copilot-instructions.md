## cairn

> Working-session crib for Claude Code / Cursor / Copilot sessions on

# CLAUDE.md

Working-session crib for Claude Code / Cursor / Copilot sessions on
this repo. **Project architecture lives in
[`ARCHITECTURE.md`](ARCHITECTURE.md)**; read that first for the design
context. This file is the smaller, mutable layer on top — the
"what's the current state of the work" and "things that bit us
recently" notes that benefit from being adjacent to the AI's reading
path.

## Status snapshot

cairn shipped to the App Store as v0.3.1, build 58 on TestFlight. The
core deletion-propagation pipeline is complete end-to-end. The two
significant in-flight workstreams are (a) closing audit findings
from a pre-launch code review (see `notes/audit-synthesis.md` for the
prioritized list), and (b) the session-auth path for `/sync/*`
endpoints (the incremental server sync feature is wired but
unreachable via API key auth — see ARCHITECTURE.md's "API endpoints"
section).

## Working assumptions you can rely on

- Repo root: `/Users/graham/code/cairn/`.
- Maintainer keeps a shallow Immich-server checkout at
  `/Users/graham/code/immich` for cross-referencing API behavior
  against actual server source rather than training data.
- `swift test` is the source of truth for behavior. Run it before
  declaring work done.
- iOS builds via `xcodebuild -project iOS/Cairn.xcodeproj -scheme
  Cairn -destination "generic/platform=iOS Simulator" build` — used
  to verify changes that touch the iOS app target (which `swift
  build` doesn't cover).
- Sensitive credentials in `.env` (gitignored). **Never echo `.env`
  contents to tool output** — read it via `source` in a subshell
  when needed.

## Open work

In rough priority order:

1. **Audit follow-ups** — see `notes/audit-synthesis.md` for the
   prioritized list. Three pre-launch bundles: docs sweep (this
   file's existence is part of that), pre-1.0 schema rename (mostly
   done — see `confirmedFromChangeLog` rename), accessibility floor
   for destructive surfaces.
2. **Session auth for `/sync/*`** — the incremental-server-sync
   feature ships off-by-default because Immich rejects `/sync/*`
   from API key auth. Adding email/password login → JWT session
   cookie would let the feature run end-to-end. See
   `docs/active-design/sync-stream-incremental-server-sync-plan.md` for the
   pre-discovery design (the auth assumption was wrong; the
   coordinator + cache infrastructure are correct).
3. **Snapshot tests for SwiftUI screens.** None yet. Good candidate
   is `swift-snapshot-testing` from Point-Free. Priority targets:
   Setup flow, DryRunSheet phases, PendingReviewScreen
   (empty / populated / mass-offload variants).
4. **Local OS notifications for backlog alerts.** In-app Status
   banner already exists (gated by
   `CairnSettings.deletionBacklogAlertThreshold`). Next: fire a
   local `UNNotificationRequest` from `handleBackgroundRefresh`
   when a scan causes the backlog to cross the threshold
   (edge-trigger).

## Release notes workflow

Two layers, with different audiences and different freshness:

- **Per-build TestFlight changelog** —
  `iOS/fastlane/beta_changelog.txt`. One paragraph describing
  what's new in *this build*. Overwritten on every bump. Goes to
  TestFlight testers via the fastlane `beta` lane. Convention: the
  bump commit (`Bump to build N (...)`) stages the new content, so
  `git show <bump-sha>:iOS/fastlane/beta_changelog.txt` resurrects
  the per-build copy.

- **Per-version App Store release notes** —
  `iOS/fastlane/metadata/en-US/release_notes.txt`. Cross-build
  prose describing what's new since the last App Store release.
  Goes to App Store reviewers + users via Apple's "What's New"
  field. Written when prepping a marketing-version push.

`iOS/scripts/release-notes.sh <from-ref> [to-ref]` aggregates every
checkpointed `beta_changelog.txt` between two refs into a draft.
Typical use when prepping a release:

```sh
iOS/scripts/release-notes.sh v0.3.1 | tee release-draft.md
# Hand-edit / group by theme.
# Paste the cleaned-up version into iOS/fastlane/metadata/en-US/release_notes.txt
```

The script tolerates both subject conventions in the history —
`Bump to build N` (current) and `Bump build to N` (pre-build-56).
Marketing-version bumps are deliberately excluded.

### Tagging App Store releases

`fastlane tag_live_release` queries App Store Connect for the
version currently in `READY_FOR_SALE`, reads its build number +
uploaded timestamp, and tags the commit that was HEAD at that
timestamp as `v<marketing-version>`. Time-correlation (not commit-
subject matching) is the reliable signal because `fastlane beta`
auto-bumps CFBundleVersion via its internal `build_ipa` →
`bump_build` chain — re-runs of the lane can silently increment
without committing, so a TestFlight build number has no guaranteed
matching git commit, but its upload wall-clock time always maps to
one git HEAD.

Idempotent on the tag name. Pass `push:true` to also push to
origin:

```sh
cd iOS && bundle exec fastlane tag_live_release         # local tag only
cd iOS && bundle exec fastlane tag_live_release push:true
```

Run this once after each App Store version is approved + released
— the tag then serves as the baseline for the next
`release-notes.sh` run.

## Memory and other forms of persistence

Memory is one of several persistence mechanisms available to AI
sessions. The distinction:

- **Plans** — for non-trivial implementation tasks where alignment
  on approach is useful. Update the plan if the approach changes
  mid-implementation.
- **Tasks** — for tracking discrete work items within a session.
  Mark completed as you go.
- **CLAUDE.md** (this file) — cross-session project state worth
  keeping adjacent to the codebase. Stays in version control so
  every contributor (and every AI session) sees the same context.
- **Claude's user-memory** — cross-conversation context about the
  *user* (their preferences, role, the kind of feedback they give).
  Not the place for project state.

If the project changes in a way that contradicts something in
ARCHITECTURE.md or this file, update them. Stale architectural notes
are worse than no notes.

---
> Source: [glarue/cairn](https://github.com/glarue/cairn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-06 -->
