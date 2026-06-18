## teams-united-league-service

> > Coverage docs at `coverage/{STATE}/{sport}.md` are the living per-combo truth — see [Coverage System](#coverage-system) section.

# Teams United League Service — CLAUDE.md

> Coverage docs at `coverage/{STATE}/{sport}.md` are the living per-combo truth — see [Coverage System](#coverage-system) section.

## Overview

Multi-platform youth sports standings collection service for [Teams United](https://teams-united.com).
Collects league standings from 9 different scoring platforms, stores them in Firestore, and serves
them via Cloud Functions API + GCS-hosted dashboard.

**GCP Project:** `teams-united` (us-central1)
**Deploy VM:** `35.209.45.82:8080` — see [Deploy VM Access](#deploy-vm-access) below
**Firestore:** 3 main collections: `leagues`, `divisions`, `standings`
**Dashboard:** GCS-hosted HTML at `https://storage.googleapis.com/tu-league-dashboard/`

## Architecture

```
Cloud Scheduler (daily) → collectAll → iterates active leagues → adapter.collectStandings()
                                                                      ↓
User/API → collectLeague(leagueId) ─────────────────────────→ adapter.collectStandings()
                                                                      ↓
                                                              Firestore (divisions, standings)
                                                                      ↓
                                                              updateSheet → Google Sheets
                                                                      ↓
Cloud Scheduler (weekly) → seasonMonitor → health checks, season discovery, auto-dormant
```

### Cloud Functions (Gen2, Node 20)

| Function | Trigger | Purpose | Memory |
|---|---|---|---|
| `collectLeague` | HTTP POST `{leagueId}` | Collect one league | 1024MB* |
| `collectAll` | Cloud Scheduler (daily) | Collect all active leagues | 1024MB* |
| `getLeagues` | HTTP GET | List leagues with filters | 256MB |
| `getDivisions` | HTTP GET `?league=` | List divisions for a league | 256MB |
| `getStandings` | HTTP GET `?division=` | Get standings for a division | 256MB |
| `seasonMonitor` | Cloud Scheduler (weekly) | Detect stale/dormant leagues, discover new seasons | 256MB |
| `updateSheet` | POST (after collectAll) | Sync Firestore → Google Sheets | 256MB |
| `discoverGC` | HTTP POST | Discover GameChanger leagues via DuckDuckGo + API | 256MB |
| `discoverGroups` | HTTP POST | Discover GotSport division groups | 256MB |

\* collectLeague/collectAll need 1024MB for Puppeteer-based adapters (SC, GC). Deploy script: `scripts/deploy-memory-upgrade.sh`

### 9 Platform Adapters (`adapters/`)

| Adapter | Method | Sports | Config Keys |
|---|---|---|---|
| **gamechanger** | Browser (Puppeteer) | Baseball, Softball | `orgId`, `allOrgIds` |
| **sportsconnect** | Browser (Puppeteer) | Baseball (Little League, PONY) | `baseUrl`, `standingsTabId`, `programs[]` |
| **sportsaffinity** | JSON API | Soccer | `organizationId`, `seasonGuid` |
| **sportsaffinity-asp** | Browser (Puppeteer) | Soccer | `organizationId` (GUID), auto-discovers flights |
| **gotsport** | HTML scraping | Soccer | `leagueEventId`, `groups[]` |
| **tgs** | Browser/API | Soccer (ECNL, GA) | `eventId` |
| **demosphere** | HTML scraping | Soccer | `baseUrl`, `divisions[]` |
| **pointstreak** | HTML scraping | Baseball, Hockey | `leagueId`, `seasonId` |
| **leagueapps** | HTML scraping | Baseball, Soccer, Basketball, Lacrosse | `baseUrl`, `programs[]` |

### Key Files

- `index.js` — Cloud Function entry point (collectLeague, collectAll, getLeagues, getDivisions, getStandings)
- `registry.js` — Adapter registry
- `browser.js` — Shared Puppeteer launcher (v2 with frame-detached resilience)
- `season-monitor.js` — Weekly health checks + auto season discovery
- `sheets-sync.js` — Google Sheets sync
- `discover-gc.js` — GameChanger org discovery via DuckDuckGo search
- `discover-groups.js` — GotSport group auto-discovery
- `lib/age-group-parser.js` — Universal age group normalization (U4-U19, HS, Adult, etc.)
- `dashboard/` — Firebase-hosted ops dashboard

### Firestore Schema

**leagues** collection:
```
{
  name, sport, state, region,
  sourcePlatform, sourceConfig: { ... platform-specific ... },
  status: 'active' | 'dormant' | 'pending_config' | 'pending_tabid' | 'pending_groups' | 'deactivated_phase1' | 'template',
  autoUpdate, lastCollected, lastDataChange, lastStandingsHash,
  monitorStatus: 'healthy' | 'stale' | 'dormant' | 'error' | 'needs_attention',
  monitorNotes, lastMonitorCheck,
  seasonStart, seasonEnd, staleDays, discoveryConfig
}
```

**divisions** collection:
```
{ id, leagueId, seasonId, name, ageGroup, gender, level, platformDivisionId, status }
```

**standings** collection (keyed by `{divisionId}-{slugified-teamName}`):
```
{ teamName, position, gamesPlayed, wins, losses, ties, points, scored, allowed, differential, ... }
```

## Current State (April 27, 2026)

> **For real-time numbers, run a fresh four-bucket audit** via the `teamsunited-league-service-ops` Hermes skill — these are point-in-time snapshots that drift weekly.

### Stats (fresh as of 2026-04-27)
- ~5,800 leagues truly collecting (>0 divisions, recent `collectedAt`)
- ~17,354 `pending_adapter` (mostly SBL-universal placeholders awaiting the ScoreBookLive Firecrawl adapter)
- ~3,848 active rows with no platform / awaiting research
- 20 platform adapters live: sportsaffinity, sportsaffinity-asp, gotsport, pointstreak, demosphere, tgs, gamechanger, leagueapps, sportsconnect, blue-sombrero, teamlinkt, sportsengine, sportngin, sporngin, leaguelineup, teamsideline, myhockey, crossbar, teamsnap, **wa-conference**
- WA HS coverage: 504 leagues collecting across 38 of 41 WIAA conferences (V/JV/C splits) via the wa-conference adapter shipped 2026-04-27.

### Daily refresh path (canonical as of 2026-04-27)
- **GCP Cloud Scheduler `league-standings-daily` was DECOMMISSIONED 2026-04-27.** The old `collectAll` Cloud Run target was structurally fragile past ~5K leagues (DEADLINE_EXCEEDED).
- Canonical replacement: Hermes cron `fc9f02a31801` ("TU League Standings Nightly Refresh") at 5:00 AM PT on PC's Mac, fanning out `collectLeague` calls in parallel. Three-tier monitoring (collect 5:00 AM → freshness watchdog 8:00 AM → data-sanity check 8:15 AM).
- All other GCP Cloud Schedulers (tournament-sweeper, freshness-sweeper, reclassify-pending, publish-summaries, batch-activate-urls, season-monitor, etc.) remain live and unchanged.

### Phase 1 Rollout States
**WA** (Washington) — primary, most coverage. Baseball ~70 active, Soccer 8 active + 13 pending, Softball 4 active
**CA** (California) — good GameChanger coverage, 4 soccer active (NorCal Premier 709 groups, SOCAL 526 groups)
**OR** (Oregon) — soccer resolved: 5 active, 2 dormant, 3 deactivated
**ID** (Idaho) — 4 soccer leagues active (ISL, D3L, SRL, IPL)
**MT** (Montana) — 1 soccer active (MSSL Spring), 1 dormant (MSSL Fall)

### OR Soccer (completed March 12)
- 5 activated: OYSA Spring Competitive, OYSA Spring South, OYSA Dev League, OYSA Valley Academy, PMSL
- 2 dormant: OYSA Winter (season ended), USYS NW Conference (2025-26 not created)
- 3 deactivated: ALBION SC Portland, GPSD (adult), Oregon Soccer Club (stale)

### CA/ID/MT Soccer Expansion (completed March 12)
- 8 new leagues registered, 2 existing updated
- 7 leagues activated with discovered groups (246 total groups)
- 2 CA leagues resolved: SOCAL Soccer League (526 groups), NorCal Premier (709 groups) — activated March 13

### Softball (March 13 — discovery complete)
- Discovery ran across all Phase 1 states (WA, OR, ID, MT, CA)
- **WA: 7 found** — all via SportsConnect (existing LL orgs with softball programs)
  - 4 registered with tabIds (active): Edmonds (1004892), Federal Way (1263355), Ballard (1218908), NW Seattle (2519652)
  - 3 more have softball content but no discoverable tabId: Spokane YSA, Pacwest LL, Bellevue National LL
- **OR/ID/MT/CA: 0 found** — no GC softball orgs, no GotSport/LeagueApps softball
- No GC softball orgs found anywhere — softball leagues likely use ASA/USSSA/NSA association websites

### Basketball (March 13 — discovery complete)
- Discovery ran across all Phase 1 states — **0 results**
- No GC basketball orgs, no LeagueApps/SportsEngine/GotSport matches via DuckDuckGo
- Youth basketball likely lives on platforms we don't yet support (SportsEngine, local rec sites)
- CA had rate-limiting (403) errors from DuckDuckGo

### Recent Additions
- **Pending resolution** (March 13): resolve-all-pending.js activated 2 CA soccer leagues (NorCal 709 groups, SOCAL 526 groups)
- **Softball discovery** (March 13): 4 WA softball leagues registered via SportsConnect
- **Basketball discovery** (March 13): ran across 5 states, 0 leagues found on supported platforms
- OR soccer resolution: all 9 pending_config leagues resolved
- CA/ID/MT soccer expansion: 8 new GotSport leagues + 2 updates
- **OR Soccer Expansion**: 6 OYSA/PMSL leagues registered via `sportsaffinity-asp` adapter
  - OYSA uses legacy ASP system (`oysa.sportsaffinity.com`), NOT the SCTour JSON API
  - New adapter: `adapters/sportsaffinity-asp.js` — Puppeteer-based, auto-discovers flight GUIDs
  - New lib: `lib/age-group-parser.js` — universal age group normalization
  - OYSA organizationId: `e458918e`
  - PMSL organizationId: `6857D9A0-8945-44E1-84E8-F3DECC87D56C`
- Fall City LL registered (SportsConnect, already active)
- WA Soccer Expansion: 19 new leagues registered across 3 tiers
- 14 WA Little League programs discovered (SportsConnect): 1 GC-matched, 13 pending tabId resolution
- Google Sheets sync RETIRED — dashboard is now GCS-hosted HTML

### SportsAffinity Platform Notes
SportsAffinity has TWO distinct systems:
- **SCTour JSON API** (`sctour.sportsaffinity.com`) — Used by WA leagues (RCL, SSUL). Currently DOWN (Azure 404). Adapter: `adapters/sportsaffinity.js`
- **Legacy ASP system** (`oysa.sportsaffinity.com`, etc.) — Used by OYSA/OR leagues. Working. Adapter: `adapters/sportsaffinity-asp.js` (Puppeteer-based)

## Scripts (`scripts/`)

Scripts are organized into subdirectories by purpose:

### `scripts/discovery/` — League & group discovery
| Script | Purpose | Usage |
|---|---|---|
| `discover-or-id-mt.js` | Discover GC + known leagues in OR/ID/MT | `node scripts/discovery/discover-or-id-mt.js [--dry-run] [--state=OR]` |
| `discover-and-activate-gotsport.js` | Discover GotSport groups & activate leagues | `node scripts/discovery/discover-and-activate-gotsport.js [--dry-run]` |
| `resolve-sportsconnect-pending.js` | Auto-discover SC standings tabIds | `node scripts/discovery/resolve-sportsconnect-pending.js [--dry-run] [--fix]` |
| `resolve-all-pending.js` | Resolve ALL pending leagues (tabid, config, groups, platform, adapter) | `node scripts/discovery/resolve-all-pending.js [--dry-run] [--fix] [--category=X]` |
| `discover-softball.js` | Discover softball leagues across Phase 1 states | `node scripts/discovery/discover-softball.js [--save] [--json] [--state=WA]` |
| `discover-basketball.js` | Discover basketball leagues across Phase 1 states | `node scripts/discovery/discover-basketball.js [--save] [--json] [--state=WA]` |
| `discovered_groups_new.json` | Pre-discovered GotSport group data | Data file for update-groups.js |

### `scripts/activation/` — League status changes
| Script | Purpose | Usage |
|---|---|---|
| `activate-or-soccer.js` | Activate/dormant/deactivate OR soccer leagues | `node scripts/activation/activate-or-soccer.js [--dry-run]` |
| `deactivate-non-phase1.js` | Deactivate leagues outside WA/OR/ID/MT/CA | `node scripts/activation/deactivate-non-phase1.js [--dry-run]` |

### `scripts/maintenance/` — Data fixes & cleanup
| Script | Purpose | Usage |
|---|---|---|
| `fix-soccer-casing.js` | Normalize sport casing ("Soccer" → "soccer") | `node scripts/maintenance/fix-soccer-casing.js [--dry-run]` |
| `fix-soccer-discovery-config.js` | Add discoveryConfig to GotSport leagues | `node scripts/maintenance/fix-soccer-discovery-config.js [--dry-run]` |
| `fix-soccer-season-dates.js` | Add missing seasonStart/seasonEnd | `node scripts/maintenance/fix-soccer-season-dates.js [--dry-run]` |
| `fix-regions.js` | Fix/add region fields | `node scripts/maintenance/fix-regions.js [--dry-run]` |

### `scripts/setup/` — League registration
| Script | Purpose | Usage |
|---|---|---|
| `wa-soccer-expansion.js` | Register 21 WA soccer leagues (3 tiers) | `node scripts/setup/wa-soccer-expansion.js [--dry-run] [--tier=1]` |
| `or-soccer-expansion.js` | Register 7 OR soccer leagues | `node scripts/setup/or-soccer-expansion.js [--dry-run]` |
| `expand-soccer-ca-id-mt.js` | Register CA/ID/MT soccer leagues | `node scripts/setup/expand-soccer-ca-id-mt.js [--dry-run]` |
| `setup-fall-city-ll.js` | Register Fall City LL (SportsConnect) | `node scripts/setup/setup-fall-city-ll.js [--dry-run]` |

### Root scripts
| Script | Purpose | Usage |
|---|---|---|
| `deploy-memory-upgrade.sh` | Deploy collectLeague/collectAll with 1024MB | `bash scripts/deploy-memory-upgrade.sh` |

## Config (`config/`)

League configuration data organized by state and sport:

```
config/
├── national/              # (future) cross-state config
└── states/
    ├── WA/soccer/leagues.json   # 21 leagues across 3 tiers
    ├── OR/soccer/leagues.json   # 7 leagues (OYSA, PMSL, USYS NW)
    ├── CA/soccer/leagues.json   # 4 leagues (SOCAL, CCSL, NorCal)
    ├── ID/soccer/leagues.json   # 4 leagues (ISL, D3L, SRL, IPL)
    └── MT/soccer/leagues.json   # 2 leagues (MSSL spring/fall)
```

Run scripts on the deploy VM via:
```bash
curl -X POST http://35.209.45.82:8080/exec \
  -H 'Authorization: Bearer $TU_DEPLOY_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{"command":"cd /home/deploy/workspace/league-standings && node scripts/setup/expand-soccer-ca-id-mt.js --dry-run"}'
```

## Coverage System

`coverage/{STATE}/{sport}.md` is the living state-of-the-union doc for every state × sport combo we support. It is seeded from Claude Research output and maintained by every agent/script that changes the league or tournament set. The coverage doc is the archive of research-only context (Type, Level, Registration URL, Age Range, Est. # Teams, Season(s), Sanctioning Body, Contact, Confidence, Notes, tournament Cadence + Host Venues) — none of which live in Firestore or in `config/states/...`.

- Canonical template: `coverage/_template.md`
- Overview + lifecycle: `coverage/README.md`
- Seeded example: `coverage/ID/softball.md` (from synthetic fixture)

### Coverage Maintenance — REQUIRED

**Any script, agent, or manual change that creates, activates, dormants, deactivates, renames, or re-configures a league OR tournament for state X sport Y MUST, before exiting or completing its task, run:**

```
node scripts/coverage/regen-coverage.js --state X --sport Y --source <agent-or-script-name>
```

**and include the updated `coverage/X/Y.md` in the same commit that changes the config. Tier-2, Dev, Eng, Discovery, and Tournament agents are all bound by this rule. If the script run fails, STOP and flag it — do not commit partial changes.**

### Ingest flow

New Claude Research markdown (per state or per state × sport) enters the system via `scripts/coverage/ingest-research.js`. It parses the YAML frontmatter + markdown tables, merges leagues and tournaments into `config/states/{STATE}/{sport}/leagues.json` + `tournaments.json` (matching by slug or website domain; existing fields win; never drops existing leagues), and writes `coverage/{STATE}/{sport}.md` from the template.

Example:

```
node scripts/coverage/ingest-research.js \
  Machine/Outputs/data/coverage-research/ID-softball.md
# or with overrides:
node scripts/coverage/ingest-research.js path/to/research.md --state ID --sport softball --dry-run
```

Re-ingesting the same file is a no-op (idempotent).

### When to regen

Run `regen-coverage.js` with an appropriate `--source` label:

- After any league `status` change (activate / dormant / deactivate / pending).
- After any tournament import or registration.
- After any discovery script run that changes the active set for a combo.
- After a rename, platform swap, or source-config (eventId/tabId/orgId) change.
- As part of `season-monitor`'s weekly pass — **hook PENDING** (not yet wired; will land in a follow-up commit after the in-flight agent work on `season-monitor.js` merges).

The script is safe to run with `--all` to refresh every existing coverage doc from current git config in one pass.

## Deploy VM Access

The deploy VM at `35.209.45.82:8080` is your primary tool for running scripts, deploying Cloud Functions, and managing the service.

### Authentication

All requests require a Bearer token via the `$TU_DEPLOY_TOKEN` environment variable (set locally on your Mac, NOT stored in this repo).

```bash
export TU_DEPLOY_TOKEN="<your-token-here>"  # Set in your shell profile
```

### Endpoints

| Endpoint | Method | Purpose |
|---|---|---|
| `/exec` | POST | Execute any shell command on the VM |
| `/upload` | POST | Upload a file to the VM |
| `/download` | GET | Download a file from the VM |

### Usage Patterns

**Run a script:**
```bash
curl -s -X POST http://35.209.45.82:8080/exec \
  -H "Authorization: Bearer $TU_DEPLOY_TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{"cmd":"cd /home/deploy/workspace/league-standings && node scripts/your-script.js --dry-run"}'
```

**Deploy code to VM (pull latest from GitHub):**
```bash
curl -s -X POST http://35.209.45.82:8080/exec \
  -H "Authorization: Bearer $TU_DEPLOY_TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{"cmd":"cd /home/deploy/workspace/league-standings && git pull origin main && npm install"}'
```

**Deploy a Cloud Function:**
```bash
curl -s -X POST http://35.209.45.82:8080/exec \
  -H "Authorization: Bearer $TU_DEPLOY_TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{"cmd":"cd /home/deploy/workspace/league-standings && gcloud functions deploy collectLeague --gen2 --runtime=nodejs20 --region=us-central1 --trigger-http --allow-unauthenticated --memory=1024MB --timeout=540s --entry-point=collectLeague"}'
```

**Upload a file to VM:**
```bash
curl -s -X POST http://35.209.45.82:8080/upload \
  -H "Authorization: Bearer $TU_DEPLOY_TOKEN" \
  -F "file=@local-file.js" \
  -F "path=/home/deploy/workspace/league-standings/scripts/local-file.js"
```

**Download a file from VM:**
```bash
curl -s http://35.209.45.82:8080/download?path=/home/deploy/workspace/league-standings/some-file.js \
  -H "Authorization: Bearer $TU_DEPLOY_TOKEN"
```

### Working Directory

The league standings service lives at: `/home/deploy/workspace/league-standings/`

Always `cd` into this directory before running scripts or deploys.

### Workflow: Code → Push → Deploy → Run

1. Write/edit code locally and push to `claude/*` branch on GitHub
2. After merge to main, pull on VM: `git pull origin main && npm install`
3. Deploy updated Cloud Functions as needed
4. Run scripts directly on VM

⚠️ **Security**: Never commit the token to this repo. It lives only in your local `$TU_DEPLOY_TOKEN` env var.

## Operational Priorities (ordered)

### Immediate (do now)
1. **Fix WA soccer data hygiene** — See `tasks/wa-soccer-seasonal-fix.md`
   - Normalize sport casing ("Soccer" → "soccer")
   - Add missing seasonStart/seasonEnd to 16 active soccer leagues
   - Add discoveryConfig to 11 GotSport leagues for season monitor auto-discovery
   - Set EWSL to dormant (fall 2025 season ended)
2. **Resolve pending SC leagues** — 13+ Little League programs need standingsTabId for spring 2026
   - Run `scripts/resolve-sportsconnect-pending.js --fix` on deploy VM
3. **Verify spring 2026 data flows** — SSUL (Apr 18), SC leagues (Apr 12-18), EWSL spring TBD

### Short-term
4. ~~**Soccer OR expansion**~~ — DONE (March 12): 5 active, 2 dormant, 3 deactivated
5. ~~**Discover groups for SOCAL and NorCal Premier**~~ — DONE (March 13): SOCAL 526 groups, NorCal 709 groups activated
6. ~~**Softball WA discovery**~~ — DONE (March 13): 4 WA softball leagues registered (SportsConnect), 0 GC orgs found
7. ~~**Basketball discovery**~~ — DONE (March 13): 0 leagues found on supported platforms across 5 states
8. Re-map 34 pending_config baseball leagues to correct GC org IDs
9. Add more OR/ID/MT leagues from other platforms
10. Resolve 3 WA softball SC leagues missing tabIds (Spokane YSA, Pacwest LL, Bellevue National LL)

### Medium-term
8. Build TeamSideline adapter (unlocks Thurston + Lewis County soccer)
9. Improve collectAll parallelism — currently sequential, could batch by platform
10. League Request feature — allow users to request a league be added
11. Phase 2 state expansion planning (CO, NV, AZ, UT)

## Important Notes

- **Puppeteer memory**: GameChanger and SportsConnect adapters use Puppeteer for browser automation. They require at least 1024MB Cloud Function memory. At 488MB they OOM.
- **GC season rotation**: GameChanger creates new org IDs each season. The adapter handles auto-rotation via `allOrgIds` + the public API. The season monitor also discovers rotations.
- **SportsConnect ASP.NET postbacks**: SC sites use ASP.NET WebForms with postback-driven dropdowns. Each dropdown selection causes a full page reload. The adapter handles this with `waitForNavigation`.
- **Rate limiting**: GC discovery uses DuckDuckGo HTML search (3s delay between searches) and GC API (300ms delay between calls). Be respectful of external services.
- **Season monitor**: Runs weekly. Auto-transitions stale leagues to dormant (3x stale threshold). Auto-discovers new seasons where possible. Generates reports in `monitorReports` collection.
- **Firestore batch limits**: Max 500 operations per batch. Code uses 400-op chunks for safety margin.

---
> Source: [helpertubot/teams-united-league-service](https://github.com/helpertubot/teams-united-league-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
