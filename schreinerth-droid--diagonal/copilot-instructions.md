## diagonal

> A single-file football statistics dashboard for **Diagonal Club Esp. A**, an infantil S14 team competing in **Infantil Primera Divisió S14, Grup 9** (Federació Catalana de Futbol, season 2025–26), based in Barcelona.

# Diagonal Club Esp. A — Football Dashboard

## Project overview

A single-file football statistics dashboard for **Diagonal Club Esp. A**, an infantil S14 team competing in **Infantil Primera Divisió S14, Grup 9** (Federació Catalana de Futbol, season 2025–26), based in Barcelona.

The dashboard is a self-contained HTML file (`v2.html`) — all data, styles, and logic live in one file with no build step or dependencies.

## Files

| File | Purpose |
|------|---------|
| `v2.html` | The dashboard — open directly in a browser |
| `update.js` | Node.js script that fetches published actas from fcf.cat and patches `v2.html` |
| `index.html` | Older version (kept for reference) |

## v2.html structure

The file is divided into three sections:

1. **CSS** — inline styles using CSS custom properties (`:root` vars). Design uses a warm neutral palette with accent blue (`--accent:#1a3a6b`).
2. **HTML** — static shell with tabs: Resumen · Partidos · Jugadores · Clasificación · Próximos · Árbitros
3. **`<script>` block** — all data and rendering logic. Key data arrays:

### Data arrays (inside `<script>`)

**`PLAYERS`** — object keyed by short playerId (e.g. `redon`, `gabriel`):
```js
var PLAYERS = {
  gabriel: {name:'Schreiner Padilla, Gabriel Mathias', activeDorsal:15, allDorsals:[15]},
  ...
};
```
- `activeDorsal`: current shirt number
- `allDorsals`: all dorsals ever worn (used for matching in actas)

**`MATCHES`** — array of played match objects, ordered by `j` (jornada):
```js
{j:21, date:'2026-03-10', loc:'C', home:'Diagonal', away:'Sant Ignasi CE A',
 gf:3, gc:0, res:'G',
 ref:'...', refDel:'...',
 titulars:[{id:'redon',d:22}, ...],
 suplents:[{id:'jorge',d:11}, ...],
 goals:[{min:13, type:'gf', playerId:'yannick'}, ...],
 yellows:[{playerId:'jordi',d:5}], reds:[]}
```
- `loc`: `'C'` = casa (home), `'F'` = fuera (away)
- `res`: `'G'` = win, `'E'` = draw, `'P'` = loss (from Diagonal's perspective)
- `type` in goals: `'gf'` = goal scored by Diagonal, `'gc'` = goal conceded

**`CLAS`** — static league table snapshot (updated manually after each jornada):
```js
var CLAS = { updated:'J21 · 12 mar 2026', teams:[...] };
```
Must be updated manually — fcf.cat does not expose a machine-readable standings endpoint.

**`UPCOMING`** — array of future fixtures (used by the Próximos tab):
```js
{j:22, date:'2026-03-22', home:false, rival:'Escola F Angel Pedraza A'}
```
`home: true` = playing at home. This array is **not** part of `MATCHES` — it only drives the upcoming fixtures display.

## update.js

Fetches actas from fcf.cat and auto-patches `MATCHES` in `v2.html` when results are published.

### Usage
```bash
node update.js            # patches v2.html if new actas are found
node update.js --dry-run  # shows what would be added, writes nothing
node update.js --debug    # saves downloaded HTML to ./debug/ for inspection
```

### How it works
1. Reads `v2.html`, extracts `PLAYERS` and the set of jornadas already in `MATCHES`
2. Checks each URL in `UPCOMING_URLS` — skips jornadas already loaded
3. For pending jornadas: fetches the acta page, checks if the scoreline is published (not just a time)
4. If published: parses teams, score, date, players, goals, cards
5. In dry-run mode: prints what would be added. Otherwise: inserts new match objects before the closing `]` of `MATCHES`

### Key implementation detail — `readV2()` jornada detection
The regex that detects already-loaded jornadas must be scoped **only to the `MATCHES` array**, not the full file. The `UPCOMING` array also uses `{j:N,` entries (J22–J29), which would otherwise be falsely reported as already loaded. The fix: bracket-count from `var MATCHES = [` to extract only that slice before running the regex.

### UPCOMING_URLS in update.js
Hard-coded list of acta URLs for J22–J29 (season 2025–26). URL pattern:
```
https://www.fcf.cat/acta/2526/futbol-11/infantil-primera-divisio-s14/grup-9/1i14/<home-slug>/1i14/<away-slug>
```
Diagonal's slug: `diagonal-club-esp-a`

## Adding a match manually (acta not published or parse failed)

If `update.js` reports "sin publicar todavía" or fails to parse, the user will paste the raw acta text (copied from the fcf.cat page) directly into the chat. When that happens:

1. **Extract these fields from the acta text:**
   - `j` — jornada number
   - `date` — format `YYYY-MM-DD` (acta shows DD-MM-YYYY)
   - `loc` — `'C'` if Diagonal is listed as local (home), `'F'` if visitant (away)
   - `home` / `away` — team names exactly as shown in the acta header
   - `gf` / `gc` — goals from Diagonal's perspective (gf = scored, gc = conceded)
   - `res` — `'G'` (win), `'E'` (draw), or `'P'` (loss) derived from gf/gc
   - `ref` / `refDel` — referee name and delegation (from the Àrbitres table)
   - `titulars` / `suplents` — Diagonal's player rows only (dorsal + name)
   - `goals` — all goals with minute, scorer name, and which team scored
   - `yellows` / `reds` — cards for Diagonal players only

2. **Resolve playerIds** using the `PLAYERS` dictionary in `v2.html`:
   - Match by normalized full name first (accents/case-insensitive)
   - Fall back to dorsal number if name match fails
   - If a player cannot be matched, use `null` for `playerId` and flag it to the user
   - Never invent a new playerId — only use keys already in `PLAYERS`

3. **Insert the new entry into `MATCHES`** in `v2.html`, following the exact serialization format of existing entries:
   ```js
   {j:22,date:'2026-03-22',loc:'F',home:'Escola F Angel Pedraza A',away:'Diagonal',gf:2,gc:1,res:'G',
    ref:'Apellido Apellido, Nombre',refDel:'Barcelona1',
    titulars:[{id:'redon',d:22},{id:'gabriel',d:15},...],
    suplents:[{id:'jorge',d:11},...],
    goals:[{min:34,type:'gf',playerId:'gabriel'},{min:67,type:'gc',playerId:null}],
    yellows:[{playerId:'luca',d:19}],reds:[]}
   ```
   - Insert before the closing `];` of the `MATCHES` array, preceded by a comma and blank line
   - Do not reformat or touch any existing entries

4. **After inserting**, remind the user to also update `CLAS` and (if needed) `UPCOMING`.

## After each match: what to update

1. **Run `node update.js`** — auto-adds the new match to `MATCHES` if the acta is published on fcf.cat (usually same day or next day).
2. **Update `CLAS`** manually — copy the standings from fcf.cat and update the `CLAS` object including the `updated` label.
3. **Update `UPCOMING`** — remove the played jornada from the array (or leave it; the dashboard filters by date).

## Season context (2025–26)

- Competition: Infantil Primera Divisió S14, Grup 9, FCF
- J1–J21: complete (data in `MATCHES`)
- J22–J29: pending (actas not yet published as of 2026-03-20)
- J22 is on 2026-03-22 (away vs Escola F Angel Pedraza A)
- Current standing after J21: 8th place, 32 pts (10W 2D 9L, 26 GF 26 GA)
- 16 teams in the group

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/schreinerth-droid) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
