## twitch-drops-miner

> This is the canonical repository harness. Read [CONTRIBUTING.md](CONTRIBUTING.md) before

# Agent instructions

This is the canonical repository harness. Read [CONTRIBUTING.md](CONTRIBUTING.md) before
planning, editing, testing or reviewing. Its PR checklist is mandatory. The user removed
the alternate agent instruction files; do not recreate copies or links.

## Workflow

- Use descriptive `feat/` or `fix/` branches, conventional commits and PRs against
  `ohne-b/twitch-drops-miner:main`. Keep branches, commits and documentation free of assistant branding.
- Preserve existing user changes, data, credentials, logs and backups. Ask before significant
  refactoring unless the current task already authorizes it. A full rewrite authorization
  covers its necessary cleanup. Never change a running deployment without authorization.
- Use one implementation agent when requested. Required independent adversarial review uses
  a separate read-only reviewer; the author cannot approve their own work.
- Integrate current main before final validation/review and again before merge if it advances.
  Keep incomplete PRs in draft. Record tested/reviewed revisions and actual results; never
  describe unrun checks, live progress or review as successful.
- Add backend unit/regression tests; cover frontend changes where practical. Always update
  README and this harness when behavior/architecture changes. Update English messages for UI
  or console changes. There are no other locales or language settings.
- Keep user guidance in README and contributor policy in CONTRIBUTING. Do not create
  `docs/`, `plans/` or repository planning documents.
- GitHub release publication uses the repository owner's scoped `prod` `RELEASE_TOKEN`;
  verify its identity before publishing images. Registry publication retains the workflow's
  built-in token. Keep tokens out of source, logs and chat; never reuse a local CLI credential
  as an Actions secret.

## Architecture

The product is Twitch Drops Miner (dashboard: Drops Miner), repository and
Cargo package/binary are twitch-drops-miner. Preserve the existing Compose service/container name,
data/log directories, TDM log prefix, auth cookie and CSRF header for upgrade compatibility.

One Rust Cargo package owns the backend. Use concrete structs with methods and composition
for domain/services (the repository's OOP requirement), typed enums and DRY shared policies.
Do not add forwarding hierarchies or speculative traits with one implementation.

| Module | Responsibility |
| --- | --- |
| `main.rs` | CLI/env, credential-safe logging, owned shutdown, healthcheck |
| `config.rs`, `dto.rs` | Typed settings, compatible migration, API snapshots |
| `domain.rs`, `policy.rs` | Campaign/drop/channel eligibility and dependency-aware ignores |
| `store.rs` | Exclusive data directory, atomic settings/history/archive/claim journal |
| `auth.rs`, `origin.rs` | Dashboard passwords/sessions, origin and cookie policy |
| `twitch/` | OAuth, bounded HTTP/GQL, inventory/catalog, beacon watch, PubSub shards |
| `app/` | Application commands/settings, typed Activity, revisioned snapshot publication and patches |
| `miner/` | One session supervisor and owned jobs; session, watch, inventory and claim responsibilities |
| `store/records.rs` | Frozen version-one archive/journal records, independent of runtime presentation fields |
| `web/` | Axum HTTP, Socketioxide, protected snapshots and embedded frontend |
| `fixture.rs`, `bin/dashboard-fixture.rs` | Feature-gated offline browser fixture |
| `frontend/`, `lang/English.json` | React/TypeScript/Tailwind and one message catalog |

Build frontend assets before backend/static tests. `web/` is ignored output. Release binaries
embed it and run without a build tool/runtime companion. Production builds never enable
`dashboard-fixture`; fixture routes must return 404 in production.

## Mining contracts

- Discovery does not select games. `games_to_watch` is the ordered Unicode-casefolded
  automatic allowlist. Opt-in auto_mine_badges/auto_mine_emotes default false and add matching
  watch rewards from other games, with dependency closure and existing benefit/ignore rules.
  Selected games rank first; other campaigns rank by soonest end. Use the shared mining
  policy for discovery, selection, estimates, progress eligibility and Up next. Display
  filters remain separate. Empty with both options off means no automatic watch events. Mine selects a game across
  eligible campaigns. An explicitly chosen manual channel overrides automatic selection
  without changing saved games, filters or priorities.
- `mining_priority_mode` defaults to `manual` and preserves the saved game order.
  `short_events` promotes active selected-game reward windows of at most 24 hours total;
  `ending_soonest` ranks active selected-game reward deadlines regardless of duration.
  Use campaign/drop date intersections, earliest deadline then saved game order, and
  preserve equal-priority watching. Required earnable prerequisites inherit target deadlines
  (bounded by their own expiry); impossible, ignored or filtered target branches cannot
  boost unrelated rewards. Confirmed completion ends a boost without inventing a claim;
  estimates do not. These modes do not predict completion feasibility, reorder saved games,
  preempt manual channels or promote optional other-game rewards above selected games.
  Share ranking across discovery before channel truncation, selection, fallback display and
  Up next; retain its grouping and upcoming visibility without preempting before opening.
- Preserve campaign/drop timing, prerequisites, claim state, benefit filters and ignore rules.
  Literal ignore substrings cascade through dependents while retaining shared prerequisites.
  Zero-minute subscription rewards are omitted from Campaigns/Up next; expired drops leave
  the queue without hiding upcoming or sequential rewards. Ignore/skip is never completion.
- Active campaigns with existing progress appear first. Campaign completion requires all watch rewards
  claimed; expiry alone never qualifies. Persistent completion archives are display-only,
  survive cache clears, and can be invalidated by newer contradictory account evidence or
  changed rewards. Older claim-only history remains completion-unverified.
- Fetch Twitch account Inventory and `https://twitch-drops-api.sunkwi.com/v2/drops` concurrently.
  SunkwiBOT is the catalog source; remove Twitch catalog/detail operations and live-channel
  campaign scans. Inventory wins as whole records, including explicit unclaimed evidence;
  malformed account records must not fall back to public account assumptions. Public HTTP
  uses an isolated client with no Twitch credentials, cookies or identifiers, no redirects,
  the configured proxy/timeouts, and bounded retries within 30 seconds. Cap bodies at 16 MiB
  and campaigns at 2000. Reject timestamps older than 30 minutes or over 5 minutes ahead.
  Strip public campaign/drop `self` records; preserve real ACLs, dependencies and timing.
  The public adapter normalizes an omitted `allow.channels` to an empty list only when
  `allow.isEnabled` is explicitly false. Never infer that from absent/invalid flags or
  replace supplied channel values; account inventory still requires its channel field.
  Reject mixed-null enabled ACLs. Shared domain parsing rejects campaign/drop dates without
  room for the scheduler's one-hour lead and the claim journal's 24-hour grace period.
  Missing restrictions/dependencies, malformed/null entries and duplicate IDs are partial,
  not empty success. Shared account parsing preserves explicit null/empty ACL and dependency
  lists and nullable/default ACL flags, while rejecting missing collections and wrong types.
  Exclude every occurrence of duplicate account campaign IDs and reserve rejected IDs against
  public replacement and award-only pending-claim recovery; confirmed durable receipts still recover.
  Keep known active/upcoming records on partial refresh; valid empty
  feeds are authoritative. A feed 401/403 never logs out Twitch; Twitch auth/cancellation
  failures propagate. Coverage can vary, and restarts need the feed for non-inventory
  campaigns. Never claim relogin/cache clearing repairs feed coverage. Unknown
  PubSub/CurrentDrop progress shares a coalesced, at-most-once-per-minute inventory refresh
  with confirmed watch completion and reported successors awaiting prerequisite claim evidence.
  Retry unresolved evidence through the claim grace period, including after failed refreshes,
  retaining unresolved expired campaigns when a partial refresh omits them,
  without blocking watch cadence. Only account claim evidence unlocks prerequisites or History.
- Unknown linkage is null and unknown progress has no confirmed timestamp. Infer claims from
  awards only when every benefit has evidence in the drop's time window and no explicit
  account record contradicts it.
- Special Events (`509663`) and IRL (`509672`) cross categories only with a nonempty enabled
  actual ACL. Automatic drops need matching category and drops-enabled status; all need live
  channels, selected games or opted-in reward types, and eligible rewards. Offline/ineligible streams yield even at tied
  fallback priority. Preserve nullable viewer counts and the watching row during rebuilds.
- Channels publishes currently eligible selected-game or automatic-reward streams plus the explicitly selected
  manual channel. Rank automatic candidates before the channel limit using matching campaign
  priority, including actual-ACL special-category streams. Mine channel accepts a validated
  Twitch login/root URL and resolves it with owned bounded work independently of inventory.
  Manual watching requires a live identity, not catalog coverage, selected games, a drops tag
  or local reward eligibility. Do not persist game selections or fabricate reward progress.
  Direct lookup and manual stream refresh use a short best-effort metadata attempt;
  missing/failed metadata never blocks watching, but auth/cancellation still propagates.
  Show a known manual reward only after Twitch reports it; do not estimate manual rewards.
  Preserve pending lookups and the confirmed channel separately across network generations;
  late/superseded results cannot restore manual mode. Manual stream metadata never overrides
  real ACLs or account records. Pending stream refresh pauses watching; retry failed refreshes.
  An optional 1..1440-minute timer uses a monotonic deadline starting at selection, survives
  network renewal/browser reconnect, and returns to automatic selection at expiry. Offline
  manual channels wait without switching targets or stopping the timer. Exit, logout, cache
  clear and process restart end manual mode. Viewer counts use channel-only events.
- Watch events use validated Twitch beacon URLs and a base64 minute-watched payload every
  59 seconds. HTTP clients expire idle pooled connections after 15 seconds. Beacons use the
  shared five-attempt transport/429/5xx retry policy with cancellable backoff (1/2/4/8 seconds,
  or Retry-After bounded to 1..60 seconds). Retry the identical payload, never an acknowledged
  204 or ordinary 4xx, and never count retries as progress. Exhaustion returns to owned watch
  recovery. No playlists/video/audio downloads. Confirm via PubSub or CurrentDrop, distinguish
  estimates, and recover at 15 unconfirmed estimates. Only currently eligible drop progress
  suppresses fallback. Failed/unacknowledged beacons invalidate the owned channel's cached
  address. Three consecutive current-stream watch failures renew the network generation;
  successful acknowledgements reset this count. Late request results cannot overwrite newer
  account/stream events or restore invalidated beacon addresses.
  Successful inventory refreshes clear stale estimate ceilings on refreshed and retained
  drops, preserving confirmed progress and evidence newer than the request. Failed
  refreshes must not clear the ceiling or fabricate confirmation.
  Fresh account inventory may correct conflicting live progress for the same campaign/drop
  and unchanged watch requirement. Keep the disputed live counter separate; use inventory
  minutes/timestamps for display and eligibility, without estimates advancing them. Reconcile
  through the existing once-per-minute refresh through the claim grace period. Older requests,
  public metadata and partial-refresh retained records cannot resolve a dispute. Preserve
  disputes across same-account network renewal; cache clear, logout and account changes cannot
  restore them. Retain fresh account claims and issued instance IDs independently, including
  completion backed by an issued instance. Claims clear disputes; runtime evidence never
  changes stored history/archive formats. Disputed live completion cannot stop watching.
  Confirmed watch completion is no longer watch-eligible, but remains unclaimed until account
  evidence arrives. The automatic Mining card prefers current eligible Twitch reports;
  reported successor progress may display before prerequisite claim reconciliation, without
  granting watch eligibility. Validate CurrentDrop's returned channel and reject polls older
  than the current watch, stream/account refresh or accepted progress. Unrelated/regressive
  progress must not restore an old card; duplicate PubSub progress cannot displace a reported
  successor or postpone polling. Stream replacements fence old polls even on the same channel.
  A null CurrentDrop session or the empty object (explicit null channel/game, empty drop ID,
  zero current/required minutes) is no result; retain validation for malformed nonempty sessions.
  Estimates alone never imply completion or a claim.
- Claims require account-issued instance IDs, skip upcoming campaigns and stop at the strict
  campaign-end + 24-hour deadline. Earned claims are independent of mining/ignore selection.
  Persist the account-scoped intent before RPC, then its success receipt and history. Keep the
  receipt until the owner acknowledges domain state and archives completion; reconcile after
  restart even without catalog metadata. Never retire it before durable history/archive writes.
- History imports Twitch-confirmed claims during inventory refresh, independent of mining
  selection and reward type. Require account claim state or complete in-window award evidence;
  stale local progress must not override newer award evidence. Deduplicate by drop ID and use
  known award timestamps, labeling unknown claim times as first observed.
  Persist cleared IDs in the compatible version-1 history file so imports and late claim
  receipt recovery cannot resurrect cleared entries. Imports never issue claim RPCs or
  fabricate missing campaign metadata.
- Every session/job/socket task is owned and drained. Logout coalesces and removes only Twitch
  credentials after drainage; concurrent shutdown cannot interrupt removal in either queue order.
  Hourly validation/network reconfiguration preserves manual selection/deadlines and queues
  new channel choices until fresh channel eligibility is available. Cache clear
  preserves settings, credentials and completed campaigns, and clears local claim history with
  durable cleared-ID tombstones before refreshing; failure to persist must fail the clear.
- Requests use bounded concurrency/rate, retries and cancellation. Connection Quality defaults
  to 3 for new or missing settings; preserve explicit saved values. Quality 1..6 controls connect
  timeout 5×quality and total 10×quality seconds; the saved refresh interval actually schedules
  inventory work. Slow discovery must not block watch cadence. Duplicate idle prompts collapse.
  Nonfatal notification dismissal failures remain visible but never extend or clear the shared
  scheduling retry deadline; authentication and cancellation still propagate.
  Interrupted response bodies retry within the existing five-attempt budget only for replayable
  GETs and known persisted read-only GraphQL operations/batches. Never replay successful OAuth
  exchanges, mutation responses or acknowledged 204 beacons. Preserve known failure statuses
  when error bodies fail, including authenticated 401/403 versus ordinary public HTTP failures.
- Upstream diagnostics use server tracing only, never dashboard console/socket payloads.
  Record operation/status/attempt, JSON syntax positions, rejected field types, typed network
  causes and catalog rejection summaries. Basic logs allowlist known GraphQL messages and
  fingerprint unknown messages. Advanced `TDM_DIAGNOSTICS=true`/`--diagnostics` is explicitly
  opt-in and stays off with ordinary verbosity. The isolated tracing target may record bounded,
  redacted JSON response previews and nested transport causes; never raw bodies, requests,
  arbitrary headers, URLs or authenticated WebSocket frames. Collect request/proxy/cookie and
  response credential values before redaction; redact sensitive fields and credential-bearing
  text, fail closed on credential-inventory limits, and withhold non-JSON bodies and oversized
  input strings. Bound redaction allocations and never reprocess inserted markers. Preserve
  correlation, timing/status and hashes; mark capture omissions. Keep public errors unchanged,
  and bound retry-body capture to one second without replacing retry/cancellation decisions.

## Authentication and storage

- Original in-app device code only, Smart TV client identity, `twitch_oauth2` types/requests.
  Keep explicit empty scopes, pending/slow-down/expiry/denial handling and token validation.
  Channel pages use the web client URL. No browser-import or browser-renewal code/services.
  HTTP 401/403 from unauthenticated pages, settings scripts or device discovery are ordinary
  HTTP failures, never grounds to discard the saved session. Authenticated API/OAuth and
  PubSub authentication failures still propagate to the session owner.
  An interrupted unauthorized token-validation response still attempts the saved refresh token
  before rejecting the session; refresh failures and account mismatches retain existing handling.
- New sessions use `twitch_session.json`; keep old credentials/backups untouched for rollback.
  Invalid new sessions are preserved separately before reauthorization. Never log OAuth tokens,
  device secrets, proxy credentials, cookie values or raw authenticated transport frames.
- Existing settings, version-1 history, completion and dashboard-auth formats remain compatible.
  Atomic replacement commits before memory/publication. Corrupt auth fails closed; corrupt
  history/archive are preserved read-only; unreadable settings never silently reset. One miner
  process per data directory. Keep retired credential filenames ignored to protect old installs.
- Optional password-only dashboard auth uses compatible scrypt and SHA-256 token digests,
  fixed 30-day sessions, HttpOnly/SameSite=Strict cookies and Secure on HTTPS. Password change
  revokes other sessions; disable requires current password. Twitch logout is separate.
- Outer middleware protects HTTP and both Socket.IO transports. Writes require `X-TDM-Request: 1`,
  same-origin/Fetch Metadata checks and bounded bodies. Rate-limit hashing by peer and globally.
  Recheck socket authorization for events/broadcasts and expire idle sessions. Enabling auth
  evicts anonymous sockets before private publication. Detached password/settings writes are owned.
- `PUBLIC_BASE_URL` is one normalized HTTP(S) root origin. Reject credentials, paths, queries,
  fragments, lists/wildcards and ambiguous numeric IPv4 forms. It controls browser origin/cookie
  scheme, not forwarded client-IP trust. Do not trust proxy headers implicitly.
- Serve SPA only on explicit dashboard routes. Preserve API/socket 404s. Public login code/fonts
  do not make account data public. HTML revalidates; hashed assets are immutable; private data
  is no-store. Dashboard protection recovery removes only web_auth.json while stopped/restricted.

## Dashboard design and contracts

- All displayed times, including tooltips, use the 24-hour `h23` clock (midnight is `00:00`),
  preserving the browser's timezone and locale date format. Use the shared date/time formatter;
  Activity's time-only display retains seconds. Keep API/storage timestamps unchanged.
- Use `frontend/src/assets/twitch-drops-miner-logo.svg` for the app, login, favicon and README.
  Preserve its artwork and aspect ratio; Vite emits one hashed asset for browser caching.
  Keep adjacent brand text accessible and sidebar navigation reachable in short windows.
- Keep the subtle charcoal/Manrope design, individual `@mdi/js` paths rendered with `@mdi/react`, and shared native controls.
  Icon-only actions use borderless transparent buttons with circular hover/keyboard backgrounds,
  accessible names and tooltips.
  History pagination uses chevrons beside the page count; channel watch/entry, return-to-auto,
  Activity follow and error retries use play-circle/plus, refresh-auto, arrow-down and reload icons.
  Inset search-clear circles within the field border; game-priority
  drag grips stay visible at all times and retain their plain six-dot pattern without pointer
  hover backgrounds. Retain native
  selects for icon-only sorting, with explicit legible dark select/option colors on light OS themes.
  Mining > Edit uses the native icon-only Priority High selector beside Add Game for
  Default (manual order), Short events first and Ending soonest. Its tooltip reports the
  selected mode; accessible help explains that drag order is preserved for ties. Reuse
  autosave/conflict handling and disable the selector until reconnect hydration.
  Mining preferences has no inline priority or ignore-name hints. Native info popovers retain
  the selected priority mode's full rules, other-game reward rules and ignore/dependency details, with
  click/tap/keyboard access and outside-click/Escape dismissal. Keep priority rules associated
  with the selector for assistive technology, and help available during reconnects. Use concise
  labels: Also mine from other games, Allowed reward types and Ignore rewards by name. Keep
  the two reward groups distinct and preserve all existing settings/mining behavior.
  Render strings as React text, validate external links/artwork, expand Twitch image placeholders.
  No injected HTML or CDN scripts. Art provides safe missing/broken-image fallbacks.
  Reward thumbnails contain their artwork; game covers retain cover sizing. Phone checkbox
  labels, game search results and drag grips have at least 44px touch targets without enlarging
  glyphs or adding pointer hover backgrounds to grips.
- Sidebar: 32px visible GitHub artwork above an equally sized, aligned Twitch avatar
  (compensate for the SVG's internal whitespace), with 4px between their controls, followed by
  equipped badges and the chat-colored name, with no divider above the footer. The identity
  links to Settings > Twitch account. Settings shows a 48px avatar, colored name, equipped
  badges and adjacent logout action, followed by the collection labeled Badges. Badge
  buttons have no borders; click/tap/keyboard reveals descriptions. Use the same account
  settings page on phones, without a separate profile popup. Omit banner, bio, account date,
  followers, roles, social links and the separate account ID row.
  Lighten dark chat colors only enough for readable contrast, preserving their hue.
  Profile metadata uses bounded, cancellable GraphQL reads owned by the authenticated network
  generation, concurrently with mining, using the existing login with no new scopes. Refresh
  once per generation (normally hourly), preserve same-account metadata during renewal, and
  clear it on logout/account changes. Never persist it or accept late data for another account.
  Query the full selectable global badge collection separately; permission/schema/network
  failures leave it explicitly unavailable while retaining equipped badges. Do not include
  channel subscription badges or infer badge ownership from the global artwork catalog.
  Authenticated 401/403 and cancellation retain existing session handling. Validate identities,
  bounded metadata, image URLs and colors; fetch only identity, color and badge fields.
  Connection status lives in Settings > Connection and is labeled Dashboard connected, separate
  from Twitch. The Twitch logout icon stays immediately beside the account identity.
  Logged-out authorization status stays visible. Device authorization keeps the
  copyable code, Twitch Activate and Done on one unboxed wrapping row with equal-height controls
  (36px desktop, 44px phone). Activate and Done use the same outlined button style. Successful
  copying shows a tick, tooltip and accessible status for three seconds; another successful copy
  restarts the timer. Restore the copy icon on expiry or failure, and clear timers and stale
  clipboard results when the code changes or the component unmounts. Success feedback never shifts the
  row or following sections; visible failure feedback retains its spacing below the row.
- Navigation is Mining (`/`), Campaigns, Activity and Settings. Keep the compact charcoal
  sidebar and the four-item phone bottom navigation, safe-area spacing and short-window access.
  Mining preferences replace the overview from Up next > Edit (`/?edit=priorities`), with a
  Back to Mining icon action. Only the game list scrolls on desktop; search/priority controls
  and the settings beside it stay fixed. In short windows the settings column scrolls independently
  to keep every field reachable, without scrolling the whole preferences panel or page.
  When error/reconnect notices leave too little height, allow the games column to scroll as
  well; search results and selected games must never overlap or become unreachable.
  Game search results appear below the search controls and above the selected games;
  bound their height so long result lists remain reachable in short windows.
  `/settings#mining` redirects there. Settings has account, dashboard access, connection and
  maintenance sections without trailing separator lines.
  Settings sections start at the same offset below their tabs, without repeated section headings.
  Keep the page title and tab labels, plus the account status/logout row. Hide inactive form
  wrappers and retain drafts when switching tabs. Tab hashes support back/forward without
  anchor-scrolling the title or navigation out of view.
- Mining: watching information only in Now mining, no status subtitle or Recent activity. Channels
  and Up next have equal desktop dimensions and internal scrolling; stack on narrow screens and
  preserve access on short windows. Show confirmed values without redundant labels
  or the Watching for this reward caption. Put the current reward's Last confirmed timestamp
  inside Now mining beside the selection mode, in smaller, darker text with readable contrast.
  Omit it when confirmation is unknown; let the card header wrap on phones with an 8px row gap.
  Now mining uses 24px vertical/20px horizontal desktop padding and 16px phone padding.
  Header-to-reward and reward-to-progress gaps are 20px desktop/16px phone, with an 80px
  contained reward thumbnail centered beside its text with a 16px desktop/12px phone gap.
  Keep the 6px progress track and 8px count gap. Group the manual return action and timer
  in one wrapping row, preserving phone hit targets. Its empty state adds 20px vertical padding.
  Known rewards without confirmed progress show an empty progress bar and `0 / required minutes`
  in Mining and Campaigns,
  with an unconfirmed tooltip and accessible description. Suppress the redundant awaiting-progress
  subtitle when a reward is shown. Keep unknown evidence unchanged; never display estimates as
  confirmed minutes or invent a confirmation timestamp or a duration for an unknown reward.
  Channels, Up next and game-priority rows have inset separators and modest 4px scrollbar
  padding, smaller than Campaigns. Up next shows a shared date once only when reward windows
  match by instant and their upcoming state agrees; retain distinct per-drop dates.
  Channel views may fill missing game artwork from catalog records with the exact same game
  ID. Never change stream identity/eligibility or add network requests for this fallback.
  Keep expanded channel-entry controls and feedback inside the scrollable list body.
  Manual lookup has no preparing message; use a play-circle icon submit button with the Mine
  accessible name/tooltip, preserving its busy/disabled state, inline
  errors and return-to-auto control while a lookup is pending.
  Channel name/URL and optional timer use accessible input placeholders. Settings autosave
  has no saving/saved notices; preserve errors, edits and Retry.
- Mining and Campaigns share inventory refresh feedback inside the button; Maintenance has no
  inventory refresh action. Track queued
  and running work through publication, coalesce requests, preserve state on reconnect and
  ignore stale completion events. An acknowledgement is not completion. Keep request errors
  and partial-catalog failures retryable without clearing previous results or adding notices.
  Use the shared fixed-size icon button: refresh/spinner, brief success tick, or retryable error.
  Keep its position and size stable across states, respect reduced motion, and preserve accessible
  state labels/live feedback plus tooltip and accessible error/catalog details.
- History lives in Campaigns in place of Finished; the old /history route redirects. Display
  recorded claims grouped by campaign (25 per page), independent of completion/catalog coverage.
  Share search, game filters, sorting and list/grid controls. History defaults to newest recorded
  claim; loading stays quiet without a Loading history caption, while errors retain Retry.
  History detail panels render recorded claims only: saved campaign/game identity, rewards,
  additional benefit names, optional artwork and claim/first-observed timestamps. Live metadata
  may supply artwork but must never add account linkage, eligibility, channels, progress,
  prerequisites, earning dates or unclaimed rewards. Deep links resolve against recorded claims;
  missing records and request failures stay explicit, with Retry available inside the panel.
  Most Drops counts recorded claims, campaign date sorts place unknown dates last. No
  CSV/JSON export, Since filter or separate history clear action. Clear all cache clears history
  and publishes the durable change to open dashboards; archives never recreate cleared rows.
- History artwork is optional; retain old rows and use matching live benefits as display fallback.
  No Telegram controls/API/credentials in responses and no dashboard updater.
- Campaigns starts with Settings-style Available/History icon tabs and the total campaign count
  immediately left of refresh on the right. Omit the filtered count; match Last confirmed's
  12px/#888888 text. Keep the page heading screen-reader-only. Clear filters stays beside All games
  inside Filters. On phones put pagination to the right of the tabs, then a full-width search
  row, followed by filter/sort/view controls and the count/refresh pair, wrapping when needed.
  Below 375px omit the decorative tab icons to make room; retain both text labels.
  Native icon-styled sorting offers Default, Newest (campaign start descending),
  Ending Soonest (end ascending), Most Drops (total descending), and A-Z (campaign name).
  Default retains progress-first ordering; ties use that same deterministic order. Sort is
  URL state preserved by searches, filter resets and tab changes, never a mining setting.
- Campaign summaries open one detail panel, alongside the list on wide screens and as a full
  page on smaller screens. Lock page scrolling while the full-page detail is open and contain
  its body scrolling; release the lock on close, navigation or return to the desktop layout.
  At desktop widths tabs, search and filters stay full width above the list and detail columns.
  Both columns fill the remaining height with a compact 12px bottom margin; details keep the
  same height for short and long campaigns, with a fixed header and separately scrollable body.
  Scroll campaign results independently with a small scrollbar gutter; allow the controls area
  to scroll in short windows and the workspace to scroll when notices or long titles leave too
  little room for usable controls and detail content. Omit selection stripes and
  confine row hover to the side-panel icon circle; preserve a visible title/icon keyboard focus cue.
  Use the Dock Right icon for opening details, with a tooltip, current-item state and panel
  control association rather than disclosure semantics.
  Mine icon hover stays separate, without a selected background strip behind it. Apply the
  same shared summary component and collection layout for Available and History: 48px artwork,
  matching text spacing/count typography and card borders, with inset separators between list rows.
  Stretch grid cards to equal row heights without clipping names. Use each summary's width
  to place counts and actions below the identity in narrow cards/rows, keeping artwork/text
  aligned at the top and footer actions aligned at the bottom. Preserve phone hit targets.
  Keep History's recorded claims/date and Available's live status/mining action distinct.
  Preserve search, filters, layout, sorting, page, scroll and trigger
  focus when closing. Keep the selected campaign visible when opening narrows the results or
  changes their layout. Restore offsets only for the matching results width and layout;
  otherwise reveal the selected item instead of depending on browser scroll anchoring. Switching
  details replaces the current detail history entry. Campaign/drop
  IDs form deep links; missing IDs and unknown account progress/linkage remain explicit.
  Links from Mining and Activity return to their originating route, scroll and trigger focus
  on Close/Back. Preserve their search/filter URL state and the return state through in-place
  Campaigns query edits; restore matching-width offsets and reveal the trigger after resizing.
  Activity uses the same Dock Right detail icon. Reward deep links retain
  scrolling and current-item semantics without a selection stripe or indentation.
  Show the campaign date range once; only show per-drop dates when the effective window differs
  from the campaign, comparing instants rather than timestamp strings. Keep claim timestamps.
  Available details have no appended History section: show claim/first-observed times inline
  for account-confirmed claimed rewards only. Never let old history mark a live unclaimed
  reward as claimed. Omit benefit names identical to the reward title, retaining other names.
  List filters/layout are shareable URL state; loading a shared URL does not autosave it.
  Available and History paginate 25 campaign groups, with pagination centered across the
  top toolbar on desktop and beside the tabs on phones, wrapping above search when needed.
  Results fill the remaining height without a pagination
  footer. Preserve full-size pagination hit targets. Mine remains a game-wide action.
  Eligibility, normalized game identity, saved ranks and priority reasons come from shared
  backend policy; prerequisite boosts identify their actual target rewards.
- Activity is a bounded session buffer of typed events with category, severity, safe message
  arguments, entity references, first/last time, repeats and explicit recovery. A matching
  successful operation recovers its failures; unrelated messages never imply recovery.
  Keep category filtering, but omit the This session heading suffix and per-event category tags.
  Inset row separators to align with the content instead of touching the panel edges.
  Fit the list to the remaining viewport on desktop and phone; keep notices and controls
  reachable in short windows. Distinguish an empty event buffer from no filter matches.
  Count adjacent repeats in Activity while suppressing duplicate server log lines.
- Maintenance checks the latest stable release's `latest.json`, compares SemVer precedence
  without build metadata, and distinguishes failure from up-to-date status. Keep requests
  bounded/coalesced and release links within this repository. No install/download execution.
  Check for updates uses the fixed-size MDI Update icon button, with a checking label/tooltip,
  busy state and spinning icon during requests (respect reduced motion); retain result text below.
- Shared Field content starts at the top; helper text cannot stretch neighboring label rows.
  No focus rings, but visible keyboard background/border changes must outrank utility layers;
  keep system focus in forced colors. Verify computed field/button/checkbox focus and axe checks.
- The application owns mining state independently of HTTP/socket transport. Publish related
  fields atomically; clone snapshots before emitting and never await network/disk under a view lock.
  Protocol 2 snapshots/patches carry process identity and monotonic revisions; per-subscriber
  patches use the last delivered revision and coalesce slow consumers. Retain the legacy event
  adapter for old tabs and recheck authorization for every emission. A gap requests a full resync;
  disconnects/process changes disable commands until hydration. Viewer-only patches retain
  frontend campaign identity. Incompatible clients can reload with a temporary unsaved draft.
- Every durable History insertion and clear publishes a history revision under the history
  lock. HTTP responses include process/revision/clear identity; stale responses cannot restore
  rows after a clear. Runtime DTO additions must not change version-one archive/journal JSON.
- One typed provider hydrates complete snapshots and incremental events. Commands stay disabled
  until reconnect hydration. Editable settings drafts are separate from live data; serialize
  autosaves with original revisions. HTTP409 preserves edits for Retry and only changes touched fields,
  including individual nested filter/benefit fields. Draft errors remain reachable across pages.

## Validation and release

Use the complete commands in CONTRIBUTING. Rust tests use temporary files and mock transports;
Playwright starts its own loopback:8765 fixture, verifies readiness/reset, and refuses reuse.
Never use live credentials, a live miner or real notifications for automated checks.

CI requires Rust fmt/Clippy/tests, frontend format/type/build/unit/browser/axe, automation
contracts, version/lock agreement, and amd64/arm64 production image builds plus isolated health.
Only PRs limited to README.md, CONTRIBUTING.md and AGENTS.md may skip code/image jobs;
scope/whitespace checks and the final Validation gate still run. All main pushes and manual
runs require the full baseline. Cache Rust dependencies, never workspace binaries or test
results; a cache hit never replaces running the checks. CI installs only Chromium's headless shell.
Build one fixture binary/dashboard bundle for both CI browser shards; each shard owns a
separate runner/server and one worker. Preserve readiness/reset checks and refused server
reuse. The Validation gate requires both shards; local Playwright still starts Cargo itself.
Use focused local checks and the final revision's CI baseline without repeating the whole
suite locally. Image builds run alongside tests. Main retains the exact tested OCI archives
for seven days; publishing downloads them from the latest successful exact-main validation,
checks both platforms/version/revision before registry writes, and copies them by digest.
Failed or pending newer validation cannot fall back to an older run. Missing/expired artifacts
require rerunning validation; release and edge publishing never rebuild images. PR artifacts
cannot be promoted. Ordinary merges only retain artifacts, never publish registry images.
Keep the project's PolyForm Noncommercial 1.0.0 license in `LICENSE.md` and the full upstream MIT
license in `NOTICE.md`; third-party components retain their original licenses;
include both with frontend asset notices in production images. Preserve 1000:1000 ownership,
mounts and port. Health does not prove earning.

Cargo manifest/lock own the version. Prepare release opens a draft PR; a maintainer can use
the same local release script and draft-PR process if its token is unavailable. Publish is manual from
validated main and uses reviewed CHANGELOG notes/GHCR. Every release attaches and verifies
`latest.json` before publishing; stable releases alone move the latest pointer. Use scoped
conventional commit messages and concise release change lists. The release script owns the
single `Changelog:` link and issue footer; do not duplicate them in CHANGELOG entries.
Accept exact-commit push or manual validation.
Publish first to GHCR at ghcr.io/ohne-b/twitch-drops-miner using the scoped workflow token.
Then mirror the published image to Docker Hub through the shared main-only `prod` workflow.
Use the configured repository/username variables and token secret; never expose credentials
to PR jobs. Copy all platforms by immutable source digest, preserve and verify that digest,
and never rebuild or recreate releases for mirroring. Only the current stable release matching
GHCR's latest digest may advance Docker Hub's latest; older releases and prereleases cannot.
Failed mirrors leave GHCR intact and can be retried through the manual mirror action.
Advance latest only after the stable release and its manifest are public. Preserve
published old-name images and document promotion recovery and first-package visibility.
Keep Buildx action pins identical between validation and publishing. README uses
a centered title/tagline/license opener and GitHub Flavored Markdown alerts. Keep upstream
attribution in License and credits; do not add a contributor/PR table or automation that
rewrites README after merges. No ordinary code merge may publish a release or bypass
independent review/checks.

Manual Publish edge image may publish only `:edge` from exact validated current main, using
the scoped workflow token and the same image platforms/action pins, then mirror it to Docker Hub.
It creates no GitHub
release, version tag or version bump and never changes `:latest`; the OCI revision identifies
the build. Ordinary merges do not publish edge images either.

For home-server work, inspect the current checkout/Compose/image before assumptions. Ask before
changing the running deployment. Build while it runs, back up data and Compose before replacement,
retain rollback, and require interactive sudo in the user's terminal. Always provide the actual
PowerShell update command in the handoff; never ask for a sudo password in chat.

---
> Source: [ohne-b/twitch-drops-miner](https://github.com/ohne-b/twitch-drops-miner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
