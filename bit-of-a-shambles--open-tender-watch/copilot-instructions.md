## open-tender-watch

> The **Open Tender Watch** is a Rails 8 application that monitors Portuguese (and broader European) public procurement data to detect corruption risk, abuse patterns, and conflicts of interest. It is designed for journalists, auditors, and civic watchdogs.

# Open Tender Watch — Context & Guidelines

## Project Overview

The **Open Tender Watch** is a Rails 8 application that monitors Portuguese (and broader European) public procurement data to detect corruption risk, abuse patterns, and conflicts of interest. It is designed for journalists, auditors, and civic watchdogs.

The approach is **risk scoring, not accusation**. The system surfaces cases for audit or journalistic review with explicit confidence levels — it does not produce conclusions.

---

## Domain Model

- **Entity**: Represents both public bodies (adjudicantes) and private companies (adjudicatários). Identified by `tax_identifier` (e.g. NIF/NIPC), scoped to `country_code` — the same numeric ID can exist in different countries.
- **Contract**: A public procurement record with metadata (object, price, dates, procedure type, CPV, location). Linked to a contracting entity and a data source.
- **ContractWinner**: Join table between `Contract` and `Entity`. A contract can have multiple winners with a `price_share`.
- **DataSource**: DB-driven registry of configured data adapters per country. Each record specifies `adapter_class`, `country_code`, `source_type`, and JSON `config`.

---

## Key Data Sources

### Portugal

| Source | What it provides | Notes |
|---|---|---|
| **Portal BASE** | Central public contracts portal — contracts, announcements, modifications, impugnations | Primary source. Data published in OCDS format on dados.gov.pt via IMPIC. API access available for bulk extraction (registration required). Data quality is the responsibility of contracting entities — late or incomplete entries are themselves risk signals. |
| **Portal da Transparência SNS** | Health-sector contracts via OpenDataSoft v2.1 | ~43,000 records. No API key required. `PublicContracts::PT::SnsClient`. |
| **dados.gov.pt** | Open data platform, includes BASE mirrors | Use for bulk OCDS downloads |
| **TED** | EU-level procurement notices | Valuable for contracts at EU thresholds and cross-checking publication consistency |
| **Registo Comercial** | Company registrations, shareholders, management | Scraped from publicacoes.mj.pt |
| **Entidade Transparência** | Public entities, mandates, persons in public roles | https://entidadetransparencia.pt/ — useful for conflict-of-interest checks |
| **RCBE** | Beneficial ownership register | Access requires authentication by legal person number; CJEU ruling limited open access — treat as a constrained layer |
| **AdC** | Competition Authority — cartel cases, sanctions | Cross-reference supplier NIFs against published AdC cases |
| **Tribunal de Contas** | Audit reports, financial liability decisions | Used to corroborate red flags |
| **Mais Transparência / Portugal2020** | EU-funded contract data | Useful for prioritising EU-funded tenders |
| **ECFP/CNE** | Political party donations | Future source for conflict-of-interest detection |

### EU / Cross-border

| Source | What it provides |
|---|---|
| **TED** | EU procurement notices via REST API and bulk XML packages |

---

## Red Flag Catalogue

Indicators are grouped into three tracks following [OECD methodology](https://www.oecd.org/en/publications/preventing-corruption-in-public-procurement_9789264059765-en.html) (see also [OCP Red Flags for Integrity](https://www.open-contracting.org/resources/red-flags-for-integrity-giving-green-light-to-open-data-solutions/)):

### Track A — Rule-based red flags (high explainability)

| # | Indicator | Data fields | Notes |
|---|---|---|---|
| A1 | Repeat direct awards / prior consultations to same supplier by same authority | `procedure_type`, `contracting_entity_id`, winner NIF, `publication_date` | OECD: repeat awards and concentration over 3 years |
| A2 | Contract published after execution starts | `publication_date`, `celebration_date`, `execution_start_date` | OECD: contract data earlier than adjudication date. **Implementation note:** Portuguese law requires signing first then publishing. Only gaps > 10 days are flagged (`MIN_PUBLICATION_DELAY_DAYS = 10`). Before this threshold was introduced, A2 fired on ~85% of all contracts (any gap). |
| A3 | Execution begins before publication in BASE | `publication_date` vs actual start | OECD: contracts implemented before publication |
| A4 | Amendment inflation — large or frequent modifications | `base_price`, amendment value, amendment count | BASE publishes modifications above thresholds |
| A5 | Contract value just below procedural thresholds | `base_price`, `procedure_type` | Threshold-splitting / fragmentation |
| A6 | Single bidder / low-competition procedure | bidder count, `procedure_type` | BASE publishes procedure type and bidder details |
| A7 | Buyer uses direct award far above peer median for same CPV | `procedure_type`, `cpv_code`, `contracting_entity_id` | Peer comparison by CPV and region |
| A8 | Long execution duration (> 3 years) | contract duration | OECD: execution length over 3 years as risk feature |
| A9 | Estimated value vs final price anomaly | `base_price`, `total_effective_price` | OECD: ratios between estimated, base, and contract price |

### Track B — Pattern-based anomaly flags (statistical / model)

| # | Indicator | Approach |
|---|---|---|
| B1 | Bid rotation — suppliers who rarely compete except with one authority | Cluster analysis of winner NIF × authority pairs |
| B2 | Supplier concentration — share of buyer's spend to one supplier | Herfindahl-style concentration index by authority + CPV |
| B3 | Unusual pricing relative to CPV and region peers | Z-score or percentile within CPV × region × year |
| B4 | Sudden procedural shifts near budget deadlines | Time series of procedure type distribution per authority |

### Track C — Integrity and compliance risk

| # | Indicator | Data fields | Notes |
|---|---|---|---|
| C1 | Missing supplier NIF / unidentified winner | `tax_identifier` null or invalid; zero `contract_winners` with positive effective price | OECD: VAT number completeness is a risk signal. Two sub-cases: (a) winner exists but NIF is blank; (b) contract was awarded (effective_price > 0) but no winner recorded. Investigation of BASE data found ~217K such contracts — primarily "Ajuste Direto Regime Geral" where the contracting authority published a notice without completing supplier fields. This is a known BASE data quality pattern. |
| C2 | Impossible date sequences | date ordering checks | Catches data manipulation or late entry |
| C3 | Missing mandatory fields by procedure type | CPV, location, base price; implausible `base_price` > €1T | Absence or corruption is a risk signal, not just a data defect. **Implementation note:** `base_price` values above €1 trillion absolute value are treated as data-entry errors (price entered in cents, extra zeros). The import service nulls these out on ingestion (`MAX_PLAUSIBLE_PRICE = 1_000_000_000_000`); C3 catches any that bypass the cap. Typical BASE examples: €147B drug contract, €80B tablet purchase. |
| C4 | Supplier overlap with AdC sanction cases | winner NIF × AdC case database | OECD: AdC sanctions enriched with NIF for cross-referencing |
| C5 | Entity name variations masking same entity | fuzzy match + NIF | Entity resolution is mandatory — names vary widely |
| C6 | Contract not submitted to TdC when required | TdC data (constrained) | Some indicators require TdC internal data |

### Data quality as a flag

Missing or inconsistent data is not just a technical defect — it can indicate evasion. Flag:
- Missing supplier NIF
- Missing CPV code
- Contract amendments with missing legal basis
- Repeated manual text variations for the same entity name
- Date fields outside plausible ranges
- `base_price` above €1 trillion (almost certainly a data-entry error — price in cents or extra zeros)
- Awarded contract (`total_effective_price > 0`) with no winner recorded (known BASE pattern: ~217K contracts as of 2025)

---

## Scoring Architecture

Each flagged case should carry:
- **Risk score** (weighted sum of active flags)
- **Evidence fields** (which fields triggered the flag)
- **Data completeness score** (fraction of expected fields present)
- **Confidence level** (low / medium / high — degrades when key fields are missing)

This prevents weak data from producing overconfident conclusions.

---

## Implementation Phases

### Phase 1 — Procurement spine *(in progress)*
Build a clean ingestion pipeline normalising core fields across all adapters (contract ID, authority NIF, supplier NIF, procedure type, CPV, prices, dates). Multi-country adapter framework in place. Domain model complete with country-scoped uniqueness. Target: full Portal BASE + SNS data loaded, all C-track data quality flags firing.

### Phase 2 — Rule-based flags *(next)*
Implement Track A flags one by one as DB-level queries or lightweight services. Each flag is a separate, independently testable unit. Deploy dashboard filter and case drill-down as flags come online. Order of implementation:
1. A2/A3 — date anomalies (already partially firing from `celebration_date < publication_date`)
2. A9 — price-to-estimate anomaly (base vs effective price ratio)
3. A5 — threshold-splitting (contract value just below €5K / €20K / €75K / €150K thresholds)
4. A1 — repeat direct awards (same authority + supplier, window of 36 months)
5. A4 — amendment inflation (requires amendment data ingestion from BASE)
6. A7 — abnormal direct award rate (peer comparison by CPV, needs sufficient data volume)
7. A6 — single bidder (requires bidder count from BASE)
8. A8 — long execution (requires execution dates)

### Phase 3 — External enrichment *(planned)*
Ingest and cross-reference additional sources:
- **TED**: cross-check publication consistency for EU-threshold tenders; flag contracts above threshold that are absent from TED
- **AdC**: match supplier NIFs against Competition Authority sanction list (C4)
- **Entidade Transparência**: link contract parties to persons in public roles; surface potential conflicts of interest
- **Mais Transparência / Portugal2020**: tag EU-funded contracts for prioritised scrutiny

### Phase 4 — Graph database layer *(planned)*
Relational SQL is sufficient for per-contract flags but breaks down when the question is about *networks* — who is connected to whom, through how many hops, and how central they are. This phase builds a parallel graph representation alongside the relational DB.

**Why a graph database:**
- Bid rotation (B1) requires detecting clusters of suppliers who co-appear without competing — this is a graph community detection problem, not a SQL aggregation
- Ownership chains require multi-hop traversal: supplier → shareholder → public official → contracting entity
- Entity resolution across sources (name variants, NIFs that appear in multiple roles) maps naturally to a property graph
- Network centrality scoring: flag entities that are unusually central in the procurement network

**Implementation approach:**
- Store the core data in SQLite (Rails ActiveRecord) as today
- Project a graph from it into Neo4j (or similar) as a read-optimised view: Entity nodes, Contract nodes, AWARDED_TO / EMPLOYED_BY / OWNS / RELATED_TO edges
- Run graph algorithms (PageRank, Louvain community detection, shortest path) against the graph layer
- Feed results back as scored flags into the Rails flag system
- Refresh the graph on each import cycle

**Key graph queries:**
- Detect supplier clusters that never compete against each other (bid rotation)
- Find shortest path between a supplier NIF and a public official NIF
- Identify entities that are both winners and connected to contracting authority management
- Score network centrality of each entity as a risk amplifier

### Phase 5 — Statistical/pattern flags *(planned)*
Implement Track B indicators once data volume is sufficient for meaningful statistics:
- B2: Herfindahl concentration index per authority × CPV (requires at least 2 years of data)
- B3: Z-score pricing anomaly within CPV × region × year
- B4: Procedural shift time-series per authority
- B1: Bid rotation (via graph layer from Phase 4)

### Phase 6 — Case triage + deployment *(parallel track)*
This runs in parallel with Phases 2–5:
- Deploy to production (Kamal config already present)
- Case triage UI: per-case evidence trail, confidence level display, flag breakdown
- Export format for referrals (PDF / structured JSON) to TdC, AdC, MENAC
- User accounts and case assignment (for newsroom / audit team use)
- Public vs restricted view split

### Phase 7 — Ownership layer *(constrained)*
RCBE beneficial ownership linkage. Access requires legal person authentication and the 2022 CJEU ruling limits public exposure. Build as a restricted layer available only to authenticated auditors. Track changes — ownership structures shift around procurement cycles.

---

## Backlog

The canonical task list is maintained as GitHub Issues:
**https://github.com/bit-of-a-shambles/open-tender-watch/issues**

Issues are labelled by:
- **priority:** `priority: now` / `priority: next` / `priority: planned`
- **type:** `type: data` / `type: flag` / `type: ui` / `type: infra`
- **difficulty:** `difficulty: easy` / `difficulty: medium` / `difficulty: hard`

Good starting points are tagged `good first issue`.

When working on a task as an LLM agent, query open issues to find what to work on next:
```bash
gh issue list --label "priority: now" --state open
gh issue list --label "good first issue" --state open
```

Close issues when complete: `gh issue close <number> --comment "Fixed in <commit>"`

---

## Data Source Architecture — ETL Pattern

Every data source is a Ruby service class in `app/services/public_contracts/<iso2>/`. It owns the full ETL pipeline for one source:

**Extract** — fetch raw records from the source. This may be a paginated REST API, a file download, or a web scrape. The service handles authentication, pagination, rate limiting, and HTTP error handling.

**Transform** — convert the raw payload into the standard contract hash (see below). Every source speaks a different language: field names differ, dates arrive in different formats, NIFs may be missing, prices may include VAT or not. All of that complexity is encapsulated in the adapter; the rest of the system never sees it.

**Load** — not the adapter's responsibility. Return normalized hashes from `fetch_contracts` and `ImportService` handles persistence, entity resolution, and deduplication.

### The three methods every adapter must implement

```ruby
def fetch_contracts(page: 1, limit: 50) # → Array<Hash>
def country_code                          # → "PT" | "EU" | "ES" …
def source_name                           # → "Portal BASE" (display name)
```

### Standard contract hash

`fetch_contracts` must return an array of hashes in this shape. Only `external_id` and `contracting_entity` fields are required; others may be nil if the source doesn't provide them.

```ruby
{
  "external_id"           => "string",      # required — unique ID in this source
  "country_code"          => "PT",          # required — ISO 3166-1 alpha-2
  "object"                => "string",      # contract description / title
  "procedure_type"        => "string",      # e.g. "Ajuste Direto", "open"
  "contract_type"         => "string",      # goods / services / works
  "publication_date"      => Date,
  "celebration_date"      => Date,          # signing date
  "base_price"            => BigDecimal,    # estimated / base value
  "total_effective_price" => BigDecimal,    # final awarded value
  "cpv_code"              => "34144210",    # 8-digit CPV (no suffix)
  "location"              => "string",      # NUTS code or free text
  "contracting_entity"    => {
    "tax_identifier" => "string",           # required — NIF/NIPC or synthetic
    "name"           => "string",           # required
    "is_public_body" => true
  },
  "winners" => [
    { "tax_identifier" => "string", "name" => "string", "is_company" => true }
  ]
}
```

### Notes for contributors adding a new adapter

- Place the file in `app/services/public_contracts/<iso2>/<source>_client.rb` and wrap it in `PublicContracts::<ISO2>::<Source>Client`.
- If the source has no buyer NIF (e.g. TED), derive a deterministic synthetic identifier: `"TED-#{Digest::MD5.hexdigest(name.downcase.strip)[0, 12]}"`. This allows entity deduplication across notices without a real NIF.
- Never make live HTTP calls in tests. Stub `Net::HTTP.new` with `Minitest::Mock` or `stub`.
- Write a `_test.rb` for every public method. Run `bundle exec rails test` — coverage must stay at 100%.
- Add a `DataSource` record to `test/fixtures/data_sources.yml` so the adapter is included in seed/import tests.
- Document the source in `AGENTS.md` (Key Data Sources table) and in both README files.

---

## Technical Standards

- **NIF/NIPC**: Always stored as a string to preserve leading zeros.
- **Currency**: `decimal` with precision 15, scale 2.
- **country_code**: ISO 3166-1 alpha-2 (PT, ES, FR…). Always 2 letters.
- **external_id**: ID from the original data source, unique within `[external_id, country_code]`.
- **adapter_class**: Must be within the `PublicContracts::` namespace and implement `#fetch_contracts`.
- **Testing**: Minitest. All HTTP stubbed — no live calls in the test suite. 100% SimpleCov line coverage enforced (CI will fail below this).
- **UI**: Rails 8 + Hotwire + Tailwind CSS. Cyberpunk-noir aesthetic (`#0d0f14` background, `#c8a84e` gold, `#ff4444` red alerts).

## File Structure

```
app/
  models/                          Contract, Entity, ContractWinner, DataSource
  services/public_contracts/
    base_client.rb                 Generic HTTP client
    import_service.rb              Ingests contracts from a DataSource record
    pt/
      portal_base_client.rb        Portal BASE API
      sns_client.rb                Portal da Transparência SNS (health sector)
      dados_gov_client.rb          dados.gov.pt API
      registo_comercial.rb         publicacoes.mj.pt scraper
    eu/
      ted_client.rb                TED API v3
  controllers/
    dashboard_controller.rb        Main insight dashboard
    contracts_controller.rb        Contracts index + show
    locales_controller.rb          Locale switcher (EN/PT)
docs/plans/                        Design docs and implementation plans
transparencia/                     Legacy Python scripts for data extraction
```

## Operational Procedures

### Mandatory workflow: always develop against the development database first

**Never run data-modifying tasks directly on production.** The production SQLite database is a promoted copy of the development database. The workflow is:

```
1. Make code changes
2. Run bundle exec rails test   # must pass 100% line coverage
3. Run the relevant rake task(s) locally against storage/development.sqlite3
4. Verify results in the development app (http://localhost:3000)
5. bundle exec rails db:sync:push   # rsync dev DB → production
6. kamal redeploy (only needed for code changes, not for DB-only syncs)
```

The `db:sync:push` task rsyncs `storage/development.sqlite3` to the production server and touches `tmp/restart.txt` to reload Puma. Never skip step 3–4: production data state must always be verified in development before promotion.

---

### Flag scoring SOP

Flag scores are stored in the `flags` table and summarised in `flag_entity_stats` and `flag_summary_stats`. They must be regenerated whenever:

- A flag action's logic changes (threshold, severity, score)
- Import data is substantially updated
- The `ImportService` normalisation logic changes (e.g. price cap, date parsing)

**Full regeneration (recommended after any flag-logic change):**

```bash
bundle exec rails flags:run_all
```

`run_all` executes actions in dependency order then calls `flags:aggregate`. It takes 10–20 minutes on the full dataset.

**Individual flag actions (for targeted reruns):**

```bash
bundle exec rails flags:run_first_action   # A2 date sequence anomaly
bundle exec rails flags:run_a2             # alias for run_first_action
bundle exec rails flags:run_a9             # A9 price anomaly
bundle exec rails flags:run_a5             # A5 threshold splitting
bundle exec rails flags:run_a1             # A1 repeat direct award
bundle exec rails flags:run_b5_benford     # B5 Benford deviation
bundle exec rails flags:run_c1             # C1 missing winner NIF / zero winners
bundle exec rails flags:run_c3             # C3 missing mandatory fields
bundle exec rails flags:run_b2             # B2 supplier concentration
bundle exec rails flags:aggregate          # rebuild summary stats + clear cache
```

**After any rerun, check the aggregate output:**

```
entity stats rows: ~27000
severity=nil: exposure=~€145B contracts=~1.9M companies=~83K public=~11K
severity=high: ...
```

If `exposure=nil: total_exposure=0` then `flags:aggregate` did not find any `FlagEntityStat` rows — check that `run_all` completed without error.

---

### Database sync SOP

```bash
bundle exec rails db:sync:push    # dev → production (rsync + restart)
bundle exec rails db:sync:pull    # production → dev  (if it exists)
```

`db:sync:push` rsyncs `storage/development.sqlite3` (~2.5 GB) to `128.140.78.94:/rails/storage/production.sqlite3` then touches `tmp/restart.txt` in the running container. Expect ~90 seconds upload on a good connection.

**Checklist before pushing:**
- [ ] `bundle exec rails test` — 457+ tests, 0 failures, 100% line coverage
- [ ] `bundle exec rails flags:run_all` — completed without error
- [ ] Dashboard at http://localhost:3000 looks correct (exposure, contract counts)
- [ ] Commit and push all code changes (`git push`) before syncing the DB

---

### Sense checks after sync

Run the sense check script to verify data integrity in production:

```bash
ssh root@128.140.78.94 'CNAME=$(docker ps -q -f name=opentenderwatch-web); \
  docker exec $CNAME bundle exec rails runner /tmp/sense_check.rb'
```

Key invariants to verify:
- **No duplicate NIFs** within the same `country_code`
- **A2 flag rate** ≤ 30% of all contracts (> 50% indicates threshold was dropped)
- **Total dashboard exposure** €100B–€200B range (outside this range = aggregate bug)
- **B5 top entity** ≤ €15B (above this = three-tier price fix may have regressed)
- **Contracts > €1T** = 0 (ImportService must cap `MAX_PLAUSIBLE_PRICE`)
- **Import freshness** — BASE updated within 7 days, TED/SNS within 14 days

---

## Key Data Quality Notes

- BASE data quality is the responsibility of contracting entities — treat inconsistencies as risk signals.
- Entity resolution is mandatory: supplier names vary; always match on NIF where available, then fuzzy name + CPV.
- Some OECD indicators could not be developed due to data availability — document which flags are constrained.
- Beneficial ownership linkage is constrained by RCBE access rules and the CJEU ruling — do not assume it can be automated.
- **A2 minimum gap**: The `DateSequenceAnomalyAction` only fires when `publication_date` is more than `MIN_PUBLICATION_DELAY_DAYS` (currently 10) days after `celebration_date`. Without this threshold, ~85% of BASE contracts fire A2 because signing before publishing is normal in Portugal. Legitimate late-publication risk starts beyond 10 days.
- **Price cap**: `ImportService` nulls out `base_price` and `total_effective_price` values above `MAX_PLAUSIBLE_PRICE` (€1 trillion). Prices above this threshold are data-entry errors (e.g. price entered in cents, extra zeroes). Any that bypass the cap fire `C3_MISSING_MANDATORY_FIELDS`.
- **Zero-winner contracts**: ~217K BASE contracts (primarily "Ajuste Direto Regime Geral") have no recorded winner. This is a known BASE data quality pattern where the contracting authority published the notice without completing supplier fields. These are flagged via `C1_MISSING_WINNER_NIF` when `total_effective_price > 0`.

---
> Source: [bit-of-a-shambles/open-tender-watch](https://github.com/bit-of-a-shambles/open-tender-watch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
