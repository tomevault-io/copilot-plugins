## retirement-planner

> A privacy-first, offline-only retirement planning desktop app built in Rust, supporting USD and INR, with multi-scenario comparison dashboards and rich visualizations.

# Retirement Planner — Project Context

A privacy-first, offline-only retirement planning desktop app built in Rust, supporting USD and INR, with multi-scenario comparison dashboards and rich visualizations.

---

## Project Constraints

- **No internet access required** — fully offline
- **No LLM calls** — no AI features
- **Data is private** — stored locally, user-controlled
- **Desktop app** — macOS, Windows, Linux
- **Simple & focused** — not a complex financial suite

---

## Decided Requirements

| # | Decision |
|---|---|
| **Tax** | Simple — user inputs a flat effective tax rate (%) |
| **Inflation** | Per-category rates (healthcare, housing, food, etc.) + toggle for one blanket rate |
| **Instruments** | Full CRUD — user creates custom instruments (name, type, expected return, etc.) |
| **Security** | OS-level file security, no app-level encryption |
| **Scenarios** | Multiple what-if scenarios per user, compared side-by-side on a single dashboard |
| **Visualizations** | Rich — charts, graphs, comparison panes from the start |
| **India instruments** | Start simple — generic instruments, no PPF/NPS/EPF rule modeling initially |

---

## Tech Stack

| Layer | Choice | Rationale |
|---|---|---|
| **Language** | Rust | Performance, safety, single binary distribution |
| **GUI** | `iced` 0.13+ | Pure Rust, Elm architecture, no webview runtime, good for form-heavy apps |
| **Storage** | `rusqlite` (bundled SQLite) | Single-file, zero-config, offline |
| **Charting** | `plotters` + iced canvas widget | Pure Rust, rich chart types |
| **Serialization** | `serde` + `serde_json` | Import/export plans as JSON |
| **Dirs** | `dirs` crate | Platform-standard app-data paths |
| **IDs** | `uuid` crate | Unique IDs for scenarios, instruments, etc. |

### GUI Framework Comparison (why iced)

| Criteria | iced | egui | Tauri | Dioxus |
|---|---|---|---|---|
| Language | Pure Rust | Pure Rust | Rust + HTML/JS | Rust (RSX) |
| Runtime | Native (wgpu) | Native | Webview required | Native or webview |
| Best for | Form-heavy apps | Dev tools | Web-tech teams | React devs |
| Offline | ✅ Fully native | ✅ Fully native | ⚠️ Webview dep | ✅ Native mode |

---

## Project Structure

```
retirement-planner/
├── Cargo.toml                  # workspace root
├── crates/
│   ├── core/                   # domain logic — zero GUI deps
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── models/
│   │       │   ├── mod.rs
│   │       │   ├── profile.rs      # age, retirement age, life expectancy, currency
│   │       │   ├── instrument.rs   # CRUD: name, type, expected return, allocation
│   │       │   ├── expense.rs      # category, amount, inflation rate
│   │       │   ├── income.rs       # salary, contributions, other income
│   │       │   └── scenario.rs     # a complete what-if configuration
│   │       ├── engine.rs           # year-by-year projection engine
│   │       ├── currency.rs         # USD/INR formatting, cents/paise storage
│   │       └── storage.rs          # SQLite CRUD operations
│   └── ui/                     # iced GUI shell
│       └── src/
│           ├── main.rs
│           ├── app.rs              # top-level iced Application
│           ├── theme.rs            # light/dark theming
│           ├── views/
│           │   ├── mod.rs
│           │   ├── dashboard.rs    # scenario comparison, summary cards
│           │   ├── profile.rs      # profile setup form
│           │   ├── instruments.rs  # CRUD list for instruments
│           │   ├── expenses.rs     # expense categories + inflation rates
│           │   ├── income.rs       # income & savings form
│           │   ├── scenarios.rs    # create/manage scenarios
│           │   └── projections.rs  # charts & graphs
│           ├── charts/
│           │   ├── mod.rs
│           │   ├── corpus_line.rs  # corpus over time (multi-scenario overlay)
│           │   ├── allocation_pie.rs
│           │   └── withdrawal_bar.rs
│           └── components/
│               ├── mod.rs
│               ├── card.rs         # summary card widget
│               ├── form_field.rs   # labeled input widget
│               └── nav.rs          # sidebar navigation
├── assets/                     # icons, fonts
└── README.md
```

---

## Data Model (core)

```rust
enum Currency { USD, INR }

struct Profile {
    id: Uuid,
    name: String,
    current_age: u8,
    retirement_age: u8,
    life_expectancy: u8,
    base_currency: Currency,
    exchange_rate: f64,       // user-supplied USD↔INR
    effective_tax_rate: f64,  // simple %
}

struct Instrument {
    id: Uuid,
    name: String,              // e.g. "US Large Cap Index", "Indian Equity MF"
    asset_class: AssetClass,   // Equity, Debt, RealEstate, Cash, Other
    expected_annual_return: f64,
    current_value_cents: i64,  // stored in cents/paise
    monthly_contribution_cents: i64,
}

enum ExpenseCategory { Housing, Healthcare, Food, Transport, Leisure, Other(String) }

struct Expense {
    id: Uuid,
    category: ExpenseCategory,
    monthly_amount_cents: i64,
    inflation_rate: Option<f64>,  // None → use blanket rate
}

struct InflationConfig {
    blanket_rate: f64,
    use_blanket: bool,  // toggle: if true, ignore per-category rates
}

struct Scenario {
    id: Uuid,
    name: String,           // e.g. "Conservative", "Aggressive", "Early Retirement"
    profile_id: Uuid,
    instruments: Vec<Instrument>,
    expenses: Vec<Expense>,
    incomes: Vec<Income>,
    inflation_config: InflationConfig,
    notes: String,
}
```

**Key principle:** All monetary values stored as `i64` cents/paise — converted to display format only in the UI layer.

---

## Core Engine Logic

The projection engine runs per-scenario:

1. **For each year** from `current_age` to `life_expectancy`:
   - Sum instrument values, apply annual returns
   - Add contributions (pre-retirement only)
   - Inflate expenses per category (or blanket)
   - Post-retirement: withdraw expenses from corpus, apply tax rate
   - Record: year, corpus balance, total income, total expenses, net withdrawal
2. **Output:** `Vec<YearProjection>` — one entry per year
3. **Comparison:** run engine for each scenario → overlay results on single chart

**Key outputs per scenario:**
- Corpus at retirement
- Years corpus lasts
- Safe monthly withdrawal (SWR)
- Corpus depletion age (if applicable)

---

## Dashboard — Single Pane of Truth

```
┌─────────────────────────────────────────────────────────────┐
│  SCENARIO COMPARISON DASHBOARD                               │
├─────────────┬───────────────┬───────────────┬───────────────┤
│             │ Conservative  │ Moderate      │ Aggressive    │
├─────────────┼───────────────┼───────────────┼───────────────┤
│ Corpus @ret │ $1.2M         │ $1.8M         │ $2.5M         │
│ Lasts until │ Age 82        │ Age 91        │ Age 100+      │
│ Monthly SWR │ $3,200        │ $4,800        │ $6,700        │
│ Depletion   │ ⚠ Age 82      │ ✅ Age 91     │ ✅ Never      │
├─────────────┴───────────────┴───────────────┴───────────────┤
│  [═══════ Corpus Over Time — Multi-line Chart ═══════════]  │
│  [═══════ Withdrawal vs Expenses — Bar Chart ════════════]  │
│  [═══════ Asset Allocation — Pie Charts ═════════════════]  │
└─────────────────────────────────────────────────────────────┘
```

---

## Visualizations (plotters)

| Chart | Purpose |
|---|---|
| Corpus line chart | Multi-scenario overlay — corpus balance year-by-year |
| Expense area chart | Stacked categories over time (shows inflation impact) |
| Asset allocation pie | Per-scenario breakdown by instrument/asset class |
| Withdrawal bar chart | Annual withdrawals vs income post-retirement |
| Depletion gauge | Simple readiness indicator per scenario |

---

## Implementation Phases

| Phase | Status | Description |
|---|---|---|
| **1. Foundation** | Done | Cargo workspace, data models, SQLite schema + CRUD, currency module, basic projection engine, 12 unit tests |
| **2. UI Shell** | Done | iced 0.13 app with sidebar navigation, Profile/Instruments/Expenses/Income/Scenarios/Dashboard views |
| **3. Scenarios & Engine** | Done | Scenario CRUD wired, engine runs from UI via "Run Projections" button, results shown in Dashboard |
| **4. Comparison Dashboard** | Done | Multi-scenario summary cards, corpus line chart, withdrawal bar chart, asset allocation pie chart |
| **5. Polish** | Done | JSON export/import, input validation, README with build instructions |
| **6. Advanced Planning & UX** | Done | One-time events (lump-sum inflows/outflows), variable return bands (pessimistic/base/optimistic), CSV export |
| **7. Tech Debt & Performance** | Done | Clippy clean, WAL mode, saturating arithmetic, release profile, 20 tests, rustfmt.toml, CI, criterion benchmarks |

---

## Phase 6 — Advanced Planning & UX

### Goals
Deepen the financial model, improve usability, and add features that make the planner genuinely useful for real-world retirement decisions — without violating the offline/no-AI constraints.

### Feature Areas

#### 6a. Richer Financial Modeling
| Feature | Description |
|---|---|
| **Step-down contributions** | Model contribution changes over time (e.g., stop contributing at age 55, increase at bonus years) |
| **One-time events** | Lump-sum inflows/outflows at specific ages (inheritance, home purchase, medical emergency fund) |
| **Variable return bands** | Per-instrument return range (optimistic / base / pessimistic) — run engine for all three, overlay on chart |
| **Sequence-of-returns risk** | Simulate bad early-retirement years (e.g., -20% in year 1) to stress-test corpus |
| **Partial retirement** | Model a "semi-retired" phase with reduced income before full retirement |

#### 6b. India-Specific Instruments (Opt-in)
| Feature | Description |
|---|---|
| **PPF modeling** | 15-year lock-in, annual contribution cap (₹1.5L), tax-free maturity |
| **EPF modeling** | Employer + employee contributions, 8.25% fixed return, withdrawal rules |
| **NPS modeling** | Tier-I lock-in until 60, 40% mandatory annuity at exit |
| **LTCG / STCG tax** | Equity gains taxed at 12.5% LTCG after ₹1.25L exemption (FY2024 rules) |

#### 6c. UX & Usability
| Feature | Description |
|---|---|
| **Onboarding wizard** | Step-by-step first-run flow: currency → profile → first instrument → first expense → first scenario |
| **Keyboard navigation** | Tab-through forms, Enter to save, Escape to cancel |
| **Undo/redo** | In-memory command stack for form edits and CRUD operations |
| **Scenario duplication** | Clone an existing scenario as a starting point for a new what-if |
| **Notes & tags on scenarios** | Rich text notes + user-defined tags for organizing many scenarios |

#### 6d. Reporting & Export
| Feature | Description |
|---|---|
| **PDF report** | Single-page printable summary per scenario (corpus table + key metrics) — using `printpdf` crate |
| **CSV export** | Year-by-year projection table as CSV for spreadsheet analysis |
| **Chart image export** | Save charts as PNG files to disk |

#### 6e. Data Management
| Feature | Description |
|---|---|
| **Multiple profiles** | Support more than one user profile (e.g., spouse planning, joint vs. individual) |
| **Profile switching** | Switch active profile from sidebar without restart |
| **Backup & restore** | One-click copy of the SQLite file to a user-chosen location |
| **Data migration** | Versioned schema migrations so future upgrades don't break existing data |

### Suggested Implementation Order
1. **6c — Onboarding wizard + scenario duplication** (highest UX impact, low risk)
2. **6a — One-time events + step-down contributions** (most-requested planning features)
3. **6a — Variable return bands** (stress-testing, builds on existing engine)
4. **6b — India instruments** (PPF/EPF/NPS, opt-in toggle per scenario)
5. **6d — PDF + CSV export** (reporting, needs `printpdf` dependency evaluation)
6. **6e — Multiple profiles + data migration** (infrastructure, lower urgency)

---

## Phase 7 — Tech Debt, Best Practices & Performance

### Goals
Audit the entire codebase for correctness, idiomatic Rust, performance bottlenecks, and visual polish. The app should feel instant and look sharp on all platforms.

### Feature Areas

#### 7a. Code Quality & Tech Debt
| Area | Action |
|---|---|
| **Error handling** | Replace all `unwrap()` / `expect()` in non-test code with proper `Result` propagation or user-facing error messages |
| **Clippy clean** | Run `cargo clippy --all-targets --all-features -- -D warnings` and fix every lint |
| **Dead code audit** | Remove unused structs, fields, functions, and `#[allow(dead_code)]` suppressions |
| **Module visibility** | Tighten `pub` surface — expose only what the UI crate actually needs from `core` |
| **Consistent naming** | Enforce Rust naming conventions throughout (snake_case fields, CamelCase types, SCREAMING_SNAKE constants) |
| **Docs** | Add `///` doc comments to all public types and functions in `core` |
| **Dependency audit** | Run `cargo audit` for known CVEs; remove unused transitive deps with `cargo machete` |

#### 7b. Performance
| Area | Action |
|---|---|
| **Engine caching** | Cache `Vec<YearProjection>` per scenario — only re-run engine when scenario data changes, not on every render tick |
| **Lazy DB reads** | Load scenario children (instruments, expenses, incomes) on demand, not all at startup |
| **Chart rendering** | Throttle canvas redraws — only redraw when projection data or window size changes |
| **SQLite tuning** | Enable WAL mode (`PRAGMA journal_mode=WAL`), set `synchronous=NORMAL`, add indexes on foreign keys |
| **Startup time** | Profile with `cargo build --release` + `hyperfine`; target <100ms cold start |
| **Release profile** | Tune `Cargo.toml` release profile: `opt-level = 3`, `lto = "thin"`, `codegen-units = 1`, `strip = "symbols"` |

#### 7c. Visual Polish
| Area | Action |
|---|---|
| **Consistent spacing** | Audit all views for uniform padding, margin, and alignment — define spacing constants in `theme.rs` |
| **Typography** | Bundle a clean variable font (e.g., Inter or Noto Sans) in `assets/fonts/`; set as default in iced |
| **Color system** | Define a full semantic color palette in `theme.rs` (primary, surface, on-surface, error, warning, success) |
| **Dark mode** | Ensure all custom-drawn canvas charts respect the active theme (dark background, light grid lines) |
| **Chart aesthetics** | Anti-aliased lines, rounded bar corners, smooth pie segments, subtle grid lines, axis labels with units |
| **Responsive layout** | Use iced `Responsive` widget so charts and cards reflow gracefully on window resize |
| **Loading states** | Show a spinner or skeleton card while engine is computing projections (async-friendly) |
| **Empty states** | Friendly placeholder UI when no scenarios exist yet — guide user to create one |
| **Micro-interactions** | Button hover/press states, row highlight on hover in tables, smooth scroll in lists |

#### 7d. Robustness & Safety
| Area | Action |
|---|---|
| **Input sanitization** | Clamp all numeric inputs to valid ranges at the model layer, not just UI layer |
| **Integer overflow** | Audit all `i64` arithmetic for potential overflow on large corpus values; use `checked_add` / `saturating_add` |
| **Schema versioning** | Add a `schema_version` table to SQLite; fail gracefully if DB was created by a newer version |
| **Crash recovery** | On panic, log to `~/Library/Logs/RetirementPlanner/` (macOS) / platform equivalent before exit |
| **Test coverage** | Expand unit tests to cover edge cases: retirement age = current age, life expectancy = retirement age, zero contributions, 100% tax rate |
| **Property tests** | Add `proptest` suite: corpus never negative with valid inputs; depletion age always ≥ retirement age |

#### 7e. Developer Experience
| Area | Action |
|---|---|
| **CI pipeline** | Add GitHub Actions workflow: `cargo fmt --check`, `cargo clippy`, `cargo test`, `cargo audit` on every PR |
| **`cargo fmt`** | Enforce `rustfmt.toml` with `edition = "2021"`, `max_width = 100`, `imports_granularity = "Crate"` |
| **Benchmarks** | Add `criterion` benchmarks for the projection engine (1 scenario, 10 scenarios, 50-year horizon) |
| **Feature flags** | Gate India-specific instruments behind a `india-instruments` Cargo feature flag |
| **Logging** | Add `tracing` crate with `tracing-subscriber`; replace all `println!` debug output with structured spans |

### Suggested Implementation Order
1. **7a — Clippy + dead code + error handling** (foundation; unblocks everything else)
2. **7b — Engine caching + SQLite tuning** (biggest perf wins, low risk)
3. **7c — Font + color system + spacing constants** (visual consistency baseline)
4. **7c — Chart aesthetics + dark mode** (builds on color system)
5. **7d — Integer overflow audit + input clamping** (correctness/safety)
6. **7e — CI pipeline + rustfmt + tracing** (dev hygiene, ongoing)
7. **7b — Release profile tuning + startup profiling** (final perf pass)

---

## Testing Strategy

- **Unit tests:** engine calculations, currency formatting, inflation compounding, CRUD ops
- **Property tests (`proptest`):** corpus monotonically grows with zero withdrawals; never negative with valid inputs
- **Integration tests:** SQLite round-trips for all models
- **Manual UI testing** initially; GUI testing deferred

---

## Conversation Changelog

### 2026-02-18 (Initial Planning)
- Defined project constraints: offline, no AI, privacy-first, desktop app
- Discussed GUI frameworks — compared iced, egui, Tauri, Dioxus — chose **iced**
- Decided: simple tax (flat rate), per-category inflation with blanket toggle, full CRUD instruments, OS-level security, multi-scenario comparison dashboard, rich visualizations, start simple on India-specific instruments
- Defined data model, engine logic, project structure
- Created 5-phase implementation plan
- Created AGENTS.md as living context document
- Created workflow to keep AGENTS.md updated

### 2026-02-18 (Phase 1 & 2 Implementation)
- Built Cargo workspace with `core` and `ui` crates
- Implemented all data models: Profile, Instrument (with AssetClass), Expense (with ExpenseCategory), Income, Scenario, InflationConfig
- Built currency module with USD comma formatting and INR lakh/crore formatting (cents/paise i64 storage)
- Built SQLite storage with full CRUD for profiles and scenarios (including child tables: instruments, expenses, incomes)
- Built year-by-year projection engine with depletion detection, SWR calculation, and summary generation
- All 12 unit tests passing (currency formatting, engine projections, storage CRUD)
- Built iced 0.13 UI with sidebar navigation and 6 views: Profile, Instruments, Expenses, Income, Scenarios, Dashboard
- "Run Projections" button wires UI data → engine → dashboard results
- Dropped `plotters-iced` (C++ build issues on macOS), using `plotters` + `plotters-backend` instead
- Note: iced 0.13 removed `Sandbox` trait — using function-based `iced::application()` API
- Note: iced 0.13 padding requires explicit `u16` or `f32` types for array literals

### 2026-02-18 (Phase 4 & 5 Implementation)
- Built 3 canvas-based charts: corpus line (multi-scenario overlay), withdrawal bar, asset allocation pie
- Charts use iced's built-in canvas widget — no C++ deps, no `plotters-iced`
- Chart structs must be stored as owned fields in `DashboardView` (Canvas borrows the Program impl)
- Rewrote dashboard with summary cards, charts, and year-by-year table
- Added `export.rs` module with `PlanExport` struct for JSON export/import
- Export writes to `~/Documents/retirement_plan.json`, import reads from same
- Added `validate()` method for imported data (age ranges, negative values, empty names)
- Added Export/Import buttons to sidebar with status messages
- Created comprehensive README.md with build instructions, usage guide, and project structure
- All 12 tests still passing, workspace compiles cleanly

### 2026-02-18 (Phase 6 & 7 Implementation)
- **Phase 6a**: Added `OneTimeEvent` model (lump-sum inflows/outflows at specific ages); engine processes events via `apply_one_time_event`
- **Phase 6a**: Added `ReturnBand` enum (Pessimistic=0.6x, Base=1.0x, Optimistic=1.4x) and `projection_with_band()` function; dashboard shows all 3 bands for single-scenario view
- **Phase 6d**: CSV export of year-by-year projection table (`projections_to_csv()`); "Export CSV" button in sidebar
- **Phase 6 UI**: Added "One-Time Events" view (7th nav item) for adding/editing lump-sum events
- **Phase 7a**: Cargo clippy clean (`-D warnings` passes); removed `plotters`/`plotters-backend` unused deps; fixed `is_multiple_of()` lint; derived `Default` instead of manual impl; removed dead `from_profile()` function
- **Phase 7b**: Added WAL mode + `synchronous=NORMAL` to SQLite; added `schema_version` table; tuned release profile (`lto=thin`, `codegen-units=1`, `strip=symbols`, `panic=abort`)
- **Phase 7b**: Storage schema updated to persist `one_time_events_json` in scenarios table
- **Phase 7d**: All arithmetic uses `saturating_add`/`saturating_mul`; corpus floored at 0 in engine output
- **Phase 7d**: Test count: **20 unit tests** (8 new edge case tests: zero contributions, 100% tax, retirement=current_age, return bands order, one-time inflow/outflow, corpus never negative, depletion age ordering)
- **Phase 7e**: Created `.rustfmt.toml` (edition=2021, max_width=100, imports_granularity=Crate); created `.github/workflows/ci.yml` (fmt check + clippy + test + release build); added `criterion` benchmarks (`engine_bench.rs`) for 1/10 instruments and all-3-bands
- Note: numeric literals in Rust tests must use consistent underscore grouping (e.g. `10_000_000` not `100_000_00`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/soumendrak)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/soumendrak)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
