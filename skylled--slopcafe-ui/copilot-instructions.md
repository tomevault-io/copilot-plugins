## slopcafe-ui

> handles it here. `ReadTextResponse` (`GET /d/{id}/text`) is likewise generated

# ♊ GEMINI.md — Repository Grounding & Codebase Map

Welcome! This file serves as a grounding and reference document for future AI developer sessions. It outlines the purpose of the Slopcafe Operator App (UI), maps key files and features to their respective implementations, and establishes the "self-healing" documentation protocol.

---

## 📌 Repository Purpose
The **Slopcafe Operator App** is a professional, feature-rich Flutter administrative application designed for operators to manage, search, monitor, and configure the Slopcafe fleet deployment.

It communicates with the Slopcafe Backend API to perform fleet management tasks, agent registrations, document revocation, and metadata indexing.

---

## 🛠️ Technology Stack
- **Framework**: Flutter SDK (>=3.0.0)
- **State Management**: `flutter_riverpod` (Riverpod 3.x). The mutable stores
  (`connectionStateProvider`, `documentsListProvider`, `agentsListProvider`) use
  the modern `Notifier`/`NotifierProvider` API — each overrides `build()` for its
  initial state and reads `ref` directly (Riverpod 3 dropped the legacy
  `StateNotifier`/`StateNotifierProvider` from the default barrel).
- **HTTP Client**: `dio` (with custom interceptors for auth and status monitoring)
- **Local Persistence**: 
  - `flutter_secure_storage` (v10; secure storage of base URLs and API operator
    tokens). v10 dropped the Jetpack-Security/`EncryptedSharedPreferences` Android
    backend (deprecated by Google) for default custom ciphers; values written by
    earlier builds auto-migrate on first access, so no `AndroidOptions` are set.
  - Custom SQLite or local file-based database for offline document caching
- **Localization**: `flutter_localizations` + `intl` via Flutter's `gen-l10n` ARB
  pipeline. Every user-facing string is centralized in `lib/l10n/app_en.arb`
  (see the **Localization (i18n)** section below).
- **API Models & Error Codes**: **generated** from the backend's canonical
  **OpenAPI 3.1** contract — `freezed` + `json_serializable` data classes plus an
  `ErrorCode` enum — by a bespoke pure-Dart emitter (`tool/generate_api.dart`)
  feeding the existing `build_runner` pipeline. The hand-written `lib/models/`
  classes are gone; see **API layer (generated from the OpenAPI contract)** below.
- **URL Launching**: `url_launcher` for external browser navigation on mobile platforms (Android/iOS).
- **Inbound web links**: `app_links` — Android App Links, so a tap on a
  `https://slopcafe.com/d/:id` or `/s/:slug` URL anywhere on the device opens
  the Reader instead of a browser. **Mobile only by design** (see **Inbound web
  links** below and [docs/deep-links.md](file:///Users/kyle/Repos/slopcafe_ui/docs/deep-links.md)).
- **Platform Targets**: macOS, iOS, Android, and Web

---

## 🎨 Design Language — "Cortado"
The mobile UI follows the **Cortado** design language (a warm café metaphor): a terracotta **clay** +
**honey** OKLCH-derived palette, serif display type, text-forward "plate" cards (no cover art), tinted
tag chips, press-cards, and a floating pill tab bar. The information architecture is three tabs —
**Library** ("The Café"), **Search**, and **Operate** ("The Pass") — plus pushed routes: a full-screen
document **Reader**, a **Collections** tag browser, a generic **document list** (a tag's collection /
"see all"), and **Settings**. Copy uses "light café flavor": café-flavored section/screen titles,
professional functional copy. Both **Cortado-light** and a derived **Cortado-dark** are supported via
`ThemeMode.system`.

### Large-screen adaptation (tablet / desktop)
Cortado is phone-first but **adaptive**, via one small system in
[lib/core/design/layout.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/core/design/layout.dart):
* **One breakpoint** — at window width ≥ `AppLayout.railBreakpoint` (600, M3's compact/medium
  boundary) the shell swaps the floating pill tab bar for a left **side rail** (same three tabs +
  desktop affordances: an explicit **refresh** action — pull-to-refresh has no mouse gesture — and a
  Settings shortcut). Below it, the phone layout is byte-for-byte unchanged.
* **Readable columns** — every scrolling screen centers its content by growing its horizontal
  gutter (`AppLayout.gutterFor` / the `AdaptiveGutter` builder): `contentMax` 760 for
  lists/stats/compose, `formMax` 640 for Settings, `readerMax` 860 for the Reader surface. On
  phones the gutter collapses to the original `AppSpacing.screenH`, so compact layouts are
  untouched. The three shell tabs also swap `AppSpacing.bottomInset` for a small
  `context.shellBottomInset` when the floating bar is gone.
* **Wide-only layout shifts** — Operate's stat grid goes 2×2 → 4-across when the content column
  fits it; modal sheets cap at `sheetMax` 560 (centered); toasts keep a compact fixed width.
* **Pointer affordances** — `PressCard` shows a click cursor + a whisper of hover lift; plain
  tap targets use the `Tappable` wrapper (a cursor-aware `GestureDetector` drop-in, in
  `press_card.dart`).
* **macOS window** — `MainFlutterWindow.swift` sets a 400×600 `minSize` so the layout can't
  collapse below its compact floor.

---

## 🌐 Localization (i18n)
The app is wired for multiple languages using Flutter's **idiomatic `gen-l10n` ARB pipeline**, even
though only English (`en`) currently ships. The driving goal is centralization: **all user-facing
copy lives in a single editable file**, `lib/l10n/app_en.arb`, so wording can be tweaked without
hunting through widgets.

* **Single source of copy**: [lib/l10n/app_en.arb](file:///Users/kyle/Repos/slopcafe_ui/lib/l10n/app_en.arb)
  — the template ARB. Edit values here to change any in-app text. Plurals (e.g. `documentCount`,
  `searchResultCount`) use ICU syntax; placeholders (`{count}`, `{error}`, `{version}`) are typed in
  the `@key` metadata. A few notable conventions: `searchSuggestionSeeds` is a comma-separated list
  split at runtime into the Search idle chips; `revokeConfirmWord` is the word an operator must type
  to confirm a document revoke; trailing/leading spaces in `cascadingDestruction`,
  `typeToConfirmPrefix`, and `typeToConfirmSuffix` are intentional (they sit next to styled spans).
* **Config**: [l10n.yaml](file:///Users/kyle/Repos/slopcafe_ui/l10n.yaml) (arb-dir `lib/l10n`,
  output class `AppLocalizations`, `nullable-getter: false`). `pubspec.yaml` sets `generate: true`.
* **Generated code** (committed, **do not hand-edit**): `lib/l10n/app_localizations.dart` +
  `app_localizations_en.dart`. Regenerate after editing any ARB with **`flutter gen-l10n`** (also
  runs on `flutter pub get`).
* **Access pattern**: call sites read copy via the `context.l10n` extension defined in
  [lib/l10n/l10n.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/l10n/l10n.dart) — e.g.
  `context.l10n.searchTitle`, `context.l10n.documentCount(n)`. In `async` methods, capture
  `final l10n = context.l10n;` *before* the first `await` to avoid `use_build_context_synchronously`.
* **Wiring**: `MaterialApp` in `main.dart` registers `AppLocalizations.localizationsDelegates` /
  `supportedLocales` and sets the OS title via `onGenerateTitle`.
* **Context-less layers**: `lib/core/format.dart` helpers `relTime(l10n, date)` and `greeting(l10n)`
  take an `AppLocalizations`; byte/date formatting (`fmtBytes`, `fmtDate`) and separators stay
  locale-neutral. The `api_client` 401 handler intentionally carries **no** copy — the UI layer
  supplies the localized `tokenRejectedDetail`.
* **Adding a language**: copy `app_en.arb` to `app_<locale>.arb` (e.g. `app_es.arb`), translate the
  values, run `flutter gen-l10n`. No Dart changes required.

---

## 📂 Codebase Map & Key Functionalities

Below is the directory mapping of the core functionalities within the `lib/` directory:

### 1. Entry & Bootstrapping
* **[lib/main.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/main.dart)**
  * Initializes the `ProviderScope`, builds the `MaterialApp` (Cortado light/dark themes, `ThemeMode.system`), and runs the application.
  * Registers the localization delegates (`AppLocalizations.localizationsDelegates` / `supportedLocales`) and sets the OS task-switcher title via `onGenerateTitle` (localized `appTitle`).
  * Contains `RootGate`: the first-launch gate that routes to `SettingsScreen` when the deployment is unconfigured (no Base URL/Operator Token in secure storage), otherwise to `AppShell`.
* **[lib/screens/app_shell.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/screens/app_shell.dart)**
  * The three-tab shell: an `IndexedStack` of Library / Search / Operate. **Adaptive**: beneath a floating pill tab bar on compact windows, beside a left side rail (`_SideRail` — logo, tabs, refresh via `refreshFleetData`, Settings shortcut) at width ≥ `AppLayout.railBreakpoint`.
  * Houses the global connection-state listener that intercepts `401 Unauthorized` token rejections and pushes the Settings screen.
  * Owns the **inbound web-link** subscription (`_openDeepLink`): resolves the link's raw name via `documentsListProvider.resolveListing` and pushes `ReaderScreen` on top of whatever is on screen (a link tap is an excursion the operator should be able to back out of). Subscribing *here* rather than at the app root is deliberate — resolving needs a Base URL and an operator token, and the shell is precisely the widget that doesn't exist until `RootGate` has both, so a link arriving on an unconfigured install waits for setup instead of failing against a deployment the app was never pointed at. Nothing is lost in that wait: the platform side holds the launching link until its first subscriber attaches. See **Inbound web links** below.

### 2. Core Services
* **[lib/core/api_client.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/core/api_client.dart)**
  * Sets up the global `dioProvider` and defines custom interceptors.
  * Implements `connectionStateProvider` to track whether the app is in `connected`, `disconnected`, or `unauthorized` states. On a 401 it flips to `unauthorized` carrying **no app copy** — it parses the typed `ApiError` envelope and forwards the backend's own `ErrorBody.message` (server-supplied detail) into `errorMessage`, leaving it null otherwise; the UI layer renders the localized `tokenRejectedDetail` fallback (this service has no `BuildContext`).
  * Appends authorization headers dynamically to all requests sent to the configured Base URL.
  * **Sends `Accept: */*` on any request that didn't set its own** — a two-line interceptor with an outsized job. `dart:io` sends *no* `Accept` header unless one is set explicitly, and Cloudflare strips the `ETag` off the response to a request that arrives without one. The ETag is how every version-resolution path in the app learns which version it is holding (see `lib/core/publication.dart`), so its absence never fails loudly — it silently disables the Reader's conditional `If-None-Match` revalidation, mis-keys the offline body cache, and flattens the served-vs-current comparison the publication gate depends on. `*/*` is the neutral value: it constrains nothing about what the server may return, it only keeps the validator intact. **Do not "clean this up"** — a caller that set its own `Accept` (a byte path asking for HTML) is never overridden.
* **[lib/core/secure_storage.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/core/secure_storage.dart)**
  * Wraps `flutter_secure_storage` to encrypt and store the API Base URL and Operator Token.
* **[lib/core/document_cache.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/core/document_cache.dart)**
  * Manages the storage and retrieval of offline-cached documents to support running the app in disconnected mode.
  * Encapsulates saving and loading the global documents metadata listing (`documents_list.json`), checking if specific document versions exist in the cache, and scanning the cache directory for version resolution.
* **[lib/core/publication.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/core/publication.dart)**
  * The shared vocabulary for contract 2.0.0's **publication gate** (the rule itself is spelled out in the **API layer** section below; the library-level doc comment restates it in full). Four functions and one extension, no Flutter imports:
    * `parseVersionTag(String?)` — the tolerant-but-strict version parser. Accepts every shape the byte path can hand back (`"v7"`, `W/"v7"`, `v7`, `7`, `"7"`, whitespace); returns null rather than salvaging digits from anything malformed, because a *wrong* version silently mislabels a cache entry or clobbers a newer write, while a *missing* one degrades to a visible unknown.
    * `resolveCurrentVersion(Headers)` — the document's head: `x-doc-current-version` first, **falling back to the `ETag`**. That fallback is load-bearing, not defensive scaffolding (see the header contract below).
    * `resolveServedVersion(Headers)` — the ETag and nothing else: what the bytes in hand actually are, which is what a cache key or an `If-None-Match` revalidation must be built from.
    * `ifMatchFor(Headers)` — the canonical `If-Match` preflight value for a write, built from the *current* version and emitted in the strong `v<n>` form so the request never depends on Cloudflare's compression decisions. **The app performs no `PUT` today**, so this exists as the tested, shared parser behind the read path rather than as live write plumbing.
    * `extension DocumentPublication on DocumentListing` — `isPublic`, `servedVer` (the 2.0.0 serving rule applied to a listing row) and `hasUnpublishedWork`, which reports **proven** divergence only: public, *and* something has been promoted, *and* it isn't the head. A null `published_ver` means the gate was never closed, so claiming withheld work there would badge documents that have none.
  * Covered by [test/publication_gate_test.dart](file:///Users/kyle/Repos/slopcafe_ui/test/publication_gate_test.dart) — hermetic (it builds dio `Headers`/`Response` objects directly, including a bodyless **304**, rather than going over the wire): header precedence and the ETag fallback, every tag shape incl. the weak `W/` form, `ifMatchFor` returning null rather than a fabricated value, the serving rule on public/private/never-promoted rows, and the `NotLiveBadge` render contract. `tool/smoke_test.dart` carries the matching live-layer fixture (a `current_ver: 8` / `published_ver: 4` public row against a private row with the same numbers).
* **[lib/core/links.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/core/links.dart)**
  * The shared vocabulary for the **link graph** (`GET /d/:public_id/links`) — the hermetic sibling of `publication.dart`, no Flutter imports. Two enums and two extensions:
    * `LinkState` — the five states an outbound link resolves to (`live` · `redirected` · `retired` · `revoked` · `missing`) plus **`unknown`**, the forward-compat sentinel. `fromWire` tolerates case/whitespace and maps anything unrecognised to `unknown`. `isBroken` is `retired|revoked|missing` — **exactly the contract's broken-link report**. Two exclusions are load-bearing: `redirected` still *reaches* a document (stale, not dead — grouping it with the corpses would tell the operator a working link needs fixing), and `unknown` is **never** broken, because flagging a state a newer backend added would put a repair prompt on a healthy link every time the contract grows a value.
    * `LinkKind` — `public_id` | `slug` | `unknown`, the namespace the href addressed.
    * `extension OutboundLinkGraph on OutboundLink` — `linkState`, `linkKind`, `isBroken`, `canOpen` (keyed on `target_public_id`, not the state: `live` carries itself, `redirected` carries the forward target, the three broken states carry null), and **`canRepairSlug`**, the rule behind every slug-tombstone affordance. It is narrower than "broken" in one direction and wider in the other: a `/d/` link has no slug to point anywhere, a `missing` name has no tombstone (so the redirect route 404s — the fix for a typo'd link is to edit the document), while `redirected` *is* included despite not being broken, because an existing redirect is the only thing clear/release act on.
    * `extension DocumentLinkGraph on DocumentLinksResponse` — `brokenOutbound`/`brokenCount`, and `hasNoGraph`, which callers must read as "nothing **recorded**" rather than "nothing links here": a corpus published before the graph shipped has no rows until the backfill runs, and the two are indistinguishable from the client.
  * Covered by [test/link_graph_test.dart](file:///Users/kyle/Repos/slopcafe_ui/test/link_graph_test.dart) (hermetic — the state matrix builds `OutboundLink`s directly); `tool/smoke_test.dart` carries the matching live-layer fixture.
* **[lib/core/changes.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/core/changes.dart)**
  * The shared vocabulary for the **corpus change feed** (`GET /admin/documents?order=updated`) — the third hermetic core module, alongside `publication.dart` and `links.dart`, no Flutter imports. Two enums, one window type, one extension:
    * `DocumentOrder` — `created` | `updated`, the `?order=` values. A typo is a `400 bad_request`, never a silent fallback, which is why the wire strings are pinned in a test.
    * `ChangeKind` — `revoked` · `content` · `classification` · `unknown`, and `extension DocumentChange on DocumentListing` deriving it from the row. **Three of the four are proven, not inferred.** A content write always stamps *both* `updated_at` and `current_version_at`, so an `updated_at` meaningfully *after* `current_version_at` means something touched the row after the last write, and only a non-content change can do that — `classification` is a deduction. `revoked` is read straight off `revoked_at` and dominates everything (revoke clears the `current_*` fields, so nothing else stays comparable). `unknown` is a null `current_version_at` — nothing to compare, and the UI says nothing rather than guessing.
    * `DocumentChange.writeSkewTolerance` (2s) is the one judgement call, and the contract forced it: `updated_at` and `current_version_at` are "stamped by different statements of one D1 batch, so a pure content write can leave them a millisecond apart **either way**". Two seconds sits far above that batch skew and far below any interval a human retags in. **The tolerance absorbs the skew; the comparison itself is deliberately signed** (`updatedAt - currentVersionAt > tolerance`, never `.abs()`) — the two are not interchangeable, and only the signed form is sound, because a classification edit is by definition something that happened *after* the last write, so only `updated_at` running **ahead** is evidence of one. Sub-tolerance skew in either direction lands on `content` under both forms, which is exactly why a ±40ms test cannot tell them apart; `test/change_feed_test.dart` pins the sign with drift *beyond* the tolerance instead. The unresolvable case (a retag seconds after the write it followed) deliberately falls to `content`: under-reporting a change the operator just made themselves is invisible, while over-reporting sends them hunting an edit nobody made.
    * `ChangeWindow` — the `updated_since` presets (`day`/`week`/`month`/`all`). `updatedSince(now)` emits a **UTC** ISO-8601 instant (a local stamp would shift the window by the machine's offset — silently wrong, since the server parses it happily) and returns **null** for `all`, so the parameter is omitted rather than sent as a very old date. Presets rather than a persisted "since you last looked" watermark: this is a human-driven console, not a resuming headless consumer, so a watermark would import the feed's inclusive-boundary/idempotency obligations for a question a fixed window already answers.
  * Covered by [test/change_feed_test.dart](file:///Users/kyle/Repos/slopcafe_ui/test/change_feed_test.dart) (hermetic — rows are built as `DocumentListing` values); `tool/smoke_test.dart` carries both a JSON-layer fixture and, with a token, live checks incl. the `bad_cursor` rule.
* **[lib/core/review.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/core/review.dart)**
  * The shared vocabulary for the **review queue** — the operator-facing consequence of the publication gate, and the fourth hermetic core module beside `publication.dart` / `links.dart` / `changes.dart` (no Flutter imports). Two enums, one extension, one function:
    * `ReviewSide` — `live` (`published_ver`) | `latest` (`current_ver`), the two panes. Both resolve through the **same** pinned route `/d/:id/v/:n/raw`, including the live side that `/d/:id/raw` would also have served: comparing a pinned read against a serve-the-published-version read would compare two code paths, and any difference on screen would be ambiguous between "the content changed" and "the route differs".
    * `SourceComparison` — `identical` | `differs` | `unknown`, derived from the `current_source_sha256`/`published_source_sha256` pair already on every listing row, so it costs no extra request. Three-valued on purpose: a pre-retention version carries a null hash, and "cannot tell" must not collapse into either answer. `identical` says the two versions were written from the same **source** — deliberately not "render the same", since the two can have been run through different sanitizer releases.
    * `extension DocumentReview on DocumentListing` — `isAwaitingReview`, `pendingSince`, `versionsAhead`, `versionFor(side)`, `sourceComparison`. `isAwaitingReview` is `hasUnpublishedWork` **plus `!isRevoked`**, and that extra term is load-bearing rather than defensive: a queue entry promises two *renderable* versions and a revoked document's byte path is gone (`/d/:id/raw` answers 404/410), so queueing one offers a review that cannot be performed. `pendingSince` reads `current_version_at` — the content clock, i.e. when the head moved ahead of the pointer — and falls back to `updated_at` only when there is none, because `updated_at` also moves on a retag or a promote and would misdate the wait. `versionsAhead` is plain arithmetic on the two numbers and says so; it `clamp`s at 0 rather than trusting that versions only count upwards.
    * `reviewQueueFrom(rows)` — filter + de-duplicate on `public_id` + sort **most-recent-pending-work first**. Newest-first rather than FIFO because every list in the app is newest-first and an operator opening the queue is nearly always reacting to a burst an agent just wrote; nothing is starved either way, since the queue is exhaustive and states its full length. De-duplication is not theoretical: a document written *during* the walk can be delivered on two pages of an `updated`-ordered fetch as its position shifts.
    * `VisibilityFilter` (`public`|`private`) and `PublicationFilter` (`pending`|`current`) — contract **2.2.0**'s query-param wire values, pinned in a test for the same reason `DocumentOrder`'s are: an unrecognised value is a hard `400 bad_request`, never a silent fallback to unfiltered, so a typo would fail the whole screen rather than quietly widen it.
  * **Since 2.2.0 the server has the predicate** (`?visibility=public&publication=pending`), so this file no longer does the heavy lifting — but `reviewQueueFrom` is still applied to the result and is **not** vestigial. The server's `pending` is `published_ver IS NOT current_ver`, and NULL is distinct from any number, so a **never-promoted public document** satisfies it while being *not gated at all* (by the 2.0.0 serving rule it already serves its head). `isAwaitingReview` excludes those. The spec is ambiguous about whether the backend special-cases this for public rows, and the client is right either way — which is the point. Taking rows rather than a query is also what let this file survive the 2.2.0 change untouched in its substance.
  * Covered by [test/review_queue_test.dart](file:///Users/kyle/Repos/slopcafe_ui/test/review_queue_test.dart) (hermetic — rows built as `DocumentListing` values): the admission matrix incl. a revoked row constructed to defeat the *incidental* exclusion, both clocks, the negative-gap clamp, the three-valued source comparison, and the ordering + de-dup rules.
* **[lib/core/deep_link.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/core/deep_link.dart)**
  * The shared vocabulary for **inbound web links** — the fifth hermetic core module beside `publication.dart` / `links.dart` / `changes.dart` / `review.dart` (no Flutter imports, no `dart:io`). One constant, one value type, one pure function:
    * `kDeepLinkHost` — the public web host this build claims. **The knob an adopter changes.** It is necessarily written twice (Gradle compiles it into the manifest filter that decides whether Android *offers* the app the tap; this constant decides whether the app *accepts* it), so the pair is pinned by a test that reads the literal back out of `android/app/build.gradle.kts`. A mismatch is silent and total — every link would open the app to nothing — which is exactly the class of bug a test, not a comment, has to prevent.
    * `DeepLinkTarget` — a `/d/<public_id>` or `/s/<slug>` **raw addressed name**, deliberately not a resolved document, for the same reason the link graph stores names: `/s/` is late-bound, so what a slug points at is a property of the read.
    * `parseDeepLink(Uri)` — the claim matrix. Accepts `/d/` and `/s/` on the pinned host over `https` **and** `http` (letting an unencrypted link escape to a browser is strictly worse than opening it here, since the app then talks to its own configured `https` Base URL regardless), tolerates trailing byte-path segments (`/raw`, `/v/:n/raw` — a copied URL still names its document), and returns **null for everything else**. Null means "not ours" and callers must no-op on it rather than surface an error: the intent-filter is already scoped to `/d/` and `/s/`, so a future widening should degrade to silence, not to a toast about a link the operator never expected the app to claim. The one accepted under-delivery is that a version-pinned link opens the *latest* view, because `ReaderScreen` owns version selection in its own state rather than in its constructor.
  * Covered by [test/deep_link_test.dart](file:///Users/kyle/Repos/slopcafe_ui/test/deep_link_test.dart): the claim/reject matrix in both directions, and the Gradle↔Dart host invariant (verified to discriminate — flipping the Gradle literal fails the test).
* **[lib/core/theme.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/core/theme.dart)**
  * Assembles the Cortado `ThemeData` (light + dark) from the design tokens: maps the palette onto a Material `ColorScheme` AND registers the raw token set as a `ThemeExtension`, plus component themes (cards, inputs, buttons, sheets).
* **[lib/core/design/tokens.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/core/design/tokens.dart)**
  * `AppColors` (a `ThemeExtension`) — the Cortado-light + Cortado-dark palettes (OKLCH→sRGB), shadows, tag tints, and decorative accents. Read anywhere via `context.colors`. Also `AppRadii` and `AppSpacing`.
* **[lib/core/design/layout.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/core/design/layout.dart)**
  * `AppLayout` — the adaptive large-screen system: the rail breakpoint, content max-widths (`contentMax`/`formMax`/`readerMax`/`sheetMax`), `gutterFor(width)` and the `AdaptiveGutter` builder widget, plus the `context.isExpandedLayout` / `context.shellBottomInset` extensions. See **Large-screen adaptation** above.
* **[lib/core/design/typography.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/core/design/typography.dart)**
  * `AppText` — named serif/sans/mono text styles (system-font fallback stacks) + the Material `TextTheme` builder.
* **[lib/core/format.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/core/format.dart)**
  * Shared display formatters: `fmtBytes`, `fmtDate`, `relTime`, `greeting`, `titleCase` (tag/collection display names). The copy-bearing helpers take an `AppLocalizations`: `relTime(l10n, date)` and `greeting(l10n)`; the rest stay locale-neutral.
* **[lib/l10n/](file:///Users/kyle/Repos/slopcafe_ui/lib/l10n/)** — localization. `app_en.arb` (the single editable copy file), `l10n.dart` (the `context.l10n` extension), and the generated `app_localizations*.dart`. See the **Localization (i18n)** section above.

### 3. Data Models — `lib/api/` (GENERATED — see the API layer section below)
The hand-written `lib/models/document.dart` / `lib/models/agent.dart` have been
**replaced by code generated from the OpenAPI contract**. App code imports the
barrel **[lib/api/api.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/api/api.dart)**.
* **[lib/api/models.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/api/models.dart)** *(generated, do not edit)* — `freezed` + `json_serializable` data classes for every JSON request/response body in the spec: `DocumentListing`, `SearchHit`, `AgentListing`, `AgentKey`, `ListDocumentsResponse`, `SearchDocumentsResponse`, `ListAgentsResponse`, `ListAgentKeysResponse`, `MintAgentKeyResponse`, `CreateOAuthClientResponse`, `CreateUnboundOAuthClientResponse`, `Revoke{,Agent,Key}Response`, `SetDocument{Visibility,Slug,Tags}Response`, `HealthzResponse`, `BackfillResponse` (the per-page `POST /admin/vectors/backfill` result — drives the vector-index backfill; see the **semantic search** note below), etc. Nullable (OpenAPI-3.1 `anyOf`-null) fields generate as nullable Dart; `created_at`/`revoked_at` (plain spec strings) are typed `DateTime`; `visibility`/`matched_field` stay `String` (not enums) to avoid call-site churn (the semantic-search update added `"semantic"` as a fourth `matched_field` value — a vector-only concept hit; because the field is kept as `String`, this was a no-op for the generated layer). `DocumentListing`/`AgentKey` expose an `isRevoked` getter; `SearchHit` is flat with a `.document` view + `SearchHit.fromDocument(...)` (used by the offline local-search fallback). `toJson` emits the same snake_case keys as before, so the offline document cache stays compatible (a stale cache missing a newly-required field just fails the parse and falls back to an online fetch — `getCachedDocumentList` swallows it). As of the authoring update both `DocumentListing` and `SearchHit` also carry a **required** `created_by_kind` (`agent`|`operator`, kept as `String`), distinguishing operator-authored documents. As of contract **0.5.0** the `AgentKey` item also carries **required** `expired` (`bool`, server-computed) and `expires_at` (nullable `DateTime`; **null = never expires** — every operator-minted key and all legacy rows; a non-null value is a short-lived publish credential's TTL). Per the contract a key authenticates **only while `revoked_at` is null AND `expired` is false**, so `AgentKey` exposes `isActive => !isRevoked && !expired` (alongside the existing `isRevoked`) — the Operate key list buckets on `isActive` so a lapsed-but-un-revoked credential isn't shown as live (see Operate below). As of contract **0.10.0** (the lifecycle-status + context-packs update) `DocumentListing`/`SearchHit`/`ReadSourceResponse` carry a **required** `status` (`active`|`deprecated`|`archived`, kept as `String` like `visibility`) and a nullable `superseded_by` (a replacement doc's `public_id`; the contract mandates it is **never auto-followed** — surface it, let the reader decide), and new generated classes exist for the status mutator (`SetDocumentStatusResponse`) and the context-pack envelope (`PackResponse`/`PackInfo`/`PackRoot`/`PackDocument`/`PackOmitted`). The status axis is consumed across the UI (see the **Lifecycle status** notes under the provider, Operate, Reader, Search, and widgets entries); the pack envelope is generated but **no UI consumes packs yet**. As of contract **2.0.0** (the publication gate) `DocumentListing`/`SearchHit`/`PackDocument` carry five more fields — **required** `updated_at` (non-nullable `DateTime`) plus nullable `current_version_at`, `published_ver`, `current_source_sha256` and `published_source_sha256` — and `WriteResponse`/`ReadSourceResponse` gained a nullable `source_sha256`. `published_ver` *is* the gate: a listing row now carries both the head and the version readers are actually served, so every "NOT LIVE" surface in the app is a pure function of the row and needs no per-document probing (the derivation lives in `lib/core/publication.dart`, not on the generated class). Because `updated_at` is **non-nullable**, every hand-built `DocumentListing` has to supply one — the Reader's not-found placeholder synthesises the same `DateTime.now()` it already used for `createdAt` (a record we invented has never been touched, and any other timestamp would be a fabrication); the `test/status_lifecycle_test.dart` fixture and the `tool/smoke_test.dart` revoked-doc JSON gained one too. Note `updated_at` and `current_version_at` are **different clocks**: the latter is the last *content write*, the former the last touch of any kind (retag, rename, visibility, status, revoke — and a promote), so a row where `updated_at` is later than `current_version_at` is the ordinary shape of a document that was reclassified since it was last written. New generated classes: `ListVersionsResponse`/`VersionListing` (the real version history — `is_current` and `is_published` are **orthogonal** booleans, and `source_present` is the gate on offering Restore), `PromoteResponse`, `RestoreResponse` (a full sanitizer report, not a bare version number), plus `ReadTextResponse` (still unconsumed — see the API-layer section's *deliberately unconsumed* list) and the four link-graph shapes (`DocumentLinksResponse`/`OutboundLink`/`OrphanDocumentsResponse`/`LinksBackfillResponse`), which the link-graph adoption now consumes. The three slug-tombstone responses (`SetSlugRedirectResponse`/`ClearSlugRedirectResponse`/`ReleaseSlugTombstoneResponse`, plus the unreferenced `RedirectTarget`) have been generated since the 2.0.0 re-pin but went unrecorded here until the link-graph work adopted them — they were never listed as unconsumed either, so **don't read the generated class list as complete evidence of what is or isn't wired**; grep for the import. Note `ClearSlugRedirectResponse.redirect_to` generates as `dynamic`: the spec types it as a bare null (it is always null by construction), which the emitter has no nullable Dart type to narrow to.
* **[lib/api/error_code.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/api/error_code.dart)** *(generated, do not edit)* — the `ErrorCode` enum: the 33 `error` discriminants of the `ErrorBody` oneOf union (0.10.0 added `bad_status` + `invalid_status`; 2.0.0 added `slug_locked`, `too_deep` and `version_not_found`), plus `unknown` (forward-compat). `ErrorCode.fromWire(String?)` maps a wire value to a code.
* **[lib/api/api_error.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/api/api_error.dart)** *(hand-written glue)* — `ApiError`: a typed view over the `ErrorBody` envelope. `ApiError.fromException(e)` parses a `DioException`; `ApiError.describe(e)` returns the backend's `message` (falling back to the raw error) for toasts; discriminant extras are exposed as getters (`clientId`, `hint`, `slug`). The OAuth-exists path keys on `ErrorCode.clientExists`.

### 4. State Management (Providers)
* **[lib/providers/document_provider.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/providers/document_provider.dart)**
  * Manages state for fetching document listings, including cursor-based infinite pagination.
  * Automatically serializes and caches the document list locally when online, and falls back to loading cached listings when offline.
  * Aggregates document tags for sidebar/filter widgets.
  * Integrates search algorithms (such as full-text BM25 keyword matching) to calculate relevance score indicators.
  * Implements local client-side search fallback over cached listings with highlight formatting when offline.
  * `documentSearchProvider` (a `FutureProvider.family` keyed by `SearchQueryParams`) calls `GET /admin/documents/search`. `SearchQueryParams` carries the `SearchMode` enum (`hybrid` (default) · `keyword` · `semantic`) which is sent as the `mode` query param — the backend's hybrid leg fuses BM25 + vector (Vectorize/Workers-AI) search via RRF. The Search tab's segmented selector drives it. The offline cached fallback is keyword-only substring matching regardless of mode (mirroring the backend's own best-effort degrade-to-keyword behaviour when embedding is unavailable).
  * Provides triggers for revoking specific documents and caching them locally.
  * Exposes `updateStatus(publicId, status, {supersededBy})` — the lifecycle-status mutator (`POST /admin/documents/:id/status`, no version bump), the fourth sibling of the `updateVisibility`/`updateSlug`/`updateTags` writes. `'deprecated'` may carry a `supersededBy` replacement public_id; the backend force-clears the pointer on `'active'` and the response is canonical for both fields (the local row + offline cache mirror it).
  * Exposes `authorDocument(...)` — the operator authoring write (`POST /admin/documents`). Builds the inline request body (`content`+`format` required; optional `title`/`description`/`tags`/`slug`/`visibility`), returns the backend's `WriteResponse`, and reloads the canonical first page on success.
  * Exposes the **version trio** added for the 2.0.0 publication gate:
    * `fetchVersions(publicId)` → `ListVersionsResponse` (`GET /admin/documents/:id/versions`; newest first, capped at the 200 most recent, no cursor). Deliberately **uncached and never folded into `DocumentsListState`** — a promote or a restore invalidates it the instant it happens, and it is only ever read by a screen that is already open, so callers re-fetch rather than hold it.
    * `promoteVersion(publicId, version)` → the updated `DocumentListing` (`POST /admin/documents/:id/promote`, hand-built body). Promote is **not a write**: no version bump, no sanitizer re-run, no FTS/vector resync — it only repoints `published_ver`, which makes it idempotent and legal on a *private* document (that stages the choice before the door opens; the later flip to public keeps it). The response is canonical for `published_ver`, so it is mirrored onto the row in state **and** into the offline cache exactly like the `visibility`/`slug`/`tags`/`status` mutators.
    * `restoreVersion(publicId, version)` → **`RestoreResponse`** (rewritten onto the operator JSON route `POST /admin/documents/:id/restore`; the old `POST /d/:id/restore` `FormData` call and its version-number guessing are gone). Restore is mandatorily **restore-as-new** — the backend re-writes the chosen version's retained source as a brand-new head and never rewinds `current_ver`, so `RestoreResponse.version` is a number that did not exist before the call. It re-runs the *current* sanitizer over *old* source, so the response carries a fresh `modified`/`stripped`/`will_not_render` report — the reason this returns the typed response instead of an `int`. It bumps `current_ver` (so it reloads the canonical first page) and deliberately leaves `published_ver` alone, meaning a restore on a gated public document is **not** what readers get. Callers must check `VersionListing.sourcePresent` first: a pre-`0008` version predates source retention and cannot be restored.
  * Exposes `resolveListing({publicId, slug})` — the **shared name→listing resolver**, the landing point for every *late-bound* reference into the corpus: a link tapped inside the Reader's WebView, an outbound edge in the link graph, and an inbound web link from outside the app (`lib/core/deep_link.dart`). All three arrive holding a raw addressed name rather than a document, which is exactly what the link graph's late-binding rule says they must. Was private to `ReaderScreen`; lifted here so "a tapped link and a tapped web link open the same document the same way" is true rather than coincidental. Three tiers, cheapest first: the loaded list → `GET /admin/documents?slug=` / `GET /admin/documents/:id` → a synthesised placeholder row. Two properties are load-bearing: it resolves through the **admin** surface, so a **private** target opens rather than dead-ending (the one thing a browser could never do with that URL); and the placeholder tier exists **only for `public_id`**, because an id is an immutable capability — a failed lookup on one means "couldn't ask" (offline, rejected token) far more often than "gone", and opening the Reader on a placeholder lets its own byte-path fetch produce the real answer, including an offline-cached body and the honest revoked/not-found page. A **slug** gets no placeholder: a name that resolved to nothing leaves no id to fetch bytes with. Returns null rather than throwing — every caller is a navigation gesture, and a network blip should cost the tap, not the screen.
  * Exposes `backfillVectors(VectorBackfillMode)` — walks the cursor-paginated `POST /admin/vectors/backfill` to completion (`missing` = embed un-vectored docs · `rebuild` = re-embed the corpus) and returns a summed `BackfillSummary`. Drives the Operate › Documents "Search index" action; touches no document state (vectors are a search-side index).
* **[lib/providers/agent_provider.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/providers/agent_provider.dart)**
  * Manages agent lists, agent creation, key minting/rotation, and key revocation actions.
* **[lib/providers/links_provider.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/providers/links_provider.dart)**
  * `LinkGraphService` / `linkGraphServiceProvider` — the link graph and the slug tombstones that repair it. A **standalone service, not a notifier** (the shape of `AgentManagerService`): none of these calls owns app state, and the graph is a live view every write invalidates — the backend re-extracts a document's links in the same D1 batch as the write — so a cached neighborhood would be wrong exactly when it mattered.
    * `fetchLinks(publicId)` → `DocumentLinksResponse` (`GET /d/:id/links`). Uncached per open, like `fetchVersions`. Auth is **any authenticated reader, never public** — backlink rows are listing rows for other documents, including private ones. Answers `404` on a revoked document (revoke deletes its outbound rows), which is why the Reader gates the entry point on `!isRevoked`.
    * `fetchOrphans()` → `OrphanDocumentsResponse` (`GET /admin/links/orphans`). No cursor by design (a curation worklist, capped at 200).
    * `backfillLinks()` → `LinksBackfillSummary`, walking `POST /admin/links/backfill` to completion. **Always rebuild semantics** — extraction is cheap and deterministic, so unlike the vector backfill there is no mode to pick and no expensive option to warn about. Note the per-page counters: `scanned`/`updated`/`links` are all **summed**, because the spec's field description is explicit that `links` is "Total link rows stored across **this page**" — the prose reference's looser "total rows stored" reads either way, and taking the last page's value would report one page of a multi-page sweep. `unreadable` (`scanned - updated`) is the contract's re-run signal: a document whose render couldn't be fetched is scanned-but-not-updated.
    * `setSlugRedirect(slug, targetPublicId)` / `clearSlugRedirect(slug)` / `releaseSlug(slug)` — the three slug-tombstone mutators. The redirect body is modelled inline in the spec, so the map is hand-built like promote/restore. All three take a **retired** slug: `404 not_found` on a slug route never means "no such document", it means the name isn't retired. Slugs are percent-encoded into the path so a hand-typed one earns its `400 bad_slug` instead of silently addressing another route.
* **[lib/providers/changes_provider.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/providers/changes_provider.dart)**
  * `ChangeFeedNotifier` / `changeFeedProvider` — one walk of `GET /admin/documents?order=updated`, with `ChangeFeedState` holding its own `documents` / `nextCursor` / `window` / `windowSince`. **`ChangeFeedState` has no `copyWith`** — every construction lists its fields explicitly, so an omitted field silently resets to its default; check the whole list when adding one.
  * **Separate state from `DocumentsListState` is a correctness measure, not tidiness.** A cursor encodes the ordering that minted it, and replaying an `updated`-ordered cursor under the default ordering is a hard `400 bad_cursor` — never a silent re-sort. Two state objects make that unreachable: there is no field the two walks' cursors can both be written to. For the same reason `order` is sent on **every** page, alongside the `cursor`, not just on the first request.
  * Changing the window (or a pull-to-refresh) **restarts the walk** rather than continuing it — the held cursor was minted under the old window and page position — and bumps a generation token so the abandoned walk's in-flight page cannot commit when it lands. `reload()` is intentionally not blocked while a fetch is running (see the API-layer bullet).
  * The resolved `updated_since` instant is **pinned once per walk** in `windowSince` and replayed on every page; recomputing it per page would advance the window's lower bound mid-walk and drop the rows nearest it.
  * De-duplicates on `public_id` across pages, because `updated_since` is **inclusive** by contract (the boundary row is re-delivered on purpose so a resuming consumer can't skip one at a shared millisecond).
  * Deliberately **no offline fallback and no write to the offline cache**, unlike `documentsListProvider`. The feed's whole claim is "this is what moved recently", which the cached listing cannot answer — it is a snapshot with no record of when anything changed relative to a window — and writing a recently-*changed* slice into `documents_list.json` would corrupt the newest-created page the app falls back to offline.
* **[lib/providers/review_provider.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/providers/review_provider.dart)**
  * `ReviewQueueNotifier` / `reviewQueueProvider` — reads the queue via contract **2.2.0**'s filter: `GET /admin/documents?visibility=public&publication=pending&order=updated`. `ReviewQueueState` carries `documents` / `isLoading` / `hasLoaded` / `isComplete` / error, and like `ChangeFeedState` it has **no `copyWith`** — every construction lists its fields, so an omitted one silently resets.
  * **It was a full corpus sweep until 2.2.0**, because no server-side predicate existed. Now the rows crossing the wire are proportional to the *queue* rather than the corpus, and `limit=200` makes it a single request in every realistic case. It still walks to exhaustion rather than offering a Load-more button: the change feed has a natural "that's enough" point and a review queue does not, since one that stops early hides work somebody is accountable for.
  * **`visibility=public` is not optional decoration.** `publication=pending` alone also matches private drafts that were never published — documents with no reader to withhold anything from. The pair is the queue; either half alone is not.
  * **`scanned` is gone** with the sweep. It existed to contextualise "0 waiting" against the size of the corpus it was drawn from; the server now answers the question directly, so there is no corpus figure to report and none is needed. **`isComplete` and the `_maxPages` backstop were kept** (now 50 pages = 10k *already-pending* documents) even though tripping them should be unreachable — the cost is nothing while they hold, and a review queue that truncates in silence reads as "all caught up" while hiding work.
  * **Separate state from both `DocumentsListState` and `ChangeFeedState`** for the change feed's two reasons — an `updated`-ordered cursor must never meet the `created`-ordered one, and a gate-filtered slice must never reach `documents_list.json` — plus its own third: this walk carries filters the others do not, and the server does not encode filters in the cursor, so a cursor swapped between them would silently return the wrong population rather than erroring.
  * `order=updated` rather than the default: the interesting rows (just-rewritten documents) land on the first page, and the walk's ordering matches the queue's own sort key, so rows arrive roughly sorted and the list stops reshuffling as pages land. **No `updated_since`** — a document rewritten last year and never approved is still waiting, so narrowing the walk would trade completeness on the one surface that cannot afford it.
  * Generation token like the change feed, but it matters more here: this is a *loop* of awaits, so an abandoned walk has many chances to commit. Every write is gated and the loop itself breaks on a generation change.
  * `resolve(publicId, publishedVer)` — a local edit that drops a row after a successful publish, taking the **canonical** `published_ver` off the `PromoteResponse` so the row is re-evaluated rather than assumed resolved (promoting v6 of a document whose head is v8 leaves it in the queue). A re-read instead would cost a round trip to learn one row's new number — and would reshuffle the queue under an operator working through it, since a promote stamps `updated_at`. No offline fallback and no cache write, for the change feed's reason: the queue's claim is "these are waiting *right now*", and a stale one would have the operator approving against numbers that have moved.
* **[lib/providers/deep_link_provider.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/providers/deep_link_provider.dart)**
  * `deepLinksSupported` — the **mobile-only rule**, stated in one place rather than left incidental. `Platform.isAndroid || Platform.isIOS`; iOS is included ahead of the target existing so that adding one is an Xcode capability plus an `apple-app-site-association` file, not a hunt through Dart for a platform check that silently excluded it.
  * `appLinksProvider` (injection seam over the `app_links` singleton) and `inboundDeepLinksProvider` → `Stream<DeepLinkTarget>`.
  * **A plain `Provider<Stream<…>>`, not a `StreamProvider`**, and that is a correctness choice: the consumer wants each link as an *event* to navigate on, while `StreamProvider` would hand it de-duplicated application *state* — re-opening the same URL twice in a row is a repeat of an equal `AsyncData<DeepLinkTarget>`, so a `ref.listen` would never fire the second time and the tap would land on nothing.
  * **No `getInitialLink()` call, and adding one would be a bug rather than belt-and-braces.** The Android plugin holds the launch intent's URL and flushes it to the *first* subscriber, once, guarded by its own `initialLinkSent` flag. So the cold-start link survives an arbitrarily long detour through first-run setup, and a later re-subscription does not replay it into a second Reader.
* **[lib/providers/health_provider.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/providers/health_provider.dart)**
  * `healthProvider` (`Notifier<HealthState>`) — the best-effort `/healthz` snapshot (sanitizer version + R2 storage cap/used) shown on the Operate stat grid + storage bar. Was Operate-local widget state; **lifted into a provider** so the shared pull-to-refresh can reload it from either home tab. `load()` is best-effort (a failure keeps the last snapshot; the opportunistic `storage_used_bytes`/`storage_bytes_used` field isn't in the contract).
* **[lib/providers/refresh.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/providers/refresh.dart)**
  * `refreshFleetData(ref)` — the single pull-to-refresh routine for both home tabs. Concurrently reloads the document list, the agent fleet, and the health snapshot (`clear: true` for the lists) — the same fetches a cold app open kicks off, so a pull behaves "as if the app were opened anew" regardless of which tab started it. Wired into both Library's and Operate's `RefreshIndicator`.

### 5. UI & Screens (Cortado IA)
* **[lib/screens/library_screen.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/screens/library_screen.dart)** — Library ("The Café") tab.
  * Greeting + connection-status pill (taps through to Settings), fleet/menu tickers, a text-forward "Today's Special" featured plate (`DocFeedCard`; currently the newest public doc — curated selection deferred to a future backend feature), a tag-based Collections carousel ("All" → `CollectionsScreen`, tiles → `DocumentListScreen`), and a "Recently plated" list ("See all" → `DocumentListScreen`; its `_DocRow` shows the `NotLiveBadge` next to the version it qualifies, and both ends of that meta line now flex so the extra pill can't overflow a narrow phone). Offline banner from cache state. No search bar (Search is its own tab) and no per-doc `OFFLINE READY` badge. **Pull-to-refresh** (a `RefreshIndicator` over the `ListView`) runs the shared `refreshFleetData` — a full document + agent + health reload, identical to an Operate pull.
* **[lib/screens/search_screen.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/screens/search_screen.dart)** — Search tab.
  * Autofocus query field, debounced live results via `documentSearchProvider` (with local cached fallback), relevance bars, matched-field pills (deprecated hits also carry the `DEPRECATED` badge, and hits with unpublished work the `NotLiveBadge` — search indexes `current_ver`, so a snippet can quote text no reader has been served yet), and highlighted snippets. Suggestion chips when idle. A **segmented Hybrid/Keyword/Semantic mode selector** (`_SearchModeSelector`, styled like `PillTone.solid`) appears beneath the field while a query is active and drives `SearchQueryParams.mode`. Semantic-only hits surface as a `SEMANTIC` matched-field pill and an **un-bracketed** snippet (the backend deliberately omits the `[term]` brackets to signal "concept match, not term match"); the snippet parser already renders an un-bracketed string as plain text, and the relevance bar normalizes against the in-set max score (so the per-mode score scales — fused RRF / negated BM25 / cosine — stay comparable within a result set).
* **[lib/screens/operate_screen.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/screens/operate_screen.dart)** — Operate ("The Pass") tab.
  * Fleet stat grid + R2 storage bar; a "Kitchen" segment (agent rows → an agent bottom-sheet with keys, mint key, OAuth client, kill; plus mint-agent and unbound-OAuth flows). The agent sheet's key list buckets on `AgentKey.isActive`: live keys (neither revoked nor expired) get the revoke action and, if they carry a future TTL, an "Expires …" sub-label; everything else falls into a single **INACTIVE** audit section where each `_KeyRow` shows the right tombstone pill — red **REVOKED** or neutral **EXPIRED** (a lapsed-but-un-revoked short-lived publish credential) and a "Documents" segment (an **"Author a document"** CTA that pushes the compose screen, a **"Search index"** entry that opens the vector-backfill sheet (`_BackfillSheet` — index-new vs rebuild, each confirmed), the admin doc list with include-revoked, a three-way **lifecycle filter** (`_StatusFilterSegmented`: All/Active/Deprecated — client-side over the loaded provider pages, like include-revoked) and a `DEPRECATED` badge on deprecated rows, + a per-doc actions sheet: visibility/**publish**/**status**/slug/tags/revoke — the status row toggles Mark deprecated (via the shared `showDeprecateSheet`, optional `superseded_by` target) / Mark active (plain confirm; the pointer clears server-side)). After authoring it surfaces the outcome (incl. any sanitizer adjustments) via toast. The `/healthz` metrics come from `healthProvider` (no longer Operate-local state). **Pull-to-refresh** (a `RefreshIndicator` over the `ListView`) runs the shared `refreshFleetData` — the same full reload as a Library pull.
  * **Review queue.** A **"Review queue"** entry leads the Documents segment's four entries, above "Search index", and pushes `ReviewQueueScreen`. First because it is the only one of the four with a **backlog** — the other three are maintenance the operator chooses to run, this one is work waiting on them. It deliberately carries **no pending count**. That was once because an honest number required a full corpus sweep; since 2.2.0 it would only cost one filtered request, so the reason is now purely that the only figure cheaply available *on this screen* is a lower bound over whatever pages the shared list happens to hold — which would sit a few pixels above the very rows whose `NOT LIVE` badges contradict it. A real count here would mean this tab issuing the queue's own query on every Operate build, which is a worse trade than one tap.
  * **Change feed.** A **"Recent changes"** entry sits between "Search index" and "Link graph" in the Documents segment and pushes `ChangesScreen`. A pushed screen rather than a sheet (unlike its two neighbours): it is a browse/triage surface with its own pagination, not an action menu — and reading its own walk is what keeps its `updated`-ordered cursor away from the `created`-ordered one `documentsListProvider` holds.
  * **Link graph.** A single **"Link graph"** entry sits beside "Search index" in the Documents segment and opens `_LinkGraphSheet` — one entry rather than three, because the three are one subsystem and only the first is an action on its own: **Rebuild link graph** (the backfill sweep; confirmed, but `danger: false` — unlike the vector rebuild this is the *cheap* option and the copy says so, while still confirming because it rewrites rows across the whole corpus), **Orphans** (pushes `OrphansScreen`) and **Retired names** (opens the shared slug-repair sheet). The result toast reports `updated`/`links`, or the re-run warning when `hasUnreadable`.
  * **Publication gate (2.0.0).** Admin rows carry the honey `NOT LIVE` pill whenever `hasUnpublishedWork` holds — honey like `DEPRECATED` (a caution about how the document is *served*), never the red of a revoked kill. The actions sheet leads with **Publish**, because it is the only action in there that changes what a reader is handed and it is what the row's marker sent the operator in to do; its header shows the head pill (`v7`) beside a `Live v5` pill (honey while the gate holds work back, and omitted entirely on a private document, which serves nobody), with a plain-language note underneath. The Pass publishes the **current** version and nothing else — picking an arbitrary version is a rollback, needs the history's dates/authors/`source_present` to be made safely, and the Reader already owns that surface; here the two numbers on the row have already said what would change. The action is offered whenever there is a head that isn't already published (including on private documents, which stages the choice), confirmed like a write even though promote isn't one, and the confirm sheet appends a lock-icon caveat on a private document so "everyone will see v4" isn't left to imply the door is already open. Toast reports `PromoteResponse.published_ver` (canonical) and maps `ErrorCode.versionNotFound` to "that version no longer exists", which tells the operator to reopen the list in a way a generic failure does not.
* **[lib/screens/compose_screen.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/screens/compose_screen.dart)** — operator authoring / **Markdown composition** screen, pushed from Operate › Documents.
  * A deliberately **forkable starting point**. The plumbing is wired end-to-end against the live contract: the metadata form (title/description/tags/slug/visibility), the `format` (Markdown/HTML) toggle, the write/preview switch, the `POST /admin/documents` call (via `documentsListProvider.authorDocument`), and sanitizer-aware result handling. The actual Markdown/HTML **rendering is intentionally left as an isolated swap seam** — `_PreviewPane`, currently a raw-source passthrough — so renderer packages (flutter_markdown, markdown_widget, gpt_markdown, …) can be A/B'd without touching the rest of the screen. **No rendering dependency was added** (deferred by design). Visibility is tri-state (Default = omit the field → deploy default · Public · Private). Pops the `WriteResponse` on success (the list is already reloaded by the provider). Render/behaviour covered by [test/compose_screen_test.dart](file:///Users/kyle/Repos/slopcafe_ui/test/compose_screen_test.dart).
* **[lib/screens/reader_screen.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/screens/reader_screen.dart)** — full-screen document Reader ("the plate"), pushed as a route.
  * WebView-only reader with deliberately minimal chrome: a compact top bar (back · version chip · more), a single-line truncated serif title, a thin meta row (visibility · author · date), and tappable tag chips (→ `DocumentListScreen`). The WebView lives in an `Expanded` inside a `Column` (not a scroll view), so it scrolls internally and owns the screen edge-to-edge. Preserves the version-first offline cache strategy (conditional `If-None-Match`/`304`, instant cached render, reload only on version change). Operator actions are consolidated in the more-sheet (copy link, make public/private, mark deprecated/active — the lifecycle-status toggle, same `showDeprecateSheet` flow as Operate — edit slug & tags, copy slug URL, open in browser, revoke) plus publish and version restore. A deprecated document shows a `DEPRECATED` badge in the meta row and a honey caution **banner** above the WebView; when `superseded_by` names a replacement, the banner's Open CTA resolves it (by public_id, via the same `_resolveDocumentListing` path link-clicks use) and pushes a new Reader — the contract never auto-follows the pointer, so the tap is the reader's explicit decision. Pops `true` after a revoke so list/search callers can refresh. Intercepts link clicks: matching configured Host URL links navigate the app internally to the document (resolving by public ID or slug), while other external links present an "Open in browser?" warning dialog before launching. (The cover, description, `OFFLINE READY` badge, bottom action bar, and the HTML/Markdown/Report view modes were removed.) **macOS caveat (fixed):** WKWebView on macOS reports our own `loadHtmlString(..., baseUrl:)` to the navigation delegate as a request for the bare base URL — the `_expectSyntheticBaseLoad` flag (armed around each programmatic load, checked via `_isBareBaseUrl`) lets that synthetic navigation through; without it the reader intercepts its own content load and the WebView stays blank behind an "Open in browser?" prompt. On wide windows the document surface centers to `AppLayout.readerMax`.
  * **Publication gate (2.0.0).** The latest view renders and caches the **served** version, not `currentVer`: `_loadHtmlIntoWebview` loads `_currentDoc.servedVer`, and on a `200` it re-files the bytes under `resolveServedVersion(response.headers)` — the server describing the exact bytes in hand outranks anything a listing predicted — with the predicted number as the fallback for a stripped or malformed header. The pinned historical view is exempt (its URL already names the version, and taking the label from a header there could file bytes under a key the cache would never ask for again). `_adoptCurrentVersion` reads `x-doc-current-version` off **both** the `200` and the `304` and adopts it only when it is *ahead* of what we hold — versions only count upwards, so a lower number is `resolveCurrentVersion` falling back to the ETag on a promoted document, and writing that back would erase the divergence the chrome exists to report. The **version chip** names what is on screen: `Live v5` in honey (with the `public` icon) whenever the served version is behind the head, plain `v7` otherwise — labelling the screen "v7" while the reader looks at v5 is the exact lie 2.0.0 makes possible. A honey **served-version banner** sits above the WebView when `hasUnpublishedWork` holds; its CTA pins the view to the head via `/d/:id/v/:n/raw` (the only way to read unpublished bytes) and deliberately publishes nothing on the operator's behalf. The version sheet is now **real history** — `fetchVersions` per open, never cached, rendered by `_buildVersionList` with a dot per row (green published / honey current / line otherwise), orthogonal `CURRENT` + `LIVE` pills, `by <author> · date · size` meta, and per-row **Publish** (hidden on the already-published row, confirmed via `showConfirmSheet`, `ErrorCode.versionNotFound` → its own toast) and **Restore** (hidden on the head; visible-but-disabled with an explanatory hint when `source_present` is false, since silently omitting it would read as an app bug). The historical banner drops its Restore button when the pinned version *is* the head, because pinning there is now an ordinary state.
  * **Link graph.** A **Links** row in the more-sheet (gated on `!isRevoked` — `/links` 404s for a revoked id, so offering it there could only produce an error) opens `_openLinksSheet` → `_buildLinkGraph`, structurally the sibling of the version sheet: fetched per open, never cached, future started outside the builder. Two sections — **Referenced by** (`_BacklinkRow`; backlinks arrive as full listing rows, so these open the Reader with **no resolution hop**, unlike an outbound link, which carries only the raw name its author typed) and **Links out** (`_OutboundLinkRow` — a state-coloured dot, the authored path `/s/name` or `/d/id` in mono *rather than* a resolved title, since that string is what the operator must go and find in the source, the `LinkStateBadge`, and a resolved title underneath as confirmation of where it lands). A honey `_SheetNote` heads the sheet when anything is broken; `hasNoGraph` replaces the whole body with the "nothing recorded" note. A repairable row carries a **Repair name** button into `showSlugRepairSheet`. Tapping a resolvable link routes through `_openLinkTarget` → the same `_resolveDocumentListing` path in-WebView taps use, so a private or unlisted target still opens. A footnote states the asymmetry the graph can't hide: backlinks reflect each document's *current* version, which on a gated public document isn't what readers are served.
* **[lib/screens/changes_screen.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/screens/changes_screen.dart)** — pushed **Recent changes** feed (`GET /admin/documents?order=updated`), reached from Operate › Documents.
  * The same `DocFeedCard` plates as every other list, with one addition no other list has a use for: a `ChangeKindBadge` + relative timestamp above each card, because "this moved" is only actionable once the operator knows whether the *bytes* changed or only the *filing* did. A segmented window selector (24 hours / 7 days / 30 days / All) drives `updated_since`; switching it restarts the walk. Cursor-paginated via an explicit **Load more** (not infinite scroll — a change feed has a natural "that's enough" point that a browse list doesn't), and a failure *after* rows are on screen keeps the rows and appends the error rather than replacing a partial answer with an error tile.
  * Two pieces of copy are load-bearing and **always visible — outside every branch of the render, including the empty and error states**: the honey note that reclassification never writes a version (the reason the screen exists at all — without it the feed reads as a redundant re-sort of the Library), and a footnote about the migration-0017 backfill, which took `updated_at` from the current version's write time for pre-migration rows, so an *older* retag reads as a content change until the document is next touched. The empty result is where the second one matters most: an operator who picks a short window, sees "nothing changed", and knows they retagged something last month needs to be told why it isn't here.
* **[lib/screens/review_queue_screen.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/screens/review_queue_screen.dart)** — pushed **Review queue**, reached from Operate › Documents. Every public document whose head the publication gate is withholding from readers.
  * The same `DocFeedCard` plates as every other list, with a meta line above each carrying a honey `v4 → v8` mono pill and "N versions ahead · written &lt;rel&gt;". The card's own `NotLiveBadge` is redundant here (every row has one by definition) but is deliberately **not** special-cased away: it is the marker the operator followed in from the Library, and suppressing it only on this screen would break that thread.
  * **Reads its own query rather than filtering `documentsListProvider`.** The shared list is paginated and the app holds only the pages somebody happened to scroll, so filtering it would make the queue a function of scroll depth — worse than no queue, because it would read "all caught up" while holding work back. Since 2.2.0 the server answers the question directly (`?visibility=public&publication=pending`); the client filter still runs over the result, for the never-promoted reason in `lib/core/review.dart`.
  * Two pieces of copy are load-bearing and **always visible, outside every branch**: the standing note that agents can write to a public document but cannot publish it (without it a list of documents that look healthy everywhere else has no visible reason to exist), and a footnote stating the two things the queue cannot say for itself — **there is nothing to reject** (leaving a document here *is* declining, and keeps readers where they are), and a public document that has never been published isn't gated at all and so never appears. The empty state is where the second matters most.
  * A truncated walk still gets its own red warning tile (unreachable in practice since 2.2.0, kept because silent truncation is the one failure this screen must never have); a failure *after* rows have landed keeps the rows and appends the error rather than replacing a partial answer.
* **[lib/screens/review_screen.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/screens/review_screen.dart)** — the pushed **two-pane review** surface: what readers are served today against what they would get if the operator approved.
  * **Two `WebViewController`s, not one driven back and forth**, swapped by an `IndexedStack`. The controller owns the underlying platform WebView (`WebViewWidget` only displays it) and `IndexedStack` lays out every child unlike `Offstage`, so both sides load concurrently on open and the hidden one stays alive and scrolled. That costs a second platform view for one pushed route and buys three things a single-WebView switcher cannot have: **each pane keeps its scroll offset** (the point of the switcher is flipping between the same passage in two versions — a shared WebView lands back at the top every time, which makes the interaction useless past the first screenful), **a flip costs nothing** (no request, no `loadHtmlString`, no re-layout of untrusted HTML), and **failures stay local** (a version whose bytes are gone leaves its own pane showing the error while the other still renders).
  * **Both panes read `/d/:id/v/:n/raw`**, the live side included — see the `ReviewSide` note under `lib/core/review.dart` for why the symmetric route matters.
  * **Nothing here touches the offline body cache.** `saveCachedHtml` evicts every other cached version of a document before writing, so caching either pane would evict the Reader's body *and* the two panes would evict each other on every load. Strictly read-only against the cache — the one screen in the app that is.
  * Identical security posture to the Reader — `JavaScriptMode.disabled` plus the injected strict CSP — since this is where an operator decides to put agent-authored bytes in front of the public. Each pane carries **its own** `_expectSyntheticBaseLoad` flag (the macOS WKWebView synthetic-base-load quirk): one shared flag would let whichever load finished first disarm the other's. Link navigation is **refused** in both panes (following one would replace the pane and destroy the comparison) with a toast saying so; the Reader owns link traversal, reachable from the chrome's article button.
  * **The approve action publishes the version it rendered, by number** — the one the Latest pane pinned on open, never "whatever is current now". If an agent writes v9 mid-review, approving still publishes v8, and a honey banner (fed by `resolveCurrentVersion` off the pane responses, guarded to versions strictly *ahead*) names v9 so the operator can come back for it. Publishing "the latest" would put unreviewed bytes on the open web — the exact failure the gate was added to prevent. The button is disabled until the Latest pane has actually rendered, since approving a pane that failed to load is approving something unread. A `SourceComparison.identical` note appears when the two versions share a source hash. On success it resolves the row through `reviewQueueProvider` and pops.
* **[lib/screens/orphans_screen.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/screens/orphans_screen.dart)** — pushed **Orphans** worklist (`GET /admin/links/orphans`), reached from Operate's link-graph sheet.
  * Live documents no live document links to, rendered as the same `DocFeedCard`s as every other list, with pull-to-refresh. Reads its own endpoint rather than filtering `documentsListProvider` — orphanhood is a property of the link graph, not of any field on a listing row, so there is nothing to compute client-side. Uncursored, matching the contract. Two pieces of copy are load-bearing and deliberately **always visible**: the honey note that the graph must be rebuilt first (before that, every pre-backfill document reads as an orphan and the client cannot tell the difference), and the footnote that an orphan is a librarian's signal rather than a defect — a document only ever shared by URL is a perfectly good one.
* **[lib/screens/collections_screen.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/screens/collections_screen.dart)** — pushed **Collections** tag browser.
  * Lists every tag (counts computed client-side from loaded docs, matching the Library carousel); each row opens that tag's collection via `DocumentListScreen`.
* **[lib/screens/document_list_screen.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/screens/document_list_screen.dart)** — pushed generic document list.
  * Renders documents as text-forward `DocFeedCard`s. Powers both a single tag's collection (`tag` set) and "Recently plated → See all" (newest-first), reading and filtering the shared `documentsListProvider` client-side. Exposes the static `DocumentListScreen.openForTag(context, tag)` — the shared tag-tap navigation used by cards, Collections, and the Reader.
* **[lib/screens/settings_screen.dart](file:///Users/kyle/Repos/slopcafe_ui/lib/screens/settings_screen.dart)** — pushed Settings ("Connection") screen.
  * Configure + double-probe test the Base URL and Operator Token (`GET /healthz` then `GET /admin/agents`), save (optional `onSaved` callback, used by the first-launch gate), and clear secure storage. Shows the unauthorized banner.

### 6. Shared Widgets
* **[lib/widgets/](file:///Users/kyle/Repos/slopcafe_ui/lib/widgets/)** — Cortado primitives composed by every screen: `pill.dart` (`Pill`/`VisBadge`/`DeprecatedBadge` — the honey-toned lifecycle pill, rendered only for `status == 'deprecated'` — /`TagChip` — the tinted, tappable tag chip), `doc_feed_card.dart` (`DocFeedCard` — the text-forward "plate" card shared by Library / Collections / document lists; deprecated docs carry the `DeprecatedBadge` beside the visibility marker — plus `NotLiveBadge`, the publication-gate marker, rendered only where `hasUnpublishedWork` holds and placed beside the version number it qualifies rather than up in the status corner; deliberately the same honey `Pill` as `DeprecatedBadge` so the two read as siblings where they land together, and suppressed on a revoked document, which serves nobody), `app_button.dart` (`AppButton`/`AppIconButton`), `press_card.dart` (`PressCard` — now with click cursor + subtle hover lift on pointer devices — /`Tappable` — the cursor-aware `GestureDetector` drop-in used by every plain tap target — /`RiseIn`), `stat.dart` (`MiniStat`/`OpStat`), `section_header.dart` (`SectionHeader`/`MetaDot`/`Eyebrow`/`BackHeader` — the pushed-screen back+title header), `sheets.dart` (`AppSheet`/`SecretSheet`/`showConfirmSheet`/`showDeprecateSheet` — the deprecate flow's sheet: explainer + optional `superseded_by` field, returns the trimmed target or null on cancel; the caller owns the POST — /`CopyField`/`SheetActionRow`; `showAppSheet` caps every modal sheet at `AppLayout.sheetMax` on wide windows), `toast.dart` (compact fixed width on wide windows), `cafe_logo.dart`, and — for the link graph — `LinkStateBadge` in `pill.dart` (a derived badge in the `VisBadge`/`DeprecatedBadge` idiom, where the tone carries the whole triage: green `LIVE`, honey `REDIRECTED` (a caution about how the link resolves, sibling to `DEPRECATED`), red for the three dead states, and **neutral** `UNKNOWN` — an unrecognised state is not evidence of link rot) plus `slug_repair_sheet.dart` (`showSlugRepairSheet` — set/clear a retired slug's redirect, or release the name). That sheet **owns its own writes**, unlike `showDeprecateSheet`, which returns a decision for the caller to act on: it offers three mutations across two endpoints and is reached from two unrelated places (a broken outbound link in the Reader's report, and Operate's link-graph sheet), so handing the POSTs back would duplicate the error mapping at both sites. It maps the two codes these routes own — `not_found` → "that name isn't retired" (on a slug route it never means "no such document") and `bad_target` → "that target isn't a live document" — and confirms the release destructively, because releasing returns the name to the pool where a later publish may claim it, silently sending every link still addressing it to a different document. It always shows the slug field even when pre-filled: the contract exposes **no way to browse retired slugs**, so typing one is a first-class way in rather than a fallback. Status-surfacing render coverage lives in [test/status_lifecycle_test.dart](file:///Users/kyle/Repos/slopcafe_ui/test/status_lifecycle_test.dart); the `NotLiveBadge`'s render contract (marked only on proven divergence, never on a private or fully-published row) lives in [test/publication_gate_test.dart](file:///Users/kyle/Repos/slopcafe_ui/test/publication_gate_test.dart). Also in `pill.dart`, for the change feed: **`ChangeKindBadge`** — the third derived badge in the `VisBadge`/`DeprecatedBadge`/`LinkStateBadge` idiom, taking a whole `DocumentListing` and reading `changeKind` off it. Red for a revoke (a kill, as everywhere else), **clay** for `RECLASSIFIED` — the accent, because a change that left no version behind is the one thing this feed reports that nothing else in the app can — and neutral for `REWRITTEN`, which every version number already reports. `ChangeKind.unknown` renders **nothing** at all (with a static `showsFor(doc)` so callers can drop the adjacent spacer): unlike `LinkStateBadge.unknown`, which is a real server-sent state a newer backend added and therefore worth showing, this `unknown` means the row carries no `current_version_at` to compare — an "UNKNOWN" pill would dress the absence of information up as a finding. Covered by [test/change_feed_test.dart](file:///Users/kyle/Repos/slopcafe_ui/test/change_feed_test.dart).

---

### 7. Launcher icons (`assets/icon/`, `tool/gen_*_icon*.dart`)

The source art is **[assets/icon/app_icon.png](file:///Users/kyle/Repos/slopcafe_ui/assets/icon/app_icon.png)** — a 1254px full-bleed design (the cup on a flat `#65381D` field). Every platform icon derives from it, but the two desktop/mobile platforms need **opposite** treatments, and that asymmetry is the whole reason there are two generator scripts:

* **Android masks for you.** The launcher applies its own shape, so the icon must bleed edge-to-edge and keep the subject inside the adaptive safe zone. **[tool/gen_adaptive_foreground.dart](file:///Users/kyle/Repos/slopcafe_ui/tool/gen_adaptive_foreground.dart)** lifts the cup out of the source and re-seats it, pre-scaled, into that zone; `flutter_launcher_icons` (configured in `pubspec.yaml`) then emits the mipmaps.
* **macOS masks nothing.** Whatever shape the PNG is, that is the shape in the Dock — a full-bleed square lands as a hard-cornered square wedged between rounded neighbours. **[tool/gen_macos_icon.dart](file:///Users/kyle/Repos/slopcafe_ui/tool/gen_macos_icon.dart)** therefore seats the art in Apple's icon grid itself and writes `macos/Runner/Assets.xcassets/AppIcon.appiconset/` **directly**; `flutter_launcher_icons` is set to `macos: generate: false` so it keeps its hands off.

Regenerate after changing the source art:

```
dart run tool/gen_adaptive_foreground.dart   # source art -> Android foreground
dart run flutter_launcher_icons              # foreground  -> Android mipmaps
dart run tool/gen_macos_icon.dart            # source art -> the macOS appiconset
```

**Why the macOS generator exists rather than a pre-shaped source image.** `flutter_launcher_icons` can only downscale a single master, and Apple's grid is not one ratio at every size. Its constants were measured off the system icons in `/System/Applications` (Podcasts / Reminders / App Store / Music / Notes all share one exact silhouette), not guessed:

* **Body ratio changes with the slot.** 128px and up put an 824px body on a 1024px canvas (~80.5%), but the 16/32/64px canvases use **7/8** — a proportional margin would burn pixels an icon that small cannot spare.
* **The margin is a whole number of pixels**, so the straight edges land on the pixel grid instead of straddling it. Visible at 128px, where the ratio wants 12.5 and Apple ships 12 — a **104px** body, not 103.
* **The corner is a continuous (G2) squircle, not a circular arc**: a cubic Bézier easing off the straight edge, a circular arc through the middle of the turn, then the mirrored Bézier. `kSmoothing` trades arc for Bézier (0 degenerates to an ordinary rounded rect). Fitted by rasterising candidates and diffing binary silhouettes against Apple's icons — the error floor is a plateau over **0.53..0.61** at 0.26px RMS on a 206px body, and the same sweep independently recovered the template radius (**185.4/824**).
* **The drop shadow is part of the art**, since macOS adds none: ~30% black, Gaussian sigma 12, pushed 11px down at 1024px, fitted to the alpha falloff around the system icons.

Each slot is rendered at its native size rather than downscaled from a master — at 16px the squircle is only ~3px of curve, and resampling that out of a 1024px alpha channel smears a rim that native supersampling resolves cleanly. Verified end-to-end against Apple's own icons: **exact** at 16 and 32px, 0.196px RMS at 128, 0.296px at 256.

---

## 🔗 Inbound web links (Android App Links)

A tap on `https://slopcafe.com/d/<public_id>` or `/s/<slug>` anywhere on an
Android device opens that document in the Reader instead of a browser. The
operator runbook — including the `assetlinks.json` that must be published and
the `adb` verification commands — is
**[docs/deep-links.md](file:///Users/kyle/Repos/slopcafe_ui/docs/deep-links.md)**.
The four things worth knowing here:

* **The app half is complete; the web-server half is not, and cannot be done
  from this repo.** `android:autoVerify="true"` is honoured only if
  `https://<host>/.well-known/assetlinks.json` names this package and this
  build's signing certificate. Until that file is live the links still work —
  they just open in a browser, exactly as before. Note the current fingerprint
  to publish is a *debug* keystore's, because `build.gradle.kts` still signs
  release with the debug config (standing `TODO` there); docs/deep-links.md
  spells out why that is fine to get working and wrong to leave live.
* **The claim is deliberately narrow**: `/d/` and `/s/` path prefixes only, one
  host, no subdomains. Claiming the whole origin would swallow the rendered
  site, `/openapi.json` and the root into an operator console that has nothing
  to show for them. `http` *is* claimed alongside `https`, because letting an
  unencrypted link escape to a browser is strictly worse than opening it here —
  the app then talks to its own configured `https` Base URL regardless.
* **macOS is excluded, by design and by circumstance.** A `https://` link only
  reaches a Mac app through Universal Links, which needs an `Associated Domains`
  entitlement, which needs a Developer Program team — and this app is ad-hoc
  signed with no team. Registering a custom URL scheme instead would hijack the
  operator's browser links system-wide, which is the opposite of what a desktop
  operator wants: on a Mac a `slopcafe.com` link belongs in the browser. The
  macOS bundle therefore declares no `CFBundleURLTypes` and no
  `associated-domains`, and `deepLinksSupported` says so in Dart as well rather
  than letting the exclusion be an accident of configuration.
* **The domain is a build-time fact, not the runtime Base URL.** Android decides
  whether the app is even offered the tap from the manifest, long before any
  Dart runs. So the host is written twice — `manifestPlaceholders["deepLinkHost"]`
  in `android/app/build.gradle.kts` and `kDeepLinkHost` in
  `lib/core/deep_link.dart` — and `test/deep_link_test.dart` reads the Gradle
  literal back to pin the pair. Change either and the test names the other. The
  Settings Base URL remains an independent knob: it can point at staging while
  the build claims production.

---

## 🔗 External Integration Dependencies

* **Canonical OpenAPI spec (machine source of truth)**:
  * Served live at **[https://slopcafe.com/openapi.json](https://slopcafe.com/openapi.json)** — OpenAPI **3.1.0**, `info.version` tracked in `tool/CONTRACT_VERSION` (currently **2.2.0**). The app's models + `ErrorCode` are generated from a pinned copy at `tool/openapi.json`. See the **API layer** section below.
* **Canonical HTTP API Reference (human reference)**:
  * **Document Slug**: `slopcafe-http-api`
  * **URL**: [https://slopcafe.com/s/slopcafe-http-api](https://slopcafe.com/s/slopcafe-http-api)
  * Consult this live document for behavioral/narrative contract the spec can't encode.

---

## 🔌 API layer (generated from the OpenAPI contract)

The app's API models and error codes are **generated from the backend's canonical
OpenAPI 3.1 spec** rather than hand-maintained, so they stay in sync with the
backend automatically (this was "Phase 3 — consumer adoption" of the backend's
code-first API-contract effort).

* **Pinned spec**: `tool/openapi.json` — an exact copy of what prod serves at
  `https://slopcafe.com/openapi.json`. Generating from the pin (not the live URL)
  keeps builds reproducible. `tool/CONTRACT_VERSION` records the `info.version`
  generated against. **Re-pin + regenerate whenever the backend surface
  changes** — not only on an `info.version` bump. (The operator authoring
  endpoints + `created_by_kind` landed under an unchanged `info.version` of
  `1.0.0`, so `CONTRACT_VERSION` stayed `1.0.0`; only update it if `info.version`
  actually changes. The semantic-search surface — the `/admin/documents/search`
  `mode` param, the new `POST /admin/vectors/backfill` path + `BackfillResponse`
  schema, and the `"semantic"` `matched_field` value — landed the same way: pin
  re-pulled, `CONTRACT_VERSION` left at `1.0.0`.) The inverse also happens: the
  backend later **renumbered** `info.version` `1.0.0` → **`0.4.0`** with **no
  surface change** — pre-launch they switched off the placeholder `1.0.0` to
  `0.x.y` nomenclature, and going forward any change (breaking or not) bumps the
  minor/tweak digits for consumer visibility; the `1.0.0` bump is reserved for
  actual launch. That re-pin touched only the version numeral (the pinned spec is
  otherwise byte-identical), so the only generated diff was the contract string in
  the `models.dart` / `error_code.dart` headers. The first **real** surface change
  under that scheme then bumped `0.4.0` → **`0.5.0`**: the `AgentKey` item (in
  `ListAgentKeysResponse.keys`) gained two **required** fields — `expires_at`
  (nullable ISO; null = never expires) and `expired` (server-computed bool) — so a
  key is now "active" only when neither revoked nor expired. The generator emits
  these automatically (`expires_at` → nullable `DateTime`; `expired` → `bool`);
  the one hand-edit was the new `isActive` convenience getter on `AgentKey`
  (`tool/generate_api.dart` `_extraMembers`). The companion agent counts
  (`active_keys`/`total_keys` on `AgentListing`) were already in the spec; their
  contract definition now also excludes expired keys (no schema diff). The next
  re-pin jumped **0.5.0 → 0.10.0** in one hop, absorbing four backend bumps:
  the operator console (HTML-only paths, invisible to codegen), the **lifecycle
  status axis** (required `status` + nullable `superseded_by` on
  `DocumentListing`/`SearchHit`/`ReadSourceResponse`; the new
  `POST /admin/documents/{id}/status` mutator; `status` filter param on
  list/search; error codes `bad_status`/`invalid_status`), **context packs**
  (the `Pack*` schemas behind search's `include_bodies` knob — note
  `SearchOrPackResponse` is a pure `anyOf` union with no `properties`, which the
  generator correctly skips: a caller knows which arm it gets from the
  `include_bodies` param it sent; `load_context_pack` is MCP-only, no HTTP
  twin), and 0.10.0's auth widening on the five content-read endpoints (prose
  only, no shape change). Generator config change: `DocumentStatus` joined
  `_stringEnumSchemas` (it's a `$ref`'d string enum — without the entry it
  emits as `dynamic`). The required `status` field also meant updating the one
  hand-built `DocumentListing(...)` placeholder (`reader_screen.dart`) and the
  `tool/smoke_test.dart` fixtures (which had silently drifted — they were
  missing `created_by_kind` since the authoring update and failing 2 checks
  before this re-pin). A stale offline `documents_list.json` missing `status`
  fails the parse and falls back to an online fetch, by design. The next re-pin
  is the first **MAJOR** one: **0.10.0 → 2.0.0**, and unlike every hop above it
  is genuinely breaking. Three things earn the major digit. (1)
  `DocumentListing`/`SearchHit`/`PackDocument` each gained **five new required
  fields** — `updated_at`, `current_version_at`, `published_ver`,
  `current_source_sha256`, `published_source_sha256` — of which only
  `updated_at` is **non-nullable**, so it is the one that breaks things: every
  hand-built `DocumentListing` in the repo stops compiling until it supplies one
  (the Reader's not-found placeholder, the `test/status_lifecycle_test.dart`
  fixture, the `tool/smoke_test.dart` revoked-doc JSON), and — as with `status`
  before it — a stale offline `documents_list.json` written by an older build
  fails the parse and falls back to an online fetch. (2) `version_not_found`
  **split out of** `not_found`: a version-targeting call (promote, restore) that
  used to be indistinguishable from "no such document" now has its own code, so
  error arms that keyed on `not_found` see a code they did not before
  (`slug_locked` and `too_deep` arrived in the same bump — see the *unconsumed*
  note below for why neither needs handling here). (3) The **publication gate**
  itself: `documents.published_ver` and the `POST /admin/documents/{id}/promote`
  mutator change what the HTML byte path serves, which is a behavioural break no
  schema diff can express — it gets its own bullet below. The re-pin also
  brought `ListVersionsResponse`/`VersionListing`, `PromoteResponse`,
  `RestoreResponse`, `ReadTextResponse` and the link-graph schemas; no generator
  config change was needed (`SourceFormat` is a `$ref`'d string enum reached
  through `VersionListing`, and the existing `_stringEnumSchemas` handling
  covered it). The next re-pin is **2.0.0 → 2.2.0**, the first hop under the
  restored strict semver (2.0.0 froze 2026-07-25), and it is **MINOR — additive
  only, nothing breaking**. Three things landed:
  * **The review-queue filters**: `visibility` (`public`|`private`, migration
    0011) and `publication` (`pending`|`current`, migration 0018) on
    `GET /admin/documents`. Both are **query params**, so the generator emits
    nothing for them and the whole adoption is hand-written consumer code — see
    the review-queue bullet below for what they mean and the one place the
    server's predicate is wider than the client's question.
  * **`unchanged` (required `bool`) on `WriteResponse` AND `RestoreResponse`.**
    This is the part worth reading twice, because it is easy to file under
    "additive" and miss: it reports that **the backend now collapses no-op
    writes**. A re-write whose source bytes, title, description, tags and slug
    all match what the document already holds stores nothing, and `version` then
    names the version that was *already there* rather than a new one. The field
    is required and non-nullable, so the generated classes gained a required
    constructor arg — harmless here only because nothing in the repo hand-builds
    either response.
    * Consequence the app has **not** yet acted on: `restoreVersion` reports
      "Restored as v\<n\>" off `RestoreResponse.version`. Restoring a version
      whose source is byte-identical to the current head now collapses, so that
      toast can name the existing head and claim a restore that did not happen.
      `RestoreResponse.unchanged` is exactly the signal needed to say so
      instead. Authoring is unaffected (`POST /admin/documents` is a new
      document, never a no-op, per the field's own description).
    * It also makes the review queue *cleaner* for free: an agent re-pushing an
      unchanged file on a timer no longer inflates the version history, so it no
      longer manufactures queue entries whose two versions differ only by
      number. `SourceComparison.identical` should now be rare rather than
      routine.
  * No path, error-code or other schema changes: 33 discriminants before and
    after, no added/removed paths, and `WriteResponse`/`RestoreResponse` are the
    only two schemas whose bytes moved at all (verified by normalized JSON diff
    against the pinned copy rather than taken on trust — the hand-off note for
    this bump described it as "no response shape changed", which the bytes
    contradict).
  ```sh
  curl -s https://slopcafe.com/openapi.json -o tool/openapi.json
  # update tool/CONTRACT_VERSION if info.version changed
  dart run tool/generate_api.dart        # spec -> lib/api/models.dart + error_code.dart
  dart run build_runner build            # -> models.freezed.dart + models.g.dart
  ```
* **`2.0.0` FROZE on 2026-07-25**, and that changes the re-pin cadence. Until
  then it was an **open major**: the backend cut it on a dedicated
  breaking-change branch where breaks *accumulated* under one version string, so
  `2.0.0` denoted a moving contract — the same version could describe different
  wire behaviour a week apart, `^2.0.0` promised nothing, and the only safe
  practice was to pin the **bytes** and diff them (which is why this repo pins
  `tool/openapi.json` rather than trusting the numeral). With the window closed,
  strict semver is back in force: MAJOR for a break, MINOR for additive
  backward-compatible surface, PATCH for docs-only. A caret range is now
  meaningful, and an `info.version` bump is once again real evidence that
  something moved.
  * The freeze was verified against prod on the day: the live `openapi.json` was
    **byte-identical** to the pinned copy, so the window closed without any
    final change relative to what the app had already adopted — no re-pin, no
    regeneration, `CONTRACT_VERSION` stays `2.0.0`.
  * **The living reference has caught up.** `slopcafe-http-api` **doc v35** now
    reads *"`2.0.0` HAS LANDED AND IS FROZEN — a caret range is safe again"*,
    replacing the open-major paragraph that lagged in v34. Prose and numeral
    agree; nothing here is left to reconcile. (The doc still notes the version
    bump is "a human obligation, not test-enforced", so the two *can* drift
    again — the byte-diff below stays the arbiter, not the narrative.)
  * Version-string equality still isn't proof of currency in general — diff the
    spec bytes (`curl` + a normalized JSON compare) when you need certainty.
* **Generator**: [tool/generate_api.dart](file:///Users/kyle/Repos/slopcafe_ui/tool/generate_api.dart)
  — a dev-only, pure-Dart script (no Flutter imports) that walks
  `components.schemas` and emits `lib/api/models.dart` (freezed) + `lib/api/error_code.dart`.
  Its header documents **why a bespoke emitter** rather than an off-the-shelf
  generator: `swagger_dart_code_generator` doesn't support 3.1 and emits
  non-nullable fields for `anyOf`-null (a runtime crash on revoked docs);
  `swagger_parser`/`swagger_to_dart` handle 3.1 nullability but force a Retrofit
  dep, `String` (not `DateTime`) timestamps, an enum (not `String`) `visibility`,
  and **still** require a hand-written 28-code error enum — net *more* churn. The
  bespoke emitter is ~thin: the real serialization/`copyWith`/equality is produced
  by the standard `freezed` + `json_serializable` + `build_runner` pipeline.
  Small app-specific config lives at the top of the script (string-enum schemas
  kept as `String`, `score` typed `double`, `_at` strings typed `DateTime`,
  inline-item name overrides `agents→AgentListing` / `keys→AgentKey`).
* **Generated outputs** (committed, **do not hand-edit**): `lib/api/models.dart`,
  `lib/api/error_code.dart`, and the `build_runner` products `models.freezed.dart`
  / `models.g.dart`. Hand-written glue: `lib/api/api_error.dart`; barrel: `lib/api/api.dart`.
* **Operator authoring (write) routes use hand-built request maps**:
  `POST /admin/documents` (author a new doc as the operator principal) and
  `PUT /admin/documents/{public_id}` (update → new version, optional `If-Match`)
  take request bodies the spec models **inline** (not in `components.schemas`),
  so the generator emits no request class — `DocumentsListNotifier.authorDocument`
  builds the JSON map directly (mirroring the existing `set{Visibility,Slug,Tags}`
  writes). Both return the generated `WriteResponse` (new `public_id`/`url`/
  `version` + the sanitizer report `stripped`/`will_not_render`). The compose UI
  lives in `lib/screens/compose_screen.dart`. (Only `POST` — author-new — is wired
  in the UI so far; `PUT`/edit is a future extension over the same `WriteResponse`.)
  2.0.0's `POST /admin/documents/{id}/promote` and `POST /admin/documents/{id}/restore`
  are the same shape of thing — a one-key `{"version": n}` body modelled inline —
  so `promoteVersion`/`restoreVersion` build their maps by hand too, while their
  *responses* (`PromoteResponse`/`RestoreResponse`) are generated.
* **Non-JSON routes stay hand-rolled** (by spec design): content-negotiated /
  raw-bytes / HTML reads (`/d/{id}`, `/d/{id}/raw`, `/d/{id}/text`, `/s/{slug}`,
  the version `/raw` reads) and `/mcp` are not JSON-modelled — the
  Reader's WebView + ETag conditional-GET cache and the 404/410 status-code
  handling there are intentionally **not** routed through `ErrorCode` (the bodies
  aren't the JSON envelope). Restore **left** this list in 2.0.0: it moved off the
  form-encoded `POST /d/{id}/restore` onto the operator JSON route, which returns a
  modelled `RestoreResponse` and a real `ErrorBody` on failure.
* **Publication gate (contract 2.0.0)** — the one behavioural break in the bump,
  and the reason `lib/core/publication.dart` exists. The backend now stores
  `documents.published_ver` alongside `current_ver`, and the HTML byte path
  (`GET /d/{id}/raw`, `GET /s/{slug}`) serves **exactly**:

  ```
  served = (visibility == "public" && published_ver != null) ? published_ver : current_ver
  ```

  There is **no principal term in that rule**. A public document serves its
  published version to *everyone* — anonymous visitor, agent key and this
  operator app alike — which is the whole point: any active agent key may
  overwrite any live document, so without the gate an agent write would be on the
  open web at the next render. A private document always serves `current_ver`, and
  so does a public document that has never been promoted (`published_ver == null`
  means the gate was never closed). **Only the HTML byte path moved.** Every
  credentialed / machine-readable surface — `/d/{id}/text`, `/d/{id}/source`,
  `/d/{id}/links`, the admin list, search, context packs — still reads
  `current_ver` and is unchanged, which is precisely why a listing row or a search
  snippet can describe bytes no reader has been served. Moving the pointer is
  **operator-only** (`POST /admin/documents/{id}/promote`); there is deliberately
  no agent-reachable twin.
  * **Header contract.** `ETag` names the **SERVED** version and is always
    present, canonically `"v7"` — but Cloudflare weakens it to `W/"v7"` whenever
    it gzips (routine for HTML), so a parser must accept the weak marker, the
    quotes, a bare `v7` and a bare `7`. `x-doc-current-version` is a **bare
    integer** naming the **CURRENT** version, it rides **304s as well as 200s**
    (a conditional request that validates still reports where the head is), and
    it is sent **only to a non-anonymous caller** — that staged unpublished work
    exists is exactly what the gate withholds from readers, so for an anonymous
    caller it is *absent*, not clamped. **The ETag fallback in
    `resolveCurrentVersion` is the correct answer, not a degraded guess**: the
    header is missing in exactly two situations — an anonymous caller, or a
    pre-2.0.0 server — and in both the gate is not in play at all, so the ETag
    *is* the current version. Never make the header mandatory; hardening this
    into an error path would break version resolution against every older
    deployment for no gain. (See also the `Accept: */*` interceptor under
    **Core Services** — without it Cloudflare strips the ETag and *both* numbers
    disappear.)
  * **Client surface.** `lib/core/publication.dart` (parsers + the
    `DocumentPublication` extension), `DocumentsListNotifier.fetchVersions` /
    `promoteVersion` / `restoreVersion`, the Reader's served-version banner +
    `Live v<n>` chip + real version-history sheet, Operate's Publish action, and
    the `NotLiveBadge` on every listing surface. Each is documented in its own
    map entry above.
  * **Review queue (the gate's worklist).** The gate answers "what is served"
    per document; the queue answers "what is the operator on the hook for"
    across the corpus, and reviewing answers "what would change". Its client
    surface is `lib/core/review.dart` (`ReviewSide`, `SourceComparison`, the
    `DocumentReview` extension and `reviewQueueFrom`),
    `lib/providers/review_provider.dart` (`ReviewQueueNotifier`),
    `ReviewQueueScreen`, `ReviewScreen`, and Operate's "Review queue" entry.
    Each is documented in its own map entry above.
    * **The corpus sweep is gone (contract 2.2.0).** `GET /admin/documents`
      gained `visibility` (`public`|`private`, migration 0011) and `publication`
      (`pending`|`current`, migration 0018), so the queue is now one filtered
      query — `?visibility=public&publication=pending&order=updated` — and the
      rows crossing the wire are proportional to the *queue*, not the corpus.
      Both compose with the existing `tag`/`slug`/`status`/`updated_since`
      filters and with the cursor. An unknown value for either is a plain
      **`400 bad_request`**, not a dedicated code and not a silent fallback to
      unfiltered, which is why the wire strings live in tested enums
      (`VisibilityFilter`/`PublicationFilter` in `lib/core/review.dart`)
      alongside `DocumentOrder`'s.
    * **`reviewQueueFrom` is still applied to the result, and that is
      deliberate.** The server's `pending` is `published_ver IS NOT
      current_ver`, and NULL is distinct from any version number, so a public
      document that was **never promoted** satisfies the server's predicate —
      while being *not gated at all*, since by the 2.0.0 serving rule it already
      serves its head to everyone. Rendering the server's answer raw would put
      such a document in the queue claiming readers are on something older when
      they are on the newest thing that exists. The spec's own wording is
      ambiguous about whether the backend special-cases this for public rows
      (`publication`'s description notes the never-published overlap only for
      *private* docs), and the client is correct under either reading, which is
      the point of keeping the check. `tool/smoke_test.dart` reports how many
      rows the client filter drops on every authenticated run, so live data
      answers the question continuously rather than the prose being trusted.
      **The division of labour: the server narrows the fetch, the client keeps
      the semantics.**
    * Two smaller filter properties, both asserted in the smoke test rather than
      assumed: **revoked documents match neither `publication` value** (revoke
      nulls both pointers), and no returned row is already fully published. The
      revoke property does *not* replace
      `DocumentReview.isAwaitingReview`'s own `!isRevoked` term — that term
      exists to keep an unrenderable document out of the queue wherever the rows
      came from, including a caller that never passed the filter.
    * **Reviewing is a read of the byte path, not of the admin surface.** The
      admin list and search still report `current_ver` by design, so they can
      describe bytes nobody has been served; the two panes deliberately go to
      `/d/:id/v/:n/raw` for both versions, because what is being approved is
      what a *reader* would receive.
    * The reviewed version is published **by number**, never as "the latest" —
      see the `ReviewScreen` entry for why that is the screen's core safety
      property rather than an implementation detail.
  * **This app is operator-only.** It authenticates with an operator token
    (`SecureStorageService.getOperatorToken`, validated in Settings against the
    operator-only `GET /admin/agents`), so promote is *always* available and
    there is no agent-key mode to branch on. Do not build an "if only an agent
    key, point at `/d/{id}/manage`" path — it is unreachable here.
* **Link graph & slug tombstones** — the wiki graph the backend keeps in sync
  with every write, and the repair actions for the link rot it reports. Six
  routes, all adopted: `GET /d/{id}/links` (a document's neighborhood —
  `backlinks` + `outbound`), `GET /admin/links/orphans`,
  `POST /admin/links/backfill`, and the three slug-tombstone mutators
  `POST`/`DELETE /admin/slugs/{slug}/redirect` + `DELETE /admin/slugs/{slug}`.
  Every response is generated (`DocumentLinksResponse`/`OutboundLink`,
  `OrphanDocumentsResponse`, `LinksBackfillResponse`, `SetSlugRedirectResponse`,
  `ClearSlugRedirectResponse`, `ReleaseSlugTombstoneResponse`); only the
  redirect *request* body is hand-built, being modelled inline.
  * **Late binding is the whole design.** Outbound targets are stored as the raw
    addressed name and resolved **per read**, so a link's `state` is a property
    of the request, not of the document — which is why nothing about the graph
    is cached anywhere in the app.
  * **Two asymmetries that look like bugs and aren't.** (1) The graph always
    reflects each document's **current** version while the HTML byte path may
    still serve an older *published* one, so a gated public document can render
    links the graph doesn't list and list links its rendered page doesn't carry
    — the Reader's links sheet says so in a footnote rather than pretending
    otherwise. (2) A link authored against a since-**renamed** slug is
    deliberately **not** counted as a backlink on the document it now reaches;
    it only gets there through the loud tombstone redirect, which is never
    followed implicitly, so it surfaces on the *source* document's outbound list
    as `redirected` — link rot to repair, not an edge in the graph.
  * **Pre-backfill corpora are indistinguishable from unlinked ones.** Documents
    published before the graph shipped have no rows until
    `POST /admin/links/backfill` runs, and the client cannot tell that apart
    from a genuinely unlinked document. Both surfaces that could mislead say so:
    the links sheet reports "nothing **recorded**", and the orphans screen
    carries a standing note that the graph must be rebuilt first.
  * **`bad_target` (422) and `not_found` (404) are the slug routes' own arms.**
    `not_found` on a slug route means *the name isn't retired* — never "no such
    document" — so both are mapped to their own messages rather than falling
    through to a generic failure. `slug_locked` remains unhandled and correctly
    so: it is an agent-key restriction an operator token is never refused for.
  * **Client surface.** `lib/core/links.dart` (the state/kind vocabulary and the
    `canRepairSlug` rule), `lib/providers/links_provider.dart`
    (`LinkGraphService`), the Reader's links sheet, `OrphansScreen`, Operate's
    "Link graph" entry, `LinkStateBadge`, and `showSlugRepairSheet`. Each is
    documented in its own map entry above.
* **Corpus change feed (`order=updated` + `updated_since`, migration 0017)** —
  adopted on the **operator list** (`GET /admin/documents`). The same two params
  also exist on the agent-reachable twin `GET /d`, which this app has never
  called at all (it is operator-only, so the admin route is always available and
  strictly wider) — that surface stays unconsumed for the same reason the rest
  of `GET /d` is. Both are **query params**,
  so the generator emits nothing for them and the whole feature is hand-written
  consumer code over the existing `ListDocumentsResponse`; the `updated_at` /
  `current_version_at` fields it sorts and reasons on were already on
  `DocumentListing`.
  * **Why it exists.** A classification edit — retag, rename, visibility flip,
    status change, **promote** — stamps `updated_at` and writes **no version**.
    Every version-based check in the app is therefore blind to it, and the
    default `created` ordering can never find it either: a document authored a
    year ago and retagged this morning sits a year deep in that list. This is
    the only surface that reports those changes.
  * **A cursor carries the ordering that minted it.** Replaying one under the
    other ordering is a hard `400 bad_cursor`, never a silent re-sort. So
    `order` rides along with `cursor` on **every** page, and changing the
    ordering or the window drops the cursor and restarts the walk. The app goes
    one further and gives the feed its own state object
    (`ChangeFeedState`, not a mode on `DocumentsListState`) so the two walks
    have no cursor field in common and the error is unreachable by construction.
  * **The window instant is pinned per walk, not recomputed per page.**
    `updated_since` is resolved once in `reload()` and stored on the state
    (`windowSince`), then replayed verbatim on every page. Recomputing `now -
    24h` on page 2 would raise the window's *lower* bound by however long the
    operator took to tap Load more — and since the feed is ordered
    most-recently-changed **first**, a walk moves *towards* that bound as it
    pages, so the rising floor silently drops exactly the rows the later pages
    exist to deliver. The server does not encode filters in the cursor, so
    nothing rejects it: the walk just ends short of its own stated window.
  * **Requests carry a generation token.** `reload()` is deliberately **not**
    gated on `isLoading` the way `DocumentsListNotifier.loadNextPage` is —
    tapping a different window mid-fetch must switch the feed, not be swallowed.
    That only works if the superseded request can no longer commit, so every
    fetch captures a monotonic id and re-checks it before each `state =` (and
    captures the rows/cursor/window it needs *before* its await, never reading
    them back afterwards). Without it a slow page from an abandoned walk lands
    later and appends its rows onto the new walk's list, restores the old
    `window`, and leaves a cursor minted under one window paired with another
    window's `updated_since`.
  * **`updated_since` is inclusive on purpose** (`updated_at >= value`) — a
    resuming consumer re-sees the boundary row rather than risking a skip at a
    shared millisecond — so the client de-duplicates on `public_id` across
    pages rather than assuming disjoint ones.
  * **Revoked documents appear** (revoke is a change), which is why the row
    badge checks `revoked_at` before anything else.
  * **The migration backfilled `updated_at` from the current version's write
    time**, so a classification change made *before* migration 0017 under-reports
    and reads as a content change until the document is next touched. The screen
    carries a standing footnote saying so, in the same spirit as the orphans
    screen's "rebuild the graph first" note.
  * **Client surface.** `lib/core/changes.dart` (`DocumentOrder`, `ChangeKind` +
    the `DocumentChange` derivation, `ChangeWindow`),
    `lib/providers/changes_provider.dart` (`ChangeFeedNotifier`),
    `lib/screens/changes_screen.dart`, Operate's "Recent changes" entry, and
    `ChangeKindBadge`. Each is documented in its own map entry above.
* **Semantic search (`mode` + backfill)**: the backend added a hybrid
  keyword+vector search. `GET /admin/documents/search` gained a `mode` query
  param — `hybrid` (default; BM25 ⊕ Vectorize/Workers-AI fused via Reciprocal
  Rank Fusion) · `keyword` (FTS only) · `semantic` (vector only). It's a query
  param (not a `components.schemas` shape), so the generator emits nothing for
  it — `documentSearchProvider` threads it by hand from the `SearchMode` enum,
  surfaced by the Search tab's segmented selector. `SearchHit.matched_field`
  gained `"semantic"` (un-bracketed snippet ⇒ concept match); kept as `String`,
  so no codegen change. Behavioural notes from the contract: search is **not
  paginated** (50-cap, raise `limit` up to 200 to widen), the query embed is
  **best-effort** (a brief embedding outage degrades `hybrid`/`semantic` to the
  keyword leg rather than erroring; the only hard failure is `422 bad_query`
  when no leg can run), and `score` scale **differs by mode and is only
  comparable within one result set** (already handled — the relevance bar
  normalizes against the in-set max). **Vector backfill:**
  `DocumentsListNotifier.backfillVectors(VectorBackfillMode)` walks
  `POST /admin/vectors/backfill` (operator; `mode=missing|rebuild`, cursor-
  paginated → the generated `BackfillResponse`) to completion, summing pages
  into a `BackfillSummary`. Docs published before semantic search shipped have
  no vectors and stay invisible to the semantic/hybrid legs until embedded.
  Surfaced as the Operate › Documents **"Search index"** entry (`_BackfillSheet`)
  with two clearly-delineated modes — **index-new** (`missing`; cheap,
  incremental) and **rebuild** (re-embed the whole corpus; flagged as the
  costly option) — **each gated by its own confirmation sheet** (rebuild gets
  the danger CTA). `BackfillSummary.suspectPartialFailure` (`vectors ≪ embedded`)
  drives a "some embeds failed, re-run" warning toast.
* **Generated but deliberately UNCONSUMED after the 2.0.0 adoption.** These are
  live contract surface the app does not touch — scoped out, not overlooked, so
  don't read the generated classes as a half-finished feature:
  * **`updated_since` on search** — the change window composes with `q` on
    `GET /admin/documents/search` too (search takes the filter but **not** `order`:
    relevance rank is its ordering, which is also why it has no cursor). Left
    unconsumed deliberately — the Search tab answers "find me this", and bolting a
    recency window onto it would duplicate the change feed's job in a surface with
    no room to explain it. The list half of the feed **is** adopted; see its own
    bullet above.
  * **Context packs** — still unconsumed, as since 0.10.0: the `Pack*` envelope
    behind search's `include_bodies`, plus `GET /d/pack`. 2.0.0's `too_deep` error
    code belongs to that surface (pack root-following depth), which is why nothing
    handles it here. `ReadTextResponse` (`GET /d/{id}/text`) is likewise generated
    and unused — the Reader renders HTML bytes, never the text projection.
  * **`slug_locked` (403)** is an **agent-key** restriction on renaming a public
    document's slug. An operator token is never refused for it, and the app's only
    auth-driven handling keys on 401, so there is nothing to misclassify. No
    speculative handling was added; map the code only where an error arm already
    exists.
* **Smoke test**: [tool/smoke_test.dart](file:///Users/kyle/Repos/slopcafe_ui/tool/smoke_test.dart)
  validates the generated layer against the **live** backend (public `/healthz`
  + an unauthenticated 401 → `ErrorCode.unauthorized`), a revoked-doc fixture
  (the 3.1-nullable risk) and, since 2.0.0, a publication-gate fixture (a public
  row with `current_ver: 8` / `published_ver: 4` beside a private row carrying
  the same two numbers, asserting `servedVer` / `hasUnpublishedWork` both ways),
  and a link-graph fixture (a five-state outbound report, asserting the broken
  set and the `canRepairSlug` rule in both directions). Run
  `dart run tool/smoke_test.dart`; set `OPERATOR_TOKEN=…` to also exercise the
  authenticated list → search flow plus the **read-only** halves of the link
  graph (`/admin/links/orphans`, and `/d/:id/links` against the first live
  document — which also fails loudly if prod ever sends an outbound `state`
  this build resolves to `LinkState.unknown`, the one way that forward-compat
  sentinel could otherwise hide a contract change), and the **change feed**
  (a JSON-layer fixture for the `updated_at` vs `current_version_at` derivation
  incl. sub-second skew in the "wrong" direction; plus, with a token, a live
  `?order=updated` page asserted to arrive most-recently-changed-first and —
  the valuable one — that page's cursor **replayed under the default ordering
  and asserted to come back `400 bad_cursor`**. That check is read-only, costs
  nothing, and is the only way this repo would notice if the trap its whole
  pagination design guards against were ever disarmed server-side). The
  backfill sweep and the three slug mutators are deliberately **not**
  exercised: this script runs against prod and every one of them writes.
  Since **2.2.0** the authenticated leg also exercises the **review-queue
  filter** (`?visibility=public&publication=pending`), asserting the filter's own
  guarantees — every row public, no revoked row, none already fully published —
  and that an unknown `publication` value is **rejected** with `400 bad_request`
  rather than silently ignored (a degrade to unfiltered would quietly turn the
  queue back into the whole corpus, and nothing else would notice). It also
  settles against live data the one thing the spec's prose leaves ambiguous —
  whether the server's `pending` admits **never-promoted** public documents — and
  it does so **discriminatingly**: it fetches every public document to establish
  whether the deciding case exists at all before drawing any conclusion, because
  "the client filter dropped nothing" is equally consistent with the server
  excluding such rows and with the corpus containing none. It prints one of three
  verdicts (`ANSWERED: DOES admit` / `ANSWERED: excludes` / `INCONCLUSIVE: no
  never-promoted public document exists`) rather than a bare count that a clean
  run could be mistaken for evidence. The first run against prod (2026-07-27,
  4 pending of 8 public) came back **INCONCLUSIVE** — the corpus had no
  never-promoted public document to test with. Deliberately a report rather than
  an assertion, since both readings are contract-legal; the hard assertion beside
  it is the invariant that holds either way — no row lacking a `published_ver`
  ever reaches the queue.

---

## 🔄 Self-Healing Documentation Protocol (CRITICAL)

> [!IMPORTANT]
> **Maintaining Documentation Integrity:**
> Whenever you modify this repository—whether introducing new features, refactoring directory structures, changing providers, or updating API integrations—**you MUST update this `GEMINI.md` file.**
> 
> Keep the codebase map, dependencies list, and design guidelines completely accurate. Ensuring that this file self-heals at the end of each session ensures seamless continuity for all subsequent AI developer agents.

---
> Source: [Skylled/slopcafe_ui](https://github.com/Skylled/slopcafe_ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
