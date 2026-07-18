## my-family-story

> Same system as the Execution Status project: local edits accumulate as ops,

# Family Tree — working notes

## Change tracking & publish ("chat = server, baseline = database")
Same system as the Execution Status project: local edits accumulate as ops,
the user copies a change-set from the Save dialog and pastes it in the chat,
and Claude "bakes" it into the shared baseline.

### Architecture
- **Working copy** (auto-saved sidecars, shared by everyone in the design tool):
  - `.image-slots.state.json` — photo drops & crops as data URLs (written by `image-slot.js`)
  - `.ft-doc.state.json` — the ops log + tree-exclusion overrides + **user-authored moments** (written by `tree-store.js`)
- **Baseline**: `tree-seed.js` (`window.FT_SEED = { seedStamp, baseV, slots, excluded }`).
  `slots[id] = { src, w, h, s, x, y }` where `src` is a REAL project file under `photos/`.
  `image-slot.js` getSlot() falls back to the seed; a `{del:1}` sidecar marker shadows a seeded photo.
- **Ops** (`tree-store.js`): every photo add/replace/remove, avatar crop, hide/show-on-tree,
  field edit, and STRUCTURAL change (person/union/side-link add+remove — Edit mode) records
  `{ts, kind, id|key|ekey|pid|uid|side, label, prev*, pub}`. `prevRaw` = raw sidecar value before the
  change (for undo/discard). Slot ids: `ph-<person>-e<idx>` milestone photo, `av-<person>-e<idx>`
  avatar crop. Exclusion keys: `<person>-e<idx>`.
- **Field edits (Records Room)**: `doc.edits[ekey] = value` + op `{ts, kind:'edit', ekey, label, prevV, prevENone?, pub}`.
  ekey grammar: `p:<personId>:<dot.path>` person field (name.en, name.he, bio.en, bio.he, birth.year,
  birth.place, death.year, death.place, sheet.look, sheet.items — the last two are the character
  sheet feeding image prompts; bake as a `sheet: { look, items }` object on the person) ·
  `m:<personId>:<idx>:<field>` milestone field (en, he, post, postHe,
  year, tier, hideStory; idx = index in that person's `milestones[]`; `he`/`postHe` are the Hebrew
  title/story shown when the app language is Hebrew; `hideStory:true` hides the
  event from the focus view — bake as a boolean field on the milestone) · `u:<unionId>:<field>` union field
  (year, endYear) · `w:<evId>` world-event on/off (bool; `wevEnabled` is wrapped to consult it) ·
  `w:<evId>:<field>` world-event text field (title, titleHe, short, shortHe, desc, descHe, long,
  longHe — bake into that event's entry in `world-events.js`; empty string → delete the field).
  Values apply LIVE onto `window.FAMILY` objects on load and on each edit. The Records Room UI
  (`records-room.jsx`, editor-mode-only "Records" button in the top bar) is the editor for these.
- **UI**: `publish.jsx` — `SaveChip` in the top bar (dirty → "Save · N changes", copied →
  "N awaiting paste", else "vN · in sync") + `PublishDialog` (op list, undo/redo, per-op discard,
  trash-all, 3-step explainer, "Copy changes (for Claude)" → `FTStore.changeSet()` JSON +
  `markPublished()`). Global ⌘Z/⇧⌘Z → FTStore.undo/redo (app.jsx).
- On load, `tree-store.js` REBASES: if `FT_SEED.seedStamp` > doc's, published ops are dropped
  (they're in the seed now), unpublished ops survive, matching exclusion overrides are pruned.

### Baking a change-set (DO THIS when a `{"type":"ft-change-set",...}` JSON is pasted)
1. Read `.image-slots.state.json` and `.ft-doc.state.json` (run_script `readFile`).
2. Collect the slot ids from the pasted `ops` (kinds `photo-*` and `crop`). For each id:
   - Sidecar value has a data URL `u` → decode the base64 into a file
     `photos/<id>.webp` (or .png/.jpeg per the data URL mime) and set
     `FT_SEED.slots[id] = { src: 'photos/<id>.webp', w, h, s, x, y }` (s/x/y from the sidecar value, default 1/0/0).
     run_script: `const b64 = u.split(',')[1]; const bin = atob(b64); const arr = new Uint8Array(bin.length); ...; await saveFile(path, new Blob([arr], {type: mime}))`.
   - Sidecar value is `{del:1}` (or the op is `photo-remove` with no sidecar value) → DELETE
     `FT_SEED.slots[id]` and the old `photos/<id>.*` file.
   - Framing-only sidecar value (`{s,x,y}`, no `u`) on a seeded slot → merge s/x/y into the seed entry.
3. Exclusion ops (`tree-hide`/`tree-show`): apply the CURRENT override from `.ft-doc.state.json`
   `excluded[key]` to `FT_SEED.excluded` (add or remove the key).
3b. Field-edit ops (`kind:'edit'`): the pasted set carries an `edits` snapshot (`ekey → value`).
   Bake each into the source file:
   - `p:<id>:<path>` / `m:<id>:<idx>:<field>` / `u:<unionId>:<field>` → edit `family-data.js`
     (person fields, that person's `milestones[<idx>]`, or the union entry).
     `p:<id>:parents` (array value) rewrites that person's `parents` list.
   - `w:<evId>` → edit `world-events.js`: value true → remove `off:1` from that event; value
     false → add `off:1`.
   After baking, remove the baked `edits` entries from `.ft-doc.state.json` (`doc.edits`) along
   with their ops.
3c. Structural ops (`person-add`/`person-remove`/`union-add`/`union-remove`/`side-add`/`side-remove`):
   the pasted set carries a `struct` snapshot — `{ people, delPeople, unions, delUnions, sides, delSides }`.
   Bake into `family-data.js`:
   - `struct.people[id]` → insert the person object into the `people` array (place by `gen`,
     formatted like its neighbours). New-person field edits may also appear in `edits` — apply
     them onto the inserted object.
   - `struct.delPeople` → delete that person from `people` AND strip the id from every other
     person's `parents` array (the `edits` snapshot usually carries the updated arrays already).
   - `struct.unions[uid]` → append to `unions`; `struct.delUnions` → remove that union.
   - `struct.sides` → append to `sideLinks`; `struct.delSides` → remove the matching
     `{from,to,kind}` entry.
   Photos for new people ride the normal slot system (`ph-<id>-e<idx>` / `av-<id>-e<idx>`) — bake
   them like any other photo. After baking, clear the baked entries from `doc.struct` in
   `.ft-doc.state.json` along with their ops.
4. `tree-seed.js`: write the updated FT_SEED with `seedStamp: Date.now()` and `baseV: +1`.
5. Clean the working copy:
   - `.image-slots.state.json`: remove every baked id (and `{del:1}` markers) — but KEEP ids that
     still have ops in `.ft-doc.state.json` NOT covered by the pasted change-set (newer edits).
   - `.ft-doc.state.json`: remove the baked ops (match by `ts` against the pasted set), clear the
     `excluded` overrides that were baked, and set `seedStamp`/`baseV` to the new values.
6. No bundling step — the app loads `tree-seed.js` directly. Confirm with a short summary
   ("Baked N changes → v{baseV}").
Idempotent: re-pasting the same change-set finds nothing new to bake (ops already removed).

## User-authored moments (undated-friendly)
Users can add their own life-story moments from the focus view (the hairline
"Add a moment" between posts). A moment can be **undated** — entered relative to
its neighbours and given a year later. Model & flow:
- **Storage**: `.ft-doc.state.json` → `doc.moments[personId] = [{ id, ei, title, text, titleHe?, textHe?, year|null, bucket, seq, place, hideStory? }]`.
  - `ei` is a STABLE integer ≥ 1000, unique per person — used for photo slots
    `ph-<person>-e<ei>` / `av-…-e<ei>`, so moment photos ride the same image-slot
    system as everything else and never collide with structural events (which
    keep `ei` 0..N-1). Moment edits log to `doc.momentLog` (mirrors `doc.ops`;
    both feed `changeSet()` / `markPublished()` / rebase-on-newer-seed).
  - `bucket` = the `key` of the dated event a moment follows (`'e'+ei` for a
    structural event, `'m'+id` for a dated user moment, or `'start'`); `seq` =
    float order among undated siblings in that bucket. `year=null` → undated.
- **`relationships.lifeEvents(id)`** now returns structural events (with stable
  `ei`/`key`/`derived:true`) MERGED with user moments. Dated items sort by year;
  undated items sit right after their `bucket` anchor (ordered by `seq`) and get
  an interpolated `effYear` + a soft `yearLabel` ("between 1934 & 1943"). Every
  consumer (projector, autoAvatar, scrubber, log) keys photos off `e.ei` and
  positions off `e.effYear`, NOT array index.
- **Baking a moment** (when a change-set carries `moment-*` ops): read
  `doc.moments[personId]` and fold each into that person's `milestones[]` in
  `family-data.js` as `{ year, en: title, tier, post: text }`. Undated moments
  have `year:null` — give them the interpolated `effYear` (or ask the user) so
  the static seed can sort them; any moment photo in `.image-slots.state.json`
  (`ph-<person>-e<ei>`) bakes to `photos/` exactly like other photos, keyed to
  the milestone's NEW structural index once inserted. Then clear the baked
  entries from `doc.moments`/`doc.momentLog`.

## App structure
- `Family Tree.html` — script order matters: tree-seed.js → image-slot.js → family-data.js →
  relationships.js → tree-store.js → (babel) tweaks-panel, publish, records-room, records-desk, tree-canvas, focus-view, app.
- `records-room.jsx` — Records Room editor overlay (rail: The desk / People / World events / Places / Photos);
  editor mode only (`window.FTEditorMode`, set via `?edit=1`, persists in localStorage `ft-editor`).
  Exports rrSlotSrc / rrAdultAvatar / RrAvatar to window for records-desk.jsx.
- `records-desk.jsx` — "The desk": Records Room opening screen — animated logo-tree score card
  (weighted coverage score, click to replay), stat tiles, coverage gauges, since-last-save feed,
  spotlight (thinnest record), Do-next issue list (absorbed the old Health screen).
- `tree-canvas.jsx` — tree nodes, FTtree (exclusions facade → FTStore), autoAvatar,
  AvatarCropOverlay (fullscreen crop editor), PhotoMenuSlot (⋯ menu), PhotoIconTools (icon row on projector).
- `focus-view.jsx` — person focus view: LifeProjector (photo stack, wheel/click nav), life story log,
  footer with prev/next event arrows, personal timeline rail.
- `image-slot.js` — modified starter: built-in Replace/Remove chrome disabled (`.ctl` display:none),
  seed fallback in getSlot/has, `{del:1}` markers, FTStore.onSlotChange hook, raw()/restore() API.
- Photos are data URLs in the sidecar until baked; after baking they're real files under `photos/`.

## App versioning & changelog
The APP (not the family data) is versioned in CHANGELOG.md and the "App version" line
near the top of README.md / README.he.md. Whenever a feature, UI change, or fix ships:
bump the version (semver-ish: features = minor, fixes = patch), add a CHANGELOG.md entry
dated that day, and keep the README version line in sync. Data bakes (tree-seed.js baseV)
are a SEPARATE counter and never touch CHANGELOG.md.

---
> Source: [aviranrevach/my-family-story](https://github.com/aviranrevach/my-family-story) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
