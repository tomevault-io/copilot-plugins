## american-indian-allotment

> Flask web app for researching Indian allotment land dispossession. Replaces a legacy PHP search at land-sales.iath.virginia.edu. Built at IATH, University of Virginia.

# Federal Register Forced Fee App — Development Guide

## What This Is
Flask web app for researching Indian allotment land dispossession. Replaces a legacy PHP search at land-sales.iath.virginia.edu. Built at IATH, University of Virginia.

## Running
```bash
cd federal-register-app
source venv/bin/activate
python3 app.py  # runs on http://localhost:5001
```

## Stack
- **Backend:** Flask + psycopg2, Python 3
- **Database:** PostgreSQL `allotment_research` (local, user=cwm6W; Cloud SQL in production)
- **Frontend:** Bootstrap 5, jQuery, DataTables (server-side), Chart.js
- **Map:** Leaflet.js, leaflet.heat, Esri ArcGIS Feature Service (standalone SPA at `/map`)
- **Deployment:** Google Cloud Run, **manual** (there is NO Cloud Build trigger — pushing to
  main does NOT auto-deploy, despite older notes). To deploy: build from a CLEAN context (the
  working tree has uncommitted junk) and update the service. Gotchas verified 2026-06-24:
  (1) `gcloud builds submit` falls back to `.gitignore` without a `.gcloudignore`, and `.gitignore`
  has `data/` — which silently drops `static/data/us-states.geojson` from the image; add a
  permissive `.gcloudignore`. (2) The Cloud SQL data and the code deploy are independent — push
  table changes via `scripts/push_new_tables_to_cloudsql.sh` (Auth Proxy on a non-5432 port),
  deploy code separately. Build+deploy used:
  `git archive HEAD | tar -x -C /tmp/ctx; printf '.git/\n' > /tmp/ctx/.gcloudignore;
   gcloud builds submit /tmp/ctx --tag us-east1-docker.pkg.dev/$PID/cloud-run-source-deploy/federal-register-app:TAG;
   gcloud run deploy federal-register-app --region us-east1 --image …:TAG` (image-only deploy
  retains the existing DATABASE_URL env var and Cloud SQL connection).
- **Virtualenv:** `./venv/`

## Architecture
Single-file Flask app (`app.py`) with Jinja2 templates. No ORM — raw SQL with psycopg2. All tables use server-side DataTables pagination via JSON API endpoints.

### Two parallel sections

**Claims section** (original) — 35,686 Federal Register claims from two 1983 publications ([March 31](https://land-sales.iath.virginia.edu/documents/federal_register/fedreg-1983_03_31.pdf) and [November 7](https://land-sales.iath.virginia.edu/documents/federal_register/fedreg-1983_11_07.pdf)):
- `/` — Claims search (DataTables + filters). Columns: BIA Agency Code, Case #, Allottee Name, Tribe, Allotment #, Claim Type, Patent Date, Map (yes/no badge). Default sort: agency code, then case number.
- `/claim/<id>` — Claim detail with linked patents. Document Source links to original FR PDF.
- `/api/search` — JSON API for claims DataTables
- `/api/search/csv` — CSV download

**Patents section** (added March 2026) — 285,870 BLM allotment patents:
- `/patents` — Patent search (DataTables + filters for name/tribe/state/type/date)
- `/patent/<objectid>` — Patent detail with PLSS land description (+ "Patent Cancelled" block for records in cancelled_patent_research)
- `/api/patents` — JSON API for patents DataTables (filters incl. `cancelled=yes`)
- `/api/patents/csv` — CSV download
- `/patents/cancelled` — Browse the cancelled-patents research dataset (cancelled_patent_research, 439 records); summary by cancellation authority + full list. Filter constant: `CANCELLED_RESEARCH_WHERE_SQL`
- `/patents/timeline` — Stacked bar chart (fee vs trust, forced fee toggle)
- `/api/patents/timeline` — JSON API for timeline

**Map (integrated Leaflet SPA):**
- `/map` — Interactive allotment patent map (Leaflet + Esri Feature Service, standalone template)

**Other pages:**
- `/tribes` — Tribe list with claim counts and BIA agency codes
- `/tribe/<slug>` — Individual tribe page with timeline, agency codes in header
- `/timeline` — Forced fee claims timeline (original)
- `/about` — About page (includes BIA agency code reference table, FR PDF links)
- `/research` — Research overview (home.html) with dataset cards and FR PDF links

### Cross-links
- Claim detail → BLM patent: "View full BLM record" link (via accession_number lookup in blm_allotment_patents)
- Patent detail → Claim: alert banner linking to Federal Register claim (via forced_fee_patents_rails)

## Key Database Tables
See `DATABASE.md` for full schemas.

- `federal_register_claims` (35,686 rows) — FR claims (all types). PK: id
- `forced_fee_patents_rails` (17,560 rows) — hand-verified claim-to-patent linkages from Rails admin
- `rails_patents` (285,870 rows) — full patent catalog with `has_plss_geometry` flag. PK: id
- `blm_allotment_patents` (239,845 rows) — BLM patent mirror from ArcGIS (mappable patents only). PK: objectid
- `all_patents` (view, 285,870 rows) — unified view joining rails_patents + blm_allotment_patents
- `fee_patents` (88,537) / `trust_patents` (95,353) — older BLM patent tables (still used for claim detail fallback)
- `trust_fee_linkages` (29,229) — trust→fee conversion records
- `parcels_patents_by_tribe` (401,811) — PLSS legal descriptions

### Claims → Patents join
```sql
LTRIM(fr.case_number, '0') = LTRIM(ffp.case_number, '0')
AND fr.allottee_name = ffp.fedreg_allottee
```

### Patent authority categories (defined in app.py)
- **FEE_AUTHORITIES:** Indian Fee Patent (and variants), Indian Homestead Fee Patent, Indian Trust to Fee
- **TRUST_AUTHORITIES:** Indian Trust Patent (and variants), Indian Allotment - General, Indian Partition, etc.
- **Forced fee:** See rule below — always use Federal Register claims, never BLM flag.

## RULE: Forced Fee Numbers Must Come From the Federal Register
The sole authoritative source for forced fee counts is the `federal_register_claims` table (35,686 total claims; forced fee claims are those WHERE `claim_type ILIKE '%FORCED FEE%'` — 9,649 rows). NEVER use the BLM `forced_fee` flag (`WHERE forced_fee = 'True'` — 15,045 rows) to count or label forced fee patents.

Why the BLM flag is inflated: a forced-fee event is a single historical act (trust patent → fee patent for the same allotment), but the BLM published layer's flag was generated by an upstream one-to-many join from FR claims to BLM patent records and flagged BOTH sides of the conversion. Verified on current data: of 5,728 distinct trust-authority allotments flagged `forced_fee=True`, **4,198 (73%) are paired with a fee-authority record at the same allotment+state+tribe also flagged true**; the symmetric figure for fee-authority is 4,198/5,280 (80%). So most flagged events show up twice.

Aggregate ratio is **15,045 / 9,649 ≈ 1.56×**, but per-tribe inflation is more lopsided where the join hit cleanly. Example — Blackfeet: 821 FR forced-fee claims (BIA agency C51201, `tribe_identified='Blackfeet'`) vs 2,886 BLM-flagged rows (1,667 fee-authority + 1,208 trust-authority) — **3.5× inflation**. The BLM-flag number is wrong as a count of forced-fee events.

**This rule applies to the map too.** The Leaflet SPA at `/map` queries BLM's Esri Feature Service directly (`tribal_land_patents_aliquot_20240304/FeatureServer/0/`), and its `forced_fee` field carries the same inflated flag — the upstream join was done before the Esri layer was published, so reading from Esri is not a workaround. The map's red parcels, "Forced fee" legend entry, and analysis-panel "forced fee rate" stats are currently derived from the BLM flag and should be treated as overcounts pending a refactor to filter by FR-claim accession numbers instead.

When showing forced fee data:
- Count from `federal_register_claims` WHERE `claim_type ILIKE '%FORCED FEE%'`
- Label as "FR forced fee claims" (these are CLAIMS — a subset of all trust-to-fee conversions)
- Never say "forced fee patents" based on the BLM flag
- Never conflate FR claims with the total number of trust-to-fee conversions

## RULE: Patent-side "Forced Fee & Related" is a wider FR bucket
The patents page (`/patents`, `/patent/<id>`, `/api/patents`, `/api/patents/csv`) flags patents linked to any of seven thematically-related FR `claim_type` buckets, not just `%FORCED FEE%`. The seven patterns live in `DISPOSSESSION_CLAIM_PATTERNS` in `app.py` and are reused by the patents filter, the JSON `is_dispossession_claim` flag, the patent-detail banner, and the CSV export:

```
%FORCED FEE%             (forced fee patent — 9,649)
%SECRETARIAL TRANSFER%   (1,327)
%UNAPPROVED%             (unapproved land sale — 980)
%WITHOUT APPROVAL%       (land sold without approval — 264; covered by UNAPPROVED-ILIKE but kept for clarity)
%TAX FORFEITURE%         (688)
%TAXATION%               (1,057)
%RECOVERY%               (claim for recovery of trust/restricted land — 935)
```

Per the 2026-06-02 decision: these buckets share the same historical phenomenon (loss of trust title), even when the FR's 1983 enumeration filed them under different administrative labels. The Ponca Agency `B07813` filed zero forced-fee claims but fourteen recovery-of-trust claims — under the narrow filter, Ponca dispossession is invisible. Trespass, welfare, timber, old-age-assistance, allotment-never-issued, and questionable-cancellation claims are NOT in this set — they don't represent loss of trust title.

When adding a new patents-page feature that filters or labels patents by FR linkage:
- Use `DISPOSSESSION_CLAIM_PATTERNS` / `DISPOSSESSION_WHERE_SQL`, never re-derive the list
- The JSON field is `dispossession_claim` (boolean); the template flag is `is_dispossession_claim`
- The badge/column label is "Forced Fee & Related" (table column header) or "Forced Fee & Related Claims" (dropdown / lead paragraph)
- The patent-detail page renders ALL linked claims; dispossession ones get the warning-style banner and a "Dispossession" badge inline; non-dispossession ones get the info-style banner with the actual claim type shown

## Templates
Most extend `base.html`. Navigation: Claims | Patents | Map | Tribes | Visualizations (dropdown) | About | Main Site.
Exception: `map.html` is standalone (does not extend `base.html`) — it has its own thin nav bar and full-viewport layout for the Leaflet map SPA. Map assets live in `static/map/js/` and `static/map/css/`.

## Patterns to Follow
- Server-side DataTables: route returns page HTML, `/api/` route returns JSON with draw/recordsTotal/recordsFiltered/data
- URL filter persistence: read from URL params on page load, update URL on search
- CSV export: same filters as search, streamed via `io.StringIO`
- Tribe slugs: `slugify()` / `unslugify_tribe()` in app.py
- GLO links: `glo_url(accession, doc_class)` builds glorecords.blm.gov URLs
- Allotment map: integrated at `/map` route, cross-linked with `url_for('allotment_map', tribe=..., accession=...)`

## Environment
- Main site: https://land-sales.iath.virginia.edu/

---
> Source: [cwmmwc/american-indian-allotment](https://github.com/cwmmwc/american-indian-allotment) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
