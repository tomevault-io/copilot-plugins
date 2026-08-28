## splunk-quest

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

**Splunk Quest** — a progress tracker for the team's Splunk certification ramp-up. Flask + SQLite,
server-rendered. No build step, no npm, no test framework, no accounts. All application state is
one SQLite file; the whole deployment is one container.

The curriculum is not invented here. It is extracted from the two `*-learning-path.md` reports at
the repo root, which are the **human source of truth**. Read *The content pipeline* below before
changing any content — it is the part that requires reading several files to see.

## Commands

Everything runs from `app/`. `seed.py`, `check.py` and `refresh_course_urls.py` are stdlib-only;
`app.py` and `smoke.py` need Flask, so a bare `python app.py` outside the venv is the usual
first-run stumble.

```bash
python3 -m venv .venv && .venv/bin/pip install -r app/requirements.txt

cd app
../.venv/bin/python seed.py     # curriculum.json -> ../splunk-quest.db (idempotent)
../.venv/bin/python app.py      # http://localhost:5000 — debug, auto-reload, dev only
```

```bash
cd app
../.venv/bin/python check.py                    # validate curriculum.json; prints a summary
../.venv/bin/python smoke.py                    # every route, against a throwaway database
../.venv/bin/python refresh_course_urls.py --dry-run   # re-scrape Splunk's catalog, report only
../.venv/bin/python refresh_course_urls.py      # ...and rewrite the urls in curriculum.json
```

`check.py` runs inside `seed.py` and refuses to load a file that fails, so seeding a broken
`curriculum.json` is not possible.

**There is no way to run a single test.** `smoke.py` is one `main()` with `ok(label, condition)`
assertions grouped under printed headings (`curriculum`, `identity`, `ticking`, `badges`,
`rollout`, `guards`, `re-seed`, …), no test framework and no selector. It builds its own database
in a tempdir, runs in a couple of seconds, and prints `✓`/`✗` per assertion with a failure list at
the end — run the whole thing. To iterate on one area, add assertions next to the matching heading.

Docker (the recommended deployment; the entrypoint re-seeds on every start):

```bash
docker compose up -d --build && docker compose logs -f
docker compose exec app python check.py
```

Port 5000 is contested — `python app.py` sits there in development and macOS AirPlay Receiver
hides there too. `SPLUNK_QUEST_PORT` moves the container's host port. Full deployment story,
including nginx, TLS and backups, is in `DEPLOY.md`.

## The content pipeline

```
splunk-security-learning-path.md        the reports — human source of truth
splunk-observability-learning-path.md
        │  hand-transcribed, section by section (every record carries `source`)
        ▼
app/curriculum.json                     the machine copy: certs, paths, tiers, items, resources
        │  app/refresh_course_urls.py writes the per-item `url` onto it
        │  app/check.py validates it (slugs, kinds, cert refs, cost tie-outs, links)
        ▼
app/seed.py                             upsert into SQLite — idempotent, re-run at will
        ▼
splunk-quest.db                         content tables read-only at runtime
        ▼
app/app.py + app/db.py + templates/     routes, queries, server-rendered pages
```

`curriculum.json` is **hand-built, not parsed**, and deliberately so: the same course appears in
up to five tables across the reports at different granularities and spellings, cells carry a
`✅ / ❌ / — / ★ / ¹²³` vocabulary, subtotals sit inline, and the free-course inventories are
`·`-delimited prose. A parser would need an alias map and constant maintenance for no benefit at
this size.

**When the plan changes, edit both the `.md` and `curriculum.json`, then re-seed.** The
`update-curriculum` skill (`.claude/skills/update-curriculum/`) walks that end to end — use it for
any content change.

## The invariant everything else rests on: content vs data

`app/schema.sql` splits the tables in two, and `seed.py` treats them differently:

| | tables | what `seed.py` does |
|---|---|---|
| **Content** — from `curriculum.json` | `path`, `tier`, `item`, `cert` | upserts by slug (or `path_id`+`code`); items absent from the JSON are **deleted**, and their progress rows cascade — *except* items in the `team-` slug namespace, which are the team's, not the file's |
| **Content, replaced wholesale** | `lean_stage`, `resource` | `DELETE` then re-insert on every seed |
| **Data — written by people** | `user`, `enrollment`, `progress`, `link`, `milestone`, `milestone_person`, `item_edit` | **never touched** |

That is why "deploy" is just `git pull && docker compose up -d --build`: there is no migration
step, and re-seeding cannot take the team's progress with it. The
`N people, N progress marks and N team milestones preserved` line `seed.py` prints is the receipt.
`smoke.py` asserts it. **Do not add a write to a data table from `seed.py`.**

**The rollout is entirely data.** A milestone is three things — which certifications, who needs
them, by when — written by the team at `/rollout` and stored in `milestone` + `milestone_person`.
Nothing about it comes from `curriculum.json`. Each report's §9 *suggested rollout* used to be
seeded into a `milestone_template` table to adopt from; that went, and with it the `label`,
`title` and `target` columns it required. `db.init()` carries an old-shape table forward and
drops the template — see *Two traps* below.

Two things follow that are easy to undo by accident:

- **The headline and the count are derived, never stored.** `db.milestone_headline()` builds the
  headline from the certifications — full name for one, joined abbreviations for several — and the
  count is how many assigned people hold the whole set. Don't add a title field back.
- **Certifications are the engine.** They are what a ticked exam is checked against, so a
  milestone must name at least one and must have at least one person on it. `milestone_board()`
  gives every person a state against the whole set — `all`, `some`, `none` — which is how the
  intersection rule (naming two means holding one counts for nothing) surfaces in the UI instead
  of needing a sentence to explain it.

## The team's edits to a path are data too

`/path` lets people reorder, remove and add items. That runs straight into the invariant
above — `seed.py` rewrites `item.ord` on every upsert and deletes anything the file no
longer has, and the container re-seeds on every start — so an edit written onto the item
row would not survive the next deploy. Three mechanisms keep it:

- **`item_edit`** holds the order override and the removed flag. Data half, never seeded.
  A move renumbers every row in the tier densely, hidden ones included, so positions
  cannot collide.
- **The `live_item` view** is `item` with the edits applied — removed rows gone, `pos`
  carrying the override. **Every read of an item goes through it**, and `ORDER BY i.pos`
  rather than `i.ord`. The two deliberate exceptions are `earned_certs()` and
  `held_by_person()`: passing an exam is a real-world fact, and taking the row off a path
  must not revoke a badge or un-meet a milestone. Writes go to the two tables underneath.
- **The `team-` slug namespace** marks an item the team added. It is on the row rather
  than in `item_edit`, so nothing — a dropped edit row, a departed teammate whose
  `added_by` goes NULL — can turn it back into content and get it swept. `check.py`
  reserves the prefix, so a slug in the file can never collide with one.

Three things that follow:

- **Removing a curriculum item hides it; removing an added one deletes it.** Deleting a
  curriculum row would only invite the next re-seed to put it back, with everyone's ticks
  on it gone — so it is hidden, stays restorable at the foot of its tier, and `progress`
  is never touched. An added row has no file to come back from, so it goes for good.
- **Added items are `is_lean = 1` with no `lean_stage`.** Somebody who puts a course on
  their path means it is part of their route; filing it behind the full-ladder toggle
  would be a surprise. The stage stays empty because the stages and their published costs
  are the report's and tie out against the report's items alone. Two `ORDER BY
  lean_stage IS NULL` guards in `app.py` stop a stage-less row jumping the queue.
- **A move swaps two positions.** Lifting the row out and re-inserting it beside its
  neighbour looks identical in the view it was done in, but drags the rows in between
  along: on the lean route, moving a rung down and back up would leave the full ladder
  rearranged. `smoke.py` asserts the round trip.

## Cost tie-outs

An item's `cost` is what you pay **taking it the way this app tracks it** — `0` where a free video
version exists, with the paid alternative recorded in the item's `note`. That makes the arithmetic
checkable, and `check.py` checks it: ticking every ⚡ lean item on a path must sum to exactly the
`lean_cost` the report publishes, and each `lean_stage` must match its own published cost. A
failing tie-out means a mis-transcribed price — fix the transcription, not the assertion.

Every money figure in `curriculum.json` is tied out this way. The one that used to be exempt —
the rollout's per-head `spend_head`, published in the reports as an approximation — left with the
rollout template.

## Canonical keys: one course, several paths

Most of the platform ladder appears on more than one path. `db.canonical_key()` derives an
item's real-world identity — `cert:<slug>` for exams, `course:<slugified title>` for everything
else — and stores it as `item.ckey`. Ticking one item ticks every twin, and `db.spend()`
deduplicates by `ckey`, so the Power User exam sits on all three paths and costs $130 once.

It lives in `db.py` rather than in `seed.py` (which calls it) because the app derives keys
too: a course somebody adds at `/path` has to land on the same key as the seeded copy
elsewhere on the ladder, or it is a second row charging a second time for one course.

**The gotcha:** the key comes from the *title*. Spell the same course two ways across paths and the
twins silently split — the tick stops propagating and the money is counted twice. No check catches
this. Titles of shared courses must match exactly — and that now includes a title somebody
types into the add form.

## Two traps in the template and schema layers

**An imported Jinja macro cannot see context processor globals.** `_macros.html` is pulled in with
`{% from "_macros.html" import ... %}`, so `current_user`, `certmap` and `all_paths` are undefined
inside those macros — *silently*, rendering as empty rather than raising. That is why `prereqs()`
is handed its `certmap` explicitly, and why "is this chip me?" is a `me` flag computed in
`db.milestone_board()` rather than a comparison in the template. If a macro needs request context,
pass it as a parameter or precompute it in `db.py`.

**There is no migration framework.** `db.init()` runs `schema.sql` with `CREATE TABLE IF NOT
EXISTS`, so changing an existing table's columns does nothing to a database that already has it.
The one migration lives in `db.init()` as `_retire_legacy_milestone()` and
`_carry_legacy_milestones()`: rename the old table aside *before* the schema script runs, let the
script create the current shape, then copy across what survives. `init()` returns notes and
`seed.py` prints them, so a deploy leaves a receipt. Any future column change needs the same
shape — detect, rename, re-create, carry, drop.

## Splunk Platform is synthesised

Neither report has a platform path; both are standalone budget plans that carry the platform ladder
inside their own architect tier (security §5 Tier 3, observability §5 Tier 3c). Pulling it out into
its own path removed the duplicate spine and made Advanced Power User and Cloud Admin earnable at
all — their exams previously lived nowhere, so those badges were unreachable. `check.py` now asserts
every certification has an exam item somewhere. Platform rows carry
`"source": "synthesised from the platform certification ladder — not in either report"` or a
certification-track PDF, not a report section.

## Where items link to

Every non-exam item must have an `https` `url` — `check.py` fails otherwise, because a row that
says "free video" and goes nowhere is a dead end. `refresh_course_urls.py` scrapes Splunk's two
public catalog pages and writes those urls; `url_kind` records what was found (`course` = a real
course page, `search` = a catalog search fallback, `null` = the item keeps its own link, e.g. free
trials and the public workshop). Exams have no `url` — they resolve to their certification track
page at render time.

The urls embed Splunk's Saba tenant id (`NA10P2PRD105`) and will eventually rot, which is why this
is a re-runnable script. **Don't hand-write course urls** — run the script; if a course lands on a
`search` fallback because our title differs from Splunk's catalog name, add the mapping to
`ALIASES` at the top of the script and re-run.

## Security posture

**There is no login.** Identity is a name typed into a box, remembered in a signed cookie. Anyone
who can reach the URL can become any teammate, tick their items, and rewrite the team's rollout.
That is a deliberate trade for a small trusted team and it does not survive the open internet —
access control belongs in the reverse proxy (`DEPLOY.md` carries an nginx site config with an
`auth_basic` block ready to uncomment). Don't build app-level auth without being asked; don't
widen the compose port binding off `127.0.0.1`.

Anyone may reorder, remove or add an item on any path, the same posture the rollout takes: a
path's item list is the team's, not one person's contribution, so there is no owner check
where `delete_link()` has one. `item_edit.added_by` records who added a row so the ladder
can say.

`clean_url()` in `app.py` is a real security boundary, not tidiness: `/links` values go straight
into an `href`, so a pasted `javascript:` URL would run in a teammate's session. It validates to
`http`/`https` with a host, and `smoke.py` asserts six rejected variants.

`splunk-quest.db` and `secret_key` are gitignored and excluded from the image — they are state, not
source. The database belongs outside the checkout and outside the image (`/data` on the volume under
Docker). `docker compose down -v` deletes it.

## Design system

Tokens at the top of `app/static/style.css`; the palette is adapted from joinastute.com. Two rules
keep it coherent, and both are easy to break by accident:

- **One accent.** Petrol `#146c72` carries everything interactive. `--path` (petrol for
  Observability, amber for Security, espresso for Platform) is path *identity* only — header rule,
  ladder rail, progress fill — and never colours a control. `--warn` means exactly one thing: a
  milestone whose date has passed.
- **Lowercase on `h1`/`h2` only.** `h3` labels real content, and acronyms (CDA, CDArch, O11y) stay
  as written.
- **A ground names its own ink.** `--champagne` is a light surface in the light theme and a dark
  one in dark, so anything sitting on it takes `--on-champagne`, never a fixed colour — the
  same shape `--on-accent` has. Pairing it with `--espresso` gave the instructor-led pill and
  every flash message 1.14:1 in dark. A new token used as a background in both themes needs the
  same companion.

Dark is the default and lives in `:root`; light is opt-in via the header toggle. Contrast ratios in
both themes are measured, not assumed. `[hidden] { display: none !important }` sits at the top of
the sheet because an author `display` beats the UA rule whatever the specificity — without it,
anything given a `display` silently stops honouring the attribute.

Every tier on a path starts **folded**. Its summary carries the bar, the count and the cost, so the
page reads as a whole path at a glance; `?open=<tier_id>` is the one thing that opens one, and it is
there so an edit redirects back to the rungs it happened to. Ticking works without JavaScript — each checkbox is a real
form POST; `app.js` upgrades it to a fetch call that updates bars, spend and badges in place.

## Further reading

| | |
|---|---|
| `README.md` | Operations — the five things that bite, quick starts, configuration, backups |
| `app/README.md` | How the app works, page by page, and what was deliberately left out |
| `DEPLOY.md` | Docker and systemd, nginx, TLS, backups, restore, troubleshooting |
| `.claude/skills/update-curriculum/` | The content-change workflow, field by field |

---
> Source: [sispehar/splunk-quest](https://github.com/sispehar/splunk-quest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
