## gloomhavenvr

> Read `CLAUDE.md`, `.planning/STATE.md`, and the newest build notes beside

# Working on GloomhavenVR

Read `CLAUDE.md`, `.planning/STATE.md`, and the newest build notes beside
`NetProtocol.ModBuild` before implementation. `CLAUDE.md` retains the project's
technical contracts and historical reasoning; the rules below adapt its workflow
to Codex and record the user's instructions of 2026-09-08.

- The primary agent is the user's contact and integrator. Delegate independent
  implementation tasks to workers with separate Git worktrees and explicit,
  disjoint file ownership. Create each worktree from the current `dev` integration
  commit, never from the older release branch. Respect the available concurrency
  limit; do not assume a tool creates an isolated worktree automatically.
- Review worker changes, integrate them into `dev`, run the complete required
  checks, and push directly to `origin/dev`. This is authorized by the user;
  another confirmation is unnecessary. Never push worker branches or force-push.
- Speak to the user in German. Write code, comments, and developer documentation
  in English. Product strings remain English and German through `Core/Loc`.
- Hardware evidence lives in the main checkout's gitignored `.planning/debug/`;
  the other player's logs are in `.planning/debug/remote/`. Verify both build
  banners and inspect supplied screenshots before assigning a cause.
- Initialize worker dependencies with `scripts/worktree-setup.sh`. Keep any new
  refactor baseline private to its worktree; never overwrite a shared symlink.
- Never use `git stash`. Preserve unrelated work and the read-only game references.
  Do not print `.env` or copy its contents into tracked files.
- Keep player documentation and release highlights focused on installation from release archives,
  controls and visible changes. Installer scripts, CI/build details and pending hardware tests
  belong in developer documentation or `.planning/`, not in player instructions.
- The maintainer always tests at Debug. Ordinary players use the normal log level, which
  must remain useful for bug reports: retain build/version, important lifecycle and flow
  context, failures, and bounded reports of significant anomalies. Deduplicate or rate-limit
  recurring reports; normal logging must be informative without growing with every frame
  or routine action. Detailed measurements and frequent state traces belong at Debug, with
  bounded repetition and guards before expensive sampling or string formatting. Do not
  promote diagnostic streams merely to simplify the maintainer's hardware tests, or hide all
  useful failure context at Debug (user clarification, 2026-09-18).
- Commit useful checkpoints. Attribute commits truthfully: the Claude-specific
  coauthor and session-link template in `CLAUDE.md` does not apply to Codex.
- Distinguish source-proven fixes, log evidence, and unverified hardware outcomes.
  A green automated check does not establish that a headset picture is correct.

## Multiplayer visual parity

The user's 2026-09-09 review instruction covers everything the owner sees: original widgets,
content, appearance, order, geometry, state, effects and intermediate animation. Only an
explicitly user-confirmed exception permits a divergence. Historical deferrals, performance
arguments and implementation comments are not approvals. Record the source of each actual
exception; fix newly discovered divergences within the authorized review. Native prefab clones
must retain original presentation without running gameplay controllers or callbacks.

The latest multiplayer test ruling (2026-09-09, ModBuild 486 evidence) supersedes older
pile/active-card face exceptions: during the action phase cards are face-up; during
ability selection, remote fans, held cards and placed cards are face-down. Remote short-rest
burn flights must remain face-down. Damage-sacrifice choices during the action phase
remain face-up. Card-face visibility and permission to name private cards remain separate.

The user clarified during the build 490 review (2026-09-09): concealment is exclusively a
remote presentation rule. Local cards of controlled characters are never concealed, including
short-rest burn flights and fallback artwork. During selection, local character switching is
already limited to controlled characters; do not add remote secrecy gates to local rendering.

The build 490 hardware report (2026-09-09) explicitly confirms that the entire 3D map
environment is public: neither local nor remote fans, held cards or other map card surfaces
may be concealed. Artwork construction failures are rendering defects, not privacy decisions.

The build 500 hardware ruling (2026-09-14) requires immediate return of a held figure to
its board position before any non-idle action starts, locally and for remote observers.
This supersedes the older forced-release glide ruling; ordinary idle manual releases keep
their usual glide. Native movement must never derive its start from a hand-held transform.

The user's 2026-09-17 window-placement request permits necessary, animated opening-time
room making when a new window would overlap an existing visible window. Keep affected
windows in view and use one author for shared movement. This narrowly supersedes the
historical fixed-window rule; it does not authorize periodic head-following or lost-window
recall. Manual grabs take precedence, and layout must never gate native continuation.
The build-526 hardware clarification fixes the newly opened window at its spawn pose: only
older, visibly overlapping windows may move. Transparent host/hit areas are not visual overlap;
a free quest-popup gaze centre takes precedence over the legacy beside-quest-log preference.

The user's 2026-09-17 tutorial clarification restores the original task-specific hands/controller
choice and animated transitions. Card handling and physical fingertip tasks show hands; key
lessons show controllers. Both tracked sides must remain represented by either a hand or a
controller. Build 518's interpretation that controller models must stay visible for the whole
lesson was incorrect and is superseded.

The user's build-529 Guildmaster clarification (2026-09-18) keeps the original quest list
visible during map browsing, including the mode's persistent ordinary dialog. Only actual
quest commitment and its subsequent story/loadout may hide it. A dialog asking to hide
other native UI is not, by itself, proof of Guildmaster quest acceptance.

The user's build-531 hardware clarification (2026-09-18, `wasser_hintergrund.jpg`)
restricts mixed-reality backings to UI elements. Never add MR background geometry inside
the play area, including water, fog/unseen terrain and preview meshes. Earlier scenery
backing implementations and comments do not authorize exceptions to this rule.

The user's build-536 options-menu clarification (2026-09-20) requires VR Options to
leave other menu entries' focus and availability untouched. A grey native button must
represent genuine native disablement, never opening the independent VR window. The VR
entry is a toggle synchronized with window closure, including X; the next press must
immediately reopen it, without an extra deselect click or a grace period.

---
> Source: [McFredward/GloomhavenVR](https://github.com/McFredward/GloomhavenVR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
