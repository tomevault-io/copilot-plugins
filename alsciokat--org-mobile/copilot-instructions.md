## org-mobile

> This file provides guidance to coding agents working in this repository.

# AGENTS.md

This file provides guidance to coding agents working in this repository.

## Project

Org Mobile — an Android viewer/editor for Emacs org-mode files (`com.alsciokat.orgmobile`). The org files live in a user-picked SAF folder and are synced by an external program; the app must never corrupt or reformat what it doesn't explicitly edit.

## Commands

```bash
./gradlew :core:org:test                  # parser/timestamp/edit tests (pure JVM)
./gradlew :core:data:testDebugUnitTest    # vault/agenda/search/reminder tests
./gradlew assembleDebug                   # build APK
./gradlew :core:org:test --tests '*OrgEditsTest*'   # single test class

adb install -r app/build/outputs/apk/debug/app-debug.apk
adb shell am start -n com.alsciokat.orgmobile/.MainActivity

cd web && npm run build          # editor bundle + themes -> app/src/main/assets/
cd web && npm test               # doc-engine unit tests; npm run test:browser for the bundle

# Convert an Emacs theme into a theme CSS file (see "Themes"), then rebuild the assets.
tools/emacs-theme-to-css.py ~/.emacs.d/elpa/modus-themes-*/modus-vivendi-theme.el \
    -o web/themes/modus-vivendi.css
```

Sample corpus for manual testing is in `sample-org/`; push with
`adb push sample-org /sdcard/Documents/org-sample`. The user's real vault is `/sdcard/Documents/org`.

## Commits

Write commit messages with a concise one-line imperative summary, followed by a blank line
and a bulleted list of only the core changes. Use the minimum number of bullets warranted by
the commit—one bullet is preferred when there is only one core change. Do not pad the list
with implementation details, verification steps, or separate restatements of the same change.

Before committing, update this AGENTS.md so its documentation matches the change—whenever the
commit adds, removes, or alters behavior described here (architecture, data flow, invariants,
toolchain, UI behavior), revise the relevant prose in the same commit. Keep the doc and the
code in sync; a stale description is a defect.

## Toolchain constraints (hard-won — do not "fix" these)

- SDK root is `/opt/android-sdk` (see `local.properties`). Platform packages are minor-versioned (`platforms;android-37.1` — there is no plain `android-37`).
- Gradle runs on **JDK 21** via `org.gradle.java.home` in `gradle.properties`; the system Java (26) is too new for AGP.
- **AGP 9 has built-in Kotlin**: applying `org.jetbrains.kotlin.android` to an Android module is a hard error. The `org.jetbrains.kotlin.plugin.compose` plugin is still required for Compose. The pure-JVM `:core:org` module uses `kotlin("jvm")` normally.
- Glance: `actionStartActivity<T>()` (reified) does not exist in glance-appwidget — use the `Intent` overload. Glance `Text` does not auto-size; widget font sizes are hard-coded sp values. Widget picker previews are **static** `android:previewLayout` XML (`res/layout/widget_*_preview.xml`) — Glance widgets render blank in the picker otherwise; keep the preview roughly in sync with the real layout by hand. Default size is set via `targetCellWidth`/`targetCellHeight` with `minWidth`/`minHeight` following Android's `70·cells − 30` dp formula (agenda widget is 3×2).
- **Android's ICU regex engine** is stricter than the JVM's: a bare `}` in a pattern crashes at runtime (`PatternSyntaxException`) though it compiles and passes JVM unit tests. Always escape `\}`; avoid `[\s\S]` (use `RegexOption.DOT_MATCHES_ALL`).
- After reinstalling the app, Samsung's launcher shows "Couldn't add widget" on existing widget instances until the app process pushes fresh content — launch the app once (or wait for `SyncWorker`). Not a bug; don't chase it.
- WebDAV uses **OkHttp** (`:core:data`; `mockwebserver` for tests), image loading uses **coil-network-okhttp**, and secrets use **androidx.security:security-crypto** (`EncryptedSharedPreferences`, still the standard despite maintenance mode). The manifest needs `INTERNET` and sets `android:usesCleartextTraffic="true"` (WebDAV is user-configured and often a self-hosted LAN box over plain `http://192.168.x.x`; Android blocks cleartext by default on API 28+, failing both OkHttp and Coil). WebDAV multistatus XML uses `javax.xml`; keep any `mobileorg.org`/index regex ICU-safe (escape `\}`, no `[\s\S]`).

## Architecture

```
:core:org    pure-JVM org parser/model/edits + MobileOrg protocol — no Android deps
:core:data   vault (SAF + WebDAV file access, cache), agenda, search, reminders — Android lib but logic is JVM-testable via FakeOrgFileStore
:app         Compose UI, notifications, widgets, settings, vault registry
```

```
Architecture Diagram:

+-------------------+                   +-------------------+                   +-------------------------+                 +-----------------------+
| /app              |                   | /app/.../data     |                   | /core/.../data          |                 | /core/.../orgmode     |
| AppRoot -> UI     | ---- access ----> | VaultManager      | ---- manage ----> | OrgVault                | --- contain --> | OrgDocument           |
|                   |               |-> | VaultActions      | ---- modify ---/  |                         |  `--- use ----> | OrgParser             |
|                   |               `-> | SettingStore      |                   |                         |                 |                       |
+-------------------+                   +-------------------+                   +---------+---------------+                 +-----------+-----------+
                                                                                          | 
                                                                                          | store and load with
                                                                                          v
                                                                                +-------------------------+                 +-----------------------+
                                                                                | /core/.../data          |		            | External Storage      |
                                                                                | OrgFileStore            | --- access ---> | Local Org Files       |
                                                                                |  |-> SafOrgFileStore    |				    | Remote Org Files      |
                                                                                |  `-> WebDavOrgFileStore |				    |                       |
                                                                                +-------------------------+				    +-----------+-----------+
```

### Round-trip safety (the core invariant)

`OrgParser` records **exact source spans** (`Span`) for every headline component (keyword, priority, title, tags, planning entries, property drawer, body, subtree). `OrgEdits` performs edits by splicing only the affected span into the original source string — every byte outside the edit stays identical, so the external sync program never sees spurious diffs. Tests assert this via `sample.replaceFirst(...)` equality. After any edit the document must be **re-parsed before the next edit** (spans go stale). `OrgEdits.setProperty` is the general property-drawer primitive (replace/add/remove one `:KEY:` line, create/destroy the `:PROPERTIES:` drawer as needed) — reuse it for any new headline property rather than hand-rolling drawer edits. `OrgEdits.setTitle` splices `titleSpan`; `OrgEdits.replaceBody` splices `bodySpan` (empty text clears it; otherwise a trailing `\n` is ensured); `OrgEdits.replaceSubtree` splices `subtreeSpan` — the generic primitive behind org commands. `OrgEdits.cutSubtree` removes the whole `[lineSpan.start, subtreeEnd)` range byte-for-byte (subtreeEnd already sits at the next sibling/ancestor headline or EOF, so no stray blank line remains) — the primitive behind `org-cut-subtree`.

The render model (`OrgContentParser` → `OrgElement`/`OrgInline`) is parse-only and never serialized back, **but it now carries source spans too** so the document view can edit in place: `OrgParagraph.span`, `OrgListItem.contentSpan`, `OrgLatexBlock.span`, and the per-inline `OrgLinkInline.span`/`OrgInlineMath.span` (populated from `parseInlines`'s `baseOffset`). These are byte offsets into the file source used purely to locate what to splice; the model is still never round-tripped back to text. `OrgColors` parses `:COLOR:` (Google-Calendar names, common names, hex) to ARGB; call sites tint TODO chips (with luminance-based text contrast) and calendar dots.

### Data flow

- `OrgVault` (`:core:data`) holds `StateFlow<Map<relPath, ParsedOrgFile>>`; `refresh()` re-reads only files whose `lastModified`/size changed. Cold-start consumers call `ensureLoaded()`, which shares one mutex-protected initial refresh even when the vault is genuinely empty; explicit `refresh()` remains unconditional for sync. `OrgVault.edit` re-reads fresh content, applies the transform, and retries on `FileChangedException` (write-safety against concurrent external sync). Edit transforms and parsing run on `Dispatchers.Default`, but the live cache is published only after the write and its matching parse are both complete, so source text, spans, screens, and widgets always observe one atomic version. Headlines are re-located inside transforms by line offset, then by title (`VaultActions.locate`). Each in-place `edit` snapshots the file's pre-edit source onto a bounded per-vault undo stack (only when the content actually changes); `OrgVault.undo()` restores the newest snapshot through the same write path without recording itself (so a second undo reaches the edit before it — there is no redo), returning the restored rel path. `VaultActions.undo` → the target-free `undo` command drives it; staging vaults never capture snapshots (undo is a no-op there), and `create` is not undoable. After any successful write (`edit`/`create`) the vault invokes its optional `onWrite` hook (fire-and-forget, `runCatching`-wrapped so it can never affect the edit); `VaultManager` wires it to `SyncthingNudge.start`, which broadcasts the documented `action.START` intent to every known Syncthing-Android package (Catfriend1 fork + official, declared under `<queries>`, with `FLAG_INCLUDE_STOPPED_PACKAGES`) to wake background sync. It is best-effort only — gated on the `SettingsStore.syncthingNudge` toggle (Settings → Sync, default on), throttled to one broadcast per 10s, and silently a no-op if no Syncthing app is installed or Syncthing is set to always run in background (which ignores the intent).
- **Persistent source cache / instant startup.** SAF reads (not parsing) dominate cold start — a real vault is hundreds of files. `VaultCache` (`:core:data`; `FileVaultCache` in `:app`, one length-prefixed binary file per tree URI under `filesDir/vault-cache/`) persists **raw source text only**, keyed by `(lastModified, size)`. **Never persist parsed structure**: re-parsing cached bytes is cheap, and a stale serialized parse could drift from the parser and drive a corrupting edit — source is the ground truth, the parse is derived. On the first `refresh()` the vault seeds reuse from the cache (unchanged files re-parse from cached bytes instead of a SAF read; only changed/new/removed files touch SAF), and re-saves only when the vault diverges from disk (a `persisted` `(lastModified, size)` signature guards against re-writing on every foreground rescan). The first `refresh()` publishes cached content (re-parsed) to `_files` up front — *inside* the same `refreshing = true` window, before the SAF walk — so screens render instantly while the reconcile continues (unchanged files are then reused from those primed entries). `edit`/`create` don't re-save the cache; the next `refresh()` (e.g. on resume) reconciles it. The agenda screen shows a small top-bar `CircularProgressIndicator` while `refreshing` **and** the view on screen is non-empty, and a centered one while `refreshing` and the list is empty (the sidebar folder tree, `VaultTree`, follows the same rule with its **Rescan button** as the small one — the spinner takes the icon's place in that slot) (the genuine first launch, before any cache exists) — the two are mutually exclusive, never both at once.
- `VaultManager` (`:app`) is the app-wide singleton owning the **registry of saved vaults** and the single live vault built from the active one. Each `VaultConfig` (persisted as JSON in the `vault` prefs; the pre-registry single `tree_uri` migrates to a `LOCAL` config on first run) is `LOCAL` (a SAF tree URI), `WEBDAV_DIRECT`, or `WEBDAV_STAGING`. A local vault is **named after its folder** — `VaultManager.folderName` reads the tree's `DISPLAY_NAME` (falling back to the tail of its document id, then a generic label), for a picked folder and for that migration alike. The name is a label and nothing else is keyed by it, so `VaultManager.rename` just rewrites the registry (no re-activation); Settings → Vaults reaches it through each row's pencil, which for a local vault *is* the rename dialog since its folder is the one that was picked, and for WebDAV still opens the full form. Switching just swaps the `vault` `StateFlow`, so every downstream collector recomposes unchanged. Each config gets its own `FileVaultCache` and `FilePendingEdits` (both keyed by config id, both deleted with the config) and, for WebDAV, its own auth'd Coil `ImageLoader` (`VaultManager.imageLoader(context)`; `OrgImageView` passes it so remote images load with credentials). WebDAV secrets (basic password / bearer token) live in `SecretStore` (EncryptedSharedPreferences), never in the JSON. An edit the backend cannot take is **queued**, not reported (see **The offline edit queue**); what still reaches `VaultManager.errors` → an AppRoot Snackbar is what queueing cannot rescue — an unsupported staging edit, a failed delete or rename, and a `PendingConflictException` naming the file the reader has to decide about. `onWrite`→`SyncthingNudge` fires for `LOCAL` only. `SafOrgFileStore` walks the tree with `DocumentsContract` directly (DocumentFile is too slow).

### Backends (`OrgFileStore` implementations)

The vault is backend-agnostic behind `OrgFileStore` (`list/read/create/createDirectory/write/meta/delete` + `resolveRelative`). `SafOrgFileStore` is the local backend (its `create` requests MIME `application/octet-stream`, **not** `text/plain` — SAF's ExternalStorageProvider otherwise rewrites `inbox.org` to `inbox.org.txt` to match the `text/plain` extension, which then dedupes to `inbox.org (1).txt` on retry; `.org` is an unknown extension whose inferred type is octet-stream, so requesting octet-stream preserves the exact name); `WebDavOrgFileStore` (`:core:data`, OkHttp) is the remote one — `list()` via `PROPFIND` (Depth `infinity`, falling back to a Depth-1 walk for servers like Nextcloud that refuse it), `read`=GET, `write`=conditional PUT (compares `getlastmodified` like SAF → `FileChangedException`; 412→same), `create`=MKCOL parents + PUT. `WebDavXml` is a pure, unit-tested multistatus parser (prefix-agnostic in the `DAV:` namespace). `WebDavException` extends `IOException`, so an offline WebDAV edit is caught by `OrgVault.edit`'s `IOException` handler — which now **queues** it rather than reporting it (see **The offline edit queue**); reads still come from the source cache offline. `uriFor` returns a `content://` (SAF) or `https://` (WebDAV) URI; the auth header rides an OkHttp interceptor shared with Coil.

### The offline edit queue

An edit made with the backend unreachable is **kept, not dropped**. `PendingEdits.kt`
(`:core:data`, pure) holds a `PendingFile` as **two whole texts**: `base`, the file as of the
last successful sync, and `local`, the reader's copy that every offline edit writes into. The
base is captured once, on the first queued edit, and is **never advanced except by a sync that
completed** — so it always describes a state the server really held, which is what makes it a
usable merge ancestor.

Two texts rather than a log of splices, deliberately. An earlier design recorded each splice
(`from`/`to`/`insert`/`expect`) and replayed the chain; it was dropped because nothing read the
individual records — replay wrote their fold — and because a three-way merge needs three
*texts*, which `diff(base, local)` recovers anyway. A chain also costs a quadratic re-fold on
every read and reports spurious conflicts for an edit that was typed and then undone.

- **Every route into the vault queues the same way.** `edit` catches `IOException` from
  `meta`/`read`/`write` and writes the transform's output into `local`; `create` queues a
  `create` entry whose base is empty. **`delete` and `rename` do not queue** — a queue holds a
  file's text, and neither of those has one to hold.
- **The queue is written to disk on every edit** (`PendingEditStore` → `FilePendingEdits` in
  `:app`, one length-prefixed binary file per config under `filesDir/pending-edits/`,
  temp-file-and-rename like `FileVaultCache`). No timer, no batching: the process can be killed
  at any moment. `writeUTF` is unusable for the payloads (64KB cap); lengths are explicit ints
  over UTF-8 bytes.
- **`inFlight` is a write-ahead marker**, saved immediately *before* each upload attempt and
  holding exactly the bytes about to be sent. If the process dies between the server accepting
  the write and the queue being cleared, the restart finds the server holding `inFlight` and
  knows the write landed — instead of merging against a base that is now a version behind and
  asking the reader about their own edit. Crash-safety rests on this and **not** on the merge
  being idempotent, which line-level diff3 cannot guarantee (see `ThreeWayMerge`).
- **The source cache still holds the server's version.** `persistCache` substitutes each queued
  file's `base` back in and drops `create` entries. The cache's contract is `(lastModified,
  size)` describing the text beside it; storing local content under a server stamp would make
  the next rescan skip a read it needed. `refreshLocked` loads the queue **before** the backend
  is asked and lays `local` over the cached source (`overlayPending`) — that is the cold start.
- **A failed `refresh()` no longer empties the vault**: the listing's `IOException` is caught,
  the cached and queued content stands, and `storage` says unreachable.

**When the queue goes up.** Coming back online is **not an event anything reports** — the app
can sit in the foreground through a whole outage and back with no rescan in between — so there
is exactly one signal that the backend can be written to again: a call to it succeeding. Every
`refresh()` flushes (which is what makes `SyncWorker` drain the queue hourly with the app
closed, and the resume rescan drain it on foreground), and so does the **first `meta` that
answers inside `edit` or `create`** — before the new edit is stored, not after.

That ordering is the point. Writing the new edit straight to the server would put it over a
version the queue is still holding a merge for, and the queued entry would then describe a file
that has moved on; the server would go on accepting edits with the offline work never merged,
which is exactly what happens without this. It costs nothing when the queue is empty (the
ordinary case) and nothing extra while offline, since the `meta` has already failed by then and
the edit is on the queue. Afterwards, **a file that still has a queue entry is edited through
the queue, never over it** — conflicted, or a write that did not land, the rule is the same, and
both branches return rather than falling through to a write.

**Flushing** (`flushPendingLocked`) tests four things per file, in order:

1. `inFlight` matches what the server holds → a previous attempt landed; finish the job.
2. The server is still at `baseLastModified` → nothing to reconcile, push `local`.
3. The server moved → read it and run `ThreeWayMerge` over base/local/remote. **A clean merge
   is written and both sides end in step**, and the file is named in `StorageState.merged`
   because what is on the server afterwards is neither version anybody typed. That notice is
   held until `acknowledgeMerges()` — it is worth seeing once.
4. The merge has conflicts → nothing is written, the entry stays queued with `conflict` set, and
   `PendingConflictException` reaches `onError` once per file (not on every rescan).

**An unresolved conflict is a state a file lives in, not a modal the reader has to clear.** Once
marked, a file is **pinned offline**: `edit` routes it to the queue whether or not the backend
answers, and `flushPending` skips it entirely. Both halves are load-bearing.

- **Editing must not write.** The overlay carries the *server's own current stamp* — it has to,
  or the rescan could not tell what changed — so an ordinary conditional write on a conflicted
  file sails straight through the `expectedLastModified` guard and puts the reader's version
  over the change the merge had just refused to overwrite. The guard cannot catch this; the
  explicit `_pending[relPath]?.conflict` test at the top of `editAttempts` is what does.
  (Without it, `local` also went stale — the write bypassed the queue, so the next
  `overlayPending` put the pre-conflict copy back and the edit vanished.)
- **Flushing must not retry.** Re-running the merge every rescan would either keep failing, or
  quietly succeed on a question the reader was told was theirs. It waits.

So the answer to "what if I ignore it" is: the file goes on being edited exactly as it was
offline, every keystroke lands in `local` and on disk, nothing reaches the server, and the
markers stay up.

Because a conflict is only ever **deferred, never cleared**, there are four ways back to it and
none of them depends on the prompt that first raised it: the top bar's indicator (which goes
straight to the file when that file is the one on screen), the **sidebar's long press** on a
marked row, the document's **⋮ → Resolve conflict…** (first in the menu, and present only for a
conflicted file), and the `resolve-conflict` command, which reaches it from any screen the
prompt opens on. The command picks its file rather than typing it, like `delete-file`. `conflictMerge` recomputes against whatever the server holds *at the time it is
opened*, so a conflict that has since become resolvable — the reader edited the clashing region
away, or the server moved again — is found by the conflict screen, which is why that screen
offers **Sync now** on a clean result instead of merely reporting one. Resolving later takes
everything typed in the meantime, since `local` is what it merges.

**`ThreeWayMerge`** (`:core:data`, pure) is a line-based diff3. Lines keep their own terminator
(`splitLines`), so joining regions is plain concatenation and a merge is byte-exact — CRLF
survives, and so does a file with no trailing newline; nothing normalizes anything, because
this runs over files an external program also writes. The diff is **patience-style**: trim the
common prefix and suffix, anchor on lines occurring exactly once on each side, recurse into the
gaps, and report a segment with no unique line in it as changed whole. Anchoring is what keeps
two edits at opposite ends of a large file independent without a matrix the size of the file
squared; the conservative fallback can only turn a clean merge into a question, never a question
into a silent overwrite. A region both sides changed is `Stable` when their texts agree (or when
one of them equals the base) and `Conflict` otherwise, keeping all three versions.

Whether two changes clash is **strict overlap** — they must actually share a base line.
Abutment is not a clash: the server rewriting the last heading while the phone appends after it
is the commonest edit an org file gets, the order of the results is not in question, and the
textbook `<=` rule (node-diff3's) put it to the reader as one. The single exception is two
**insertions at the same point**, which share no line but do share a position, so nothing says
whose text goes first.

Two other things in it are easy to get wrong and are covered by tests: `sideSlice` needs **two
separate sums** — the region's start moves by that side's *earlier* changes, its length by this
region's own — since a single "everything ending at or before this point" rule mistakes a pure
insertion for something that happened before the region it is inside. And re-merging a merge
that already landed is usually but **not always** clean, which is why `inFlight` exists.

**Atomicity.** Every write path — `edit`, `create`, `refresh`, `flushPending` — holds the
vault's `Mutex` from the `meta` that decides what to do, through the read and the merge, to the
write. A Kotlin `Mutex` is **not** released on suspension, so the network I/O is inside the
critical section and an edit arriving mid-flush waits rather than interleaving: it cannot be
lost, and it cannot produce a conflict that would not have happened anyway. The cost is that an
edit blocks for the length of a flush on a slow link — a stall, not a race, and the right
trade, since the alternative is a merge computed from inputs that moved under it.

**The apply is the one exception, and has to be**: it spans `conflictMerge` and
`resolveConflict` with a person deliberating in between, so both of the merge's inputs can move
in that window and both are checked against what the screen was shown. `seenRemote` catches the
server moving again — a stamp check would be no use here, since the reader has just decided
*against* part of the very change it would trip on. `seenLocal` catches the file being edited
on this device meanwhile: a conflicted file goes on being edited, and those edits arrive from
places that are not this screen (a widget's mark-done, a notification action, the agenda), so
writing text composed from a stale copy would discard the edit *and* drop the queue entry
holding it. `ResolveOutcome` names which happened; neither is an error, so the screen reloads
the merge and re-asks rather than reporting a failure, wording the two apart because one is
somebody else's change and the other is your own. `discardPending` is the one call that loses
work, so nothing reaches it but an explicit choice.

**The queue belongs to the backend it was made against**, not to the vault's registry entry.
`PendingFile.origin` records it (`VaultConfig.backendOrigin()` — the SAF tree URI, or the WebDAV
root plus its user; never the secret, since this sits in plain prefs beside the queue), and
`flushPendingLocked` skips any entry stamped with a different one.

This is not defensive programming, it is a hole with teeth. **A vault's address is editable
while its identity is not**: `WebDavVaultForm` keeps `initial?.id`, so a vault re-pointed at
another server keeps its config id and therefore its queue file — whose `base` and
`baseLastModified` describe files on the *old* server. Flushing that against the new one would
read a same-named file, three-way-merge the reader's notes into somebody else's document and
write the result back. And the worst shape is the quietest: if the other server reports the same
`getlastmodified` (second-resolution HTTP dates, over files two servers may well have synced
from the same source), the flush takes the "the server is where we left it" branch and writes
`local` over it with **no merge at all**.

Foreign entries are **held, not dropped**: they still show, still take edits, and go up on their
own if the address is put back — which is the likely case, since the usual way to get here is a
typo. `StorageState.foreign` reports them, the indicator shows the error-tinted `SyncProblem`,
and the dialog explains rather than offering a *Resolve* button, since settling one would write
it to the very server it must not reach.

Wrong **credentials** are not guarded and do not need to be: a 401 is a `WebDavException`, which
is an `IOException`, so it reads as "offline" — the queue is safe and waits, though the wording
is about reachability rather than auth.

**The server changing underneath an unchanged address** is the other half, and `origin` cannot
see it: the config is right, so nothing on this device has anything to compare. Only the contents
can say. `refreshLocked` therefore checks the **listing** — before a single file is read — and a
listing that shares **no** path with the vault it knows is treated as a different folder
(`StorageState.remoteMismatch`): a moved document root, a remounted share, an alias edited on the
box. While it stands, nothing syncs at all — `flushPendingLocked` refuses outright, `edit` and
`create` queue instead of writing, `_files` and the source cache keep the vault the reader knows
— and the indicator shows it above every per-file state, since none of those are the thing to
read first.

It costs **nothing extra**: `store.list()` is called from exactly one place (`refreshLocked`),
so the check is a set comparison over a listing already fetched. An ordinary edit never lists —
it is `meta` + the conditional write — which is also why the check cannot be the only guard.
Between rescans an edit meets the changed folder on its own, through `meta`, and finds nothing;
that path **queues** rather than throwing, because a `NoSuchElementException` escapes
`editInternal` (which catches `IOException` and the staging exception and nothing else) into
`DocumentScreen`'s `scope.launch`, which has a `finally` and no `catch` — a crash on a keystroke,
in the very situation the detection exists for. A path the vault never knew still throws, since
that is a caller error rather than a sync one.

Zero overlap rather than a ratio, on purpose: a false positive costs one tap on a dialog that
explains itself, a false negative writes the reader's notes into a stranger's directory. A
wholesale rename does trip it, and should, since nothing distinguishes the two from here. An
*empty* listing over a known vault trips it too, which is right — every file vanishing from the
server is worth stopping for — and incidentally fixes what used to be a quiet nuisance, where
an empty listing was published and `persistCache` then wiped the offline copy.

`acceptRemote()` is the way through, and the dialog's confirming button. It says "this folder is
my vault" — **not** "these are my files" — so it also **drops every queued entry's ancestor**
(`base = ""`, `baseLastModified = NO_ANCESTOR`; `create` entries are left alone, a new file being
new wherever it lands). A queued base came from the folder that was here before and is not an
ancestor of anything on this one, and an empty ancestor is what makes each side's content an
insertion at the same point — a question the merge already knows how to put. It is one-shot and
needs no persistence: once adopted the listing *is* what the vault knows, so the next rescan
matches it.

**A merge's unstated premise is that the base is a common ancestor, and nothing in the mechanics
checks it** — two unrelated documents merge perfectly happily. `ThreeWayMerge.sharesAnyLine`
(non-blank lines, or the test would pass on almost any pair of org files) is the guard, and
`flushPendingLocked` refuses to merge without it. This is not belt-and-braces for the accept
path: the listing check only fires on **zero** overlap, and `inbox.org` is a popular name, so a
folder swap sharing one filename reaches the ordinary flush with a base that is nobody's
ancestor. The shape that makes it dangerous rather than merely wrong is the commonest edit there
is — an append is a pure insertion at the very end while the wholesale difference covers the
whole file, and those *abut* rather than overlap, so the merge comes out **clean**, writes the
stranger's file with the reader's line on the end, and reports it as a successful merge. The fast
path (push the copy, no merge) matches **size as well as stamp** for the same reason, at no cost.

**Staging vaults do not queue** (`pendingStore = null`): a MobileOrg edit *is* a write to the
server (`mobileorg.org`), so there is nothing to queue to.

### Backends (`OrgFileStore` implementations)

The vault is backend-agnostic behind `OrgFileStore` (`list/read/create/createDirectory/write/meta/delete` + `resolveRelative`). `SafOrgFileStore` is the local backend (its `create` requests MIME `application/octet-stream`, **not** `text/plain` — SAF's ExternalStorageProvider otherwise rewrites `inbox.org` to `inbox.org.txt` to match the `text/plain` extension, which then dedupes to `inbox.org (1).txt` on retry; `.org` is an unknown extension whose inferred type is octet-stream, so requesting octet-stream preserves the exact name); `WebDavOrgFileStore` (`:core:data`, OkHttp) is the remote one — `list()` via `PROPFIND` (Depth `infinity`, falling back to a Depth-1 walk for servers like Nextcloud that refuse it), `read`=GET, `write`=conditional PUT (compares `getlastmodified` like SAF → `FileChangedException`; 412→same), `create`=MKCOL parents + PUT. `WebDavXml` is a pure, unit-tested multistatus parser (prefix-agnostic in the `DAV:` namespace). `WebDavException` extends `IOException`, so an offline WebDAV edit is caught by `OrgVault.edit`'s `IOException` handler — which now **queues** it rather than reporting it (see **The offline edit queue**); reads still come from the source cache offline. `uriFor` returns a `content://` (SAF) or `https://` (WebDAV) URI; the auth header rides an OkHttp interceptor shared with Coil.

### The offline edit queue

An edit made with the backend unreachable is **kept, not dropped**. `PendingEdits.kt` (`:core:data`,
pure) holds the model: a `PendingEdit` is one splice (`from`/`to`/`insert`/`expect` — the same shape the
document editor reports over the bridge), and a `PendingFile` is one file's **base** plus the ordered
**chain** of splices that turns it into what the reader is looking at.

Both halves are load-bearing. Each splice's offsets are relative to the state the one before it produced,
so a queue is a strict sequence replayed from one base — never a set of independent changes that could be
applied out of order or in part. The base is the file's source *as the server last showed it*, captured
once on the first queued edit and extended by every later one, and `baseLastModified`/`baseSize` are what
tell a reconnect whether the server is still where we left it.

- **Every route into the vault queues the same shape.** The splice is derived inside `OrgVault` by
  `minimalEdit(before, after)` — a character-for-character port of the engine's own
  (`web/packages/doc-engine/src/text.js`), so a keystroke, a heading marked done, a property set and an
  undo all become one chain. `edit` queues on any `IOException` from `meta`/`read`/`write`; `create`
  queues as a `create` entry whose base is empty. **`delete` and `rename` do not queue** — a queue holds
  edits to a file's text, and neither of those has one to hold.
- **The queue is written to disk on every entry** (`PendingEditStore` → `FilePendingEdits` in `:app`, one
  length-prefixed binary file per config under `filesDir/pending-edits/`, temp-file-and-rename like
  `FileVaultCache`). The process can be killed at any moment, so there is no timer and no batching.
  `writeUTF` is unusable for the payloads (it caps a string at 64KB); lengths are explicit ints over UTF-8
  bytes.
- **The source cache still holds the server's version.** `persistCache` substitutes each queued file's
  `base` back in, and drops `create` entries entirely. The cache's contract is `(lastModified, size)`
  describing the text beside it, and storing local content under a server stamp would make the next
  rescan skip a read it needed. The queue is what carries the local side across a restart; `refreshLocked`
  loads it **before** the backend is asked and lays it over the cached source (`overlayPending`), so a
  cold start shows what was last typed.
- **`overlayPending` applies each chain to its own base**, never as a splice onto whatever is in the map.
  That is the safety property: after an external change the map holds the server's newer text, and
  replaying stale offsets onto it would splice at positions that no longer mean anything. The metadata
  stays the server's so the next rescan can still tell what changed, and the overlay is applied *before*
  the reconciled listing is published, or a file created offline would blink off screen for the length of
  the flush.
- **A failed `refresh()` no longer empties the vault**: the listing's `IOException` is caught, the cached
  and queued content stands, and `storage` says unreachable.
- **Reconnect replays the chain** (`flushPending`, run by every `refresh()` — so `SyncWorker` drains the
  queue hourly with the app closed). Each file is one conditional write of the chain's result, guarded by
  the base's own stamp, exactly as an online edit is. A file whose stamp moved is marked
  `PendingFile.conflict` and **left on the queue**: its splices describe text that is gone, and there is
  no honest way to apply them without being told which version wins. `PendingConflictException` reaches
  `onError` once per file (not on every rescan), and `resolveConflict(relPath, keepLocal)` is the way out
  — the user's call, never a rule the vault applies.
- **Staging vaults do not queue** (`pendingStore = null`): a MobileOrg edit *is* a write to the server
  (`mobileorg.org`), so there is nothing to queue to.

`OrgVault.storage` (`StorageState`: `reachable`, `queuedFiles`, `conflicts`, `merged`, `foreign`,
`remoteMismatch`) is what the top bar reads — see **The top bar's storage indicator**.

`OrgVault.resolve` maps a `LinkTarget` to a `LinkResolution`: a `file:` link's path is resolved directory-relative (org semantics). When that exact path has no file and the **lenient file link resolution** toggle is on (`SettingsStore.linkBasenameFallback`, Settings → Files, default on; injected into `OrgVault` as a live `linkBasenameFallback: () -> Boolean` supplier so it applies without rebuilding the vault), it falls back to a file with the same **basename** elsewhere in the vault — but only when that match is **unique** (ambiguous basenames stay `NotFound`, never guessed). This makes real-world links written one directory level off (e.g. `file:x.org` for a sibling dir's `../x.org`) still open. The default `OrgVault` supplier returns false, keeping resolution strictly org-faithful in tests.

**MobileOrg staging (`WEBDAV_STAGING`).** Faithful to Emacs `org-mobile.el`: source `.org` files are read/rendered normally, but the four protocol files (`index.org`, `agendas.org`, `checksums.dat`, `mobileorg.org` — `MobileOrg.PROTOCOL_FILES`) are hidden from the browsable list (`WebDavOrgFileStore.hiddenRootFiles`) and **never edited in place**. Instead `OrgVault` runs with a `StagingWriter`: an `edit` still runs the same content transform, but the before/after docs are diffed by `MobileOrgDiff` into `F(edit:todo|priority|tags|heading|body)` change records (targeting the entry by old `:ID:` or `olp:file:outline`) appended to `mobileorg.org`; `create` becomes a capture appended there. The edited content is then published **optimistically** carrying the *server's* current metadata, so the ordinary refresh keeps showing it until Emacs pulls and re-pushes the file (its `lastModified` changes → overlay auto-drops). Structural edits (heading count changes) can't be expressed as MobileOrg field records → `StagingUnsupportedEditException`, reported and no-op (never a silent divergence). `MobileOrg`/`MobileOrgDiff`/`MobileOrgIndex` are pure `:core:org` (byte-exact to `org-mobile-apply`, unit-tested); `org-mobile-use-encryption` is out of scope.
- **Settings are filed under six collapsible groups** (`SettingsGroup` in `ui/settings/SettingsScreen.kt`):
  Appearance, Vaults & sync, Files & links, Document, Agenda & calendar, Capture & journal — named for what a
  user came looking for rather than for the code behind them, each with a one-line summary, and all closed until
  asked for, so the screen opens as six lines instead of one long scroll. Which are open is saved as their joined keys
  (`rememberSaveable` carries only what a Bundle can), and the route can name the one to open with:
  `NavHostController.openSettings(group)` seeds it, so the document view's ⋮ → Settings arrives on **Document**
  and the agenda's on **Agenda & calendar**, while the sidebar's gear — the general way in — opens with
  everything closed. Every option's explanation lives **behind a `?` button
  beside its name** (`SettingLabel` → the shared `ui/HelpButton.kt`, a persistent `RichTooltip`) rather than in a paragraph over
  the row: a wall of prose pushes the settings themselves apart, and the explanation is wanted once, not on every
  visit. `SwitchSetting` and `SurfaceCheckRow` are the two shapes almost every setting has — the latter drawing
  no checkbox where a column returns null, which is how a setting that cannot apply to a surface keeps the matrix
  aligned.
- **Deleting, and creating a file by hand.** `OrgFileStore.delete(relPath, directory)` removes a file, or a
  folder **and everything inside it** — non-org files included, which is what deleting a folder means (SAF's
  `deleteDocument` and WebDAV's `DELETE` on a collection both do it in one call; the flag only tells WebDAV to
  address the collection with a trailing slash). `OrgVault.delete` drops the path (or the whole prefix) from the
  live map and returns whether it went, reporting a transport failure through `onError` like an edit does; a
  staging vault refuses outright, since MobileOrg has no change record that says a file is gone. It is
  deliberately **not undoable** — the undo stack restores a file's *content* through the ordinary write path, and
  a deleted file has nothing to write into — so every route to it goes through `ui/ConfirmDialog.kt` and asks
  first. Three routes: the document view's ⋮ (which then shows the startup screen in the file's place — the
  screen that did the deleting should not report its own file missing), and a **long press** in the sidebar on a
  file (Delete) or on a folder (New file here / Delete folder). `NewFileDialog` (`ui/files/`) is the one way a
  file is named, shared by the folder menu and the startup screen: it writes an empty file, appending `.org` if
  that was not typed, and opens it. The folder menu also offers **New folder here** (`createDirectory` →
  `DocumentsContract.createDocument` with the directory MIME type, or MKCOL; parents made as needed) and both
  menus offer **New file alongside**, which files into the selected item's *parent* — the vault root for a
  top-level one. A new folder hands straight on to the file prompt inside it, and has to: the tree is built from
  `files.keys`, so a folder holding no org file is not in the vault's map and cannot be shown until something
  lands in it.
- **The startup screen** (`StartupScreen` in `DocumentScreen.kt`) is what stands in for a document when there is
  none — a fresh vault, a bad path, or a file just deleted. It carries the two things that can be done from an
  empty screen as buttons rather than as instructions: **Select file** opens the sidebar, **Create file** prompts
  and files into the vault root.
- `SettingsStore` filters which files feed agenda/calendar/widgets/notifications (org-agenda-files style). Every consumer calls `SettingsStore.filterAgendaFiles(...)`. The sidebar folder tree (`VaultTree`) lists every file as a collapsible directory tree, prefixed by a **Recent** section, over a compact `SidebarActions` row along the **bottom** — the top of a list of files is where files are looked for, and there is no title row at all; the row stays put over the empty state too, which is exactly when Rescan and Settings are wanted. It is painted
in the theme's own `--sidebar-bg`/`--sidebar-fg` rather than in the drawer's Material default (see **Themes**). Its left half is the **active vault's name, which is the vault switcher**: `VaultManager.configs` drops down on a press and picking one is `VaultManager.activate`, which only swaps the `vault` StateFlow, so the tree recomposes onto the new vault's files with the drawer still open on it (the document behind still shows the old vault's path until a file is picked). Rescan and Settings sit at the right, Rescan doubling as the refresh spinner: `NavHostController.openDocument` (the single choke point for every document open — files/search/agenda/widgets/deep links) calls `SettingsStore.recordRecentFile(relPath)`, a newest-first ring of rel paths (`recentFiles`, capped at `MAX_RECENT_FILES`, persisted newline-joined). The Recent section shows the still-present subset `.take(recentFilesCount)`; the count is set in Settings → Files (`setRecentFilesCount`, default `DEFAULT_RECENT_FILES_COUNT`, 0 hides the section). **Cursor positions are a separate store** — that ring is an ordered list of paths for the sidebar, this a bounded, expiring map keyed by path (`CursorPositions.kt`: `CursorPosition(offset, whenMillis)`, `CursorJson` newest-first JSON, capped at `MAX_CURSOR_POSITIONS`). `DocumentScreen` reads `SettingsStore.cursorPosition(relPath)` once per file and writes `recordCursorPosition` from a `LifecycleResumeEffect`'s `onPauseOrDispose` — not on every caret move, which happens on each keystroke. Settings → Document controls both halves: `restoreCursor` (default on; off means neither restore nor record) and `cursorRetention` (`CursorRetention`, default 30 days, `FOREVER` never expires), which prunes on change and on `init`. Search keeps file and ordinary-note results from every file but passes the agenda-file key set to `VaultSearch`, which excludes TODO/DONE headings and their body content from non-agenda files. `SearchFilter` is a **kind of result**, not a filter over one list — `ALL`, `FILES`, `TODOS` (headlines with a keyword, active or done), `HEADINGS` (every headline) and `TEXT` (body lines only) — so asking for headings never costs a body scan; the three `wants*` predicates are what each group's pass is gated on. UI search uses the cancellable `VaultSearch.searchIncrementally` flow on `Dispatchers.Default`, debounces typing briefly, and appends small batches as file, headline, and body scans progress; never move a whole-vault scan back into Compose `remember`.
- **Category filtering** is a second, orthogonal filter: `OrgDocument.headlinesWithCategory()` resolves the nearest inherited `:CATEGORY:` heading-drawer property; file names and `#+CATEGORY:` keywords are not categories. `AgendaBuilder.categories` lists the values in effect on anything that can reach an agenda surface — a headline that is scheduled, has a deadline, **or carries a TODO keyword** (planning alone was the old rule, which left an undated TODO's category unlistable and so un-hideable, though the list surfaces fold such entries onto today); a plain note's category stays out, the matrix being a filter over the agenda rather than an index of the vault. `AgendaBuilder` stamps that nullable value onto `AgendaItem.category`, and `SettingsStore.filterAgendaCategories(days)` (default "all") drops non-matching items *after* the agenda is built — so it wraps every `AgendaBuilder.build(...)` call in the two screens and `WidgetData`, but not `ReminderScheduler` (notifications ignore the category filter by design). The Settings matrix always includes an `Uncategorized` row for entries with no effective drawer property.
- **The agenda and the calendar are one screen** (`ui/agenda/AgendaScreen.kt`, route `agenda`): its bar carries a toggle whose icon is the view it goes to (the chip's own two glyphs), and the position persists as `SettingsStore.agendaView` (`AgendaView.LIST`/`CALENDAR`, default list) — a single chip stop cannot re-open the view you left otherwise. The month grid is `ui/calendar/CalendarView.kt`, now a **presenter**: the screen builds the data (and only for the view that is showing — a vault refresh is frequent, switching is not) and passes it in, which is also what lets the bar's spinner tell an empty month from a loading one. The two keep **separate `AgendaSurface`s** (`AgendaView.surface`), since every display preference below is still per surface. The month names itself in a header **inside the content**, with the arrows that step it — they label and drive the grid, so they belong to it rather than to the screen's bar, which stays "Agenda" in both views.
- **Overdue placement** differs by surface via `AgendaBuilder.build(carryOverdue = …)` (default `true`): the agenda view/list widget fold missed occurrences and pre-warned deadlines onto **today** (org-agenda style, negative `daysOffset`), but the calendar view and calendar widget pass `carryOverdue = false` so every occurrence — past included — sits on its **real day** (`daysOffset = 0`) and no cluster piles onto today.
- **Agenda-item visibility** is also independent per `AgendaSurface`: `SettingsStore.showDone(surface)` (default false) controls completed TODOs, while `SettingsStore.showNonTodo(surface)` (default true) controls scheduled/deadline entries without a TODO keyword. A third, `SettingsStore.showUnscheduledTodo(surface)` (default false), folds **active TODOs that have no scheduled/deadline planning** onto today (a new `AgendaKind.TODO` item with a null `timestamp`); it is honored only on the two list surfaces (`AgendaSurface.isList` — Agenda view and list widget) because such items have no real day and the builder gates it on `carryOverdue`, so calendar surfaces never place them. All three are persisted separately for Agenda view, Calendar view, list widget, and calendar widget, and each consumer passes them to `AgendaBuilder.build(includeDone = ..., includeNonTodo = ..., includeUnscheduledTodo = ...)` before applying its category filter. The four-column Settings rows mirror category visibility; the unscheduled-TODO row leaves the two calendar columns blank. Notifications ignore these display preferences by design.
- **List periods** are independently configurable for the Agenda screen (default 14 days) and list widget (default 7 days) through `SettingsStore.agendaPeriod()` / `listWidgetPeriod()`. The screen collects its flow directly; changing the widget period increments `displaySettingsVersion` so Glance reloads immediately.
- `OrgMobileApp` wires the reactive refreshes: committed vault files + agenda-file selection refresh widgets immediately, while `ReminderScheduler.reschedule` consumes a separately debounced (1s) collector over the same inputs. Keep them separate because reminder scheduling can take many seconds on a large journal and must never delay home-screen state. `WidgetData` serializes programmatic Glance refresh requests with one mutex. Before every `GlanceAppWidget.update`, `refreshWidget` increments the installed instance's persisted `WIDGET_DATA_VERSION`; each running widget composition observes it through `currentState<Preferences>()` and reloads external agenda data with `produceState`. This invalidation is the actual fix for delayed TODO updates—`update()` alone only recomposes the data captured by an active Glance session. A widget action acquires the mutex **before** its vault edit, invalidates the tapped instance first, then every peer and sibling instance once, and records the rendered files/settings version so the automatic files-flow refresh becomes a no-op. Widget agenda construction uses `IncrementalAgendaCache`, keyed by query (including completed/non-TODO visibility) and `ParsedOrgFile` identity; after an edit it walks only the changed file's headlines and reuses expanded items for every unchanged file. Category/completed/non-TODO display changes (debounce 300ms) refresh widgets only. Screens collect their own surface flows directly. Notifications therefore need no manual refresh calls after edits.

### Notifications

`UpcomingReminders` (pure, tested) computes alarms directly from each parsed headline. One `:REMINDER:` property stores a comma-separated list: relative entries such as `SCHEDULED -1h` / `DEADLINE -30min` follow planning repeaters, while `<2026-08-06 Thu 9:30>` entries are one-off absolute reminders. The property drawer is the only reminder store—never add a parallel JSON/preferences store. All UI changes use `VaultActions.setReminder` → `OrgEdits.setProperty` for round-trip safety. `ReminderScheduler` converts results to exact alarms (idempotent: cancels the previous batch recorded in prefs). `ReminderReceiver` shows the notification purely from intent extras (works when the process is cold); Done/Snooze actions live in `NotificationActionReceiver`, which waits for the vault before editing and removes only the completed absolute reminder. Reboots are covered by `BootReceiver`, background sync by the hourly `SyncWorker`.

### Widgets

There are three Glance widgets: the agenda list, the month calendar, and the **capture** widget. Both agenda widgets expose a separate mark-done action (`✓`, a tinted `@drawable/ic_check` vector via Glance `Image` — a flat Material check, not the curved `✓` glyph) for active TODO items while the rest of the row still opens the document. `MarkDoneAction` (`widget/WidgetActions.kt`) receives rel-path/offset/title/widget-kind action parameters, waits for a cold-start vault through `WidgetData.vault()`, then calls `WidgetData.editAndUpdate`, which holds refresh priority while running the same round-trip-safe `VaultActions.markDone` used by the app UI. It calls `refreshWidget` for the tapped instance first, then each peer instance individually, followed by every instance of the sibling kind; direct per-instance updates avoid the launcher bug where `updateAll` misses the action's instance. Each invalidation/update is wrapped in `runCatching` so one widget can't block the others. The calendar widget exposes the action in the selected day's item list. A tapped day is only a transient selection: `SelectDayAction` records the day plus a `SELECTED_AT_KEY` timestamp, and the widget re-focuses today on its own once `SELECTION_RESET_MS` (60s) of idle elapses. The read side treats a selection older than that window as expired (falls back to today on any re-render), and `SelectDayAction` schedules a one-shot inexact alarm (`CalendarResetReceiver` → `WidgetData.updateCalendarWidgets`) to force that render — the alarm need only *cause* a render since the read-side timeout does the actual reset. Only show the mark-done action when `todoKeyword != null && !isDone`; the same rule applies to the agenda screen's completion actions (list and month alike) and the heading edit sheet. Completed items are styled with the variant color when that widget's `showDone` preference includes them, while items with `todoKeyword == null` follow that widget's independent `showNonTodo` preference. Future rows in the list widget use a fixed-width two-line date gutter: weekday plus a tiny `M/d`, so the same weekday in later weeks remains distinguishable. Keep the static picker previews (which use the same `ic_check` drawable and compact future date) in sync with the real layout.

**The widgets wear the app's theme.** `orgWidgetColors(context)` (`widget/WidgetTheme.kt`) builds a Glance
`ColorProviders` from the same two slots the app uses — `SettingsStore.lightThemeId` and `darkThemeId`, each
either a CSS theme's `OrgThemes.colorScheme` or `systemColorScheme` (Material You where the device has it) — and
every widget passes it to `GlanceTheme(colors = …)`. The shape fits exactly: Glance takes a light scheme and a
dark one and picks between them by the **host's** day/night state, which is the same question the app asks of
`isSystemInDarkTheme()`, so a widget follows the phone's switch on its own with the app not running. The slots
are read as **snapshots** (`.value`) because a Glance composition is a one-shot render rather than a
subscription; `OrgMobileApp` collects both flows and re-renders every widget kind — colour is not content, so it
reaches the capture widget as well as the two agenda ones. The root background is `GlanceTheme.colors.surface`
and **not** Glance's own `widgetBackground`: that one is not a Material role, so the providers cannot fill it and
it stays a launcher-flavoured default (a pale green, on the device this was found on) with nothing to do with the
theme.

The **capture widget** (`widget/CaptureWidget.kt`, `CaptureWidgetReceiver`) is a launcher, not an editor: it renders one `+`/`@drawable/ic_add` button per `SettingsStore.captureTemplates` entry plus a `LazyColumn` of `SettingsStore.recentCaptures`. Buttons/rows are plain `actionStartActivity` launches (no `ActionCallback`/vault edit) into `MainActivity` — template buttons carry `EXTRA_CAPTURE_TEMPLATE` (each Intent gets a unique `orgcapture://template/<id>` data URI so PendingIntents don't collide), recent rows carry `EXTRA_OPEN_PATH`/`EXTRA_OPEN_OFFSET`. It reads settings snapshots directly (same process, `SettingsStore` is init'd in `OrgMobileApp`) and reloads via the shared `WIDGET_DATA_VERSION` + `produceState` pattern. `WidgetData.updateCaptureWidgets` refreshes only this widget kind; `OrgMobileApp` drives it from a `combine(captureTemplates, recentCaptures)` collector, independent of the agenda/calendar refresh paths. Keep `res/layout/widget_capture_preview.xml` (static picker preview) hand-synced with the real layout.

### Repeater semantics (match Emacs exactly)

`OrgTimestamp.advancedForDone`: `+` shifts once (even into the past), `++` shifts until future preserving cadence, `.+` restarts from today. Marking a repeating task done advances the timestamp and **keeps** the TODO state; non-repeating tasks flip to DONE and get a `CLOSED:` stamp. `occurrencesBetween` expands repeaters for agenda/calendar windows.

### Navigation

The app revolves around a **single home screen — the document view** — rather than bottom tabs. Routes in `AppRoot`: `home` (the start destination, a `DocumentScreen` showing the last-opened file — `SettingsStore.recentFiles.value.firstOrNull()`, captured once; `null` → an empty placeholder), `doc/{path}?offset=&edit=`, `agenda`, `editor/{path}?offset=`, `settings?group=`, `capture?template=`, `conflict/{path}` (settling offline edits that clash with the server), `help` (the index of every help page), `help/{name}`. `DocumentScreen` wraps its content in a `ModalNavigationDrawer` whose sidebar is the vault **folder tree** (`ui/files/VaultTree.kt` — collapsible directories built from `files.keys`, an optional Recent section, and a bottom row of the vault-name switcher plus Rescan/Settings; the old Files tab/`FilesScreen` is gone). The drawer opens **only via the top-bar hamburger** (`gesturesEnabled = drawerState.isOpen` — edge-swipe-to-open is off, swipe-to-close stays on). Both bars float over the document and auto-hide together while scrolling down — a sticky bottom **chip** (`ui/BottomChip.kt` — the places to go, as against the sidebar's file list) and the view's own top bar (`DocumentBar`, the shared `ScreenBar` made to float) — driven by the scroll direction the editor reports over the `scrolled(dir)` bridge method → `chromeVisible`. While the editor holds focus the chip's place is taken by the **editing toolbar** (see below), which does *not* auto-hide. Both sit `BOTTOM_BAR_GAP` off the bottom edge. The chip is **shared by the document and agenda screens**: each passes the `ChipStop` it *is* and the chip offers the other, then Search, then the command prompt — so the same pill navigates both ways and never offers the place you are standing. The agenda's list and month views are **one** stop, not two: they are one screen with a toggle in its own bar. The agenda is pushed over the document, so `NavHostController.openChipStop(DOCUMENT)` is a plain `popBackStack` — which keeps the back stack shallow and returns you to the *file that was open* rather than to the start destination. Only the document view hides the chip on scroll; the agenda reserves room below its last row instead. `openDocument` pops back to `home` first (`popUpTo(HOME_ROUTE)`) so the back stack stays shallow; `agenda` is pushed with a back arrow. `offset` is a headline's `lineSpan.start` — the universal way to address a headline across screens, widgets, and notification deep links (via `MainActivity.EXTRA_OPEN_PATH`/`EXTRA_OPEN_OFFSET` → `PendingNavigation`). The `edit` flag on that route means **open there and start typing**: `DocumentScreen` places the caret at `offset` and raises the keyboard once the editor holds the file, once per destination. It rides the destination rather than going through `PendingNavigation.requestCaret` because that request only reaches a document that is *already* open — when the file being opened is the one already showing, the composition the navigation is about to replace consumes it first and the new one arrives with nothing left to place. The capture screen is reached the same way: `MainActivity.EXTRA_CAPTURE_TEMPLATE` → `PendingNavigation.requestCapture` → the `capture` route (a blank template id shows the template picker). (`SettingsStore.startScreen`/`StartScreen` is now unused — a single screen has no launch-tab choice.)

### Document view (WebView + the `doc-engine` live-preview editor)

`DocumentScreen` is a **WebView** hosting our own live-preview editor. There is no CodeMirror (or any other
third-party editor) any more; the old native Compose renderer in `OrgRendering.kt`/`OrgContentParser` is likewise
not wired into this screen — it is dead until reused elsewhere.

The web front end lives in `web/`:

```
web/packages/doc-engine/   the editor engine — standalone, dependency-free, app-agnostic
  src/text.js              immutable buffer + line index + minimalEdit()
  src/deco.js              decoration kinds (hide / mark / replace / rowClass / affix)
  src/render.js            row -> DOM, with signature-keyed row reuse
  src/dom.js               source offset <-> DOM position mapping
  src/fold.js  history.js  fold ranges; undo/redo over splices
  src/engine.js            the editor: input, selection, rendering, commit pipeline
  src/lang/org.js          the org language plugin (the only org-aware file)
  test/engine-test.mjs     `npm test` — the DOM-free parts
web/src/main.mjs           app glue: Android bridge, KaTeX, vault images, window.orgApi
web/test/browser-test.mjs  `npm run test:browser` — the built bundle in headless Chromium
```

`doc-engine` is deliberately **packageable on its own** (its own `package.json`, README and tests, no imports
outside itself): markup support is a *language plugin* and persistence/links/math are a *host*, so a markdown
plugin is an added file rather than an engine change. `web/src/main.mjs` imports it by relative path, so no npm
workspace setup is needed. Everything is bundled **offline** into `app/src/main/assets/editor/` by `web/build.mjs`
(esbuild → one IIFE `editor.js`, plus `document-editor.html` and vendored KaTeX css/fonts); run `cd web && npm run
build` after editing anything under `web/src/` **or** `web/packages/`, then rebuild the APK so the packaged asset
updates (the assets are committed so a plain Gradle build needs no npm). `WebViewAssetLoader` serves the bundle
from `https://appassets.androidplatform.net/assets/editor/`, so there is **no network dependency and no
CDN/import-map at runtime**.

**The rendering invariant.** The buffer is the file source **verbatim**, so a document offset equals the
corresponding `OrgHeadline.lineSpan.start`. Live preview hides markup by wrapping it in `display:none` spans
rather than removing it, so **the text nodes of a row always concatenate to that row's source, character for
character** (only `data-deco` elements — chevrons, pencils, widget bodies — are outside the text, and the offset
walk skips exactly those). Everything else follows: the caret can sit at any source offset, `offsetFromDOM` is a
walk rather than a guess, and an IME composition can be read back out of the DOM exactly. Do not "optimize" a
hidden marker away.

**The soft keyboard and widgets (hard-won).** A widget must **not** be `contenteditable=false`. A soft keyboard
can walk the caret through the document with no DOM input event at all — GBoard's swipe along the spacebar does
this via repeated `InputConnection.setSelection` — and when that caret crosses a non-editable island the WebView
reports a non-editable focus, so Android hides the keyboard (visible in logcat as `ImeTracker … onRequestHide at
ORIGIN_CLIENT reason HIDE_SOFT_INPUT` from the app's own pid). `DocumentScreen`'s keyboard-hidden collector then
does its documented job and blurs, so the editing session ends under the user's finger — approaching a link or a
formula from the right would simply drop you out of the editor, while approaching from the left (which stops at
the plain-text boundary before the widget) was fine. Nothing is lost by letting the widget inherit editability:
the offset walk skips `data-deco` either way, every input is intercepted at `beforeinput` and applied to the
model, and a caret that does land inside a widget is written back out to the construct's edge. Verified on device
over the CDP endpoint (`adb forward tcp:… localabstract:webview_devtools_remote_<pid>`); the same crossing driven
by arrow keys never reproduced it, only a real IME gesture did.

**Which edge a widget is entered at.** A widget covers its whole construct, so a caret inside one says only
"here", not "where" — and for a table or a display formula, which occupy entire rows, that is the *only* kind of
position caret movement can produce. `Engine._offsetAt` therefore resolves such a position to the edge the caret
arrived from: the construct's **start** when coming from before it (from the left, or from the line above), its
**end** when coming from after it (from the right, or from the line below), comparing against where that endpoint
last was. `decoRangeAt` recovers the construct's range by pairing the widget with its source span — they carry the
same row-relative offset (`data-w` / `data-src`), which is what keeps the pairing correct after a row element has
been recycled to another position. Reveal runs *before* the rescue write, deliberately: while the construct is
still collapsed its source is hidden, and a row that is nothing but a widget has no visible text for the caret to
land in at all.

**A construct the browser steps over.** Hidden source offers the caret no position to stop at, so one arriving
from beyond a widget crosses the whole construct in a single move and the browser reports its **start** — the far
edge from where it came, which reads as a jump to the front of it. (A checkbox is the sharpest case: its widget is
an SVG with no text of its own, so unlike a link there is nothing inside for the caret to land in either, and the
construct could only ever be entered from the left.) `Engine._offsetAt` answers with the near edge instead when
the position it is handed is a hidden `replace`'s start and that endpoint was last at or beyond its end, so a
construct is entered from the right exactly as it is from the left and the caret's walk stays continuous. Only a
`replace` counts: a bare `hide` (a heading's stars) has a reachable start of its own, and answering past it would
move what a Backspace struck there deletes. The model then says something the browser did not, so the
`selectionchange` handler **reads the selection back after revealing** and writes the model in when the two
disagree — otherwise the caret is drawn in one place and the next arrow key moves from another. A caret parked
inside a widget is caught by `_selectionStranded`; this one is not stranded at all, which is why the test is a
read-back rather than a look at where the node sits.

A DOM position *inside* a `data-deco` element is the one case with no source of its own, and it happens for real:
a widget covers its construct, so hit-testing a tap or a caret-handle drag at its right edge can land the
selection in the widget's own DOM, and so can a soft keyboard's cursor gesture. It resolves **by document
order** — the end of the last source run before it,
which for a widget is the end of the construct it covers, since a `replace` renders its hidden source immediately
before the widget. Answering with the row start (the old fallback) threw the caret to the beginning of the line
whenever it arrived at a link or a formula from behind, and on Android a caret that lands nowhere drops the IME,
which the keyboard-hidden collector then turns into a blur — the editing session simply ended. `[data-deco]` is
also `user-select: none`, so the browser prefers a position beside the widget in the first place; the offset rule
is what makes the remaining cases land somewhere true.

An **affix** (a heading's fold and edit buttons) is the other `data-deco` shape, and it stands for no source at
all rather than for a construct — so the caret **passes through** it in the direction it is travelling
(`affixRowAt` + `Engine._offsetAt`). Which way "through" points depends on the end of the row it stands at, which
`render.js` records as `data-affix`: a **leading** affix (org's two heading buttons) sits before the row's first
character, so the caret arriving from the line above enters the row and one arriving from within it leaves
backwards; a **trailing** one sits after the last, so the directions are the mirror image. Resting in one would
make each button a caret stop to arrow past. The buttons also carry **no text node** (an inline SVG has none),
which is what keeps ordinary arrow keys from stepping into them in the first place; the direction rule covers taps and IME gestures. They are still never
`contenteditable=false` — that rule holds for every `data-deco`.

The end of a **range** never passes through — `_offsetAt` takes the selection's `collapsed` state and clamps a
non-collapsed endpoint to the row's own edge (leading affix → `row.from`, trailing → `row.to`). The fringe is
exactly where a selection handle dragged left to take in a heading's stars ends up, and stepping out of the row
from there put the newline *above* the heading inside the selection: deleting the heading then took the line
break with it and pulled the heading onto the line before.

**Reveal is per construct, not per row.** Standing in one emphasis shows *that* emphasis's markers and leaves the
rest of the line rendered. A decoration's construct is its own range unless it names a wider one
(`hide(from, to, reveal)` — both `*` of an emphasis reveal together), and **a heading's stars name the whole
heading line**: editing the title shows the marker, so the level being typed under is visible while it is edited,
and a selection dragged over the heading takes in text that is on screen rather than markup that is not. The engine
decides this centrally in `_lineDecorations`, so `decorateLine` knows nothing about the caret: a revealed `hide`
becomes a dimmed `mark`, a revealed `replace` is dropped in favour of its source. Multi-line blocks (tables,
display math) stay row-level, since a decoration cannot cross a line break: touching one re-emits its lines as
source rows. The consequences that are easy to break:

- `_lineDecorations` also **resolves overlaps** (`normalize`) before the spec is stored, so a row's decorations are
  exactly what it renders. A decoration the renderer would drop must not go on claiming a construct: the caret
  standing in it asks to reveal source that was never painted, `updateReveals` cannot find it, and the row is
  rebuilt under the caret. That is not hypothetical — `***` matches the emphasis pattern (a `*` wrapped in a pair
  of `*`), so a level-3 heading carried a phantom bold over its own stars, and arrowing into one dropped the caret
  onto the fold chevron and then out of the document. `decorateLine` therefore matches inline constructs over the
  line's *content* — after a heading's stars — and the engine normalizes on top of that.
- The reveal test (`_touches`) is **inclusive at both ends**. A widget covers its own source, so the only position
  a link or a formula can be entered from is its boundary offset — an exclusive test would make hidden-source
  constructs unreachable by arrow key.
- A widget can also take the click itself: `replace(…, edit)` (and every block, unconditionally) hands the caret
  to the source it stands for via `api.select`. That is the only way into a widget with a pointer. Leave `edit`
  off where the click already means something else, as a link's does; a widget that is `edit`-flagged must not
  consume the click itself, or the two handlers both fire.
- **A widget is never `contenteditable=false`** (hard-won — see the IME note below). It carries `data-deco` and
  nothing else, inheriting the host's editability.
- Flipping a construct is done **in place** (`updateReveals`): a `hide`/`replace` always renders its source into
  the same span and its widget into the same element, so revealing is a class change (`de-hidden` ⟷ `de-marker`,
  and `de-off` on the widget) with no node created, moved or destroyed. The caret keeps its text node, so the
  selection does not have to be rewritten afterwards — it is written back only when the flip stranded it somewhere
  unpainted (`_selectionStranded`), or when the in-place flip failed and the row had to be rebuilt after all. Only
  a *block* reveal is structural (one row becomes several and back) and goes through `_renderRange`; that is what
  `row.reveals` tracks, while inline state rides on the decorations.
- Reveal is gated on editor focus as well as on the selection: blur re-renders, or `*bold*` and heading stars stay
  on screen with nothing editing them.
- Revealing (or re-hiding) **replaces the row's element**, which destroys the browser's selection inside it, so
  `_refreshReveals` writes the model's selection back afterwards. Without that the caret collapses to the top of
  the document on the first tap into a heading. Each row spec records the construct ranges it asked about and the
  answers it got (`row.reveals`), so a selection move re-tests a handful of ranges instead of re-parsing, and
  rebuilds the constructs the caret left and entered as separate runs — never the span between them, which on a
  distant tap is the whole file.

`lang/org.js` matches org by hand (line + inline regexes — org has no ready incremental grammar, and folded or
off-screen lines aren't parsed anyway): headings (stars hidden, styled by level, led by a **fold button** and an
**edit button** — flat inlined Material paths, sized in `rem` so the heading's font size does not scale them; the
fold glyph *is* the state, `chevron_right` collapsed / `expand_more` open / dimmed `expand_more` for a heading
with nothing to fold. Both are `side: -1` affixes, but only
the pencil takes room in the line, at the head of the heading text where the stars were: the fold toggle is
lifted out of the flow into the **left fringe** (`.de-fold-toggle` is `position: absolute; right: 100%`, filling
the content's own `--fringe` left padding), so its hit area owns the whole gutter for the height of the row —
the largest target the layout has — at no cost to the heading. The two `::after` hit areas are cut so they never
overlap: the gutter is the fold's, whole, and the pencil's grows only rightwards. The pencil is the host's to
switch off (`host.headingEditButton`, `orgApi.setEditButton`, Settings → Document); the fold toggle is not. The
toggle is the **only** thing that folds: the rest of a heading row, empty run past the end included, belongs to
the text, so a press out there is an ordinary click that puts the caret at the end of the heading and lets it be
edited in place), inline emphasis
(`*`/`/`/`_`/`=`/`~`/`+`), `[[target][desc]]` links, inline `\(…\)`/`$…$` KaTeX math, image links rendered as `<img>`, and muted metadata lines (`#+…`, planning, drawers) — of which a
**drawer's opening line** also carries the fringe toggle (`drawerFoldRange`: `:NAME:` line end → `:END:` line end,
null when nothing closes it before the next heading, since an unterminated `:NAME:` is just text and folding it
would swallow the entry). It is the same `foldAffordances` a heading gets, minus the outline spacing. Multi-line
constructs — display math (`\[…\]`/`$$…$$`/`\begin{}…\end{}`) and org tables — come from `blocks()` instead,
because **a decoration may not cross a line break**; `blocks()` walks back over a table when a re-render window
opens mid-table so a partial table is never emitted. Math (inline, nested and display), images and tables are
`edit`-flagged, so tapping one reveals its source and puts the caret in it. Tapping an image therefore **edits**
it rather than opening it (which is what reaches the `[[…]]` behind a broken image); only ordinary links still
open on tap.

A **table cell renders inline markup** — emphasis, links, images, inline math — through `fillInline`/`inlineRuns`,
the same three passes `decorateLine` makes, resolved into a list of non-overlapping runs instead of emitted as
decorations. A widget has no source on screen to keep in step with, so inside one the markers are **dropped**
rather than hidden; the way to the real text is to tap the table, which reveals it like any other block. Overlaps
are settled leftmost-and-longest, mirroring the engine's `normalize` — which is also what keeps `=verbatim=` and
`~code~` literal, since the run covers its whole construct. `linkElement` therefore stops the press propagating:
inside an `edit`-flagged widget it would otherwise open the link *and* reveal the table's source (the engine's own
press tracking listens in the capture phase, so it still sees it).

**A heading's trailing tags are formatted, not just carried.** `* Title :work:home:` — each tag becomes a small
quiet chip (`de-org-tag`) and the colons that separate them recede (`de-org-tag-sep`), both `mark`s, so every
character stays its own source text the way the TODO keyword does. `HEAD_TAGS` in `lang/org.js` is deliberately
the same shape as `OrgParser`'s own `TAGS` (whitespace, `:word:` runs, end of line): the two halves of the app
must agree on what a tag *is*, or the editor would format something the parser does not report. The tag run also
**ends the line's content** for the inline passes — `contentEnd`, the mirror of `starsEnd` — since `:a_b_c:` is a
tag and not an underline, and brackets in there are not a link.

**A priority cookie is `#A` with its brackets out of the way.** `* TODO [#A] Title` — `[` and `]` recede like a
tag's colons (`de-org-priority-mark`) and what is left reads as the agenda's own priority does, `--error` in bold
(`de-org-priority`), so one priority looks like one thing wherever it is read. Marks again, so every character
stays its own source text. `HEAD_PRIORITY` is the same shape as `OrgParser`'s `PRIORITY` for the same reason
`HEAD_TAGS` is, and it is matched exactly where org puts it — after the TODO keyword if there is one, before the
title — so a cookie written anywhere else in the line stays title text. Unlike the chips it is sized in `em`: it
is a word in the heading rather than a label beside it, and it should follow the heading's own size.

**TODO keywords are chips, checkboxes are boxes.** A heading's keyword is drawn with a `mark`, not a widget —
the word stays its own source text, so it is still selected, typed over and read back like any other, and only
the box around it is ours. Which words count comes from the **host**, not a guess in the page:
`orgApi.setTodoKeywords({active, done})` is pushed from `DocumentScreen` out of `file.doc.todoConfig`, so the
editor agrees with `OrgParser` about `#+TODO:` sequences (the page's own default is org's `TODO | DONE`). A
`:COLOR:` property on the heading (read from the drawer that must follow it, org's own rule) tints the chip —
the same palette, hex forms and luminance contrast rule as `OrgColors.kt`, ported into `lang/org.js` and kept in
step with it — arriving as `--todo-bg`/`--todo-fg` row variables the chip's CSS falls back from. Without one
the chip is the **agenda screen's chip**: the theme's own `--todo-default-bg`/`-fg` and `--todo-done-bg`/`-fg`,
which are the same two colours Kotlin lifts into `errorContainer`/`onErrorContainer` and
`secondaryContainer`/`onSecondaryContainer` for `headlineChipColors` (see **Themes** below), plus the same 8dp
corner and bold sans label — one keyword should look like one thing wherever it is read. Those properties are
named apart from `--todo-bg`/`--todo-fg` on purpose: a `:COLOR:` sets the latter on the row, so defining them at
`:root` too would leave the row-level fallback chain nothing to fall back to (and paint done chips in the active
colour). A list
checkbox is the one widget that is a **control** rather than a view of one: it covers `[ ]`/`[X]`/`[-]`, and a
tap rewrites its own state character through `api.replaceRange` (the ordinary splice), so the toggle undoes,
coalesces and reaches Kotlin as the `checkbox` edit context it already knows. That press passes
`{keepSelection: true}`: pressing a control is not a caret move, and a caret dropped inside the box would reveal
the very `[ ]` the widget stands in for — the source flashing up on every toggle.

**Blocks and drawers are structure, and are resolved before anything else in `decorateLine`.** `enclosing()`
walks back from a line to the first thing that settles the question — a `#+begin_`/`#+end_`, a drawer opener or
`:END:`, or a heading — gated on the line's first non-blank character being `#`, `:` or `*`, so plain prose
costs one character test per line looked at (capped at `CONTEXT_SCAN` lines so a pathological file cannot make a
render quadratic). A **drawer's contents** then read as the muted `:PROPERTIES:` line that opens them
(`de-org-meta`, no inline preview). A **block** puts its classes on every one of its rows — its `#+begin_`/
`#+end_` lines included, with the ends rounded off — so it renders as one slab with the delimiters as its edges;
quote and verse are prose (left rule, no slab) and keep their org markup, while the **verbatim** kinds (`src`,
`example`, `export`, `comment`) have none: no emphasis, no links, and `blocks()` skips them too, so a `| a | b |`
inside a source block stays code instead of becoming a table. `src` gets **light** highlighting — comments,
strings, numbers and a language's keywords from a small alias table (`CODE`/`CODE_ALIAS`), lexed per line
because a decoration cannot cross a line break, so multi-line state (a block comment, a heredoc) is out of
scope by construction; an unknown language still gets strings and numbers. The `de-code-*` colors come from the
active theme (`--code-comment`/`--code-kw`/`--code-str`/`--code-num`), so a block is highlighted in the palette
the file is read in. A heading written *inside* a block ends that block's styling early — cosmetic, and the price of
not scanning the file.

**Plain lists.** A bullet (`-`/`+`/an indented `*`/`1.`/`1)`) is org's own markup and stays exactly as written,
merely dimmed (`.de-org-bullet`) — nothing is replaced by a glyph of ours. What the row does get is a **hanging
indent**, so a wrapped item lines up under its own text instead of running back to the margin: the bullet's
rendered width goes on the row as `padding-left` with the same amount of negative `text-indent`, which leaves the
first line exactly where it was. In a proportional font only the DOM can say how wide that prefix is, so the
engine measures it (`api.measure`, memoized against a hidden probe in the scroller, dropped on every full render)
and the language emits it as `rowStyle` — a decoration kind for layout a class cannot express because it depends
on the row's own content. A host that cannot measure simply gets no hanging indent. **Enter continues the list**
through the language's `newline(doc, at)` hook (`{from, to, insert}` or null, consulted for a collapsed caret
only): same indentation, the bullet at that level (counted on for an ordered list), an empty `[ ]` when the item
had a checkbox — and an item with no text of its own has its bullet **cleared** instead of duplicated, which is
the only way out of a list that leaves no empty item behind. A break struck inside the bullet itself is an
ordinary break.

Rendering is **incremental**: an edit re-renders only the rows it overlaps (±1 row of slack) and shifts the rest
by the delta; rows are recycled by a signature of their rendered form. If the rebuilt window fails to join the
tail exactly (a block grew across the boundary) the engine falls back to a full render, which is why that check
must stay. That signature is deliberately **position-independent** (decoration offsets are relative to the row),
so two identical headings swap elements on a re-render: a widget handler must therefore read its offset off the
row it is currently in (`rowOffsetOf`) instead of closing over the `line.from` it was built at — otherwise the
edit button edits, the fold button folds, and a clicked formula opens whichever twin happened to render first. A
row the browser has written into behind the engine's back (an IME composition, an input that outran `beforeinput`) is
marked `data-dirty` and kept out of the pool, since its signature no longer describes what it holds.

Inline images: the editor's `<img>` fetches `https://appassets.androidplatform.net/__orgimg__/?p=<link>`;
`DocumentScreen`'s `WebViewClient.shouldInterceptRequest` catches that sentinel path and streams the bytes from
`VaultManager.imageBytes(relPath, rawTarget)` (SAF `content://` via ContentResolver, or WebDAV via
`WebDavOrgFileStore.readBinary` on the auth'd OkHttp client), guessing the MIME from the extension and returning a
404 response (not null) on failure so the WebView doesn't hit the network. Remote `http(s)` image links load
directly. The SAF side goes through `SafOrgFileStore.resolveUri`, **not** `uriFor`: the persistent source cache
publishes a document before the SAF walk has indexed anything, so on a cold start an inline image would resolve
against an empty `docIds` and 404 until the file was opened a second time. `resolveUri` falls back to a targeted
walk (one child query per path segment) instead of waiting for the listing.

Bridge (`OrgBridge`, a `@JavascriptInterface` marshalled to the main thread): JS→Kotlin
`ready`/`applyEdit(json)`/`editHeading(offset)`/`linkClick(target)`/`caretMoved(offset)` (where the caret
is — see the resting caret below)/`scrolled(dir)` (scroll direction, drives the
auto-hiding bottom chip)/`focusChanged(focused)` (editor focus, drives the Back-to-exit-editing behavior);
Kotlin→JS `window.orgApi.load/reload/setVisibility/setTheme/setThemeCss/setFont/setEditButton/setTodoKeywords/setFooter/focus/setCaret/showAt/cmd/blur/flush/text` (`setCaret(offset)` = `Engine.setCaret`: select, focus, reveal — how a command that edited the file says "carry on typing here"; `cmd(name)` = `Engine.command`, what the editing toolbar is wired to; `showAt(offset)` = `Engine.showAt`, the same move **without focus**, which is how the search prompt walks the document while holding the keyboard itself) via
`evaluateJavascript`. The system Back button exits editing before it navigates, and a single Back must both drop the keyboard and blur (never a
two-step). Three cooperating paths, because Back reaches the app differently depending on how it's issued: (1)
**button/key Back** while the keyboard is up — the WebView's `onKeyPreIme` catches it *before* the IME can swallow
it, consuming the **whole** gesture (both DOWN and UP; if DOWN falls through, the IME self-hides on the down edge
and tears down its input connection so the UP never returns) and calling `orgApi.blur()` on DOWN. It is **gated on
`editing`** so that once blurred — or with a physical keyboard, whose input connection persists with no visible
IME — it stops swallowing Back and navigation is possible; (2) **edge-swipe (gesture) Back** produces no
`KeyEvent` at all, and while the keyboard is up the system IME window consumes the swipe just to dismiss the
keyboard — so instead a `snapshotFlow { WindowInsets.ime.getBottom(density) > 0 }` fires `orgApi.blur()` whenever
the keyboard goes shown→hidden by any means; (3) a Compose `BackHandler` (enabled while `editing`) covers Back
with no IME up — a gesture/button Back after the keyboard is down, or a **physical keyboard** where the editor is
focused but no soft keyboard ever showed. `editing` is editor focus tracked via the `focusChanged` bridge,
independent of IME visibility, so it is true in the physical-keyboard case; it is a plain `MutableState` so
`onKeyPreIme` can read its current value. A subsequent Back then navigates normally. The **edit button** posts
`editHeading(offset)`; Kotlin locates the headline by `lineSpan.start` and opens the same bottom
`HeadlineEditSheet` (state/priority/dates/tags/category/color/reminders) the native renderer used. It carries
**no title field**: heading text is edited in place in the document, so the sheet is for everything a heading
holds *besides* its text (`VaultActions.setTitle` went with it — `applyEngineEdit` reaches `OrgEdits.setTitle`
directly). `setThemeCss(css)` hands the page the active theme's stylesheet, and `setTheme` now carries only what
is *not* a theme — the room the floating bar covers (see **Themes** below). `setEditButton(on)` mirrors `SettingsStore.headingEditButton`
(Settings → Document, default on) into the page — `DocumentScreen` pushes it once the page is ready and on every
change; the engine's `refresh()` re-renders from its own model, so the buffer, the caret and the folds are
untouched.

**What a delete removes comes from the model, not from the browser (hard-won).** Android's IME deletes a
selection in two steps: it collapses the selection to one end, then asks for a deletion so many characters wide —
and that width was counted against the **rendered** text as it stood *before* the collapse. The collapse re-hides
whatever markup the selection had revealed (the stars of a heading it reached into), so the same count now reaches
further back than it did, by exactly the markup that disappeared, and `beforeinput`'s own `getTargetRanges()`
carries the shifted range. Measured on a device: a 56-character selection ending at offset 76 arrived as a target
range of `[16, 76)` once `*** ` had hidden itself again — the delete ate the end of the heading *above* the
selection. The engine's selection is in source offsets, which no re-render can move, so `Engine._imeTarget` makes
it the authority: a delete arriving while the model holds a range removes that range, and a **wide** delete
arriving on a caret sitting at an end of the range the engine last held (`_imeRange`, kept by `_rememberRange`
across exactly that collapse) removes that range. An ordinary one-character backspace is left alone, and the
memory is dropped by anything the user does himself — a press, a real key (an IME's own key events identify no
key: `keyCode` 229), a blur, or the edit itself — so it only ever answers for a collapse the platform made on its
own.

**Pasted text arrives in `data`, not in `dataTransfer` (hard-won).** The content element is
`contenteditable="plaintext-only"`, and for such a host Chromium puts the pasted string in `beforeinput`'s
`data` and gives the event **no `dataTransfer` at all** — the mirror image of an ordinary contenteditable, where
`data` is null and the transfer carries it. Reading only the transfer therefore pasted the empty string, which
`_apply` discarded as a no-op splice: paste did nothing whatever, in the app and in the standalone page alike.
`_onBeforeInput` takes whichever the browser filled in (`e.data ?? e.dataTransfer?.getData('text/plain')`), and
`browser-test.mjs` dispatches that exact event shape.

**Edits are incremental, never whole-buffer.** The engine coalesces a 600ms debounce window of keystrokes into the
*single smallest splice* that turns the last-reported text into the current one (`minimalEdit`, common
prefix/suffix) and posts `applyEdit({from, to, insert, expect, context})`. `expect` is the text the editor
believed occupied `[from, to)`. `VaultActions.applyEngineEdit` re-checks it inside `OrgVault.edit` and **refuses**
the write when it no longer matches (external sync landed in between), returning false so `DocumentScreen` bumps
`reloadTick` and re-seeds the editor instead of splicing at an offset that no longer means what it did. Never
replace this guard with a blind splice. `context` (from `orgLanguage.editContext`) says whether the splice landed
on a heading line, in a body, on a checkbox, or across lines; `applyEngineEdit` uses it to express the change as
the org operation it actually is — `OrgEdits.setTitle(trim = false)` when the range sits inside `titleSpan`,
`OrgEdits.replaceBody` when it sits inside `bodySpan` *and* the body's trailing-newline normalization would be a
no-op — falling through to a plain range splice otherwise. Every route is byte-exact outside `[from, to)`, which
is the only invariant that matters; `setTitle`'s `trim` parameter exists precisely so a keystroke's trailing space
is not silently swallowed. Kotlin mirrors each landed splice into `editorText` so the reconcile
`LaunchedEffect(file.source, …, reloadTick)` can still tell the editor's own writes (skip) from external/sheet
edits (push in via `orgApi.reload`) without clobbering newer in-flight typing.

**The editing toolbar, and commands as splices.** While the editor holds focus the bottom chip is replaced by
`EditToolbar` (`DocumentScreen.kt`) — undo, redo, bold, italic, plain/ordered/checkbox lists, heading, dedent,
indent, and the command palette — a horizontally scrolling pill in the chip's own place, `BOTTOM_BAR_GAP` off the
bottom edge. Unlike the chip and the top bar it is **not** tied to `chromeVisible`: it stays put while the
document scrolls under the keyboard, since that is exactly when it is wanted. Settings → Document → *Editing
toolbar* (`SettingsStore.editToolbar`, default on) turns it off and keeps the chip. `EDIT_TOOLBAR` is a plain list
of `ToolbarItem`s, so a button is one entry plus one command.

Every button but the palette is `orgApi.cmd(name)` → **`Engine.command(name)`**: `undo`/`redo` are the engine's
own, everything else is `orgLanguage.commands[name]` — a **pure function** of `(doc, {from, to}, api)` answering
with **one splice** plus the selection to leave behind, applied through the ordinary `_apply`. So a button press
undoes, coalesces, and reaches `VaultActions.applyEngineEdit` exactly as typing does; nothing about a command
touches the DOM, which is what makes the whole set unit-testable (`engine-test.mjs`) with the browser test
covering the path through the live editor. They write **org's own markup** and toggle back off to the line as it
was: `*bold*`/`/italic/` (the selection, or the word the caret stands in, or an empty pair; markers are recognized
inside *or* just outside the selection), `- `/`1. `/`- [ ] ` over the selected lines (a mixed selection turns the
list *on*, headings and blank lines are skipped, an ordered list is recounted), a heading at the level of the
nearest heading above (so the button writes a *sibling*), and indent/dedent as org's `org-metaright`/
`org-metaleft` — a heading demotes/promotes by one star (never below level 1), anything else shifts by two
spaces. On an **empty line** a list or heading command has nothing to convert, so it *starts* one instead: the
bare marker is written (a list keeps the line's indentation; a heading's stars go to column 0, as org requires)
and the caret lands after it, ready for the item to be typed. That is only when the range holds nothing else to
act on — a blank line between two items is still skipped, or toggling a list on would fill the gaps in it with
empty bullets.

**Neither bottom bar carries a navigation-bar inset** (`Modifier.imeInset`/`bottomBarInsets`), the rule
`DocumentBar` already follows at the top: `AppRoot`'s Scaffold has padded the NavHost for the system bars, so a
`navigationBarsPadding()` here counts the gesture bar twice and leaves the bar floating a finger's width off the
edge. For the same reason the keyboard inset excludes the navigation bar — `WindowInsets.ime` is measured from the
*window's* bottom and swallows the nav bar the container is already lifted by, so with both, a bar riding the
keyboard sits a gesture bar's height above it. The WebView takes the same inset, or its bottom edge floats there
instead. What is left is `BOTTOM_BAR_GAP`.

Because the toolbar stands over the foot of the page, `DocumentScreen` pushes its height in as `--foot`
(`orgApi.setFooter`, 0 when it goes) and every caret reveal clears `CARET_ROOM + footRoom()` rather than
`CARET_ROOM`. The keyboard announces itself as a viewport resize; the toolbar does not, so without this a line
typed at the foot of the screen would sit behind it. `setFooter` re-reveals, so the toolbar appearing scrolls the
caret out from under it.

Two things keep a press from ending the editing session, since the toolbar lives in the app's chrome rather than
in the page. `orgApi.cmd` **focuses before it acts on the selection** and reads that selection from the engine's
model when the page is not focused (a blur clears the browser's), so a command lands where the caret was either
way. And the Kotlin side treats a keyboard dismissal within `TOOLBAR_PRESS_WINDOW_MS` of a toolbar press as that
press's doing rather than the user's, so the IME-hidden collector does not blur the editor out from under the
button; the toolbar itself outlives a blur by `TOOLBAR_BLUR_GRACE_MS`, so a focus lost for a frame does not swap
the bar out under the finger.

**The top bar is our own** (`ui/ScreenBar.kt`), not a Material `TopAppBar`: an `APP_BAR_HEIGHT` (48dp) `Surface`
row plus a hairline, a `navigation` slot, the title (`titleMedium`, ellipsized) and `RowScope` actions. It is
**every screen's bar** — Agenda, Calendar, Help and the raw editor put it in their `Scaffold(topBar = …)` with
`navigation = { BackAction(…) }`, and the document view wraps it in the `AnimatedVisibility` that floats it (the
`DocumentBar` below). One bar, so one set of proportions; a Material `TopAppBar` is 64dp *plus* the status-bar
inset it re-applies, which is why every screen that used one stood a third taller than the document's. (Settings wears it too, its template form included; Capture is the last `TopAppBar` left.)

`DocumentBar` is that bar holding a **hamburger** (opens the sidebar folder tree), the title, the **global
visibility cycle**, and a **three-dot overflow menu** (`DocumentMenu`, built on the shared `OverflowMenu` in
`ui/ScreenBar.kt` — it owns its own open state and hands each item the dismissal). The agenda's bar ends with the
same menu, holding **Refresh** and **Settings** — errands, rather than ways of reading what is on screen, which
leaves that bar the view toggle alone. It has no Search button either: Search is the bottom chip's, on every
screen that carries one. The document's menu holds what is a *destination* rather than a way of looking at the file in front
of you: the whole-file raw editor (`RawEditorScreen`, still
`replaceFile` — a raw editor genuinely does replace the file), which keeps its **`code_xml`** glyph
(`@drawable/ic_code_xml`, the Material Symbol not present in the frozen `material-icons-extended` artifact —
vendored as a `0 -960 960 960` vector shifted by a `translateY` group) as the item's leading icon, and **Help**
(the `help` index route). The visibility cycle stays a button of its own beside the menu, since its icon *is* the
state and it is read as much as it is pressed. Three things about the bar are load-bearing:

- It **floats over the WebView** and slides away while the reader scrolls down, on the same `scrolled(dir)` signal
  as the bottom chip (`chromeVisible` in `DocumentScreen` drives both). The room it occupies is the page's own top
  padding — `--head`, pushed in with the theme vars, in CSS px, which a WebView at `textZoom` 100 maps 1:1 onto dp
  — so it is part of the scrollable content and scrolls away with the text. **Never make the WebView shrink to fit
  the bar:** resizing it re-lays out the whole page and re-pins its JS-computed height, mid-scroll, every frame of
  the animation.
- Because the bar covers the top of the viewport, `orgApi.load`'s scroll-to-heading passes that same `--head` as
  `Engine.scrollTo(offset, margin)`, or the heading a deep link asked for lands behind the bar. That scroll reports
  **nothing** over `onScroll` (`Engine._silentScroll`): a jump the host asked for is not the reader scrolling, and
  without the guard every document opened at an offset would start with its chrome already hidden.
- `ScreenBar` carries **no window insets of its own**. `AppRoot`'s Scaffold has already padded the NavHost for the
  system bars, and a `TopAppBar` re-applying its own status-bar inset on top of that is what made the old bars so
  tall. Its own `BAR_EDGE_INSET` (10dp) is what lines the outermost icons up with the status bar's clock and
  battery: an `IconButton` centres a 24dp glyph in a 48dp target, so hard against the screen edge it sits further
  out than the system's own icons do.

**The top bar's storage indicator.** `StorageStatusAction` (`ui/StorageStatus.kt`) leads the bar's actions and
is **absent while there is nothing to say** — a vault that answers, holds no queue and has nothing to report needs
no icon, and a permanent green tick would only train the reader to stop looking at the corner the warning appears
in. When it is there its glyph *is* the state, the way the visibility cycle's is: `CloudOff` (the backend did not
answer; anything typed since is queued and on disk), `CloudUpload` (it answers again and edits are still waiting),
`CallMerge` (offline edits were reconciled with the server's own changes and both sides are in step — reported
rather than swallowed, since the file is now neither version anybody typed, and cleared by `acknowledgeMerges()`
when the dialog is closed), and `SyncProblem` tinted `error`, which outranks the rest because it is the only state
waiting on a person. The badge counts **files**, not edits. A press retries a plain queue and opens
`StorageStatusDialog` for the two states that are there to be read; that dialog names the files and leads to the
conflict screen rather than offering a whole-version answer inline. When the file *on screen* is the conflicted
one there is nothing to pick from, so the press goes straight to settling it — and that bar **stops auto-hiding**
(`visible = barVisible || conflicted`): the indicator is the only thing on the document view that says the
reader's edits are not reaching the vault, and scrolling it away while typing into that very file is how somebody
works for an hour without knowing. The **sidebar marks the files themselves** (`VaultTree`'s `SyncMark`): a small
`CloudUpload` for one waiting on a network, an error-tinted `SyncProblem` for one waiting on a decision — the top
bar says the vault has something outstanding, and only the tree says *which file*. The **agenda's bar carries the
same pair**:
marking a TODO done there is an edit like any other, and the indicator has to be wherever an edit can be made.
Both read `OrgVault.storage`/`OrgVault.pending` directly, so nothing has to be told about an edit.

**Settling a conflict** is its own screen (`ui/conflict/ConflictScreen.kt`, route `conflict/{path}`, pushed by
`NavHostController.openConflict`). Only the regions that actually clash are asked about — everything the merge
placed on its own is already decided, which is the point of merging at all: a file edited in two places usually
clashes in one small part of itself, and choosing between two whole versions would throw away a side's work over
a line. Each region offers **Mine**, **The server's** and **Keep both**, shows the original underneath when there
was one, and every version has a **copy button**: the decision is final for the file and the version not chosen
has nowhere else to live, so copying it out first is how a paragraph that really wants reconciling by hand
survives to be pasted back in the editor. Apply stays disabled until every region has an answer — defaulting them
to "mine" would be one tap from silently discarding the server's work — with *All mine* / *All the server's* for
when that genuinely is the answer. Applying calls `OrgVault.resolveConflict`, which refuses if the server moved
again while the screen was open; the screen then reloads the merge rather than writing a decision about text that
has been replaced.

Startup / visibility folding: on the initial seed only, Kotlin passes `orgApi.load(text, offset,
folding)` a `{fold, body}` spec built from `OrgVisibility.initial(file.doc)` (offsets are `lineSpan.start`), where
`fold` entries collapse a whole subtree (the engine's `foldRange`) and `body` entries collapse only a heading's
own body while its child headings stay visible (`bodyFoldRange`, which stops at the first heading). This realizes
`#+STARTUP: overview/contents/showall` and per-heading `:VISIBILITY: folded/children/content/all`.
`orgApi.reload` (external/sheet edits) does **not** re-apply folding. The `offset` that seed carries is the route's
`targetOffset` when there is one — an agenda item, a link, a notification always opens at the heading it points to —
and otherwise the caret this file was last left at (`SettingsStore.cursorPosition`), which is what makes reopening a
document resume where the reader stopped. **That offset then opens whatever the spec folded over it**
(`Engine.unfoldAt`, org's `org-show-context`): the heading a link or a notification points at is very often inside
a subtree `#+STARTUP: overview` has just collapsed, and arriving at a row nothing paints is no arrival at all — the
document would sit at the fold above it with no sign of what was asked for. `orgApi.setCaret` does the same before
selecting, so a command that edited a folded heading can be typed into. Only the fold *covering* the target opens,
so the rest of the file keeps the visibility the file asked for (and the top bar's cycle position, which is a
position and not a live query, is left alone — the same as when a reader opens one heading by hand). A stale offset
is still harmless: `setDoc` clamps it to the buffer, `unfoldAt` finds nothing over it, and `scrollTo` no-ops on a
row that is not there.

**Drawers fold too, and start folded.** They are the other thing in an org file that hides a region of its own,
so they get the same fringe toggle — and org's own default: hidden in overview, contents *and* showall, opened
only by `#+STARTUP: showeverything`. The engine folds them through the language's secondary range provider
(`drawerFoldRange`), which `foldAll` applies in **every** scope and `toggleFold` falls back to when a line opens no
outline range. The load spec carries `drawers` as a **flag, not a list** — the language knows where they are; the
file only decides whether they start open (`StartupVisibility.showsDrawers`).

In the **outline modes** (overview, contents — `StartupVisibility.isOutline`, `host.hideDrawers`, and
`hideDrawers` in the load spec) a folded drawer is not left as a one-line `:PROPERTIES:` row: the row is taken out
of the flow altogether (`rowClass('de-drawer-hidden')` → `display: none`). A fold can never do this by itself — it
always leaves its own first line showing — and the case that needs it is the **file-level drawer** an org-roam
note opens with, the one drawer no outline fold covers (any drawer under a heading is already inside one). The row
stays in the DOM with every character in it, so the buffer still reads back exactly, and the caret cannot reach
what is `display: none`. `showall` keeps the folded one-liner, since that chevron is the way back into the
drawer. This is why `showeverything` is now
its own `StartupVisibility` rather than a synonym for `SHOWALL`; it is a startup state, not a stop on the cycle,
so `next()` leads out of it and never back.

**The global visibility cycle** is the same three states, in org's own lingo: the top-bar button walks OVERVIEW →
CONTENTS → SHOWALL (`org-cycle-global`'s order) and its **icon is the state** — `UnfoldLess` / `Toc` /
`UnfoldMore` — the way a heading's chevron is. `orgApi.setVisibility(state)` maps them onto `Engine.foldAll(
'subtrees')` (nested ranges collapse into the outermost, so only the top level survives), `foldAll('bodies')`
(every heading visible, every body folded) and `foldAll('none')` (the outline open, drawers still folded);
`showeverything` alone is `unfoldAll()`. The cycle position lives on the **Kotlin** side
(`StartupVisibility` in `:core:org`, reused as the state type), seeded from the file's own `#+STARTUP:` state
(`InitialFolding.startup`) on the initial load so the icon means something before the first press, and reset to
SHOWALL on a `reload` — which drops every fold (`setDoc` clears them) and does not re-apply the spec, so any other
value would be a lie. A folded row is simply its first line with
the rest hidden — the engine adds no marker of its own (no CodeMirror-style `…` chip); the fold button's glyph is
the state, and a language that wants a marker knows the row is folded through `api.isFolded` and can emit its own
affix from `decorateLine`.

A fold range **includes the blank lines before the next heading** (as Emacs does), and **blank lines alone are
enough to make one**: a heading trailed by nothing but whitespace folds it away like any other. That is what keeps
a collapsed outline evenly spaced — the file's own punctuation (two blank lines here, none there) folds away with
everything else, and each heading carries the same `de-folded` row class worth one line of `margin-bottom`
instead. The gap is margin, not a row — it holds no source offset, so there is nothing there to click into or type
in.

The spacing rule that makes overview and contents uniform is **"the heading's own body region is not on screen"**:
`de-folded` goes on a heading that is folded *or* that has no body region at all (its next line is already the
next heading). Every heading in a collapsed outline is in one of those two states, so they all sit one line apart;
and in showall it also makes `* A` + `** B` sit exactly as far apart as `* A` + blank + `** B`, whose blank row is
the same height. Both halves of the rule are decided from the **next line alone** (`isFoldable`, `hasBodyRegion`)
— `decorateLine` asks for every heading of a full render, so neither may walk a subtree.

**Every heading carries the fold mark**, so the fringe reads as one straight column rather than a dotted one. A
heading with nothing under it at all keeps the glyph as an *alignment mark*: `de-fold-toggle.is-empty`, dimmed,
built with no press handler — so it is never a button that does nothing, and a tap on it walks the caret into the
row through the ordinary leading-affix rule. The row's trailing run stops folding there too.

**The selection is always on screen, focused or not — and it is always the engine's.** Blur takes the browser's
own selection away with it (`_clearDomSelection`): the native highlight left standing over a document nothing is
editing is noise, and it goes on reacting to gestures. The document still keeps its position, and the app *acts*
on it, so it has to stay visible. Whenever `focused` is false `Engine` draws its own, on a coalesced rAF: a bar
(`.de-rest-caret`, placed from `caretRect`) for a collapsed selection, otherwise one tinted piece
(`.de-rest-selection`) per line box of `selectionRects` — inactive but tinted, the same accent the live caret and
`::selection` use, weaker. Both are hidden when the browser's own painting takes over. They live in the
**scroller**, not the content: a render replaces every row, and nothing that is not the buffer belongs inside the
contenteditable — being absolutely positioned in the scroller they still scroll with the text. The tint sits in a
`z-index: -1` layer (which is what `.de-scroller`'s own `z-index` is for: it makes the scroller a stacking
context), so it paints *behind* the glyphs rather than dimming them.

Because blur clears it, **focus has to put a selection back** — and what it must not do is adopt whatever the DOM
says, which is measurably worthless at that moment: at focus time a press has not placed its caret yet
(Chromium sets it around `mouseup`), and a programmatic focus gets a synthetic one at the start of the document.
So the model is the authority and the *cause* decides. A press sets `_pressFocus` (it precedes the focus it
causes), and that focus is left alone — the press is about to say where the caret goes. Any other focus —
`orgApi.focus()`, the WebView regaining it — writes the model's selection back, so editing resumes exactly where
the resting bar said it was rather than at offset 0. `_select` sets the model *before* focusing for the same
reason.

**A touch the system takes over must not move the caret** (`_onGestureCancel`). The edge-swipe Back gesture ends
as `pointercancel`/`touchcancel` on the WebView, and the browser both drags the caret while the finger moves and
then drops it somewhere of its own a frame or two later — measured on device, the row sitting on top of the
viewport, ~50ms after the cancel and a *second* before the keyboard closed, so the caret visibly ran to the top of
the screen mid-gesture. The engine remembers the selection each press starts on, puts it back on a cancel, and
refuses the browser's selection for `GESTURE_SETTLE_MS` after one (real input — a key, a `beforeinput`, a
composition, the next press — ends that window early). An ordinary flick-scroll cancels the very same way and
never moves the caret, so it costs that nothing.

A **range** is the exception, and the reason this is not simply "undo whatever the cancelled gesture did": the
platform's own text selection takes the touch over identically. A long press selects a word and *then* hands the
gesture to the selection UI (on device the word lands ~50ms before the cancel, so the undo was killing the
selection under the finger, before the user let go), and a handle drag is a range throughout. So the test is on
what the selection is **now** — read from the DOM, since `selectionchange` is queued and a word selected in the
same breath as the cancel may not have reached the model yet. A range found there is adopted and *becomes*
`_pointerSelection`, what the gesture should end with; anything collapsed is the browser's and `_pointerSelection`
goes back. What the selection was before the press decides nothing on its own: a Back gesture that starts on a
range collapses it like any other, and that range is the one to put back.

`touchcancel` and `pointercancel` **both** fire, in either order and a few ms apart, and the selection can
collapse between them — a right-edge Back swipe over a selected word cancelled with the range still intact and had
collapsed by the second event, while a left-edge one had already collapsed by the first. So the handler runs for
each (the press's selection is kept until the gesture is really over — `pointerup`, the next press, blur — rather
than consumed by whichever cancel arrives first), and either event opens the settle window, which refuses only
*collapsed* strays so a range landing on either side of the cancel is safe.
Every caret move is reported to the host
(`host.onCaret` → `caretMoved`, deduped, so it costs one bridge call per moved offset); `DocumentScreen` keeps it
in a `mutableIntStateOf` it never reads during composition — it changes on every keystroke and nothing on screen
depends on it. What reads it is `CommandContext.targetAtCaret`, a lambda resolved when the palette opens: the
innermost heading enclosing the caret (the last one starting at or before it) becomes the command's target, so
`org-insert-heading` and friends run where the cursor is instead of asking which heading to run on.

**A tap on empty space starts editing at the nearest end of the text** (`Engine._clickEmpty`). "Empty" is
anywhere in the **scroller** that is not a row — the content's `--head` and `--tail` padding, and, on a document
shorter than the screen, the whole expanse below the content element itself, which is the case that matters most
since a *new* file is nothing but that. What the browser makes of such a press is not to be relied on: it
resolves to a position on the content element that means nothing in source offsets, or to nothing at all, and the
caret then stays where it was while the reader has plainly asked to type here. Above every row is the start of
the document, below them all is the end, and beside one (a wide window's side margins) is that row's own end.
Two things are load-bearing: it listens on the **scroller** (the content element does not reach that far down),
and it answers on **click** rather than on the press, with nothing preventDefault'ed — a drag starting in that
space is a scroll, and a scroll fires no click. It clears `_pressFocus` as it goes: the focus the click causes
must write the model's selection back rather than defer to a caret the press never placed.

Because it answers on the click, **the click's own target cannot settle the question — where the gesture began
does** (`_pressedRow`, recorded on the scroller so a press outside the content element counts too). A click is
dispatched on the *common ancestor* of the press's target and the release's, so a gesture that starts on one row
and ends on another reports the content element, which is indistinguishable here from a tap in the padding
beside it. Tapping a widget is how that arrives for real: the reveal hands the caret to source a fraction of the
widget's height, so the rows below jump up into the finger before the release is hit-tested. Measured on device,
tapping the foot of an image put `mousedown` on the `<img>` and `mouseup` on a table cell two hundred pixels of
collapsed image further down, and the caret the widget had just placed was replaced with the end of that table
row — while tapping its head landed on the following line, which is why this read as a bug about the bottom of
an image rather than about every widget tall enough to move the page.

**The soft keyboard shrinks the viewport out from under the caret.** Opening it resizes the WebView (the
`AndroidView` is `imePadding`-ed), which fires `resize`/`visualViewport` → `fitHeight` → `Engine.revealSelection({
top, bottom })`, the smallest scroll that puts the caret back inside the viewport, clear of the floating bar above
(`--head`) and of the keyboard's edge below. Nothing does this for us: the focused element is a single
contenteditable the size of the whole document, so the browser considers it already in view — without the call, a
tap in the bottom half of the screen opens the keyboard directly over the line you just tapped. `caretRect` falls
back to the row element, because a collapsed range on an empty row measures as nothing.

Toolchain gotcha (hard-won): **Android WebView resolves every CSS viewport unit (`vh`/`svh`/`lvh`/`dvh`) to `0`**,
so a `height:100%`/`100dvh` chain collapses the editor to zero height (blank screen), and a `40vh` bottom padding
gives no scroll room past the last line. `#editor`'s height and the content's `--tail` padding are both pinned
from `window.innerHeight` in JS and re-fit on `resize`/`visualViewport` changes instead.


### Themes — a theme is a CSS file

`assets/themes/<name>.css`, one `:root` block of literal colours and nothing else. The **document editor is the
half that uses one fully** and the app's chrome is the half that borrows from it:

- `DocumentScreen.documentThemeCss()` hands the page the file's **text, verbatim** (`orgApi.setThemeCss`), which
  drops it into the empty `<style id="org-theme">` that follows the page's own defaults. It wins on document
  order, so every property takes effect and neither Kotlin nor the page has to know what any one of them means —
  adding a property to the contract is an edit to `document-editor.html` and the converter, never to Kotlin.
- `ui/theme/ThemeCss.kt` parses the same file for the **few** colours the app itself draws with and maps them
  onto a Material `ColorScheme` (`OrgThemes.colorScheme`), so the agenda, the sidebar and the settings are the
  same theme as the document they surround rather than a second one. The two TODO chip containers are the sharp
  case: `.de-org-todo` and `headlineChipColors` read the same two properties out of the same file, which is why
  a keyword looks identical in the document and in the agenda.

**A theme also carries what Material has no role for**, reached through `LocalOrgTheme` (provided by
`OrgMobileTheme`, null under System) rather than through the scheme. Both cases so far are the same trap:
Material draws a surface's *content* in **`onSurfaceVariant`**, which on a Material You palette is a near-white
grey and reads fine — but a theme's dim colour is dim *on purpose* (modus's `fg-dim` is `#989898`, picked to
recede from `fg-main`), so anything Material calls secondary came out at 4.3:1 on a theme built for 7:1 and up.

- the **sidebar** (`--sidebar-bg`/`--sidebar-fg` → `sidebarColors()` → `VaultTree`'s `ModalDrawerSheet`): the
  theme names its own panel and text — its dim background, its **main** foreground — which keeps the panel a
  shade off the document behind it while the file list reads at full contrast. A file list is not secondary text;
  under System the fallback is `surfaceContainerLow` with `onSurface`.
- the **outlined controls** (`--control-fg` → `controlColors()` → every `OutlinedButton`, chiefly the settings
  rows' value chips): the value a setting holds is the answer to its row, not an aside, so it should not read
  fainter than the label naming it. The outline is what tells the chip from the text around it. System falls back
  to `onSurface` here too.
- a **date picker's two heading lines and its weekday letters** (`datePickerColors()` in `HeadlineEditSheet`):
  they are what the dialog is about, and Material gives all three the dim role. Everything else in the picker
  keeps Material's own defaults.

The mirror-image mistake is as easy to make from the other side: **Material's `surfaceContainer*` roles are a
ramp of small elevation steps over the background** — a sheet, a dialog, the bottom chip — and an Emacs palette's
`--bg-active` is *not* one of them. It is the active mode line and the region, a deliberately strong colour
(standard-dark-tinted's is `#5f6580` over a `#182440` background), and mapping it onto `surfaceContainerHigh` made
every raised surface glare. The ramp is built from the theme's own two backgrounds instead (`bg` → halfway →
`bg-dim` → two small steps past it), and `--bg-active` keeps `surfaceBright`, which Material reaches for rarely
and which is what that colour actually is.

These properties live in the page's `:root` with the rest even though the page never draws them, so a theme file
has one contract and not two.

**The one colour a theme cannot carry is the WebView's text-selection handle.** It is not drawn by the page at
all — Chromium puts it in a window of its own, tinted from `android:colorControlActivated` on the **activity
theme** — so left alone it is the platform's teal under a themed caret. (Measured, not assumed: an
`accent-color` on the page leaves it untouched.) A theme attribute can only hold a **compiled** resource —
`Theme.applyStyle` takes a style id and there is no API to set an attribute to a runtime colour — so the value has
to exist by build time. `ui/theme/HandleTint.kt`, called from `OrgMobileTheme`, picks a style and lays it over the
activity's theme; Chromium reads the attribute **when it shows a handle**, so a theme switch reaches an open
editor with no WebView rebuild (verified on device, both directions). Two sources, in order:

- a theme **compiled into the app** has a style of its own, written from its `--accent` by `web/build.mjs`
  (`res/values/theme-handles.xml`, **generated — don't edit**), plus the hand-written `HandleTint_system` whose
  four variants track what `OrgMobileTheme` hands Compose as `primary` (the dynamic palette's
  `system_accent1_600`/`_200` on API 31+, `Theme.kt`'s own `LightColors`/`DarkColors` primary below it). Exact.
- a theme **imported at runtime** cannot have one, so the handle takes the nearest point of a **16×16×16 grid**
  of tints (`res/values/handle-tint-grid.xml`, generated: 4096 one-line styles, ~283KB of resource table).
  Nearest is arithmetic rather than a search — round each channel to a level — and lands within **8/255 per
  channel**, usually much closer. Nothing else moves: a theme's own colours are always exactly what its author
  wrote, and the grid is fine enough that the handle meets them rather than the other way about. Raising
  `LEVELS` in `web/build.mjs` (and `GRID_LEVELS` beside it) is the dial if that is ever not close enough; the
  cost is cubic.

The contract — every property, with the page's own values as the fallback — is documented at the top of
`web/src/document-editor.html`; `web/themes/README.md` says how to add a theme. Two properties are the theme's
own: `--theme-name` (what the settings list shows) and `--theme-dark` (`1`/`0`, which decides the Material base
and hence every role the app does not claim). Colours **do not** go through `orgApi.setTheme` any more: that sets
inline properties on the root, which beat every stylesheet, so anything pushed that way could never be themed.
What still goes through it is layout the page cannot know — `--head`, the room the floating bar covers (and
`setFooter`'s `--foot`).

**Themes can also be imported at runtime**, which is the whole point of a theme being a file: Settings →
Appearance → *Import theme…* (at the foot of either theme picker) opens a document picker, and `OrgThemes.import`
copies the `.css` into `filesDir/themes/` and sets it as **the slot that asked for it**. `OrgThemes.reload` reads both directories — `assets/themes` first, so a
shipped theme can never be shadowed by an import of the same name (the ids the exact handle tints are keyed by
always mean what they say) — and each imported theme is removable from its own row in the picker, since nothing
else in the app can remove it (removing one that either slot holds returns that slot to System first, so no id is
left dangling). The file is validated by *being parsed*: a sheet that declares no colour at all is
refused rather than installed as one that renders as the page's defaults. Importing an existing name replaces it,
which is how a theme is updated after editing it on the desktop.

**The theme is chosen twice**, since a theme is light or dark in itself and the phone says which is wanted:
`SettingsStore.lightThemeId` and `darkThemeId` are two independent slots (Settings → Appearance, *Light theme* and
*Dark theme*), and `activeTheme()` reads the one `isSystemInDarkTheme()` names. That is the **only** place the
choice is made — `OrgMobileTheme` publishes the answer as `LocalOrgTheme`, and the handle tint and the page's
stylesheet read it from there rather than asking again and risking a different answer. Nothing stops a dark theme
being picked for light mode (the picker labels every entry `dark`/`light` and leaves it to you), because the
alternative is refusing a choice for the sake of a rule. The single `theme` setting these replaced is migrated by
being read into **both** slots, so an upgrade looks exactly as it did.

**The system bars follow our surface, not the phone's mode** (`SystemBarAppearance()` in `ui/theme/SystemBars.kt`,
called from inside `OrgMobileTheme`). The status and navigation bars are the system's, painted over whatever is
beneath them, and `enableEdgeToEdge()` picks light or dark icons from the phone's own light/dark setting — the
wrong question the moment the theme is chosen per mode, since a dark theme worn in light mode got dark icons on a
dark surface and a light theme at night got white on white, the clock and battery vanishing either way. The
luminance of `colorScheme.surface` answers it for every theme, System included. A **dialog is its own window**
with its own appearance, so the full-bleed prompts (search, the command palette) call it again for themselves —
otherwise opening one flips the icons back to whatever the phone's mode implies while the prompt stands over the
page. `LocalView.current` names the right window either way: a dialog's parent is a `DialogWindowProvider`,
everything else belongs to the activity.

**Every screen has to paint its own ground somewhere**, because the activity window's is not ours to theme: it
comes from `Theme.OrgMobile` (`android:Theme.Material.Light.NoActionBar`, whose one job is to carry the WebView
handle tint), so it is light whatever the app theme says. For all but one screen `AppRoot`'s Scaffold covers it —
its default container is `colorScheme.background`, which is also where their window insets come from.
`OnboardingScreen` is the exception: `AppRoot` composes it **in place of** that Scaffold while there is no vault,
so it carries its own full-screen `Surface` in the same colour and its own `safeDrawingPadding()` — `safeDrawing`
rather than the Scaffold's `systemBars`, since the `WebDavVaultForm` it leads to is reached with no Scaffold in
between and is typed into. Without the surface a dark theme drew its light text straight onto that white window.

Each slot holds a file's name without `.css`, or `SettingsStore.SYSTEM_THEME` ("system", the default) for **no**
CSS theme: the app is then Material You (or the built-in green fallback) exactly as it was
before themes, and `documentThemeCss` writes that palette out as the same kind of sheet
(`materialThemeCss`) so the document follows the platform's light/dark switch. Headings stay `onSurface` there on
purpose — colouring an outline by level is something a theme says, and the platform palette says nothing about
it. The theme is pushed on **every** change, not only at startup, so picking one in Settings repaints a document
that is already open; the page swaps one `<style>` and repaints, so the buffer, the caret and the folds are
untouched (asserted in `browser-test.mjs`). `OrgThemes.init` reads every theme once in `OrgMobileApp.onCreate`,
before anything can draw — a theme that arrives late is a visible flash of the wrong palette. Widgets are not
themed too — see **Widgets** — through the same two slots.

**`tools/emacs-theme-to-css.py`** converts an Emacs theme into one of these files, and is meant to work on any
reasonable theme rather than on modus alone. It reads a theme's **palette** (`(name "#hex")` plus `(mapping
name)` aliases — modus/ef/standard-themes style, whether it is a package `defconst` or a `let*` around the face
list) *and* its `custom-theme-set-faces` **faces**, folds both into one lookup, and answers each CSS property
from an ordered list of candidates (`MAPPING`), so a theme that says nothing about org headings still gets them
from its rainbow/accent colours (and the chrome properties from its `bg-dim`/`fg-main`). A name is tried exactly and
then by suffix, which is how prefixed-`let` themes (`zenburn-bg`) work with no special case; chip containers are
derived by mixing the TODO/DONE colour into the background, so a theme need only name a colour and not a
container. It reads the theme's sibling `.el` files too,
because the palette usually lives in the package's main file rather than in the nine-line theme file. Regenerate
with `tools/emacs-theme-to-css.py THEME.el -o web/themes/<name>.css`, then `cd web && npm run build` (which
copies `web/themes/` into `app/src/main/assets/themes/` and regenerates the handle-tint styles) and rebuild the
APK. The settings list is built from
whatever is in that assets directory, so a new theme needs no code change.

**The font is the app's choice, not the theme's.** `SettingsStore.documentFont` (`DocumentFont`, Settings →
Appearance, default Serif) is pushed into the page by `DocumentScreen` as `orgApi.setFont(...)`, which writes
`--doc-font` — an inline root property, like the other host-set values — and the page's prose (`body`, and a
table) is `var(--doc-font, var(--serif))`. What it holds is **`var(--serif)` / `var(--sans)` / `var(--mono)`**
rather than a font stack: the app says *which* of the page's three faces the prose uses, while the page — or a
theme, if one ever carries faces — says what each of the three is. Code, verbatim text and metadata lines stay
`--mono` whichever is picked, since those are monospace because of what they are.

**Nothing is packaged.** Android has none of the fonts named in those stacks (Iowan Old Style, JetBrains Mono,
Menlo), so each resolves to a platform family — Noto Serif, Roboto, Roboto Mono — which is also what
`/system/etc/fonts.xml` offers by name (`serif`, `sans-serif`, `monospace`, plus `sans-serif-condensed`,
`-light`, `cursive`, `casual`, `source-sans-pro`, and aliases like `georgia`/`times new roman` that resolve onto
the same faces). Fonts a user installed themselves (Samsung's) are **not** reachable: `/data/fonts` is not
readable by apps and those families are not in the app-visible config. A specific typeface would have to be
vendored into `assets/editor/` with an `@font-face`, the way KaTeX's fonts already are.


### Search — `ui/search/SearchDialog.kt`

Search is a **prompt over the screen that asked for it**, not a screen of its own: the bottom chip's magnifier
sets `AppRoot`'s `searchOrigin` (the `ChipStop` that asked) and `SearchDialog` opens on top. It has to be a
dialog, because it acts on what is behind it — see the two kinds below that answer *there* rather than in a list.

The window behind is dimmed by the dialog's own `setDimAmount` (`DialogProperties` has no say over it) rather
than by the platform's 0.6, which buries the very document a search is walking: `LIST_DIM` with a panel up,
`DOCUMENT_DIM` — lighter still — for the document kind, where the page *is* the answer.

Its input layer is two rows deep — the kind chips ride above the field — so it passes `PaletteList` that much
more `bottomRoom` (`CHIP_ROOM + KIND_CHIP_ROOM`), or the last rows would sit under the chips with no way to
reach them.

The scan streams its hits in batches, so the panel carries a **progress bar along its top edge** while one is
running (`PaletteList(loading = …)`) — a short list is not the same thing as a finished one, and a full-text
search over a real vault is the case that needs saying. Its room is reserved whether or not it shows, so rows
never jump as a scan starts and stops; when it is gone and there are no rows, the panel says "No matches".

It is built from the same two layers as the command palette (`ui/PaletteChrome.kt`, shared by both:
`PaletteList`, `PaletteInput`, `PaletteRow`/`PaletteRowView`) — a fixed list panel that the keyboard covers
rather than pushes, and an input riding the keyboard — with the **kind chips carried by the input's own layer**
(`PaletteInput`'s `above` slot), since choosing what to look for and typing it are one gesture.

The panel **pulls up to the full screen by its handle** (a drag on the grabber at its top, or a tap on it),
between `LIST_HEIGHT_FRACTION` and `LIST_EXPANDED_FRACTION` — both prompts, since it lives in the shared chrome.
Two positions rather than a free height, and the drag decides on *release* against `DRAG_THRESHOLD`, so a finger
on its way to the list never resizes the panel; only the handle drags, the list keeps its own scrolling.

A `SearchKind` is a *kind of answer*, and each maps to a `SearchFilter` except the first:

- **Document** — walks the open document and shows **no list at all**: typing moves the viewport to the first
  match and **highlights it**, and the arrows inside the input (`PaletteInput`'s `trailing` slot, shown for this
  kind alone — the list kinds answer with rows, where an arrow would be scrolling rather than searching) step to
  the next match and the previous, wrapping at either end since a hunt that stops at the last match has no way
  back to the first. **Enter accepts and leaves**, as isearch's RET does: the caret stays at the end of the match
  and the prompt takes the highlight with it — a reveal of an *empty* range is a plain caret move, which is what
  drops `de-found`. One `found` offset is what everything moves, and one effect shows it. It goes through `PendingNavigation.requestReveal` → `orgApi.showAt` →
  **`Engine.showAt(from, to)`**, which differs from `setCaret` in exactly one way that matters: it takes **no
  focus**. The prompt owns the keyboard, and an unfocused editor already paints its own resting selection, so the
  match is a *range* rather than a caret and that painter is the highlight — carrying a `de-found` class on the
  scroller for the stronger of the two tints, since it is being read through a prompt standing over the page.
  `_markFound(false)` drops it again the moment the reader speaks (`_select`, or the editor taking focus). The
  chip is offered only when a document is behind the prompt — which the visible screen says through
  `CommandContext.relPath`, the channel it already uses to tell a prompt what it offers.
- **Filename**, **TODO**, **Heading**, **Full text** — the four list kinds, each one `SearchFilter` group.
  **Filename** answers before anything is typed: a blank query lists the recently opened files,
  newest first (`SettingsStore.recentFiles` less what the vault no longer holds, as `SearchHit.File`
  rows, so picking one opens it like any hit) — the whole ring rather than the sidebar's
  `recentFilesCount` slice, which is how much room the folder tree gives that section and not a
  statement about this panel, which scrolls. Nothing is selected, as with any blank query, so Enter
  still does nothing until something is typed.

The **default kind is the asking screen's own likely question**: `Filename` over a document, `TODO` over the
agenda. And a TODO picked **over the agenda** is handed back to the agenda (`PendingNavigation.requestAgendaFocus`)
rather than opening a file: if the view on screen holds that entry, moving to it *is* the answer — the list
animates to its row (counted the way the `LazyColumn` is built: one header per day, then its items), the month
selects its day (which is why `CalendarView`'s selected day is hoisted into `AgendaScreen`), and the row is tinted
for `FOCUS_TINT_MS`. Only when the view does not hold it — another month, or past the list's period — does the
document open at it. Everywhere else, and for every other kind, a hit opens its document.

### Command prompt (M-x) — `ui/command/`

A native, org-mimicking command registry (**not** an elisp interpreter — the user rejected that). `OrgCommands.all` is a plain list; each `OrgCommand` declares a plain-English `label` (two to four words — what the palette reads as), the emacs `name` it stands for, a longer `description`, `CommandArg`s, whether it requires a heading target, and an execute function. **A command owns its own prompting**, which is what keeps the palette generic: an arg is `ArgType.INT`/`TEXT` (typed into the args panel) or `ArgType.CHOICE`, whose `choices: () -> List<ArgChoice>` and `prompt` come from the command definition and are shown as one more list stage. A typed arg may also carry `suggest: (typed) -> List<String>` — **completions**, offered under the field and filling it when tapped, which is how `find-file` and `rename-file` complete a path against the vault (`OrgCommands.pathCompletions`: files and the folders on the way to them, prefix matches first, a trailing `/` on a folder so tapping one carries on). Tapping one puts the **caret at its end** — the field holds a `TextFieldValue` for exactly that reason, since a caret left where the finger was is a caret in the middle of a word the user never typed. A suggestion is never a restriction: an argument can still name a path that does not exist yet, which is the whole point of `find-file`. The panel's own **description is a `?` tooltip** (the shared `ui/HelpButton.kt`, the same one the settings use) rather than a paragraph over the fields — a form is filled in, not read. A future command that picks a file, a tag or a template is a new `choices` lambda, not a new palette stage. Targeted commands run against a `CommandTarget` (`title`, the document's first active TODO keyword, `commit(caret) { subtree -> newSubtree }` that sources/commits one subtree's text, and an optional `cut: suspend ()->Unit` that removes the whole subtree). `commit`'s optional `caret` maps the *new subtree text* to an offset inside it; the vault-backed target turns that into a file offset (`VaultActions.transformSubtree` returns `subtreeSpan.start + caret(new)`) and posts `PendingNavigation.requestCaret`, which the open document applies as `orgApi.setCaret` — the editor takes focus and `DocumentScreen` raises the IME, so an inserted heading can be typed into straight away. That effect **consumes the request last**: the request is one of its own keys, so consuming it earlier cancels the effect mid-wait and the caret is never placed. It waits for `hasWindowFocus` first, because the palette dialog is still up when a command returns and owns both the focus and the keyboard. A buffer-backed target (the raw editor) ignores the caret. Two providers: **vault-backed** (`OrgCommands.vaultTarget` → `VaultActions.transformSubtree` → `OrgEdits.replaceSubtree`, and `cut` → `VaultActions.cutSubtree` → `OrgEdits.cutSubtree`, with the removed text copied to the system clipboard via `Clipboard.copy`) for the document view, and **buffer-backed** (`text = transform(text)`, saved later; no `cut`) for a raw subtree editor. `AppRoot` owns `CommandHostState` and the `onCommand` that opens the palette (reached from the bottom chip's terminal button, on all three of its screens); the visible document publishes its heading choices **and a `targetAtCaret` lambda** (the heading its cursor is in — `AppRoot` resolves `target ?: targetAtCaret()` as the palette opens, so a targeted command runs where the caret is and the heading-pick stage is skipped), a raw subtree editor publishes its live buffer target, and all other routes publish no target. `CommandPalette` is built in **two layers**, Obsidian-style, as a full-bleed `Dialog` (`usePlatformDefaultWidth = false` *and* `decorFitsSystemWindows = false` — the second is what makes the keyboard an inset inside the dialog instead of a resize of it). The **list** is a fixed panel over the dimmed document that takes no keyboard inset at all: the keyboard covers its lower part rather than pushing it, so dismissing the keyboard just reveals more of the same list, with nothing reflowing. The **input chip** floats above it and rides the keyboard, dropping to the bottom of the screen when the keyboard goes. Only the args stage rides the keyboard as a panel of its own, since every field in it is being typed into. A row shows the `label` with the emacs `name` beside it in monospace, each taking half the width so the names line up as a column; a row that cannot run here is dimmed, shows why in place of the name, and is sorted **after** every runnable row. There are two such reasons: a heading the route cannot offer ("needs a heading"), and the command's own `available: () -> Boolean` saying it has nothing to act on, with its `unavailableNote` ("nothing to merge" for `resolve-conflict`). The second exists because a `CHOICE` argument whose list comes back empty is a dead end — the palette shows a stage with no rows and no explanation, and the only way out is Back — so a command that can say in advance that there is nothing to pick belongs in the dimmed group with a reason. Dimmed rows are sorted after runnable ones (a stable sort, so the fuzzy ranking survives inside each group) — what can be run is never scrolled past to reach. `fuzzyCommands` scores the query against **both** `label` and `name` and keeps the better (with a small tiebreak to the label, since that is what the row reads as), so "cut subtree" and "ocs" both find `org-cut-subtree`; `fuzzyHeadings` and `fuzzyChoices` apply the same ranking to the heading-pick and choice stages, which reuse the one input as their filter. Ordered-character matching gives word starts (after a space or `-`) the same bonus as a name segment. Every list stage renders the same `PaletteRow`s over the same input, so **typing selects the best match (the tinted top row) and Enter — the input's Go action — picks it**, whichever stage is showing; with a blank query nothing is selected and Enter does nothing, since an alphabetical first row is not a choice the user made. Its stages are search → optional heading pick → optional `ArgType.CHOICE` pick → typed args → run. **A command with nothing left to ask runs the moment it is picked** — there is no confirmation step, so the args panel appears only when something must be typed, or when a run failed and its message needs somewhere to live (`failed` is sticky for exactly that reason: it keeps a retry from dropping back into the instant-run branch and firing the command twice). Heading long-press remains a direct pre-targeted shortcut. Commands include `find-file` (one typed path: opens it, and — as emacs's own does with a name that names nothing — creates it first when it is not there), `delete-file` (the file **picked** rather than typed, since a typo in a path that opens a file costs nothing and a typo in one that deletes a file costs the wrong file; a command cannot put a dialog on screen, so it names the file through `PendingNavigation.requestConfirmDelete` and `AppRoot` does the asking, through the same `ConfirmDialog` every other deletion goes through) `beginning-of-buffer`/`end-of-buffer` (no arguments, no target, so they run the instant they are picked: `PendingNavigation.requestPoint` names **no file** — the command means the buffer on screen, and the document view is what knows which that is and how long it is — and the caret lands there with the keyboard up, through the same `setCaret` an inserted heading uses), `customize` (a `CHOICE` over the five `SettingsGroup`s, so the palette's own list *is* the group list and picking one opens the settings there — one list, one tap, nothing typed), `resolve-conflict` (the way back to a merge decision that was put off, from any screen the prompt opens on; the file is picked like `delete-file`'s, and `PendingNavigation.requestConflict` → the `conflict/{path}` route does the navigating), and `rename-file` (the file **picked** from what the vault holds, since only the new path is one that does not exist yet, then the new path typed; `OrgVault.rename` does it as copy-then-delete, portable across SAF and WebDAV alike, and ordered so a half-failure can leave a duplicate but never a lost file). A path typed without `.org` gets it, since the vault lists nothing else. Also `org-clone-subtree-with-time-shift`, the `OrgInsert` heading commands (which take **no title** — like Emacs they leave an empty heading whose text is then typed in the document, so all three are instant, and the caret lands at the end of the heading line they wrote with the keyboard up; the TODO variant's keyword therefore always carries a trailing space, or the first thing typed would run into it), `describe-function` (whose argument is another command — a `CHOICE` arg whose `choices` are `OrgCommands.all`, so it reuses the same list to pick one, then `PendingNavigation.requestHelp` → the `help/{name}` route → `ui/help/HelpScreen`, an emacs `*Help*` buffer showing the command's label, name, description, whether it needs a heading, and what it asks for — descriptions live on the `OrgCommand`, so adding a command documents it; the document menu's **Help** opens `ui/help/HelpIndexScreen` at the `help` route instead, which lists every `OrgCommands.all` entry the way the palette does — label plus the emacs name in monospace, over its description — each row opening that command's page, so the pages are reachable without knowing what to ask for), `org-cut-subtree` (targeted; cuts the subtree to the system clipboard, reversible via undo), the target-free `undo`, the target-free `org-journal-new-entry` (which asks for **nothing** either — like
the insert commands it files the entry and hands the caret to the end of its `** <time>` heading with the
keyboard up, so the entry is typed in the document rather than into a prompt; `OrgJournal.newEntry`'s
`roomToType` writes that heading with a trailing space for the same reason the TODO insert command's keyword
carries one, and `VaultActions.JournalLocation` carries both the heading's `lineSpan.start` to open at and its
`lineSpan.end` to type at), and the target-free `org-capture` (opens the capture screen's template picker). `JournalFileType` rotates daily (`%Y-%m-%d.org`), ISO-weekly (`%G-W%V.org`), monthly (`%Y-%m.org`), or yearly (`%Y.org`, the default); file type—not a contradictory free-form filename—decides when a new file starts. Date/time formats remain configurable and `OrgJournal.formatPattern` supports `%x` as `MM/dd/yyyy`. `OrgJournal` appends at EOF, adds `:CREATED: YYYYMMDD` to every new date heading, keeps blank-line separation around appended entries (optionally carrying a multi-line `body` under the `** <time>` entry), and preserves existing bytes and line endings. Journal-path resolution (dir + rotating file + date/time title) lives in `JournalCapture.fileEntry`, shared by the command and the JOURNAL capture target. The command safely creates missing SAF directories/files through `OrgVault.create` and opens the inserted entry
**at its caret offset with `edit = true`** (`PendingNavigation.request` → the route's `edit` flag), so the document
arrives scrolled to the new entry with the caret after its timestamp and the keyboard up. One request, carried by
the destination: an entry is usually written into the journal that is already open, and a separate `requestCaret`
would be eaten by the composition that navigation replaces. It is intentionally a practical subset rather than full org-journal parity. Add a command by writing a pure `:core:org` transform + one `OrgCommand` entry (label, emacs name, description).

### Capture (org-capture) — `ui/capture/`, `orgmode/OrgCapture.kt`

Basic org-capture templates. `OrgCapture` (pure `:core:org`, unit-tested) holds the `CaptureTemplate` model (`JOURNAL`/`FILE_HEADLINE`/`FILE` target, target file/headline, `prepend`, template `body`) plus three pure functions: `prompts(body)` (ordered, de-duplicated `%^{...}` labels), `expand(body, now, content, answers)` (escapes `%t %T %u %U` via `OrgTimestamp.format()`, `%^{prompt}`, `%?`/`%i` for where the typed body lands, `%%`→`%`; unknown `%x` left literal), and byte-preserving `insert(source, template, entry)` — `FILE` splices a top-level heading at EOF/before-first-heading; `FILE_HEADLINE` locates the heading by title, **re-levels** the entry to `targetLevel+1`, and files it at the subtree boundary (or after the heading's drawer when `prepend`), creating the heading if missing. Templates and a bounded ring of `RecentCapture`s persist as JSON in `SettingsStore` (`CaptureJson`, seeded with a Journal + Inbox TODO default), edited in Settings → Capture templates. `VaultActions.capture` files FILE/FILE_HEADLINE entries (create-if-missing, else `OrgVault.edit`); JOURNAL captures go through `JournalCapture.fileEntry`. `CaptureScreen` expands the template, collects the body + one field per prompt, files it, records the recent capture, and opens the new heading. **Staging caveat:** on a `WEBDAV_STAGING` vault a `FILE_HEADLINE` capture changes heading count → `StagingUnsupportedEditException` (reported via the AppRoot Snackbar); a capture into a *new* file still works through `OrgVault.create` (MobileOrg capture). Add a capture escape by extending `OrgCapture.expand`.

---
> Source: [alsciokat/org-mobile](https://github.com/alsciokat/org-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
