## syntheticdata

> Guidance for Claude Code working with this repository.

# CLAUDE.md

Guidance for Claude Code working with this repository.

## Build Commands

```bash
cargo build --release          # Build binary
cargo test                     # All tests
cargo test -p datasynth-core   # Specific crate
cargo test test_name           # Single test
cargo check                    # Check only
cargo fmt && cargo clippy      # Format + lint
cargo bench                    # Benchmarks
```

## CLI Usage

Binary: `datasynth-data` (at `target/release/datasynth-data`)

```bash
datasynth-data generate --demo --output ./output
datasynth-data init --industry manufacturing --complexity medium -o config.yaml
datasynth-data validate --config config.yaml
datasynth-data generate --config config.yaml --output ./output
kill -USR1 $(pgrep datasynth-data)  # Pause/resume (Unix)
```

### Group audit (v5.0+)

```bash
datasynth-data group manifest --config group.yaml --out manifest.json
datasynth-data group shard --manifest manifest.json --shard-id S_SIG_0001 --out ./shards/S_SIG_0001/
datasynth-data group aggregate --manifest manifest.json --shards-dir ./shards --out ./group_archive
datasynth-data group generate --config group.yaml --out ./group_archive   # in-process pipeline
```

## Server

```bash
cargo run -p datasynth-server -- --port 3000 --worker-threads 4
```

## Architecture

Rust workspace with 19 active crates:

```
datasynth-cli             → Binary (generate, validate, init, info, fingerprint, templates, data)
datasynth-server          → REST/gRPC/WebSocket server
datasynth-runtime         → EnhancedOrchestrator coordinates workflow (~30 phases)
datasynth-generators      → Data generators (JE, Document Flows, Subledgers, Anomalies, Audit)
datasynth-banking         → KYC/AML banking with fraud typologies
datasynth-ocpm            → OCEL 2.0 process mining
datasynth-fingerprint     → Privacy-preserving fingerprint extraction/synthesis
datasynth-standards       → Accounting/audit standards (IFRS, US GAAP, French GAAP, German GAAP, ISA, SOX, PCAOB)
datasynth-graph           → Graph export (PyTorch Geometric, Neo4j, DGL)
datasynth-graph-export    → Hypergraph → bulk node/edge export pipeline (RustGraph wire format; GH #218)
datasynth-eval            → Evaluation framework with auto-tuning
datasynth-config          → Configuration schema, validation, presets
datasynth-core            → Domain models, traits, distributions, resource guards, templates
datasynth-output          → Output sinks (CSV, JSON, Parquet)
datasynth-test-utils      → Test utilities
datasynth-audit-fsm       → YAML-driven audit state machines (engagements, blueprints)
datasynth-audit-optimizer → Risk-scoping / portfolio / Monte-Carlo / conformance analytics
datasynth-audit-triage    → Client-GL onboarding + fit-on-self triage (profile, calibrate, materiality, stress-MC, structural-change, relational fidelity)
datasynth-group           → Group audit simulation engine (manifest / shard / aggregate phases)
```

### Key Models (datasynth-core/src/models/)

| Category | Models |
|----------|--------|
| Accounting | JournalEntry, ChartOfAccounts, ACDOCA |
| Master Data | Vendor, Customer, Material, FixedAsset, Employee, EntityRegistry |
| Document Flow | PurchaseOrder, GoodsReceipt, VendorInvoice, Payment, SalesOrder, Delivery, CustomerInvoice, CustomerReceipt, DocumentReference |
| Sourcing (S2C) | SourcingProject, SupplierQualification, RfxEvent, SupplierBid, BidEvaluation, ProcurementContract, CatalogItem, SupplierScorecard, SpendAnalysis |
| Financial Reporting | FinancialStatement, FinancialStatementLineItem, CashFlowItem, ManagementKpi, Budget, BudgetLineItem |
| HR/Payroll | PayrollRun, PayrollLineItem, TimeEntry, ExpenseReport, ExpenseLineItem, BenefitEnrollment |
| Manufacturing | ProductionOrder, RoutingOperation, QualityInspection, InspectionCharacteristic, CycleCount, CycleCountItem, BomComponent, InventoryMovement |
| Sales | SalesQuote, QuoteLineItem |
| Bank Reconciliation | BankReconciliation, BankStatementLine, ReconcilingItem |
| Intercompany | IntercompanyRelationship, ICTransactionType, ICMatchedPair, TransferPricingMethod, GroupStructure, SubsidiaryRelationship, NciMeasurement |
| Subledger | AccountBalance, TrialBalance, AR*/AP*/FA*/Inventory* records, ARAgingReport, APAgingReport, DepreciationRun, InventoryValuation |
| FX/Close | FxRate, CurrencyTranslation, CurrencyTranslationResult, FiscalPeriod, AccrualEntry |
| Anomalies | AnomalyType, LabeledAnomaly, QualityIssue |
| Controls | InternalControl, ControlMapping, SoD |
| COSO Framework | CosoComponent, CosoPrinciple, ControlScope, CosoMaturityLevel |
| Vendor Network | VendorNetwork, VendorRelationship, VendorCluster, VendorLifecycleStage, VendorQualityScore, VendorDependency, SupplyChainTier |
| Customer Segment | SegmentedCustomer, CustomerValueSegment, CustomerLifecycleStage, CustomerNetworkPosition, CustomerEngagement, SegmentedCustomerPool |
| Tax | TaxJurisdiction, TaxCode, TaxLine, TaxReturn, TaxProvision, WithholdingTaxRecord, UncertainTaxPosition, TemporaryDifference, DeferredTaxRollforward, TaxRateReconciliation |
| Treasury | CashPosition, CashForecast, CashPool, CashPoolSweep, HedgingInstrument, HedgeRelationship, DebtInstrument, DebtCovenant |
| ESG | EmissionRecord, EnergyConsumption, WaterUsage, WasteRecord, WorkforceDiversityMetric, PayEquityMetric, SafetyIncident, SafetyMetric, GovernanceMetric, SupplierEsgAssessment, MaterialityAssessment, EsgDisclosure, ClimateScenario |
| Project Accounting | Project, ProjectCostLine, ProjectRevenue, EarnedValueMetric, ChangeOrder, ProjectMilestone |
| Audit (ISA 220/300) | AuditScope |
| Audit (ISA 600) | ComponentAuditor, GroupAuditPlan, ComponentInstruction, ComponentAuditorReport, Misstatement |
| Audit Documentation | EngagementLetter, SubsequentEvent, ServiceOrganization, SocReport, GoingConcernAssessment, AccountingEstimate, AuditOpinion, KeyAuditMatter, Sox302Certification, Sox404Assessment |
| Audit Methodology | CombinedRiskAssessment, MaterialityCalculation, SamplingPlan, SampledItem, SignificantClassOfTransactions, UnusualItemFlag, AnalyticalRelationship |
| Financial Reporting | ConsolidationSchedule, OperatingSegment, SegmentReconciliation, FinancialStatementNote |
| Business Combinations | BusinessCombination, PurchasePriceAllocation, FairValueAdjustment, ContingentConsideration |
| Accounting Standards | EclModel, ProvisionMatrix, EclProvisionMovement, Provision, ProvisionMovement, ContingentLiability |
| HR/Pensions | DefinedBenefitPlan, PensionObligation, PlanAssets, PensionDisclosure, StockGrant, StockCompExpense |
| Relationships | EntityGraph, GraphEntityType, GraphEntityId, RelationshipEdge, RelationshipType, RelationshipStrengthCalculator, CrossProcessLink |
| Graph Properties | ToNodeProperties, GraphPropertyValue, EdgeConstraint, Cardinality |
| Intercompany v5.0 | IcPairId, IcPairPlan, IcRole, ShardContext |
| Group Manifest | GroupManifest, ManifestEntity, ManifestPeriod, OwnershipGraphSection, ShardPlan, ShardAssignment, ChartOfAccountsMaster, FxRateMaster, AuditEngagementPlan, TaxGroupPlan |
| Group Aggregate | AggregatedTb, AggregatedAccount, DeferredEntity, IcMatchResult, IcMatchedPair, UnmatchedSide, UnmatchedReason, EliminationResult, ConsolidatedBalanceSheet, ConsolidatedIncomeStatement, ConsolidatedCashFlow, StatementOfChangesInEquity, ConsolidationSchedule, NotesToConsolidatedFs, NciRollforward, EquityMethodInvestment, CtaRollforward, TranslatedTb, CoverageReport |

### Core Infrastructure (datasynth-core/src/)

- **uuid_factory.rs**: FNV-1a hash-based deterministic UUIDs with generator-type discriminators
- **memory_guard.rs**: Memory limits (Linux /proc/self/statm, macOS ps)
- **disk_guard.rs**: Disk space monitoring (statvfs/GetDiskFreeSpaceExW)
- **cpu_monitor.rs**: CPU tracking with auto-throttle at 0.95 threshold
- **resource_guard.rs**: Unified resource orchestration
- **degradation.rs**: Graceful degradation (Normal→Reduced→Minimal→Emergency)
- **accounts.rs**: GL account constants (AR_CONTROL="1100", AP_CONTROL="2000")
- **graph_properties.rs**: ToNodeProperties trait, GraphPropertyValue enum for typed model→graph property mapping
- **templates/**: YAML/JSON template loading with merge strategies

### Generator Modules (datasynth-generators/src/)

| Directory | Purpose |
|-----------|---------|
| (root) | je_generator, coa_generator, company_selector, user_generator, control_generator, sales_quote_generator, kpi_generator, budget_generator, bank_reconciliation_generator |
| master_data/ | vendor, customer, material, asset, employee generators |
| document_flow/ | p2p_generator, o2c_generator, three_way_match, document_chain_manager |
| sourcing/ | spend_analysis, sourcing_project, qualification, rfx, bid, bid_evaluation, contract, catalog, scorecard generators |
| hr/ | payroll_generator, time_entry_generator, expense_report_generator, benefit_enrollment_generator |
| manufacturing/ | production_order_generator, quality_inspection_generator, cycle_count_generator, bom_generator, inventory_movement_generator |
| standards/ | revenue_recognition_generator, impairment_generator |
| intercompany/ | ic_generator, matching_engine, elimination_generator |
| balance/ | opening_balance, balance_tracker, trial_balance generators |
| subledger/ | ar, ap, fa, inventory generators + reconciliation |
| fx/ | fx_rate_service, currency_translator, cta_generator |
| period_close/ | close_engine, accruals, depreciation, year_end, financial_statement_generator |
| anomaly/ | injector, types, strategies, patterns |
| data_quality/ | missing_values, format_variations, duplicates, typos, labels |
| audit/ | engagement, workpaper, evidence, risk, finding, judgment generators |
| relationships/ | entity_graph_generator for cross-process links and relationship strength |

- priors_loader.rs: SP3 — loads industry-priors `.dsf` bundle into runtime samplers (`ConditionalIETSampler`, `BipartiteFanoutSampler`, `MultiSegmentActiveWindow`, `CrossEntityMotifSampler`, optional `VelocityCalibrator`). Opt-in via `industry_profile.priors.enabled: true`. The shipped bundles are **behavioral-only** — they carry `behavioral.yaml` + `privacy_audit.json` only; `schema.yaml`/`statistics.yaml` are intentionally absent on the parquet-extraction path. See [docs/real-world-priors.md](docs/real-world-priors.md) for the full layout.

### Server (datasynth-server/src/)

- REST: `/api/config`, `/api/stream/{start|stop|pause|resume}`, `/api/stream/trigger/{pattern}`
- WebSocket: `/ws/events`
- Features: API key auth (`X-API-Key`), rate limiting, request timeout

### Graph Module (datasynth-graph/src/)

Builders: transaction_graph, approval_graph, entity_graph
Exporters: pytorch_geometric (.pt), neo4j (CSV + Cypher), dgl

### Banking Module (datasynth-banking/src/)

KYC/AML generator with typologies: structuring, funnel, layering, mule, round_tripping, fraud, spoofing

ERP volume coupling (opt-in, `banking.erp_coupling`): when `enabled`, the
banking population (retail/business/trust counts) is derived from ERP-side
scale — planned JE volume, vendor/customer/employee master counts, and the
global window — instead of the static `banking.population` counts
(anchor-tenant semantic; coverage rates + sqrt volume factor + floors/caps,
see `coupling.rs::derive_population`). `volume.align_period` (default true)
copies the global window over the banking window. Default off → legacy
output byte-identical. Smoke: `datasynth-runtime/tests/banking_coupling_smoke.rs`.

```yaml
banking:
  enabled: true
  erp_coupling:
    enabled: true          # master switch (off ⇒ byte-identical legacy path)
    volume:
      align_period: true   # copy global start_date/period_months
      business_coverage: 0.30   # share of vendors+customers that bank here
      retail_coverage: 0.60     # share of employees that bank here
      background_share: 0.30    # uncoupled background population share
      trust_ratio: 0.05
      reference_erp_annual: 100000  # JE volume at which volume_factor = 1.0
    identity:
      enabled: true          # Stage B: anchor banking identities to the ERP world
      derive_persona_mix: true  # background business persona mix from customer_segmentation shares
```

ERP identity coupling (opt-in, `banking.erp_coupling.identity`): when `enabled`,
the banking population is anchored to real ERP identities instead of being
fully synthetic — the audited companies bank as house customers
(`enterprise_company_code`), a `volume.business_coverage` share of ERP
vendors/customers bank as business customers reusing their ERP
name/country/tax-id (`enterprise_vendor_id`/`enterprise_customer_id`), and a
`volume.retail_coverage` share of employees bank as retail
(`enterprise_employee_id`); the synthetic background fills the remainder, with
the background business persona mix optionally following the
`customer_segmentation` value-segment shares (`derive_persona_mix`). Anchored
customers use a dedicated UUID/RNG stream (seed + 8400). When `identity.enabled`
the runtime skips the legacy modulo cross-reference overlay (which renamed every
banking customer to a rotating core-customer name); off ⇒ legacy overlay +
byte-identical output. Anchor projection: `enhanced_orchestrator::build_erp_anchors`
→ `coupling`/`erp_anchor_generator`; `CustomerGenerator::generate_with_anchors`.

### Process Mining (datasynth-ocpm/src/)

OCEL 2.0 event logs with P2P/O2C process generators

### Fingerprint Module (datasynth-fingerprint/src/)

Privacy-preserving extraction (differential privacy, k-anonymity) → .dsf files → synthesis

```bash
datasynth-data fingerprint extract --input ./data.csv --output ./fp.dsf --privacy-level standard
datasynth-data fingerprint validate ./fp.dsf
datasynth-data fingerprint evaluate --fingerprint ./fp.dsf --synthetic ./synthetic/
```

- behavioral: per-industry behavioral priors (SP2) — source-mix, per-Source IET, lines-per-JE, active lifetime, fan-out, posting-lag. Bundles in `crates/datasynth-generators/resources/priors/`. CLI: `datasynth-data fingerprint extract --behavioral --industry X` / `aggregate-industry` / `info --behavioral`. Bundles are behavioral-only — see [docs/real-world-priors.md](docs/real-world-priors.md).

### Evaluation Module (datasynth-eval/src/)

- statistical/: Benford's Law, distributions, temporal patterns
- coherence/: Balance validation, IC matching, document chains
- quality/: Completeness, duplicates, format validation
- ml/: Feature distributions, label quality, splits
- enhancement/: AutoTuner generates config patches from evaluation gaps
- behavioral_fidelity/: Sajja 2026 P1-P4 metrics adapted to GL (Source / TP entity profile) with degradation-ratio noise-floor normalisation. CLI: `datasynth-data behavioral score`.

### COSO Framework (datasynth-core/src/models/coso.rs)

COSO 2013 Internal Control-Integrated Framework:
- **CosoComponent**: ControlEnvironment, RiskAssessment, ControlActivities, InformationCommunication, MonitoringActivities
- **CosoPrinciple**: 17 principles (IntegrityAndEthics through DeficiencyEvaluation) with `component()` and `principle_number()` helpers
- **ControlScope**: EntityLevel, TransactionLevel, ItGeneralControl, ItApplicationControl
- **CosoMaturityLevel**: NonExistent, AdHoc, Repeatable, Defined, Managed, Optimized

Standard controls include 12 transaction-level (C001-C060) and 6 entity-level (C070-C081) controls with full COSO mappings.

### Standards Module (datasynth-standards/src/)

Accounting and audit standards framework:

| Directory | Purpose |
|-----------|---------|
| framework.rs | `AccountingFramework` (UsGaap, Ifrs, DualReporting), `FrameworkSettings` |
| accounting/ | Revenue (ASC 606/IFRS 15), Leases (ASC 842/IFRS 16), Fair Value (ASC 820/IFRS 13), Impairment (ASC 360/IAS 36) |
| audit/ | ISA references (34 standards), Analytical procedures (ISA 520), Confirmations (ISA 505), Opinions (ISA 700/705/706/701), Audit trail, PCAOB mappings |
| regulatory/ | SOX 302/404 compliance, `DeficiencyMatrix`, Material weakness classification |

Key types:
- **Accounting**: `CustomerContract`, `PerformanceObligation`, `Lease`, `ROUAsset`, `LeaseLiability`, `FairValueMeasurement`, `ImpairmentTest`
- **Audit**: `IsaStandard`, `IsaRequirement`, `AnalyticalProcedure`, `ExternalConfirmation`, `AuditOpinion`, `KeyAuditMatter`, `AuditTrail`
- **Regulatory**: `Sox302Certification`, `Sox404Assessment`, `DeficiencyMatrix`, `MaterialWeakness`

### Standards Configuration

```yaml
accounting_standards:
  enabled: true
  framework: us_gaap  # us_gaap, ifrs, french_gaap, german_gaap, dual_reporting
  revenue_recognition:
    enabled: true
    generate_contracts: true
    avg_obligations_per_contract: 2.0
  leases:
    enabled: true
    lease_count: 50
    finance_lease_percent: 0.30
  fair_value:
    enabled: true
    level1_percent: 0.60
    level2_percent: 0.30
    level3_percent: 0.10

audit_standards:
  enabled: true
  isa_compliance:
    enabled: true
    compliance_level: comprehensive  # basic, standard, comprehensive
    framework: dual  # isa, pcaob, dual
  analytical_procedures:
    enabled: true
    procedures_per_account: 3
  confirmations:
    enabled: true
    positive_response_rate: 0.85
  sox:
    enabled: true
    materiality_threshold: 10000.0

audit:
  enabled: true                          # Enable audit engagement generation

hr:
  enabled: true
  payroll:
    enabled: true
  time_attendance:
    enabled: true
  expenses:
    enabled: true

treasury:
  enabled: true
  cash_positioning:
    enabled: true
  cash_forecasting:
    enabled: true

project_accounting:
  enabled: true
  change_orders:
    enabled: true
  milestones:
    enabled: true
  earned_value:
    enabled: true
```

### Distributions (datasynth-core/src/distributions/)

| File | Purpose |
|------|---------|
| amount.rs | AmountSampler (log-normal + Benford compliance) |
| benford.rs | BenfordSampler, EnhancedBenfordSampler, BenfordDeviationSampler |
| mixture.rs | GaussianMixtureSampler, LogNormalMixtureSampler (weighted components) |
| copula.rs | Gaussian, Clayton, Gumbel, Frank, StudentT copulas |
| correlation.rs | CorrelationEngine (cross-field dependency modeling) |
| pareto.rs | ParetoSampler (heavy-tailed distributions) |
| weibull.rs | WeibullSampler (time-to-event modeling) |
| beta.rs | BetaSampler (proportions, percentages) |
| zero_inflated.rs | ZeroInflatedSampler (excess zeros) |
| conditional.rs | ConditionalDistribution (breakpoint-based generation) |
| drift.rs | DriftConfig, RegimeChange, EconomicCycle parameters |
| industry_profiles.rs | Pre-configured profiles for Retail, Manufacturing, Financial Services |
| temporal.rs | TemporalSampler (seasonality), HolidayCalendar |
| business_day.rs | BusinessDayCalculator (T+N settlement, month-end conventions) |
| period_end.rs | PeriodEndDynamics (decay curves: exponential, extended_crunch) |
| processing_lag.rs | ProcessingLagCalculator (event-to-posting lag modeling) |
| timezone.rs | TimezoneHandler (multi-region timezone handling) |
| holidays.rs | HolidayCalendar (15 regions: US, DE, GB, FR, IT, ES, CA, CN, JP, IN, BR, MX, AU, SG, KR) |
| fraud.rs | FraudAmountGenerator |

### Distributions Configuration

```yaml
distributions:
  enabled: true
  industry_profile: retail        # retail, manufacturing, financial_services
  amounts:
    enabled: true
    distribution_type: lognormal
    components:
      - { weight: 0.60, mu: 6.0, sigma: 1.5, label: "routine" }
      - { weight: 0.30, mu: 8.5, sigma: 1.0, label: "significant" }
      - { weight: 0.10, mu: 11.0, sigma: 0.8, label: "major" }
    benford_compliance: true
  correlations:
    enabled: true
    copula_type: gaussian         # gaussian, clayton, gumbel, frank, student_t
    fields:
      - { name: amount, distribution_type: lognormal }
      - { name: line_items, distribution_type: normal, min_value: 1, max_value: 20 }
      - { name: approval_level, distribution_type: normal, min_value: 1, max_value: 5 }
    matrix:                         # Full symmetric n×n matrix (not upper-triangular)
      - [1.00, 0.65, 0.72]
      - [0.65, 1.00, 0.55]
      - [0.72, 0.55, 1.00]
  regime_changes:
    enabled: true
    economic_cycle:
      enabled: true
      cycle_period_months: 48
      amplitude: 0.15
      recession_probability: 0.1
      recession_depth: 0.25
  validation:
    enabled: true
    tests:
      - { type: benford_first_digit, threshold_mad: 0.015 }
      - { type: distribution_fit, target: lognormal, significance: 0.05 }
      - { type: correlation_check, significance: 0.05 }
      - { type: chi_squared, significance: 0.05 }
      - { type: anderson_darling, significance: 0.05 }
    fail_on_violation: false
```

### Temporal Patterns Configuration

```yaml
temporal_patterns:
  enabled: true

  business_days:
    enabled: true
    half_day_policy: half_day       # full_day, half_day, non_business_day
    month_end_convention: modified_following  # modified_following, preceding, following, end_of_month
    settlement_rules:
      equity_days: 2                # T+2
      government_bonds_days: 1      # T+1
      fx_spot_days: 2
      wire_cutoff_time: "14:00"

  calendars:
    regions: [US, DE, BR, SG, KR]   # 11 regions available

  period_end:
    model: exponential              # flat, exponential, extended_crunch, daily_profile
    month_end:
      start_day: -10
      base_multiplier: 1.0
      peak_multiplier: 3.5
      decay_rate: 0.3
    quarter_end:
      inherit_from: month_end
      additional_multiplier: 1.5
    year_end:
      start_day: -15
      peak_multiplier: 6.0

  processing_lags:
    enabled: true
    sales_order_lag: { mu: 0.5, sigma: 0.8 }
    goods_receipt_lag: { mu: 1.5, sigma: 0.5 }
    invoice_receipt_lag: { mu: 2.0, sigma: 0.6 }
    cross_day_posting:
      enabled: true
      probability_by_hour: { 17: 0.7, 19: 0.9, 21: 0.99 }

  fiscal_calendar:
    calendar_type: custom           # calendar, custom, four_four_five
    year_start_month: 7
    year_start_day: 1

  timezones:
    enabled: true
    default_timezone: "America/New_York"
    consolidation_timezone: "UTC"
    entity_timezones:
      "EU_*": "Europe/London"
      "APAC_*": "Asia/Singapore"

  intraday:
    enabled: true
    segments:
      - { name: morning_spike, start: "08:30", end: "10:00", multiplier: 1.8 }
      - { name: lunch_dip, start: "12:00", end: "13:30", multiplier: 0.4 }
      - { name: eod_rush, start: "16:00", end: "17:30", multiplier: 1.5 }
```

## Key Design Decisions

1. **Deterministic RNG**: ChaCha8 with configurable seed
2. **Precise Decimals**: rust_decimal serialized as strings (no IEEE 754)
3. **Balanced Entries**: JournalEntry enforces debits = credits at construction
4. **Benford's Law**: Amount distribution follows first-digit law
5. **Document Chain Integrity**: Proper payment→invoice reference chains
6. **Balance Coherence**: Assets = Liabilities + Equity validation
7. **Collision-Free UUIDs**: Generator-type discriminators prevent ID collisions
8. **Graceful Degradation**: Progressive feature reduction under resource pressure
9. **Three-Way Match**: PO/GR/Invoice matching with configurable tolerances

## Configuration

YAML sections: `global`, `companies`, `chart_of_accounts`, `transactions`, `output`, `fraud`, `internal_controls`, `enterprise`, `master_data`, `document_flows`, `intercompany`, `balance`, `subledger`, `fx`, `period_close`, `graph_export`, `anomaly_injection`, `data_quality`, `business_processes`, `templates`, `approval`, `departments`, `distributions`, `temporal_patterns`, `accounting_standards`, `audit_standards`, `vendor_network`, `customer_segmentation`, `relationship_strength`, `cross_process_links`, `source_to_pay`, `financial_reporting`, `hr`, `manufacturing`, `sales_quotes`

v5.0+ Group simulation: `id`, `presentation_currency`, `period`, `seed`, `defaults`, `scoping_profiles`, `ownership`, `intercompany`, `fx`, `audit`, `tax`, `output` (all under top-level keys; no `group:` envelope — the file IS a `GroupConfig`)

Presets: manufacturing, retail, financial_services, healthcare, technology
Complexity: small (~100 accounts), medium (~400), large (~2500)

### Internal Controls Config

```yaml
internal_controls:
  enabled: true
  coso_enabled: true                    # Enable COSO 2013 framework
  include_entity_level_controls: true   # Include C070-C081 entity-level controls
  target_maturity_level: "managed"      # ad_hoc|repeatable|defined|managed|optimized|mixed
  exception_rate: 0.02
  sod_violation_rate: 0.01
```

### Interconnectivity Config

```yaml
vendor_network:
  enabled: true
  depth: 3                              # Tier1/Tier2/Tier3 supply chain
  tiers:
    tier1: { count_min: 50, count_max: 100 }
    tier2: { count_per_parent_min: 4, count_per_parent_max: 10 }
    tier3: { count_per_parent_min: 2, count_per_parent_max: 5 }
  clusters:
    reliable_strategic: 0.20
    standard_operational: 0.50
    transactional: 0.25
    problematic: 0.05
  dependencies:
    max_single_vendor_concentration: 0.15
    top_5_concentration: 0.45

customer_segmentation:
  enabled: true
  value_segments:
    enterprise: { revenue_share: 0.40, customer_share: 0.05, avg_order_min: 50000.0 }
    mid_market: { revenue_share: 0.35, customer_share: 0.20, avg_order_min: 5000.0, avg_order_max: 50000.0 }
    smb: { revenue_share: 0.20, customer_share: 0.50, avg_order_min: 500.0, avg_order_max: 5000.0 }
    consumer: { revenue_share: 0.05, customer_share: 0.25, avg_order_min: 50.0, avg_order_max: 500.0 }
  lifecycle:
    prospect_rate: 0.10
    new_rate: 0.15
    growth_rate: 0.20
    mature_rate: 0.35
    at_risk_rate: 0.10
    churned_rate: 0.08
    won_back_rate: 0.02
  networks:
    referrals: { enabled: true, referral_rate: 0.15 }
    corporate_hierarchies: { enabled: true, hierarchy_probability: 0.30 }

relationship_strength:
  enabled: true
  calculation:
    transaction_volume_weight: 0.30     # Log scale
    transaction_count_weight: 0.25      # Sqrt scale
    relationship_duration_weight: 0.20
    recency_weight: 0.15               # Exp decay, 90d half-life
    mutual_connections_weight: 0.10    # Jaccard index
    recency_half_life_days: 90
  thresholds:
    strong: 0.7
    moderate: 0.4
    weak: 0.1

cross_process_links:
  enabled: true
  inventory_p2p_o2c: true              # GoodsReceipt → Delivery links
  payment_bank_reconciliation: true
  intercompany_bilateral: true
```

### Validation Rules

- period_months: 1-120
- compression level: 1-9
- rates/percentages: 0.0-1.0
- approval thresholds: ascending order
- distribution sums: 1.0 (±0.01)

### Fraud rate math (line-level vs document-level)

`fraud.fraud_rate` flags individual JE lines. `fraud.document_fraud_rate`
(optional) flags source documents (POs, invoices, payments, deliveries)
— when `fraud.propagate_to_lines = true` (default), every JE derived
from a fraudulent document inherits `is_fraud = true` and
`is_fraud_propagated = true`.

Observed line-level fraud prevalence is approximately:

```
P(line is_fraud) ≈ fraud_rate + (doc_fraud_rate × fraction_of_lines_from_docs)
```

For a typical P2P/O2C job where ~30 % of JE lines come from
document-derived postings, setting `fraud_rate = 0.02` +
`document_fraud_rate = 0.05` yields ~3.5 % total line-level fraud —
not 2 %. To target a specific line-level fraud prevalence X:

```
fraud_rate = X - (doc_fraud_rate × ~0.30)
```

Config keys accept both snake_case and camelCase (e.g. `fraud_rate` or
`fraudRate`, `document_fraud_rate` or `documentFraudRate`,
`propagate_to_lines` or `propagateToLines`) so SDK clients that follow
camelCase conventions don't silently fall through to defaults.

### Behavioral biases on fraud entries

Every JE flagged `is_fraud = true` — regardless of path (anomaly
injector, document-level fraud propagation, je_generator intrinsic
fraud, create_self_approval, create_sod_violation) — passes through
`datasynth_core::fraud_bias::apply_fraud_behavioral_bias` which applies:

| Bias | Default | Effect |
|------|---------|--------|
| `weekend_bias` | 0.30 | Shift posting_date to Sat/Sun |
| `round_dollar_bias` | 0.40 | Rescale entry to land max line on $1K/$5K/$10K/$25K/$50K/$100K (balance preserved) |
| `off_hours_bias` | 0.35 | Shift created_at to 22:00–05:59 UTC |
| `post_close_bias` | 0.25 | Set `is_post_close = true` |

These probabilities are **config-overridable** via the `fraud.bias`
section (defaults above; output is byte-identical unless overridden):

```yaml
fraud:
  fraud_rate: 0.04
  bias:                      # behavioral-bias signatures on fraud entries
    enabled: true            # master switch (false ⇒ no bias ⇒ residual-faint fraud)
    weekend_bias: 0.30
    round_dollar_bias: 0.40
    off_hours_bias: 0.35
    post_close_bias: 0.25
```

Lowering them yields *subtler* fraud (fewer forensic signals — the
adversary's lever in detector co-training); `enabled: false` removes the
lift entirely. Wired through the enhanced + streaming orchestrators and
the anomaly injector. Lift on these signals is smoke-tested in
`crates/datasynth-runtime/tests/fraud_bias_smoke.rs`; CI failure means
bias wiring regressed on one of the fraud paths.

### Fraud difficulty axis (A2)

`fraud.difficulty` is a single hardness knob that resolves to a `fraud.bias`
preset via `FraudConfig::effective_bias()`. Default `standard` uses the
explicit `bias` field (byte-identical); the other levels override it:

```yaml
fraud:
  difficulty: standard   # standard | forensic | subtle | adversarial
```

- `forensic` — loud signatures (weekend 0.55 / round 0.65 / off-hours 0.55 / post-close 0.45), easiest to detect.
- `subtle` — faint signatures (0.10 / 0.10 / 0.10 / 0.05).
- `adversarial` — bias **off** → residual-faint fraud (the hardest, label-free-defeating case; FINDINGS §44).

### Fraud campaigns (A1)

`fraud.campaigns` plants persistent, relocation-structured fraud: a pinned
counterparty (beneficiary) account stays fixed across periods while the
booking leg rotates from a pool — turning the i.i.d.-in-time DGP into a
campaign simulator so cross-period / relational / memory detectors can be
benchmarked (FINDINGS §33/§36/§40). Off by default → byte-identical output.

```yaml
fraud:
  campaigns:
    enabled: false           # master switch
    count: 1                 # distinct campaigns
    per_period_count: 2      # fraud JEs restructured per period per campaign
    booking_leg_pool: 6      # size of the rotating booking-leg account pool
    rotate_every_periods: 1  # relocate the booking leg every N periods
    period_days: 30          # JE-timeline bucket length
```

Restructured campaign JEs are flagged `is_fraud` and labelled
`RoundTripping` with `observability = memory_only` and campaign metadata
(id / period / counterparty / booking-leg). Planner:
`datasynth-generators/src/anomaly/campaign.rs`; wired through both
orchestrators.

### Observability-class labels (A4)

Every `LabeledAnomaly` carries an `observability` class —
`per_je_density` / `relational_graph` / `temporal` / `memory_only` —
derived from the anomaly type (routing/observability thesis, FINDINGS §12),
emitted in the labels output. `datasynth-eval`'s
`ml::DetectabilityAnalyzer` (A6) profiles a planted population across these
arms; `statistical::RelationalFidelityAnalyzer` (A5) measures account-flow
graph realism vs an optional corpus reference band.

### External expectations — ISA-520 substantive-analytics layer (Phase 2)

`financial_reporting.external_expectations` emits, per material GL account, an
exogenous ISA-520 expectation — an expected period total derived from a driver
(`prior_year` / `market_index` / `macro_series` / `budget`) plus a materiality
tolerance band — with the realized deviation and the **ground-truth fraud
contribution** (`actual − legitimate`). The expectation is anchored to the
account's *legitimate* (non-fraud) level perturbed by a forecast error, so a
mimetic aggregate inflation (invisible to the per-JE residual arms) deviates
beyond the band: `exceeds_band` is the "investigate" trigger, a true positive
iff `is_fraud_inflated`. This is the engine-side counterpart to the perfect-crime
countermeasure (`prop:counter`; see `docs/phase2-ledger-evidence-assurance.md`).
Off by default → byte-identical output. Output: `sales_kpi_budgets/external_expectations.json`.

A second arm, `financial_reporting.evidence_anchors`, is the **ISA-505 external-corroboration**
layer (existence/occurrence): per material account it records whether the activity is corroborated by
exogenous evidence; a material, uncorroborated account is a **dangling node** (`is_dangling`). Genuine
accounts corroborate at `corroboration_rate`; fraud-linked accounts only at `fabrication_evade_rate`
(the forged-evidence "perfect audit crime"). Ground truth: `fraud_activity`, `is_fraud_linked`. The two
arms are complementary — ISA-520 catches *concentrated aggregate inflation*, ISA-505 catches the
*fabricated-counterparty existence gap*. Output: `sales_kpi_budgets/evidence_anchors.json`. Both are
JSON outputs (gated by `output.formats`, like budgets/KPIs).

```yaml
financial_reporting:
  enabled: true
  external_expectations:
    enabled: false           # master switch
    driver: prior_year       # prior_year | market_index | macro_series | budget
    tolerance_pct: 0.10      # materiality band (ISA-520 investigate threshold)
    forecast_noise: 0.05     # expectation forecast-error std (→ realistic false positives)
    growth_rate: 0.05        # expected period-over-period growth framing the driver
    min_materiality_share: 0.005  # only material accounts are scored
  evidence_anchors:
    enabled: false           # master switch (ISA-505 corroboration)
    corroboration_rate: 0.92      # genuine accounts corroborated (1 − this = clean dangling/FP rate)
    fabrication_evade_rate: 0.10  # fraud-linked accounts corroborated anyway (forged evidence / FN)
    min_materiality_share: 0.005
```

## Anomaly Categories

- **Fraud**: FictitiousTransaction, RevenueManipulation, SplitTransaction, RoundTripping, GhostEmployee, DuplicatePayment
- **Error**: DuplicateEntry, ReversedAmount, WrongPeriod, WrongAccount, MissingReference
- **Process**: LatePosting, SkippedApproval, ThresholdManipulation
- **Statistical**: UnusualAmount, TrendBreak, BenfordViolation
- **Relational**: CircularTransaction, DormantAccountActivity

## Data Quality Variations

- **Missing**: MCAR, MAR, MNAR, Systematic
- **Formats**: Date (ISO/US/EU), Amount (comma/period), Identifier (case/padding)
- **Typos**: Keyboard-aware, transposition, OCR errors, homophones
- **Encoding**: Mojibake, BOM issues, HTML entities

## Export Files

Output files are organized by domain directory. All files are JSON unless otherwise noted.

| Category | Directory | Files |
|----------|-----------|-------|
| Transactions | (root) | journal_entries.csv, journal_entries.json, acdoca.csv |
| Master Data | master_data/ | vendors, customers, materials, fixed_assets, employees, cost_centers |
| Employee History | hr/ | employee_change_history |
| Document Flow | document_flows/ | purchase_orders, goods_receipts, vendor_invoices, payments, customer_receipts, sales_orders, deliveries, customer_invoices, document_references |
| Sourcing (S2C) | sourcing/ | spend_analyses, sourcing_projects, supplier_qualifications, rfx_events, supplier_bids, bid_evaluations, procurement_contracts, catalog_items, supplier_scorecards |
| HR/Payroll | hr/ | payroll_runs, payroll_line_items, time_entries, expense_reports, benefit_enrollments |
| HR/Pensions | hr/ | pension_plans, pension_obligations, plan_assets, pension_disclosures |
| HR/Stock Comp | hr/ | stock_grants, stock_comp_expense |
| Manufacturing | manufacturing/ | production_orders, quality_inspections, cycle_counts, bom_components, inventory_movements |
| Subledger | subledger/ | ap_invoices, ar_invoices, fa_records, inventory_positions, inventory_movements, ar_aging, ap_aging, depreciation_runs, inventory_valuation, dunning_runs, dunning_letters |
| Balance | balance/ | opening_balances, subledger_reconciliation |
| Financial Reporting | financial_reporting/ | financial_statements, bank_reconciliations, notes_to_financial_statements |
| Financial Reporting — Standalone | financial_reporting/standalone/ | {entity_code}_financial_statements (one per entity) |
| Financial Reporting — Consolidated | financial_reporting/consolidated/ | consolidated_financial_statements, consolidation_schedule |
| Financial Reporting — Segments | financial_reporting/segment_reporting/ | segment_reports, segment_reconciliations |
| Period Close | period_close/ | trial_balances |
| Sales / KPIs / Budgets | sales_kpi_budgets/ | sales_quotes, management_kpis, budgets, external_expectations, evidence_anchors |
| Intercompany | intercompany/ | group_structure, ic_matched_pairs, ic_seller_journal_entries, ic_buyer_journal_entries, ic_elimination_entries, nci_measurements |
| FX | fx/ | fx_rates, cta_entries |
| Tax | tax/ | tax_jurisdictions, tax_codes, tax_provisions, tax_lines, tax_returns, withholding_records, temporary_differences, etr_reconciliation, deferred_tax_rollforward, deferred_tax_journal_entries |
| Treasury | treasury/ | cash_positions, cash_forecasts, cash_pools, cash_pool_sweeps, debt_instruments, hedging_instruments, hedge_relationships, bank_guarantees, netting_runs |
| Project Accounting | project_accounting/ | projects, cost_lines, revenue_records, earned_value_metrics, change_orders, milestones |
| ESG | esg/ | emission_records, energy_consumption, water_usage, and others |
| Accounting Standards | accounting_standards/ | customer_contracts, impairment_tests, business_combinations, business_combination_journal_entries, ecl_models, ecl_provision_movements, ecl_journal_entries, provisions, provision_movements, contingent_liabilities, provision_journal_entries |
| Accounting Standards — FX | accounting_standards/fx/ | currency_translation_results |
| Controls (CSV) | internal_controls/ | internal_controls.csv, control_account_mappings.csv, control_process_mappings.csv, control_threshold_mappings.csv, control_doctype_mappings.csv, sod_conflict_pairs.csv, sod_rules.csv, coso_control_mapping.csv |
| Controls (JSON) | internal_controls/ | internal_controls.json, sod_violations.json |
| Banking | banking/ | banking_customers, banking_accounts, banking_transactions, aml_transaction_labels, aml_customer_labels, aml_account_labels, aml_relationship_labels, aml_narratives |
| Banking Reconciliation | financial_reporting/ | bank_reconciliations (embedded in financial_reporting/) |
| Process Mining | process_mining/ | event_log.json (OCEL 2.0), process_variants, and others |
| Audit — Core | audit/ | audit_engagements, audit_scopes, audit_workpapers, audit_evidence, audit_risk_assessments, audit_findings, audit_judgments |
| Audit — Confirmations | audit/ | audit_confirmations, audit_confirmation_responses |
| Audit — Procedures | audit/ | audit_procedure_steps, audit_samples, audit_analytical_results |
| Audit — Internal Audit | audit/ | audit_ia_functions, audit_ia_reports, audit_related_parties, audit_related_party_transactions |
| Audit — ISA 210 | audit/ | engagement_letters |
| Audit — ISA 315 | audit/ | combined_risk_assessments, significant_transaction_classes |
| Audit — ISA 320 | audit/ | materiality_calculations |
| Audit — ISA 402 | audit/ | service_organizations, soc_reports, user_entity_controls |
| Audit — ISA 520 | audit/ | unusual_items, analytical_relationships |
| Audit — ISA 530 | audit/ | sampling_plans, sampled_items |
| Audit — ISA 540 | audit/ | accounting_estimates |
| Audit — ISA 560 | audit/ | subsequent_events |
| Audit — ISA 570 | audit/ | going_concern_assessments |
| Audit — ISA 600 | audit/ | component_auditors, group_audit_plan, component_instructions, component_reports |
| Audit — ISA 700/701 | audit/ | audit_opinions, key_audit_matters |
| Audit — SOX | audit/ | sox_302_certifications, sox_404_assessments |
| Audit — Standards Reference | audit/ | isa_mappings, isa_pcaob_mappings |
| Labels | labels/ | anomaly_labels, fraud_labels, quality_issues, quality_labels |
| Graphs | graphs/ | PyTorch Geometric (.pt), Neo4j CSV+Cypher, DGL, RustGraph JSON, hypergraph |
| Events | events/ | process_evolution_events, organizational_events, disruption_events |
| Compliance | standards/ | standards, cross_references, jurisdiction_profiles, audit_procedures, compliance_findings, regulatory_filings |
| Group Audit — Per-entity | entities/{code}/ | full single-entity archive per shard |
| Group Audit — Consolidated | consolidated/ | consolidated_financial_statements.json, consolidation_schedule.json, notes_to_consolidated_fs.json, nci_rollforward.json, cta_rollforward.json, translation_worksheet.json, equity_method_investments.json |
| Group Audit — IC Eliminations | ic_eliminations/ | ic_matching_coverage.json |
| Group Audit — Manifest | (root) | manifest.json (when group generate is used) |
| Group Audit — Shard Summary | (root) | shard_summary.json (per shard) |

## Performance

~200K+ entries/second single-threaded, scales with cores, memory-efficient streaming

## Python Integration

The open-source `datasynth-py` wrapper has been retired. For Python integrations use the official commercial SDKs from [VynFi](https://vynfi.com), or invoke the `datasynth-data` CLI from Python via `subprocess` and read the generated CSV/JSON/Parquet outputs with pandas / polars / pyarrow.

---
> Source: [mivertowski/SyntheticData](https://github.com/mivertowski/SyntheticData) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
