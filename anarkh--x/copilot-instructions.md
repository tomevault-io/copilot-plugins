## x

> Guidance for AI coding agents working on this repository.

# AGENTS.md

Guidance for AI coding agents working on this repository.

## Project Overview

Street Tasks is a native WeChat mini program for short-lived neighborhood tasks. The current product is intentionally small: a map-first feed, a publish flow, a task detail page, comments, structured trust actions, lightweight admin moderation, profile/activity surfaces, and feedback. There is no fixed service area; users can browse and publish from any current location.

The app can run locally from mock data and `wx` local storage, and it also has CloudBase-backed paths for shared posts, reactions, comments, feedback, viral attribution events, images, and admin role checks. Treat `utils/store.js` as the main persistence boundary; page code should not duplicate storage or cloud fallback logic.

## Harness Operating Loop

This repository keeps agent state, verification, and handoff files under `harness/`. Treat those files as the durable source of truth for long-running AI-assisted work.

Before changing code:

1. Run `pwd` and confirm the repo root is `/Users/bytedance/git/x`.
2. Read `harness/claude-progress.md` for the latest verified state, blocker, and next action.
3. Read `harness/feature_list.json` and pick the highest-priority unfinished feature. Keep at most one feature `in_progress`.
4. Check recent history with `git log --oneline -5`.
5. Run `bash harness/init.sh`. If it fails, fix the base state before adding feature work.

Required harness files:

- `harness/feature_list.json`: machine-readable feature status and verification evidence.
- `harness/claude-progress.md`: session log and current verified state.
- `harness/init.sh`: single bootstrap and baseline verification entrypoint.
- `harness/session-handoff.md`: short handoff summary for longer sessions.
- `harness/clean-state-checklist.md`: closeout checklist before handing work back.
- `harness/evaluator-rubric.md`: acceptance rubric for completed work.
- `harness/quality-document.md`: project quality snapshot by product area and architecture layer.

Completion definition:

- The target behavior is implemented.
- Required verification actually ran.
- Evidence is recorded in `harness/feature_list.json` or `harness/claude-progress.md`.
- The repo can still be restarted from `bash harness/init.sh`.
- Any skipped manual WeChat DevTools checks are called out as unverified, not implied passing.

Session closeout:

1. Update `harness/claude-progress.md`.
2. Update `harness/feature_list.json` when feature status or evidence changes.
3. Note unresolved risks or blockers.
4. Run the relevant verification commands.
5. Leave the next session able to continue from repo files alone.

## Tech Stack

- Native WeChat mini program files: `.js`, `.json`, `.wxml`, `.wxss`
- JavaScript ES modules
- Local persistence through `wx.getStorageSync` / `wx.setStorageSync`
- WeChat location APIs using `gcj02`
- No build framework and no frontend package bundler

## Important Files

- `app.js`: app bootstrap, guest user initialization, and global center state.
- `app.json`: page registry, tab bar, window style, and location permission declaration.
- `project.config.json`: public WeChat DevTools config. Keep `appid` as `touristappid` for GitHub.
- `project.private.config.json`: local-only WeChat DevTools config. This may contain the real AppID and must stay ignored.
- `utils/config.js`: nearby feed config, categories, and expiry options.
- `utils/store.js`: post, comment, reaction, image upload, local storage, and CloudBase fallback APIs.
- `utils/viral-attribution.js`: best-effort share landing, conversion, and relay attribution events with local fallback and a strict field whitelist.
- `utils/auth.js`: local user, profile completion, admin role refresh, and permission helpers.
- `utils/feedback.js`: user feedback creation and admin feedback listing.
- `utils/geo.js`: distance calculation and map marker conversion.
- `utils/format.js`: category and time display helpers.
- `utils/post-presenter.js`: shared presentation helpers for profile/activity pages.
- `utils/diagnostics.js`: runtime diagnostics used by map startup and fallback paths.
- `utils/mock-posts.js`: seed data used when local storage is empty.
- `harness/*`: agent harness state, verification, closeout, and quality tracking files.
- `DESIGN_SYSTEM.md`: current visual design rules and native/TDesign-style component patterns.
- `PROJECT_SUMMARY.md`: high-level project summary for humans and future agents.
- `pages/map/*`: map feed, marker interactions, and list overlay.
- `pages/publish/*`: task creation flow.
- `pages/detail/*`: detail view, images, comments, confirm/stale/report actions, and resolve flow.
- `pages/admin/*`: admin-lite moderation view.
- `pages/me/*`: login, profile, admin entry, and personal stats.
- `pages/my-posts/*`: current user's posts.
- `pages/activities/*`: current user's trust-action history.
- `pages/feedback/*`: user feedback form.
- `cloudfunctions/posts/index.js`: CloudBase actions for posts, reactions, comments, feedback, viral attribution events, and image upload preparation.
- `cloudfunctions/getMyRole/index.js`: CloudBase admin role lookup from the `admins` collection.
- `scripts/check-json.mjs`: JSON syntax check for project and page config files.

## Local Development

1. Open the repository in WeChat DevTools.
2. Keep `project.config.json` with the placeholder AppID:

   ```json
   "appid": "touristappid"
   ```

3. Put the real local AppID in `project.private.config.json`. WeChat DevTools gives this file higher priority for local settings, and `.gitignore` excludes it from version control.
4. Run the JSON sanity check after editing any `.json` config:

   ```bash
   npm run check:json
   ```

There is no general unit test suite yet.

## Verification

Use these baseline checks for most changes:

```bash
bash harness/init.sh
node scripts/check-json.mjs
node harness/check-harness.mjs
git diff --check
```

After editing JavaScript, run targeted syntax checks, for example:

```bash
node --check pages/map/map.js
node --check utils/store.js
```

User-visible mini program behavior still needs WeChat DevTools or real-device verification. Record any manual evidence, skipped checks, or remaining risks in `harness/claude-progress.md` or `harness/feature_list.json`.

## Data Model Notes

Posts are plain objects with these important fields:

- `id`: string post id such as `post_001` or `post_${Date.now()}`.
- `markerId`: numeric map marker id.
- `title`, `body`, `category`, `placeName`.
- `intent`: optional subtype, currently used by `lost_found` as `lost` or `found`.
- `latitude`, `longitude`.
- `imageUrls`: optional image file IDs or URLs. Shared image posts should use `cloud://` file IDs.
- `status`: `active`, `stale`, `resolved`, `expired`, or `hidden`.
- `confirmations`, `lastConfirmedAt`, `staleCount`, `reportCount`.
- `createdAt`, `expiresAt`: timestamps in milliseconds.
- `publisherId`, `publisher`, `publisherAvatarUrl`: publisher identity and display metadata.

`listPosts(center)` computes derived status and distance, filters hidden posts, sorts newest first, and caps results by `config.maxVisiblePosts`.

Comments are stored locally under `post_comments` or in the CloudBase `post_comments` collection. Feedback is stored locally under `feedback_items` or in the CloudBase `feedback_items` collection. Trust reactions are stored locally under `post_reactions` or in the CloudBase `post_reactions` collection. Viral attribution events are stored locally under `viral_attribution_events` or in the CloudBase `viral_attribution_events` collection and must not contain comment body, contact/group data, raw OpenID, or precise coordinates.

## Behavior Rules

- Confirming a post increments `confirmations`.
- Confirming also stores `lastConfirmedAt` and one local user cannot repeat the same trust action on the same post.
- Marking a post stale increments `staleCount`; after 3 stale reports, status becomes `stale`.
- Reporting a post increments `reportCount`; after 2 reports, status becomes `hidden`.
- Resolving a post sets status to `resolved`.
- Expired posts are marked as `expired` when listed.
- Hidden posts should not appear in normal list results.
- Closed posts (`hidden`, `resolved`) should not accept comments or trust actions.

## CloudBase And Images

CloudBase is optional for local development but required for shared multi-user data. Expected collections:

- `posts`
- `post_reactions`
- `post_comments`
- `feedback_items`
- `viral_attribution_events`
- `admins`

Text-only posts may fall back to local storage when cloud APIs are unavailable. Image posts are stricter: selected images are compressed, capped at 4 files under 1.5MB each, uploaded to CloudBase Storage, and saved as `cloud://` file IDs. If cloud upload or cloud post creation fails for an image post, fail explicitly instead of saving local-only temp image paths.

## Coding Conventions

- Follow the existing native WeChat mini program style.
- Keep shared behavior in `utils/*` instead of duplicating page logic.
- Keep public user-facing copy in Chinese unless the surrounding file is developer documentation.
- Avoid adding dependencies unless the feature clearly needs them.
- Use `wx.navigateTo` for non-tab pages and `wx.switchTab` for tab pages.
- Preserve the local-storage API shape in `utils/store.js` unless replacing the persistence layer deliberately.
- Keep JSON files strict JSON with no comments.

## Security And Git Hygiene

Do not commit local secrets or machine-specific files.

Ignored local files currently include:

- `project.private.config.json`
- `.agents/`
- `.claude/`
- `skills-lock.json`
- `log/`
- `*.log`
- `node_modules/`
- `miniprogram_npm/`

The file `aaa` is also ignored and may contain local sensitive proxy configuration. Do not add it with `git add -f`.

Before pushing, run:

```bash
rg --no-ignore -n -i "(api[_-]?key|secret|token|password|passwd|pwd|private[_-]?key|session|cookie|authorization|bearer|access[_-]?token|refresh[_-]?token|client[_-]?secret|appsecret|wx[0-9a-f]{16,}|sk-[A-Za-z0-9_-]{20,}|AKIA[0-9A-Z]{16})" .
npm run check:json
git status --short --ignored
```

Expected secret-scan caveat: `TODOS.md` may mention future `appid`, private key, and CI secret handling as plain text. That is documentation, not an actual secret.

## Common Tasks

When changing categories:

- Update `utils/config.js`.
- Check formatting behavior in `utils/format.js`.
- Verify publish picker and map/detail/admin labels.

When changing location or app identity copy:

- Update `defaultCenter` and `appInfo` in `utils/config.js`.
- Verify the map info card, publish header, profile header, admin header, and share titles.

When changing post storage:

- Keep `listPosts`, `getPost`, `createPost`, `reactToPost`, and `hidePost` as the page-facing API unless there is a deliberate refactor.
- Make sure hidden and expired post behavior stays consistent.

When changing map behavior:

- Update marker generation in `utils/geo.js`.
- Verify `pages/map/map.js` marker taps still navigate with the post id.

When changing app identity:

- Public project metadata should stay generic.
- Real WeChat AppID belongs in `project.private.config.json`, not `project.config.json`.

---
> Source: [anarkh/x](https://github.com/anarkh/x) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
