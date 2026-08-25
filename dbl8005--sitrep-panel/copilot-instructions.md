## sitrep-panel

> A live local HTML report you keep updated as you work. It lives at

# sitrep-panel

A live local HTML report you keep updated as you work. It lives at
`.sitrep-panel/report/` inside whatever project you're invoked from, served
on localhost, and it live-reloads in the browser every time you update it —
the user never has to hit refresh.

This is not a hosted artifact. It never leaves the user's machine, needs no
publish step, and works offline. Use it whenever the user wants to *watch*
a task happen rather than read about it after the fact — they'll typically
ask by name ("start a sitrep-panel", "/sitrep-panel") or describe wanting a
"progress report", "status page", or "live doc" for a task.

## Invocation

`sitrep-panel [start | open | stop | archive]` — default (no argument) is
`start`.

- **`start`** (default) — scaffold a new report if none is active, or resume
  the active one. Always ends by printing the localhost URL prominently.
- **`open`** — just print the URL of the active report (start the server if
  it isn't running); never touches content.
- **`archive`** — move the current active report to
  `.sitrep-panel/archive/<timestamp>-<slug>/` and scaffold a fresh one. Use
  this when starting genuinely new, unrelated work rather than continuing
  the current task.
- **`stop`** — kill the background server. Content on disk is untouched;
  `start` or `open` later brings it back.

## First run — scaffold

If `.sitrep-panel/report/` doesn't exist yet in the current project:

1. Create `.sitrep-panel/report/screenshots/`.
2. Copy `assets/template.html` (from this package's own directory) to
   `.sitrep-panel/report/index.html`. Replace `{{TITLE}}` with a short name
   for the task, and `{{SUBTITLE}}` with one line of context (what this
   report is tracking, and when it started).
3. Write `.sitrep-panel/report/meta.json`:
   ```json
   {"updated_at": "<current ISO-8601 UTC timestamp>"}
   ```
4. If the project is a git repo and `.sitrep-panel/` is not already covered
   by `.gitignore`, append it — this is local scratch, not something to
   commit. Mention you did this; don't ask first, it's trivially reversible.

If `.sitrep-panel/report/` already exists, skip scaffolding — resume it. Use
`archive` first for unrelated new work if you want a clean slate.

## Serving it

Start the bundled server in the background, from this package's own
directory (so the relative path to `scripts/serve.py` resolves regardless of
the caller's cwd):

```bash
python3 <package-dir>/scripts/serve.py <project>/.sitrep-panel/report
```

Run this via your background-execution mechanism — it blocks forever. It
prints exactly one line, `SERVING http://localhost:<port>/`, before
blocking; capture that for the URL. It auto-picks a free port (tries 8934
first, falls back to any free port on conflict), so several reports can run
for different projects at once. Stdlib-only Python, no dependencies.

Record the port for reuse: write `.sitrep-panel/report/.server.json`
(`{"port": <port>, "started_at": "<iso8601>"}`), gitignored along with the
rest of `.sitrep-panel/`. On a later `start`/`open`, read this file first —
if a `GET` to `http://localhost:<port>/meta.json` succeeds, the server is
already up; reuse that URL instead of spawning a second one.

**Always tell the user the URL after `start` or `open`, even if you just
reused an existing server.**

## The update protocol — what makes this useful

A sitrep-panel that only gets written once at the start is worthless.
Update it at every real step:

1. **Update "What's happening now"** (`#current-work-body`) with prose —
   what you're doing right now and *why*. Replace the block's content each
   time; it always reflects the present, not history.
2. **Prepend a new log entry** to `#entries` (newest-first — insert right
   after the `<div id="entries">` opening tag), using the exact `.entry` /
   `.entry-rail` / `.entry-dot` markup shown in the template's commented
   example — the timeline rail depends on that structure. One entry per real
   step: what happened, what was decided and why, what was verified. Skip
   entries for trivial reads; log entries for writes, decisions, and
   verification.
3. **Save screenshots** into `.sitrep-panel/report/screenshots/` and
   reference them with a relative `<img src="screenshots/whatever.png">` —
   inline in a log entry, appended to `#shots-body`, or both.
4. **Update the board** — the board is four columns, each with its own
   container: `#board-todo`, `#board-progress`, `#board-done`,
   `#board-blocked`. Add/move/remove a plain `<div class="board-card">…</div>`
   in the container matching its current status as steps start, block, or
   finish — moving a card means removing it from one column's container and
   appending it to another's, not changing a class. See the commented
   example markup in the template.
5. **Touch `meta.json` last, always.** Rewrite `updated_at` to the current
   ISO-8601 UTC timestamp as the final write of every update — the page
   polls this file every 2s and reloads on change. This is the one step
   you must never skip.

## Tracker-synced board (optional)

If the project has a working issue-tracker connection available right now
and there are real tickets relevant to this task, render the board from
that real data — real key, real title, real link, real status. Refresh it
each time you update the doc.

If no tracker is connected, or nothing relevant is filed there, fall back
to a plain manually-maintained checklist kept in sync by hand. Never
fabricate ticket data, and don't silently guess which tracker to use if
more than one is plausible — ask.

## Notes

- One active report per project at a time by design (`.sitrep-panel/report/`
  is a fixed path) — the URL stays stable across a whole session. Use
  `archive` to start clean.
- Genuinely static HTML plus polling JS — no build step, no external
  requests, works fully offline once loaded.
- If the user wants something shareable with people not at their machine,
  that's a hosted artifact, not this — this is specifically for
  local/private, zero-publish visibility.

---
> Source: [dbl8005/sitrep-panel](https://github.com/dbl8005/sitrep-panel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
