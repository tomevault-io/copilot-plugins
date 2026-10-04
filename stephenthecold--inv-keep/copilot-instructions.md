## inv-keep

> Per-session ramp-up. **Read this first**, then [docs/PROJECT_STATE.md](docs/PROJECT_STATE.md)

# Working on Inv-Keep with Claude

Per-session ramp-up. **Read this first**, then [docs/PROJECT_STATE.md](docs/PROJECT_STATE.md)
for the full architecture digest. CHANGELOG.md tells you what each version added; this
file tells you how to actually *work* in the repo without re-learning the foot-guns.

## Verification recipe (run this on every meaningful change)

```bash
# Python syntax (uses the working venv at C:\temp\inv-keep-venv on this machine —
# the project's own ./.venv is a broken Linux symlink farm from OneDrive sync).
/c/temp/inv-keep-venv/Scripts/python.exe -m py_compile app/*.py

# JS syntax
node --check app/static/app.js

# Live UI verification — the Claude Preview MCP launch config is at the WORKSPACE
# root (../.claude/launch.json), NOT inside inv-keep/.  Starts uvicorn on :8096
# with data/preview.db.  See preview_start, preview_screenshot, preview_eval.
```

CI runs the equivalent on every push (`.github/workflows/ci.yml`) plus a growing
regression suite: cart-flow end-to-end, NaN-geo rejected, CSRF rejected without
token, XSS payload kept as JSON-escaped data, kiosk PIN lockdown enforces the
view/view_catalog/checkout floor (admin paths 403, catalog browse 200), per-item
stock-modal payload renders on `/parts?cat=all`, per-location stocktake adjusts
counts + writes audit rows. **When you change a URL or perm, expect to update
CI assertions too** — that's how every CI failure since v1.17 has played out.

## Known gotchas on this Windows + OneDrive box

- **Watchfiles + OneDrive miss new-file create events**. After a `Write` of a new
  module (vs an `Edit` of an existing one), restart the preview server cleanly:
  `preview_stop` → delete `app/__pycache__` → `preview_start`. Otherwise you'll see
  `ImportError: cannot import name 'foo' from 'app'` even though the file exists.
- **PowerShell drops Secure cookies over plain HTTP**. The session cookie is
  `Secure=True` unless `DISABLE_AUTH=1`. So PowerShell-driven smoke tests against
  `http://localhost:8000` get CSRF-rejected — POST handlers never see the session.
  Either set `DISABLE_AUTH=1` in `.env` for local smoke OR test through the browser
  via the Preview MCP.
- **The project's `.venv/` is unusable** on this machine (OneDrive turned the Linux
  symlinks into 8-byte text files). Use `C:\temp\inv-keep-venv` instead, or rebuild
  the venv freshly inside the project on a non-OneDrive disk.
- **gh CLI and Docker aren't on the default PATH.** Use the full paths:
  - `C:\Program Files\GitHub CLI\gh.exe`
  - `C:\Program Files\Docker\Docker\resources\bin\docker.exe`

## Known gotchas in production / on the docker host

- **`data/` ownership.** The container runs as uid `10001` (see Dockerfile);
  the bind-mounted `./data` host directory MUST be writable by that uid or
  every write throws `sqlite3.OperationalError: attempt to write a readonly
  database` — and the user-visible symptom is a 500 on whatever POST tries
  to flush an audit-log row (settings saves were the first place we saw
  this). Fix on the host: `sudo chown -R 10001:0 ./data && sudo chmod -R
  u+rwX ./data && docker compose restart inv-keep`. `install.sh` does the
  chown for fresh installs; restores / manual file ops can re-flip it.
- **GitHub Releases creates the tag, not the other way round.** The
  release pipeline (`.github/workflows/release.yml`) triggers on push of
  any `v*` tag and publishes a multi-arch image to GHCR. The Releases UI
  is the ONLY way to create the tag from the browser: Draft new release →
  type the version into "Choose a tag" → click **"+ Create new tag:
  vX.Y.Z on publish"** → Publish. Editing the *title* of an existing
  release does nothing — the workflow doesn't fire and `:latest` on GHCR
  stays put.
- **The tag name is typed by hand into a text box, and nothing validates
  it.** Every release mishap in this repo so far has been a typo in that
  box, and each fails differently:

  | Typed | What happened |
  |---|---|
  | `v,1.40.0` | stray comma — still matched `v*`, so it published under a junk tag |
  | `v1.14.1` | transposed digits for what the CHANGELOG calls v1.41.1 — published under the wrong version |
  | `V1.42.0` | **capital V** — `tags: ['v*']` is case-sensitive, so the pipeline never fired at all |

  Lowercase `v`, then the exact version from `app/version.py`. Nothing
  downstream will correct you.
- **Verify the release actually published — two checks, both needed.**
  1. The tag exists *and* has the right case. Grep case-insensitively so a
     wrong-case tag shows up instead of looking absent:
     `git ls-remote --tags origin | grep -i 'X\.Y\.Z'`
  2. The **Release image** workflow ran with event `push` (not
     `workflow_dispatch`) on that tag, and its "Derive image tags" step
     lists `:vX.Y.Z` + `:vX.Y` + `:latest`.
- **A manual `workflow_dispatch` run is NOT a substitute for the tag
  push.** It's tempting when you notice the pipeline didn't fire, and it
  looks like it worked — the run goes green and `:latest` does get
  updated. But `docker/metadata-action` derives tags from `github.ref`,
  not from the `ref` input, so a dispatch on `main` produces only
  `:main`, `:latest` and `:sha-<short>`. The `type=semver` patterns never
  match, so **`:vX.Y.Z` and `:vX.Y` are never created** and the image is
  mislabelled `org.opencontainers.image.version=main`. Hosts on the
  default `:latest` are fine; anyone pinning `INV_KEEP_VERSION=vX.Y.Z` in
  `docker-compose.yml` gets a pull failure. The fix is always to delete
  the bad tag + release and re-publish with the correct lowercase tag —
  that fires the real pipeline. (v1.42.0 hit exactly this.)
- **"docker compose pull" silently no-ops when the registry hasn't
  changed.** Tells: `Pulled` finishes in <1s AND `up -d` reports
  `Container inv-keep Running` (not `Recreated`/`Started`). Means GHCR's
  `:latest` SHA matches what's already on disk — almost always because the
  release workflow hasn't published a new image yet. Confirm with
  `docker image inspect ghcr.io/stephenthecold/inv-keep:latest --format
  '{{.Created}}'` — old date = no new image.

## When you add X, also do Y

| Adding… | …also do |
|---|---|
| A new POST route | Nothing extra for CSRF — the middleware covers everything except `/auth/callback`. Add `@app.post(...)` and you're done. |
| A new HTML form | Include `<input type="hidden" name="_csrf" value="{{ csrf_token }}">` inside the `<form>`. Forms with the split `<form id=...></form>` + `<input form=...>` pattern need the csrf hidden put INSIDE the empty form element. |
| A new `fetch(..., {method: 'POST'})` in `app.js` | Pass `headers: csrfHeaders(...)` (helper at top of the file). Multipart POSTs use `{'X-CSRF-Token': csrfToken()}` directly — don't override the multipart `Content-Type`. |
| A new column on an existing table | Add it to `app/database.py:_ADDED_COLUMNS` for the additive migration. SQLite can't ALTER a column's NOT NULL — see `_relax_transactions_customer_id` for the rebuild pattern if you need to relax a constraint. |
| A new Jinja template rendering dollar values | Use `{{ x | money(cfg.currency) }}`. Never `'%.2f'|format(x)`. Mirror in JS as `money(n)` (calls `ceilCents` internally — rounds UP to next cent). |
| A new endpoint accepting lat/lng | Validate with `_finite()` not just range checks — NaN compares False against everything, so `lat < -90 or lat > 90` is bypassable. |
| A new Jinja global / window.* | Goes in `<head>` of `base.html`, NOT after `{% block content %}` — block-internal IIFEs run BEFORE the `</main>`-and-after script tags. Bit me twice this build. |
| A new inline `<script>` inside a `{% block content %}` that calls helpers from `app.js` | Wrap in `document.addEventListener('DOMContentLoaded', ...)` — `app.js` is loaded after `</main>` and isn't defined when block scripts execute. |
| A new Leaflet popup or DOM render with user-controlled strings | Build via `document.createElement` + `.textContent`. Template literals + `innerHTML` are an XSS vector — that's the v1.10.1 fix you're not allowed to re-break. |
| A new top-level nav link (Items / Clients / Records sections) | Also add it to the mobile drawer in `base.html` (the `<aside class="nav-mobile">` block). The desktop `<nav>` is hidden below 720px and the drawer is a *separate* flat link list — adding to one and not the other will leave phone users unable to reach the page. v1.12.2 pattern. |
| A new user-uploadable brand asset (favicon, logo, …) | Mirror the favicon route in `app/main.py:settings_branding_favicon`: allowlist of image content-types **excluding SVG** (SVG executes script when fetched from `/uploads/` — stored XSS), size cap, stable filename so the old one is overwritten, cache-bust via `?v=<hex>` query string on the stored path, `audit.record(...)` + `db.commit()`. Add the setting to `settings_store.DEFAULTS` and the upload form to `templates/settings.html`. |
| A new HTML response served behind auth | Nothing extra — `auth_middleware` already stamps `Cache-Control: no-store, must-revalidate, max-age=0` + `Pragma: no-cache` on every `text/html` response so mobile bfcache can't hand back a stale CSRF token. Don't strip those headers downstream. |
| A new GET route exposing the catalog / clients / jobs | Route is gated by `view_catalog` via `rbac.required_perm` (path-prefix rule). Default Kiosk role has `view_catalog`; default everyone else has it too. If you want a path to require *more* than `view_catalog`, add an explicit `if path.startswith("/your-path"): return "manage_X"` branch *above* the catalog block in `required_perm`. v1.21. |
| A new column that exposes our cost / margin to the UI | Gate the render with `{% if can('see_cost') %}`. Default Kiosk does NOT have `see_cost`, so a shared front-desk device shows client price only. Mirror this on the matching Add/Edit input — when hidden it should still POST as `<input type="hidden" name="unit_cost" value="0">` so the form-body shape doesn't change. v1.21. |
| A new path that kiosks should reach by default | Add it to `_KIOSK_ALLOWED_PREFIXES` or `_KIOSK_ALLOWED_EXACT` in `app/main.py`. The allowlist is the **floor** that applies while the Kiosk role still has only its built-in perms (`{view, view_catalog, checkout}`); adding any extra perm to the Kiosk role under /users#roles lifts the lockdown and RBAC alone governs. Conservative default: leave it OUT of the allowlist so admins opt in by editing the role. v1.20.1 / v1.21. |
| A new section to `/settings` | Wrap in `<section class="card settings-tab" id="tab-<slug>" data-tab-pane="<slug>" data-tab-title="<title>">…</section>` and add a matching `{{ item("<slug>", "<label>", "<emoji>") }}` line to `_settings_nav.html`. The tab-switching JS at the bottom of `settings.html` flips visibility per `data-tab-pane`; localStorage remembers the last tab across POST→redirect, so form handlers can keep redirecting to `/settings` (no `?tab=` needed). v1.20.0. |
| A bulk action on selected items | Add a `/parts/bulk/<thing>` POST (manage_items, since it's under `/parts`), a modal in `parts.html` whose `.bulk-ids` div + `.bulk-next` input the `openBulk()` JS fills from the ticked `.bulk-check` rows, and a button in the `.bulk-bar`. Route back via `_bulk_return(form)` so the operator lands on the same `?cat=` view. v1.24.0. |
| A new role-scoped thing on `/users` | The page is one nested view (v1.24.0): each role is a `<details class="role-group">` listing its users (grouped in `users_page` via `role_users` / `no_role_users`) with the permission editor in a nested `<details class="role-perms">`. The shared user row is the `user_row()` macro — reuse it. One "Users & roles" sidebar entry now, no `#roles` anchor. |
| A new place that lists items / parts the operator can pick | If the source is `/api/search`, the suggestion row already dims + strikes through zero-stock items via the `.oos` class. Match it elsewhere: check `qty <= 0` and apply class `oos` + a `<span class="tag oos-tag">Out of stock</span>` badge. Do NOT auto-archive items at zero stock — archive is a deliberate admin action. v1.21. |
| A new place that displays item on-hand counts | If `pack_size > 1`, show the derived `qty // pack_size` packs + remainder hint under the count (see parts.html "On hand" cell). Stock + billing stay per-unit — the pack-size is a display convenience, not a separate currency. v1.25. |
| A new path that prints labels for arbitrary user input | Reuse `/labels/print?value=…&name=…` (ad-hoc, no Part required). Sanitize the value with the same length + control-char guard already in `labels_print_adhoc` before handing to `labels.render_svg` — the Code128 encoder will happily emit a 50k-pixel SVG for a 200-char string. v1.25. |
| A new test against the items table in CI | Hit `/parts?cat=all`, not `/parts` — v1.18 turned the root URL into a category browser (cards, not item rows). The flat table only lives under `?cat=all`. The stock-modal regression test learned this the hard way in PR #6. |
| A new `<select>` or `<input type="checkbox">` anywhere | Nothing extra — the app-wide chrome in `style.css` (custom caret + drawn checkmark) covers every control on every page. Do **not** re-scope those rules to one page or hand-roll a look-alike; identical controls must look identical everywhere (Law of Similarity). v1.42. |
| A new `<form method="post">` that navigates | Nothing extra — the delegated `submit` handler in `app.js` puts the submitter into a spinner + "Saving…" state and blocks double-submits. If the form is JS-handled, call `preventDefault()` (the handler skips `defaultPrevented` events) and own the busy state yourself. Never call `form.submit()` — it bypasses the handler; use `form.requestSubmit()`. v1.42. |
| A new icon-only control (emoji link, ✕, 💬) | Give it a real hit box, not bare glyph text: `class="icon-link"` for anchors, or a `button` that inherits the button system. The `@media (pointer: coarse)` block enforces a 44px floor — don't opt a control out of it with `min-height: 0`. v1.42. |
| A new cluster of ≥3 related settings fields | Wrap it in `.subgroup` (always visible) or `<details class="subgroup-fold">` + `.subgroup-body` (collapsed by default, with a state chip in the `<summary>` so the closed state still reports what's configured). Prefer the fold once a tab shows more than ~7 controls. v1.42. |
| A one-off `style="…"` you're about to type in a template | Don't. Add a named class to `style.css` instead — see the design-principles section below. The only surviving inline styles are genuinely per-row computed values (e.g. `.cat-indent` depth). |

## Design principles (apply to every UI change)

The UI is held to a fixed set of usability laws. They're not decoration: each
one below cashes out to a concrete, checkable rule in *this* codebase. A change
that breaks one is a bug even when the feature works.

**Vocabulary note.** The brief these came from is written in WordPress-admin /
Gutenberg / Tailwind terms. None of those exist here — this is FastAPI + Jinja2
+ one hand-written `app/static/style.css`. Translate before you go looking:

| Brief says | Here it means |
|---|---|
| `PanelBody`, Card | `.card` (a page section) · `.subgroup` (a related field cluster inside a form) · `details.subgroup-fold` (the same, collapsed) |
| `gap-2`, `px-4`, `space-x-2` | the rem spacing scale already in `style.css` — `.25 / .4 / .5 / .75 / 1 / 1.5rem`. There is no utility framework; add a **named class**, never an inline `style=`. |
| `text-lg font-semibold` | the type scale: `h1` → `h2` → `h3.settings-subhead` → `.section-h2` → `small.muted`. Lives in CSS, never inline. |
| WP list table | `table.lines` wrapped in `.table-scroll` |
| WP top bar + sidebar | `<header>` in `base.html` + `.settings-nav` (the shared `/settings` + `/users` shell) |
| Modal | `<dialog class="modal">` opened via `openModal(id)`, actions in a `<menu class="modal-actions">` |
| Toggle / status switch | plain `input[type=checkbox]` — the app-wide chrome makes it read as a switch |

### The laws, as rules

- **Aesthetic-Usability — spacing and type carry the perceived ease.** Forms are
  `.grid-form` (flex + `gap: .75rem`), sections are `.card`, sub-headings inside
  a card are `h3.settings-subhead` (which draws the separator rule for you).
  Reach for a named class; an inline `style=` in a template is the smell that
  says "this spacing is unique", and it almost never is.
- **Hick's Law — fewer visible choices per screen.** `/settings` shows exactly
  one tab at a time; `/parts` drills down instead of listing everything;
  `/report`'s client filter is a `<details>` popover, not 40 inline checkboxes.
  When a tab grows past ~7 controls, fold the optional ones.
- **Jakob's Law — behave like the admin UI people already know.** "+ Add …" is
  a filled primary button top-right in `.page-head > .head-actions`; destructive
  actions are `.danger` and last in the row; rows live in a table; edits happen
  in a `<dialog class="modal">`; the left sidebar is the section switcher.
  Don't invent a new interaction where one of those fits.
- **Fitts's Law — important targets are big and close to the pointer.** The
  button system already sets `min-height: 2.4rem`; `@media (pointer: coarse)`
  raises everything (including `.small` variants, `.ms-tab`, `.comment-btn`,
  `.tile-remove`, `.cat-card-admin` buttons) to a **44px floor**. Icon-only
  targets get `.icon-link` so the glyph has a box around it. Primary action sits
  where the thumb already is — that's why `.cart-footer` sticks to the bottom of
  the viewport on phones and why `.stocktake-bar` / `.bulk-bar` are sticky.
- **Law of Proximity — group by containment, not by hoping.** Related inputs go
  in one `.subgroup` (dashed border) or `.subgroup-fold`. Button clusters use
  `.row-actions` / `.head-actions` / `.modal-actions` so the gap is uniform;
  never let a Save and a Delete butt together with no gap.
- **Zeigarnik Effect — never leave an action in limbo.** Every navigating POST
  gets the delegated busy state from `app.js` (spinner + "Saving…" + the form's
  submit buttons disabled until navigation). Async work reports inline: see
  "Check for updates" on the Version tab and `#cart-submit-hint` on the scan
  page. A collapsed `.subgroup-fold` must still show its on/off chip in the
  summary — hiding a section is fine, hiding its *state* is not.
- **Goal-Gradient — always name the next step.** A disabled primary button must
  say why it's disabled and what unblocks it (`#cart-submit-hint`: "Add an item
  to continue" → "Pick a client to continue"). The scan page's `.ct-pill`s light
  up (`.is-set`) as each target is chosen — that's the progress bar.
- **Law of Similarity — one control, one look, everywhere.** Checkbox and
  `<select>` chrome is defined **app-wide**, not per page. It was once scoped to
  `.settings-shell`, which left `/parts`, `/jobs`, `/clients` and the scan page
  rendering raw OS controls next to styled ones — that's the exact bug the rule
  exists to prevent. Same for buttons: use `.btn` / `.ghost` / `.danger`, don't
  hand-roll a fourth variant.
- **Miller's Law — chunk, and default the advanced chunk closed.** The Alerts
  tab's daily / weekly / monthly schedules are three `details.subgroup-fold`s
  that open only when that report is enabled — ~20 controls become 3 lines plus
  whatever you actually use.
- **Doherty Threshold — under 400ms, or show something.** Debounce typeahead at
  180ms (`/api/search`, global search) and never block on it. Anything slower
  than a paint gets a visible state before the request goes out: disable the
  button, swap the label, show the spinner. `tryGetGeo` is capped at 4s and
  resolves `null` rather than stalling a scan.

### Quick design review (run before committing a UI change)

1. Any new `style="…"` in a template? Move it to a named class.
2. New button — does it use `.btn` / `.ghost` / `.danger`, and is it ≥44px on a
   coarse pointer?
3. New form — does it navigate? Then it must inherit the busy state (don't call
   `form.submit()`).
4. New `<select>` / checkbox — does it look identical to the ones on `/settings`?
5. Did the screen gain more than ~7 visible controls? Fold the optional ones.
6. Is there a disabled primary button anywhere with no explanation next to it?
7. Desktop `<nav>` **and** the mobile drawer both updated? (see the table above)

## Conventions

- **Versioning**: SemVer. Bump `app/version.py`, add a CHANGELOG entry under a new
  `## [X.Y.Z] — YYYY-MM-DD` heading, commit. Tag + image-publish is driven by
  GitHub Releases: Draft new release → "Create new tag vX.Y.Z on publish" → target
  `main` → Publish, which fires `release.yml` and pushes a multi-arch image to
  `ghcr.io/stephenthecold/inv-keep` as `:vX.Y.Z` + `:vX.Y` + `:latest`. Per-version
  commits, one per release, is the established pattern.
- **Git author** (per-repo, already set in this checkout): `Inv-Keep <noreply@anthropic.com>`.
- **CHANGELOG voice**: bullets describing the **why** + the **what**, not file lists.
  Group user-facing features above plumbing.
- **Secrets** (`.env`, certs, keystores, `data/*.db`, `data/uploads/`) are gitignored
  and must NEVER be committed. The local `.env` also includes a 64-char dev
  `SESSION_SECRET`; the app refuses to start with anything shorter or the placeholder.
- **Currency display**: ceiling-rounded to the next cent everywhere (so client-facing
  totals never under-bill). Stored values keep full precision; rounding is at the
  presentation layer (`money_filter` in main.py / `ceilCents` in app.js).
- **Time display**: UTC in the DB, configured-tz in templates via `| local_dt(cfg.timezone)`.
  The scheduler uses server clock (UTC in Docker) regardless of the UI tz setting.

## How the post-v1.16 surfaces actually work

These changed enough that the old mental model from v1.16 will mislead you.

### `/parts` is a drill-down browser, not a flat table (v1.18+)

`/parts` has four views, picked by the `cat` query param:

| URL                | View                                                         |
|--------------------|--------------------------------------------------------------|
| `/parts`           | Root — top-level category cards (no item rows)               |
| `/parts?cat=<id>`  | Detail — sub-category cards + items directly in that category |
| `/parts?cat=none`  | Uncategorized items only                                     |
| `/parts?cat=all`   | Legacy flat table, items grouped by category path            |

Two behaviors that surprise people:

- **Empty categories are hidden in browse mode** (subtree-count 0).
  Toggle **Manage** (`?manage=1`) to see every category and get inline
  rename / + Sub / Delete on each card. The Manage toggle is also the
  way to add a sub-category to a parent that has no items in it yet.
- **Single-child chains auto-skip**. Tapping `Wiring` when its only
  populated descendant is `Wiring → Ethernet → Cat 6 → 7ft` drops the
  user straight at `7ft`; the breadcrumb still shows the full path so
  navigation history isn't lost. Disabled in Manage mode so you can
  park on an intermediate to rename / add siblings.

### `/locations/<id>` is the per-location audit + stocktake page (v1.19+)

Every row on `/locations` is a link into `/locations/<id>` which shows
everything stocked there with a bulk stocktake form (prior / new /
live Δ chip per row, single Save commits every changed row). Add `?zero=1`
to also show items that aren't stocked here yet — useful during a count.
`POST /locations/<id>/stocktake` walks `qty_<part_id>` form fields and
writes one `part.stock_set` audit row per change + a `location.stocktake`
rollup so the audit page shows the count as a single event.

### `/settings` and `/users` share a sidebar shell (v1.20+)

`_settings_nav.html` is included by both pages. On `/settings` tabs are
client-side: sections render with `display: none` and only the one
matching the URL hash / localStorage value is shown. POST handlers can
keep redirecting to plain `/settings` — the JS reads
`localStorage['inv-keep:last-settings-tab']` to restore where the user
was. Add a section: see the table above. `/users` is a single combined
"Users & roles" page (v1.24.0) — roles are expandable cards with their
users nested inside; the sidebar has one Access entry for it (no more
separate Users / Roles tabs or `#roles` anchor).

### Kiosk permissions (v1.20.1 + v1.21)

Two layers, in order:

1. **`auth._kiosk_user(db)`** loads the live perm set from the Kiosk
   role row in the DB. It used to hardcode `{view, checkout}` — that
   was the bug v1.20.1 fixed. Whatever permissions you set on the
   Kiosk role under `/users#roles` are what the session gets.
2. **`auth_middleware` lockdown floor**. The hardcoded path allowlist
   (`_KIOSK_ALLOWED_EXACT` + `_KIOSK_ALLOWED_PREFIXES`) is enforced
   **only while** `_kiosk_lockdown_active()` is true — i.e. the Kiosk
   role still has only its built-in floor permissions
   (`{view, view_catalog, checkout}`). The moment an admin grants any
   other permission to the Kiosk role (e.g. `view_audit`,
   `manage_items`, `manage_locations`), the lockdown lifts and
   standard RBAC alone governs the session. Settings → Kiosk PIN
   explains this to the operator.

Default Kiosk reach under the lockdown: `/`, `/transactions`,
`/parts`, `/categories`, `/clients`, `/jobs`, `/api/cart*`,
`/api/search*`, `/api/checkout*`, `/api/void*`. Not in the floor:
`/locations`, `/transfers`, `/map`, `/report`, `/labels`, `/audit`,
`/settings`, `/users` — those require explicit perm grants on the
Kiosk role.

### Item lifecycle (v1.21)

Items can now be Archived (reversible, history preserved) or Deleted
(permanent, refused if any `Transaction` or `TransferLine` references
the part — the error message tells the user to archive instead). Routes:

```
POST /parts/<id>/archive   → archived=True   (manage_items)
POST /parts/<id>/restore   → archived=False  (manage_items)
POST /parts/<id>/delete    → row gone        (manage_items, refused on history)
```

The Edit-item modal renders Archive + Delete buttons that retarget
their `form.action` via JS based on `data-archived`. Items at zero
stock are NOT auto-archived — they get an amber "Out of stock" tag
and dim in the cart-bar search. Archive is always an explicit choice.

### Update check (v1.20+)

`GET /settings/check-update` proxies the GitHub Releases API
(`/repos/stephenthecold/inv-keep/releases/latest`) with a 5-minute
DB cache (`store.get/set("_update_cache")`) and returns
`{current, latest, behind, url, error?}`. The Settings → Version &
updates tab calls it via fetch and renders the result. The app
deliberately does not self-update — the recipe is `docker compose pull
&& docker compose up -d` on the host.

## File layout (10-second tour)

```
app/
  main.py         routes + middleware (incl. /welcome splash + no-store stamp) + cart API + map endpoint + helpers
  models.py       Order, Transaction (w/ order_id + lat/lng), Part (w/ archived), …
  database.py     ensure_columns() additive migrations + a SQLite-rebuild path
  csrf.py         pure ASGI CSRFMiddleware + self-recovering 403 page (meta-refresh to same-origin Referer)
  orders.py       cart helpers + ORD-YYYYMM-NNNN generator
  reports.py      build_report_range — outerjoins Order and excludes open/cancelled carts
  static/
    app.js                cart UI + Leaflet popups + global search + mobile nav drawer toggle
    style.css             desktop styles + the < 720px breakpoint where the header collapses to brand + hamburger, plus print rules that hide app chrome for label printing (v1.20)
    vendor/leaflet/       1.9.4, self-hosted (no CDN)
  templates/
    base.html             header + mobile drawer + favicon link
    scan.html             cart card (cart-lines table hides barcode + unit cols on phones)
    parts.html            drill-down category browser (v1.18) — cat-grid + breadcrumb + Edit/Stock/Archive/Delete modals
    location_detail.html  per-location stocktake form (v1.19)
    settings.html         tabbed settings shell — sections wrapped in .settings-tab[data-tab-pane]
    users.html            users + roles inside the same settings shell
    _settings_nav.html    shared sidebar partial — both settings.html and users.html include this
    label.html            label print page (sized + sheet variants; @page CSS pins physical dimensions)
docker-compose.yml   inv-keep (pulls ghcr.io/stephenthecold/inv-keep:${INV_KEEP_VERSION:-latest}) + optional caddy (profile "ssl")
docker-compose.dev.yml override that adds `build: .` for local-source iteration
.github/workflows/release.yml  tag-push trigger that builds + pushes the multi-arch image to GHCR
install.sh           interactive installer (hostname, TLS, OIDC → .env → compose up)
scripts/quickstart.sh  one-line bootstrap (clone + install.sh) for curl-pipe / git-clone
```

## Don't break

- The cart submit retry on `IntegrityError` (`api_cart_submit`) — pins the order#
  race fix from v1.10.1.
- The `_finite()` geo guard — pins the NaN bypass fix.
- The `{% if user.is_admin %}` gate around `window.DEFAULT_MARKUP_PCT` in `base.html` —
  pins the markup % leak fix.
- The `c.lines.forEach` row builder in `app.js` using `createElement` + `textContent` —
  pins the cart XSS fix.
- The `Cache-Control: no-store` stamp on `text/html` responses inside
  `auth_middleware`, and the `wants_html` → `/welcome` branch — pins the
  mobile sign-in fix (bfcache restoring a stale CSRF token, plus the
  multi-redirect login loop). v1.12.1.
- The favicon / logo upload allowlist that excludes SVG. SVG served from
  `/uploads/` (same origin as the app) is a stored-XSS vector. v1.12.2 +
  v1.10.1.
- The mobile drawer markup (`<aside class="nav-mobile">`) and its
  `< 720px` CSS in `style.css`. If you move things around in `base.html`,
  keep both the desktop `<nav>` and the drawer in sync so mobile users
  retain every link. v1.12.2.
- The `_kiosk_lockdown_active()` check in `auth_middleware`. It must
  read the *live* perm set from `user["perms"]`, not a snapshot, or
  edits to the Kiosk role under `/users#roles` won't take effect on
  existing sessions. v1.20.1.
- The `Part.archived` flag is set ONLY by explicit user action (the
  Archive button on the edit modal, or the CUSTOM-… walk-in item
  creation at `/api/cart/custom`). No code path archives a part on
  zero stock — items hitting zero just render with the OOS badge.
  v1.21.
- The `/parts/<id>/delete` "has history" refusal — items with any
  `Transaction.part_id == id` or `TransferLine.part_id == id` must
  not be deletable, since their history would lose the part name.
  v1.21.
- The `view_catalog` permission gates GET on `/parts`, `/categories`,
  `/clients`, `/jobs`, `/labels`, `/map`, `/report` in
  `rbac.required_perm`. Every default role except Kiosk also has
  `see_cost` — gating cost columns / inputs behind `can('see_cost')`
  is what keeps a kiosk session from seeing margin on a shared
  device. v1.21.
- The `.label-barcode svg` print rule that scales by **height** and
  centers via `margin: 0 auto`. The earlier `width: 100%` rule
  left-anchored the python-barcode SVG's fixed-mm output and was the
  centering complaint from the user. v1.21.
- The `Part.archived == False` filter inside `/api/search`. Without
  it, archived CUSTOM-… walk-in items keep showing up in the
  cart-bar autocomplete after their order has closed. v1.20.1.
- The control-char + length guard in `labels_print_adhoc`
  (`/labels/print`). Letting a 200-char or NUL-laced value reach
  `labels.render_svg` produces an enormous SVG or a crashed
  encoder. Re-validating before encode is part of the route, not the
  encoder. v1.25.
- The barcode-rebrand path on `/parts/<id>/edit` clears
  `barcode_generated=False` whenever a value is supplied. The
  `/labels` sheet trusts that flag to decide what to bulk-print by
  default — leaving it True after a rebrand silently includes the
  manually-typed code in every bulk run. v1.25.
- The **app-wide** (not `.settings-shell`-scoped) `input[type="checkbox"]`
  and `select` chrome in `style.css`. Re-scoping either one to a single
  page is what produced the v1.42 Law-of-Similarity bug: styled controls
  on `/settings`, raw OS controls on `/parts`, `/jobs`, `/clients` and
  the scan page. The `.grid-form` / `.lines` / `dialog.modal` select rules
  must keep using `background-color:`, never the `background:` shorthand —
  the shorthand wipes the caret `background-image`. v1.42.
- The delegated `submit` listener in `app.js` and its
  `if (e.defaultPrevented) return` guard. The guard is what keeps
  JS-handled forms (`#custom-form`, `#oc-form`) from being frozen in a
  "Saving…" state that never resolves, and the deferred disable
  (`setTimeout(…, 0)`) is what keeps a `formaction` submitter's value in
  the POST body. v1.42.
- The `@media (pointer: coarse)` 44px tap-target floor. Adding
  `min-height: 0` to a control to make it visually compact opts it out on
  phones — set the compact size inside the default (fine-pointer) rule
  instead and let the coarse block win. v1.42.

## Where to push back

If the user asks for something that breaks the patterns above, raise it before
doing it. Same if they ask to commit `.env` / `data/` / `certs/` — those are
gitignored for good reason.

---
> Source: [stephenthecold/inv-keep](https://github.com/stephenthecold/inv-keep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-04 -->
