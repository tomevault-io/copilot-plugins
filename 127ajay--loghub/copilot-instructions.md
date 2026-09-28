## loghub

> Guidance for Claude Code (or any future agent) working in this repository. Read this before

# CLAUDE.md

Guidance for Claude Code (or any future agent) working in this repository. Read this before
making changes - it explains what LogHub is, what's actually built vs. planned, and the decisions
already made so they don't get silently re-litigated or reverted.

## What this project is

LogHub is a centralized, web-based log viewer. Internal Web Applications on a server write their
own `*.log` files to disk; today, checking them means RDPing into the server and hunting through
folders. LogHub reads those files directly and shows them in a browser - live tail plus searchable
history - so developers and support staff never need server access just to read a log.

Full requirements live in `docs/prd/`, split by phase (see below). This file is the "how the
codebase actually works right now" companion to those requirement docs.

## Vision / where this is going

Phase 1 (pilot) is implemented: Web Applications only, single server, no auth, no database. See
`docs/prd/phase-1-pilot.md` for exact scope and `docs/prd/00-overview.md` for the architecture
decisions that shape everything (single-server deployment, no Elasticsearch/SQL, no login).

Phase 2 (`docs/prd/phase-2.md`) is **mostly implemented** as of 2026-07-20: search refinements
(regex + AND/OR/NOT), date-range export, index caching, and redaction. **Alerting was explicitly
deferred** by the user and has no design work - don't start it without scoping it first.

Phase 3 (`docs/prd/phase-3.md`) is **partially resolved as of 2026-07-21**:

- **Non-Web-App log types: resolved, no code was needed.** There is no application-type concept in
  the codebase - `LogApplicationConfig` is a name plus folder paths. Any application writing flat
  `*.log` files (Windows Service, WCF, Web API) works today by registering its folder. Verified
  against NLog, log4net, and bracketed ASP.NET layouts on a running instance. Applications logging
  **only to the Windows Event Log** are still unsupported and would need a real second collection
  mechanism. Don't describe the tool as "Web Applications only" - that was pilot scoping, not a
  code constraint.
- **Multi-server collector agent and authentication: deferred by the user (2026-07-21)**, with a
  full plan in `docs/plans/2026-07-21-phase-3-future-multiserver-and-auth.md`. Both are still gated
  on their trigger actually firing - don't build ahead of them. Note the plan's key finding: a UNC
  root path already works with no code change (`ResolveRoot` passes rooted paths through), so a
  collector agent should never be the first thing tried.

## Current implementation status (Phase 1)

Working, in `src/LogViewer.Web/`:

- Live Tail - polls each app's today's log file(s) every few seconds, pushes new lines to the
  browser over SignalR.
- History - date picker, keyword/level filter, reads matching file(s) directly on request.
- Admin page - register/remove applications (name + one or more log folder paths) from the
  browser. **This diverged from the original PRD draft**, which assumed `appsettings.json`
  editing; a user explicitly asked for frontend-managed registration instead. See "Key decisions"
  below.
- Handles flat log files (`Root\app-2026-07-20.log`), one-folder-per-day layouts
  (`Root\2026-07-20\*.log`), and split `Root\2026\07\20\` nesting. The scan is **recursive**
  (depth-capped at 6) - it used to look only one level deep, which meant any nested layout indexed
  to nothing and History showed "No log data found". Each file's date falls back through:
  date in file name -> date in folder path -> last-write time, so every readable file lands
  somewhere in the index rather than disappearing.
- Group-by-tag in History: the app offers grouping on whatever structured fields it finds in the
  selected day's logs (see `ExtractTags` below), and no grouping at all for formats that carry none.
- Multi-line log entry grouping - a line without a timestamp is treated as a continuation of the
  previous entry (stack traces, wrapped messages), not a separate broken entry. This was added
  after Serilog's own multi-line output format broke naive one-line-per-entry parsing; the fix is
  general-purpose, not Serilog-specific. See "How log parsing works" below.
- CSV export from the History view.
- Files page - read-only browsing of folders that hold `.txt`/`.json`/`.xml` documents rather
  than log streams. Separate registry, separate page, no tailing, no date index. See "How the
  file viewer works" below and `docs/plans/2026-07-27-file-viewer-design.md`.
- LogHub's own operational logging goes through Serilog to `Logs/loghub-*.log` (rolling daily) -
  separate from the application logs it monitors.
- No authentication. No database. No collector agents.

## Key decisions (don't relitigate these without a reason)

These came out of explicit stakeholder Q&A during the PRD process (`docs/prd/00-overview.md` has
the full decision log):

1. **Single server.** The web app and the log files it reads live on the same machine. No remote
   agents, no network log shipping.
2. **No database, no search engine.** Elasticsearch/OpenSearch/SQL were explicitly ruled out. The
   app reads `*.log` files directly from disk on every request - no persistence layer beyond the
   small JSON file that stores registered application names/paths.
3. **No authentication.** Internal-only tool, reachable only on the internal network. If this ever
   changes, Windows Integrated Auth was the suggested lightweight path (no extra licensing).
4. **Application registration is UI-managed, not config-file-managed.** `App_Data/applications.json`
   is written by `LogAppRegistry` when someone uses the Admin page; nobody should need to hand-edit
   JSON or restart the app to add a monitored application. Don't reintroduce an `appsettings.json`-based
   app list - that was tried first and explicitly replaced.
5. **Polling, not FileSystemWatcher**, for tailing files (`LogTailBackgroundService`). Deliberate:
   simpler to reason about, tolerates locked/rotating files without missed events. Don't swap this
   out for a watcher-based approach without a concrete reason - it was a considered trade-off, not
   an oversight.
6. **The index cache is a TTL cache, not an invalidation-on-change index.** `LogFolderScanner`
   holds a built index for `IndexCacheSeconds` (default 10). This is consistent with decision 5 -
   the app polls rather than watching the filesystem - and it means a brand-new log file becomes
   visible within the TTL rather than instantly. That's a deliberate trade, not a bug. Registering
   or removing an application invalidates the cache immediately.
7. **Redaction is a safety net, never a guarantee.** `LogRedactor` masks what it can recognize on
   the way out; it cannot catch a secret logged as a bare unlabelled string. Don't let its presence
   justify relaxing the expectation that applications shouldn't log secrets, and don't describe logs
   as "safe to share" because it exists.
8. **Log parsing is heuristic and format-agnostic on purpose.** LogHub has no per-application
   format configuration. It must keep working against log formats it's never seen before (that's
   literally why the multi-line grouping fix exists - see below). Don't add a hard dependency on
   any one framework's output shape (Serilog, NLog, log4net, etc.).
9. **Files-page folders live in their own registry, not as a flag on an application.**
   `FileFolderRegistry` (`App_Data/filefolders.json`) is separate from `LogAppRegistry`, and
   `LogFolderScanner` was deliberately *not* taught about `.xml`. "These files don't need Live
   Tail" is therefore structural - the tailer never sees them - rather than a setting someone can
   forget to tick. Don't merge the two registries to remove the duplication: the log registry is
   on the path Live Tail depends on, and keeping them apart is what guarantees a change to file
   browsing can't reach it.
10. **The file viewer has no download endpoint.** Nothing streams raw bytes off the server - the
    tool is for viewing files, not distributing them. That is why an oversized file pages via
    `?offset=` rather than offering a download link. Don't add one as a convenience.

## How log parsing works (read this before touching `Services/`)

- `LogLineParser.HasTimestamp(line)` - a line "looks like" the start of a new entry if it begins
  with a recognizable timestamp. This is the one heuristic the whole system hangs off of. It
  tolerates leading `[`/`(`/`<`/quote characters (bracketed layouts like `[2026-07-20 09:00:00]`
  are extremely common and previously read as "no timestamp at all", which collapsed a whole file
  into one entry), and accepts ISO, `yyyy/MM/dd`, `dd/MM/yyyy`, compact `yyyyMMdd HHmmss`, syslog
  (`Jul 20 09:00:00`) and bare `HH:mm:ss` forms.
- **Timestamps keep their wall-clock value.** A time written without a timezone is not shifted to
  UTC - doing so made a line reading `09:00` render as `03:30` for anyone east of UTC. Formats that
  do carry an offset (`Z`, `+05:30`) are converted normally. Don't reintroduce
  `AssumeLocal`/`AdjustToUniversal` here.
- JSON-per-line logs (Serilog compact `@t`/`@l`/`@mt`, pino, Bunyan, Docker) are detected and read
  as real fields rather than pattern-matched as text.
- `LogLineParser.DetectLevel(line)` - checks Serilog's bracketed 3-letter codes first
  (`[INF]`/`[WRN]`/`[ERR]`/`[DBG]`/`[VRB]`/`[FTL]`), then falls back to full words
  (`ERROR`/`WARN`/`INFO`/`DEBUG`/`FATAL`/`TRACE`) for other frameworks. Word matching is
  boundary-anchored on purpose - matching "error" anywhere in the text made every message that
  merely mentions the word an Error entry. Unrecognized lines are `LogSeverity.Unknown`, not an
  error - the app never rejects a line for not matching a schema.
- `LogLineParser.ExtractTags(line)` - best-effort structured fields: `key=value` pairs, bracketed
  context tokens (numbered `context`, `context2`, ...), and JSON properties. A format carrying none
  yields an empty dictionary, which is a normal outcome rather than a failure. These populate the
  History page's "group by" dropdown, so the grouping options are always whatever the logs actually
  contain - there is still no per-application format configuration.
- `LogEntryGrouper.Group(lines, sourceFile)` (used by the History/`logs` API) - groups raw lines
  into entries: a line with a timestamp starts a new entry, a line without one is folded into the
  previous entry's `Message` (with a newline). If a file has **no** timestamped lines at all, every
  line is treated as its own entry instead (so plain unstructured logs still work sensibly).
- `LogTailBackgroundService` does the same grouping, but streaming: per file, it tracks whether
  timestamps have been seen yet, and holds a "pending" entry across poll ticks while continuation
  lines might still be coming. A pending entry is flushed to the browser when either (a) a new
  timestamped line closes it, or (b) it's been pending longer than ~2 poll intervals with nothing
  new (so a straggling last line never gets silently lost), or (c) the file rotates.
- Frontend rendering relies on `white-space: pre-wrap` (both `.logline` in Live Tail and `td` in
  the History table) so multi-line `Message` values with embedded `\n` actually show as line
  breaks. If you add a new place that renders `entry.message`, it needs the same CSS rule or
  multi-line entries will look squashed.

If you're debugging "why did an entry look wrong/split/merged," start with these files, in this
order: `LogLineParser.cs` -> `LogEntryGrouper.cs` -> `LogTailBackgroundService.cs`.

## How the file viewer works (the Files page)

Everything lives in `FileBrowserService.cs` and shares no code with the log pipeline above.
Folders are registered separately (`FileFolderRegistry`) and never enter the date index.

- **Paths from the browser are `<rootIndex>/sub/dir/file.txt`.** The leading segment picks which
  of the registration's root paths applies, so a folder registered with several roots browses
  each of them. An empty path lists the roots, or descends into the only one when there is just
  one.
- **`FileBrowserService.Resolve` is the security boundary and every read goes through it.** It
  resolves with `Path.GetFullPath` and refuses anything not still under the root, then walks the
  path segment by segment refusing to cross any reparse point. Both halves are needed:
  `GetFullPath` handles `..` but not junctions, and checking only the final segment misses a
  junction in the middle of the path. Don't add a second resolution path that skips this.
- **Formatting never fails.** `.json` goes through `JsonDocument`, `.xml` through `XDocument`
  (keeping the `<?xml ?>` declaration, which `ToString()` drops on its own). A document that
  doesn't parse is returned exactly as read with a note - a malformed file is precisely when
  someone opens it, so a parse error must not become an error page.
- **Binary files are detected by a NUL byte in the first 8 KB, not by an extension allowlist.**
  A text file with an odd extension still opens, consistent with the rule that content is never
  rejected for failing to match a known shape.
- **Paging is byte-offset based and trims back to the last newline**, so the next page resumes on
  a clean boundary rather than mid-line or mid-character. The byte accounting is computed
  *before* redaction, because masking changes the text's length and the offset has to describe
  the file on disk. Pretty-printing is skipped for a paged read - a fragment isn't parseable.
- **Redaction is applied to viewer content and to search previews**, honouring `RedactSecrets`.
  The pane shows a visible marker when anything was masked rather than quietly differing from
  the file on disk.
- The frontend renders content as two `<pre>` columns (gutter + code), one text node each. An
  element per line locks the tab up on a file at the size cap.

## Repo layout

```
LogViewer/
  CLAUDE.md                          This file
  docs/prd/                          Requirements, split by phase (see below)
  Log_Viewer_WebApp_PRD.docx         Original PRD (v1.0) - superseded, kept for history
  Log_Viewer_WebApp_PRD_v1.1.docx    Updated PRD after stakeholder Q&A - source for docs/prd/
  LogHub_Design_Mockup.html          Early UI mockup (dashboard/live tail/search/admin screens)
  src/LogViewer.Web/                 The actual application (ASP.NET Core / .NET 8)
    Program.cs                      Minimal API + Razor Pages + SignalR + Serilog wiring
    Models/                         LogApplicationConfig, LogEntry, LogViewerOptions
                                    FileFolderConfig (Files page registration)
    Services/
      LogAppRegistry.cs             App registration store (App_Data/applications.json)
      FileFolderRegistry.cs         Files-page folder store (App_Data/filefolders.json)
      FileBrowserService.cs         Browse/read/search for the Files page; owns the path
                                    containment check that keeps reads inside a registered root
      LogFolderScanner.cs           Builds date -> files index (flat + dated-folder aware)
      LogLineParser.cs              Timestamp/level detection for a single line
      LogEntryGrouper.cs            Groups lines into multi-line-aware entries (batch/History)
      LogFileReader.cs              Shared-mode reads (logs are usually held open by their writer)
      LogQuery.cs                   Search expressions: AND/OR/NOT, phrases, optional regex mode
      LogRedactor.cs                Masks credential-looking values before entries leave the server
      LogTailBackgroundService.cs   Polls today's file(s), streams new entries via SignalR
    Hubs/LogHub.cs                  SignalR hub (per-app groups) + backlog replay on join
    Pages/                          Dashboard (Index), LiveTail, History, Files, Admin (Razor Pages)
    wwwroot/                        site.css, livetail.js, history.js, admin.js, files.js
    wwwroot/lib/signalr/            SignalR JS client, vendored (@microsoft/signalr 8.0.7).
                                    _Layout.cshtml references it on every page; when this folder
                                    was empty the 404 left `signalR` undefined and Live Tail died
                                    silently at startup. Keep it checked in - there is no CDN
                                    fallback and the tool is meant to run on an isolated network.
    App_Data/applications.json      Registered applications (created/edited via Admin page)
    App_Data/filefolders.json       Registered Files-page folders (same, separate store)
    Logs/loghub-*.log               LogHub's own Serilog output (created at runtime)
    sample-logs/                    Demo data used by the seeded CustomerPortal entry
    README.md                       Build/run instructions, setup gotchas (SignalR client lib, etc.)
```

## Working in this codebase

- **A .NET SDK is now available** (10.0.110 as of 2026-07-20; the project targets net8.0 and builds
  fine on it). Earlier changes were authored blind without an SDK - that is no longer the case, so
  always `dotnet build` and actually exercise the endpoints before claiming a fix works.
- Keep changes minimal and traceable to a specific request - this codebase has deliberately avoided
  speculative abstractions (see "Key decisions" above for examples of trade-offs that were made on
  purpose, not by accident).
- When extending log parsing/tailing, prefer generalizing the existing heuristics
  (`LogLineParser`/`LogEntryGrouper`) over adding format-specific special cases - the whole point is
  that LogHub shouldn't need to know in advance what's writing the logs it's reading.
- `docs/prd/00-overview.md` has the non-functional requirements (latency, resource footprint, no
  persisted log content) - check new features against these before assuming they're in scope.

## graphify

This project has a knowledge graph at graphify-out/ with god nodes, community structure, and cross-file relationships.

Rules:
- For codebase questions, first run `graphify query "<question>"` when graphify-out/graph.json exists. Use `graphify path "<A>" "<B>"` for relationships and `graphify explain "<concept>"` for focused concepts. These return a scoped subgraph, usually much smaller than GRAPH_REPORT.md or raw grep output.
- If graphify-out/wiki/index.md exists, use it for broad navigation instead of raw source browsing.
- Read graphify-out/GRAPH_REPORT.md only for broad architecture review or when query/path/explain do not surface enough context.
- After modifying code, run `graphify update .` to keep the graph current (AST-only, no API cost).

---
> Source: [127Ajay/LogHub](https://github.com/127Ajay/LogHub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-28 -->
