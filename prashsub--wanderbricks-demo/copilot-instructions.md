## wanderbricks-demo

> Comprehensive methodology for creating multi-phase project plans for Databricks data platform solutions


# Project Plan Methodology for Databricks Solutions

## Pattern Recognition

This rule documents the comprehensive methodology for creating multi-phase project plans for Databricks data platform solutions. The patterns were discovered and refined during the Databricks Health Monitor project planning process.

**Key Assumption:** Planning starts AFTER Bronze ingestion and Gold layer design are complete. These are prerequisites, not phases.

---

## When to Apply This Rule

Apply this methodology when:
- Creating architectural plans for Databricks data platform projects
- Building observability/monitoring solutions using system tables
- Planning multi-artifact solutions (TVFs, Metric Views, Dashboards, etc.)
- Developing agent-based frameworks for platform management
- Creating frontend applications for data platform interaction

---

## Plan Structure Framework

### Prerequisites (Not Numbered Phases)

Before planning begins, these must be complete:

| Prerequisite | Description | Status |
|--------------|-------------|--------|
| Bronze Layer | Raw data ingestion from source systems | ✅ Complete |
| Silver Layer | DLT streaming with data quality | ✅ Complete |
| Gold Layer | Dimensional model (star schema) | ✅ Complete |

### Standard Project Phases

```
plans/
├── README.md                              # Index and overview
├── prerequisites.md                       # Bronze/Silver/Gold summary (optional)
├── phase1-use-cases.md                    # Analytics artifacts (master)
│   ├── phase1-addendum-1.1-ml-models.md   # Machine Learning
│   ├── phase1-addendum-1.2-tvfs.md        # Table-Valued Functions
│   ├── phase1-addendum-1.3-metric-views.md # UC Metric Views
│   ├── phase1-addendum-1.4-lakehouse-monitoring.md # Monitoring
│   ├── phase1-addendum-1.5-ai-bi-dashboards.md # Dashboards
│   ├── phase1-addendum-1.6-genie-spaces.md # Natural Language
│   └── phase1-addendum-1.7-alerting-framework.md # Alerting
├── phase2-agent-framework.md              # AI Agents
└── phase3-frontend-app.md                 # User Interface
```

### Phase Dependencies

```
Prerequisites (Bronze → Silver → Gold) → Phase 1 (Use Cases) → Phase 2 (Agents) → Phase 3 (Frontend)
         [COMPLETE]                               ↓
                                           All Addendums
```

---

## Agent Domain Framework

### Core Principle

**ALL artifacts across ALL phases MUST be organized by Agent Domain.** This ensures:
- Consistent categorization across 100+ artifacts
- Clear ownership by future AI agents
- Easy discoverability for users
- Aligned tooling for each domain

### Standard Agent Domains

| Domain | Icon | Focus Area | Key Gold Tables |
|--------|------|------------|-----------------|
| **Cost** | 💰 | FinOps, budgets, chargeback | `fact_usage`, `dim_sku`, `commit_configurations` |
| **Security** | 🔒 | Access audit, compliance | `fact_audit_events`, `fact_table_lineage` |
| **Performance** | ⚡ | Query optimization, capacity | `fact_query_history`, `fact_node_timeline` |
| **Reliability** | 🔄 | Job health, SLAs | `fact_job_run_timeline`, `dim_job` |
| **Quality** | ✅ | Data quality, governance | `fact_data_quality_monitoring_table_results` |

### Agent Domain Application

Every artifact (TVF, Metric View, Dashboard, Alert, ML Model, Monitor, Genie Space) must:
1. Be tagged with its Agent Domain
2. Use the domain's Gold tables
3. Answer domain-specific questions
4. Be grouped with related domain artifacts in documentation

**Example Pattern:**

```markdown
## 💰 Cost Agent: get_top_cost_contributors

**Agent Domain:** 💰 Cost
**Gold Tables:** `fact_usage`, `dim_workspace`
**Business Questions:** "What are the top cost drivers?"
```

---

## Agent Layer Architecture Pattern

### Core Principle: Agents Use Genie Spaces as Query Interface

**AI Agents DO NOT query data assets directly.** Instead, they use Genie Spaces as their natural language query interface. Genie Spaces translate natural language to SQL and route to appropriate tools (TVFs, Metric Views, ML Models).

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     USERS (Natural Language)                             │
│  "Why did costs spike last Tuesday?"                                    │
└─────────────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────┐
│              PHASE 2: AI AGENT LAYER (LangChain/LangGraph)              │
│                                                                          │
│  ┌────────────────────────────────────────────────────────┐            │
│  │ ORCHESTRATOR AGENT                                     │            │
│  │ • Intent classification (cost, security, performance)  │            │
│  │ • Routes to specialized agents                         │            │
│  │ • Coordinates multi-agent workflows                    │            │
│  │ • Uses: Unified Health Monitor Genie Space             │            │
│  └────────────────────────────────────────────────────────┘            │
│                               │                                          │
│   ┌───────────────────────────┼───────────────────────────┐            │
│   ▼                           ▼                           ▼            │
│  ┌─────────┐          ┌─────────────┐           ┌─────────────┐        │
│  │ Cost    │          │ Security   │           │ Performance │        │
│  │ Agent   │          │ Agent      │           │ Agent       │        │
│  │         │          │            │           │             │        │
│  │ Uses:   │          │ Uses:      │           │ Uses:       │        │
│  │ Cost    │          │ Security   │           │ Performance │        │
│  │ Intel   │          │ Auditor    │           │ Analyzer    │        │
│  │ Genie   │          │ Genie      │           │ Genie       │        │
│  └─────────┘          └─────────────┘           └─────────────┘        │
└─────────────────────────────────────────────────────────────────────────┘
                                  │
                                  │ Natural Language Queries
                                  ▼
┌─────────────────────────────────────────────────────────────────────────┐
│            PHASE 1.6: GENIE SPACES (NL Query Execution)                 │
│                                                                          │
│  • Translates natural language → SQL                                    │
│  • Understands domain terminology and synonyms                          │
│  • Routes to appropriate tools (TVFs, Metric Views, ML)                │
│  • Returns structured results to agents                                 │
│                                                                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │
│  │ Cost Intel │  │ Security   │  │ Performance│  │ Job Health │   │
│  │ Genie     │  │ Auditor    │  │ Analyzer   │  │ Monitor    │   │
│  │ 15 TVFs   │  │ 10 TVFs    │  │ 16 TVFs    │  │ 12 TVFs    │   │
│  │ 2 MVs     │  │ 2 MVs      │  │ 3 MVs      │  │ 1 MV       │   │
│  │ 6 ML      │  │ 4 ML       │  │ 7 ML       │  │ 5 ML       │   │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
                                  │
                                  │ SQL + Function Calls
                                  ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                PHASE 1: DATA ASSETS (Agent Tools)                        │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │ METRIC VIEWS (Pre-aggregated analytics - use FIRST)             │   │
│  │ 10 metric views with 30+ measures each                          │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │ TABLE-VALUED FUNCTIONS (Parameterized queries)                  │   │
│  │ 60 TVFs with business logic encapsulation                       │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │ ML PREDICTIONS (ML-powered insights)                            │   │
│  │ 25 ML models with prediction tables                             │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │ LAKEHOUSE MONITORS (Drift detection)                            │   │
│  │ 8 monitors with profile + drift metrics                         │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
                                  │
                                  │ SQL Queries
                                  ▼
┌─────────────────────────────────────────────────────────────────────────┐
│              PREREQUISITES: GOLD LAYER (Foundation)                      │
│              38 dimension + fact tables (star schema)                    │
└─────────────────────────────────────────────────────────────────────────┘
```

### Genie Space → Agent Mapping

Each specialized agent has a corresponding Genie Space that serves as its query interface:

| Agent | Genie Space | Agent Purpose | Tools (via Genie) |
|-------|-------------|---------------|-------------------|
| 💰 **Cost Agent** | Cost Intelligence | Analyze costs, forecast spending, optimize resources | 15 TVFs, 2 MVs, 6 ML |
| 🔒 **Security Agent** | Security Auditor | Monitor audit logs, detect threats, ensure compliance | 10 TVFs, 2 MVs, 4 ML |
| ⚡ **Performance Agent** | Performance Analyzer | Monitor job/query/cluster performance, identify bottlenecks | 16 TVFs, 3 MVs, 7 ML |
| 🔄 **Reliability Agent** | Job Health Monitor | Track SLAs, detect incidents, analyze failures | 12 TVFs, 1 MV, 5 ML |
| ✅ **Data Quality Agent** | Data Quality Monitor | Ensure data quality, track lineage, enforce governance | 7 TVFs, 2 MVs, 3 ML |
| 🤖 **ML Ops Agent** | ML Intelligence | Monitor experiments, model performance, serving endpoints | Integrated across spaces |
| 🌐 **Orchestrator Agent** | Unified Health Monitor | Route queries, coordinate multi-agent workflows | All 60 TVFs, 10 MVs, 25 ML |

### Agent Query Flow

**Example: User asks Cost Agent about cost spike**

```
1. USER: "Why did costs spike last Tuesday?"
              │
              ▼
2. ORCHESTRATOR AGENT: Intent classification → "cost"
              │
              ▼
3. COST AGENT: Uses Cost Intelligence Genie Space
              │
              ▼
4. GENIE SPACE: Translates to SQL/TVF calls
              │
              ├── get_cost_trend_by_sku TVF (14-day trend)
              ├── cost_analytics Metric View (aggregated data)
              └── cost_anomaly_predictions ML Table (anomaly check)
              │
              ▼
5. DATA ASSETS: Execute queries, return results
              │
              ▼
6. COST AGENT: Synthesizes results with context
              │
              ▼
7. RESPONSE: "Costs increased 35% on Tuesday. ML detected anomaly.
              Root cause: 3 new pipelines launched in 'data-eng' workspace.
              Recommendation: Review pipeline efficiency."
```

### Why Genie Spaces (Not Direct SQL)?

| Without Genie Spaces | With Genie Spaces |
|---------------------|-------------------|
| Agents must write SQL | Agents use natural language |
| Agents must know schema | Genie understands semantics |
| Agents must handle synonyms | Genie handles synonyms |
| Agents must select right tool | Genie routes to best tool |
| Complex agent development | Simpler agent development |
| Hard to maintain | Easy to update instructions |

### Multi-Agent Workflow Pattern

For complex queries spanning multiple domains:

```python
# User: "Why did costs spike AND were there job failures?"
                            │
                            ▼
ORCHESTRATOR AGENT:
  1. Intent classification → ["cost", "reliability"]
  2. Fork to multiple agents (parallel)
                            │
          ┌─────────────────┴─────────────────┐
          ▼                                   ▼
    COST AGENT                        RELIABILITY AGENT
          │                                   │
          ▼                                   ▼
    Cost Intelligence Genie           Job Health Monitor Genie
          │                                   │
          ▼                                   ▼
    get_cost_trend_by_sku()           get_failed_jobs()
    cost_anomaly_predictions          job_failure_predictions
          │                                   │
          └─────────────────┬─────────────────┘
                            ▼
ORCHESTRATOR AGENT:
  3. Correlate results
  4. Check: Did failures cause cost spike?
  5. Synthesize unified response
                            │
                            ▼
RESPONSE: "Costs spiked 35% on Tuesday. 5 job failures detected.
           Correlation: Failed jobs retried 12x, causing extra compute.
           Root cause: Cluster timeout in 'data-eng' workspace.
           Actions: Fix cluster config, review retry policy."
```

### Deployment Order (Critical!)

**Genie Spaces MUST be deployed BEFORE agents can use them.**

```
Phase 0: Prerequisites (Complete)
    └── Bronze → Silver → Gold Layer

Phase 1: Data Assets (Deploy First)
    ├── 1.1: ML Models (25 models → prediction tables)
    ├── 1.2: TVFs (60 functions)
    ├── 1.3: Metric Views (10 views)
    ├── 1.4: Lakehouse Monitors (8 monitors)
    ├── 1.5: AI/BI Dashboards (11 dashboards)
    ├── 1.6: Genie Spaces (7 spaces) ← Critical for agents
    └── 1.7: Alerting (40 alerts)

Phase 2: Agent Framework (Deploy After Genie Spaces)
    ├── 2.1: Agent framework setup (LangChain/LangGraph)
    ├── 2.2: Specialized agents (6 agents)
    ├── 2.3: Orchestrator agent
    └── 2.4: Deployment to Model Serving

Phase 3: Frontend (Deploy Last)
    └── Unified UI consuming agents
```

### Agent Integration Testing

**Three-level testing strategy:**

| Level | What to Test | When to Test |
|-------|--------------|--------------|
| **L1: Genie Standalone** | Genie Space returns correct results for benchmark questions | After Genie deployment |
| **L2: Agent Integration** | Agent successfully uses Genie and formats response | After agent deployment |
| **L3: Multi-Agent** | Orchestrator coordinates multiple agents for complex queries | After all agents deployed |

### Genie Space Configuration for Agents

**Each Genie Space must include agent-optimized configuration:**

```yaml
# General Instructions (≤20 lines for Genie to process)
## Query Routing
- Cost questions → cost_analytics metric view + cost TVFs
- Job questions → job_performance metric view + job TVFs
- Query questions → query_performance metric view + query TVFs

## Tool Selection Priority
1. Metric Views (fastest, pre-aggregated)
2. TVFs (parameterized, business logic)
3. ML Predictions (anomalies, forecasts)
4. Gold Tables (only if above insufficient)

## Response Format
- Include time context
- Format currency as USD with 2 decimals
- Format percentages with 1 decimal
- Provide recommendations when applicable
```

### Success Metrics for Agent-Genie Integration

| Metric | Genie Only | With Agents | Target |
|--------|-----------|-------------|--------|
| User adoption | 50-100 power users | 500+ all users | 5-10x |
| Query success rate | 85% | 95% (agents retry) | +10% |
| Time to insight | 2-5 minutes | 10-30 seconds | 10x faster |
| Proactive alerts | 0 (reactive) | 100+ daily | ∞ |
| Complex analysis | 10% of queries | 40% of queries | 4x |

---

## Plan Document Template

### Standard Structure for Each Phase

```markdown
# Phase N: [Phase Name]

## Overview

**Status:** 📋 Planned | 🔧 In Progress | ✅ Complete  
**Dependencies:** [List dependencies]  
**Estimated Effort:** [Duration]  
**Reference:** [Official docs or cursor rules]

---

## Purpose

[2-3 sentences explaining why this phase exists]

---

## [Domain-Specific Sections]

### 💰 Cost Agent: [Artifact Name]

[Artifact details organized by agent domain]

### 🔒 Security Agent: [Artifact Name]

[Continue for all domains]

---

## Implementation Details

[Code examples, SQL, YAML configurations]

---

## Success Criteria

| Criteria | Target |
|----------|--------|
| [Metric] | [Value] |

---

## References

- [Official Docs]
- [Cursor Rules]
```

---

## Enrichment Methodology

### Source Categories for Enrichment

When enriching plans, gather information from these sources (in priority order):

1. **Official Documentation** (highest priority)
   - Databricks docs (docs.databricks.com)
   - Microsoft Learn MCP tool
   - Context7 MCP for library docs

2. **Reference Dashboards**
   - Extract SQL query patterns from `.lvdash.json` files
   - Identify visualization types and KPIs
   - Note filtering and slicing patterns

3. **Community Resources**
   - Blog posts (data engineering blogs)
   - GitHub repositories (dbdemos, etc.)
   - Sample solutions

4. **User Requirements**
   - Specific use cases provided by user
   - Business-specific terminology
   - Custom tag keys and values

### Dashboard Pattern Extraction Process

When provided with dashboard JSON files:

1. **Read the JSON file** to extract:
   - Dataset queries (SQL)
   - Visualization configurations
   - Filter parameters
   - Dashboard title and purpose

2. **Categorize patterns by Agent Domain**

3. **Convert queries to Gold layer references**
   ```sql
   -- FROM dashboard (system tables)
   FROM system.billing.usage
   
   -- TO plan (Gold layer)
   FROM ${catalog}.${gold_schema}.fact_usage
   ```

4. **Document the pattern** with:
   - Source dashboard name
   - Business question answered
   - Visualization type
   - Agent domain classification

### Enrichment Workflow

```
1. Receive reference materials (blogs, repos, dashboards)
                    ↓
2. Extract patterns and queries
                    ↓
3. Categorize by Agent Domain
                    ↓
4. Convert to Gold layer references
                    ↓
5. Add to appropriate addendum
                    ↓
6. Update summary tables and cross-references
```

---

## User Requirement Integration

### Process for Adding User-Specific Use Cases

When users provide specific requirements:

1. **Understand the Business Need**
   - What question are they trying to answer?
   - What decisions will this inform?
   - Who is the end user?

2. **Identify Required Artifacts**
   - Configuration tables (if user-configurable)
   - TVFs (for parameterized queries)
   - Metric Views (for self-service analytics)
   - Dashboards (for visualization)
   - Alerts (for proactive monitoring)
   - ML Models (for predictions/forecasting)

3. **Update ALL Relevant Addendums**
   - A single use case often spans multiple addendums
   - Ensure consistency across artifacts
   - Update summary tables

4. **Add Cross-References**
   - Link related artifacts
   - Document dependencies
   - Update main phase1-use-cases.md

### Example: Commit Tracking Use Case

**User Requirement:** "Track actual spend vs Databricks commit amount with forecast"

**Artifacts Created:**

| Addendum | Artifact | Purpose |
|----------|----------|---------|
| Gold Schema | `commit_configurations` table | Store commit amounts |
| 1.2 TVFs | `get_commit_vs_actual` | Query commit status |
| 1.2 TVFs | `get_commit_forecast` | Query ML forecast |
| 1.3 Metric Views | `commit_tracking_metrics` | Self-service analytics |
| 1.5 Dashboards | Commit Tracking Dashboard | Visualization |
| 1.7 Alerts | COST-009/010/011 | Proactive monitoring |
| 1.1 ML Models | Budget Forecaster enhancement | Prediction capability |
| 1.4 Monitoring | Budget variance metrics | Drift detection |

---

## SQL Query Standards

### Gold Layer Reference Pattern

**ALWAYS use Gold layer tables, NEVER system tables directly.**

```sql
-- ❌ WRONG: Direct system table reference
FROM system.billing.usage

-- ✅ CORRECT: Gold layer reference with variables
FROM ${catalog}.${gold_schema}.fact_usage
```

### Standard Variable References

```sql
-- Catalog and schema (from parameters)
${catalog}.${gold_schema}.table_name

-- Date parameters (STRING type for Genie compatibility)
WHERE usage_date BETWEEN CAST(start_date AS DATE) AND CAST(end_date AS DATE)

-- SCD Type 2 dimension joins
LEFT JOIN dim_workspace w 
    ON f.workspace_id = w.workspace_id 
    AND w.is_current = TRUE
```

### Tag Query Patterns

For systems with custom tags (e.g., billing):

```sql
-- Tag existence check
WHERE custom_tags IS NOT NULL AND cardinality(custom_tags) > 0

-- Tag value extraction
custom_tags['team'] AS team_tag
COALESCE(custom_tags['cost_center'], 'Unassigned') AS cost_center

-- Tag coverage calculation
SUM(CASE WHEN cardinality(custom_tags) > 0 THEN cost ELSE 0 END) / 
    NULLIF(SUM(cost), 0) * 100 AS tag_coverage_pct
```

---

## Artifact Count Standards

### Minimum Artifacts Per Domain

| Artifact Type | Per Domain | Total (5 domains) |
|---------------|------------|-------------------|
| TVFs | 4-8 | 20-40 |
| Metric Views | 1-2 | 5-10 |
| Dashboard Pages | 2-4 | 10-20 |
| Alerts | 4-8 | 20-40 |
| ML Models | 3-5 | 15-25 |
| Lakehouse Monitors | 1-2 | 5-10 |
| Genie Spaces | 1-2 | 5-10 |

### Artifact Naming Conventions

| Artifact | Pattern | Example |
|----------|---------|---------|
| TVF | `get_<domain>_<metric>` | `get_cost_by_tag` |
| Metric View | `<domain>_analytics_metrics` | `cost_analytics_metrics` |
| Dashboard | `<Domain> <Purpose> Dashboard` | `Cost Attribution Dashboard` |
| Alert | `<DOMAIN>-NNN` | `COST-001` |
| ML Model | `<Purpose> <Type>` | `Budget Forecaster` |
| Monitor | `<table> Monitor` | `Cost Data Quality Monitor` |
| Genie Space | `<Domain> <Purpose>` | `Cost Intelligence` |
|| AI Agent | `<Domain> Agent` | `Cost Agent` |

### Agent Tool Discovery Pattern

Agents discover tools via Genie Space data assets:

```
AGENT: "What are the top cost drivers?"
         │
         ▼
GENIE SPACE: Routes based on question type
├── Aggregations → Metric Views (cost_analytics)
├── Rankings → TVFs (get_top_cost_contributors)
├── Anomalies → ML Tables (cost_anomaly_predictions)
└── Trends → Monitors (fact_usage_profile_metrics)
         │
         ▼
AGENT: Receives results, synthesizes response
```

---

## Documentation Quality Standards

### LLM-Friendly Comments

All artifacts must have comments that help LLMs (Genie, AI/BI) understand:
- What the artifact does
- When to use it
- Example questions it answers

```sql
COMMENT 'LLM: Returns top N cost contributors by workspace and SKU for a date range.
Use this for cost optimization, chargeback analysis, and identifying spending hotspots.
Parameters: start_date, end_date (YYYY-MM-DD format), optional top_n (default 10).
Example questions: "What are the top 10 cost drivers?" or "Which workspace spent most?"'
```

### Summary Tables

Every addendum must include:
1. **Overview table** - All artifacts with agent domain, dependencies, status
2. **By-domain sections** - Artifacts grouped by agent domain
3. **Count summary** - Total artifacts by type and domain
4. **Success criteria** - Measurable targets

---

## Plan Maintenance

### When to Update Plans

1. **New use case identified** - Add to relevant addendums
2. **Reference material provided** - Enrich with patterns
3. **Implementation starts** - Update status to "In Progress"
4. **Implementation completes** - Update status to "Complete"
5. **Requirements change** - Update affected artifacts

### Version Control

- Plans are documentation, not code
- Track major changes in commit messages
- Reference issues/PRs when available

---

## Validation Checklist

Before finalizing any plan document:

### Structure
- [ ] Follows standard template
- [ ] Has Overview with Status, Dependencies, Effort
- [ ] Organized by Agent Domain
- [ ] Includes code examples
- [ ] Has Success Criteria table
- [ ] Has References section

### Content Quality
- [ ] All queries use Gold layer tables (not system tables)
- [ ] All artifacts tagged with Agent Domain
- [ ] LLM-friendly comments on all artifacts
- [ ] Examples use `${catalog}.${gold_schema}` variables
- [ ] Summary tables are accurate and complete

### Cross-References
- [ ] Main phase document links to addendums
- [ ] Addendums link back to main phase
- [ ] Related artifacts cross-reference each other
- [ ] Dependencies are documented

### Completeness
- [ ] All 5 agent domains covered
- [ ] Minimum artifact counts met
- [ ] User requirements addressed
- [ ] Reference patterns incorporated

---

## Common Mistakes to Avoid

### ❌ DON'T: Mix system tables and Gold tables

```sql
-- BAD: Direct system table
FROM system.billing.usage u
JOIN ${catalog}.${gold_schema}.dim_workspace w ...
```

### ❌ DON'T: Forget Agent Domain classification

```markdown
## get_slow_queries (BAD - no domain)

## ⚡ Performance Agent: get_slow_queries (GOOD)
```

### ❌ DON'T: Create artifacts without cross-addendum updates

When adding a TVF, also consider:
- Does it need a Metric View counterpart?
- Should there be an Alert?
- Is it Dashboard-worthy?

### ❌ DON'T: Use DATE parameters in TVFs (Genie incompatible)

```sql
-- BAD
start_date DATE

-- GOOD
start_date STRING COMMENT 'Format: YYYY-MM-DD'
```

---

## References

### Official Documentation
- [Databricks System Tables](https://docs.databricks.com/administration-guide/system-tables/)
- [Databricks SQL Alerts](https://docs.databricks.com/sql/user/alerts/)
- [Lakehouse Monitoring](https://docs.databricks.com/lakehouse-monitoring/)
- [Metric Views](https://docs.databricks.com/metric-views/)
- [Table-Valued Functions](https://docs.databricks.com/sql/language-manual/sql-ref-syntax-ddl-create-sql-function.html)

### Related Cursor Rules
- [15-databricks-table-valued-functions.mdc](mdc:.cursor/rules/semantic-layer/15-databricks-table-valued-functions.mdc)
- [14-metric-views-patterns.mdc](mdc:.cursor/rules/semantic-layer/14-metric-views-patterns.mdc)
- [17-lakehouse-monitoring-comprehensive.mdc](mdc:.cursor/rules/monitoring/17-lakehouse-monitoring-comprehensive.mdc)
- [18-databricks-aibi-dashboards.mdc](mdc:.cursor/rules/monitoring/18-databricks-aibi-dashboards.mdc)
- [16-genie-space-patterns.mdc](mdc:.cursor/rules/semantic-layer/16-genie-space-patterns.mdc) - **Genie Space setup for agents**

### Agent Framework Documentation
- [Phase 4 - Agent Framework](../plans/phase4-agent-framework.md) - Multi-agent architecture
- [Genie Spaces Deployment Guide](../docs/deployment/GENIE_SPACES_DEPLOYMENT_GUIDE.md) - Agent integration guide

### Project Examples
- [plans/README.md](../plans/README.md) - Index of all plans
- [plans/phase1-use-cases.md](../plans/phase1-use-cases.md) - Master Phase 1 document
- [plans/phase1-addendum-1.7-alerting-framework.md](../plans/phase1-addendum-1.7-alerting-framework.md) - Complete alerting example

---

## Rule Improvement History

**Created:** December 2025  
**Trigger:** Comprehensive plan creation for Databricks Health Monitor project  
**Updated:** December 2025 - Restructured phases to start from Use Cases (Bronze/Silver/Gold as prerequisites)  
**Updated:** December 30, 2025 - Added Agent Layer Architecture pattern (Agents → Genie Spaces → Data Assets)

**Patterns Documented:** 
- 3-phase project structure (Use Cases → Agents → Frontend)
- Prerequisites section for completed data layers
- Agent domain framework (5 domains)
- 7 Phase 1 addendums
- 100+ artifacts planned
- Dashboard pattern extraction methodology
- User requirement integration process
- **NEW:** Agent Layer Architecture (Agents use Genie Spaces as query interface)
- **NEW:** Genie Space → Agent mapping (1:1 correspondence)
- **NEW:** Multi-agent workflow coordination via Orchestrator
- **NEW:** Three-level testing strategy (Genie → Agent → Multi-Agent)
- **NEW:** Deployment order requirements (Genie Spaces before Agents)

**Key Learnings:**
1. Agent Domain framework provides consistent organization across all artifacts
2. Gold layer references (not system tables) ensure consistency
3. User requirements often span multiple addendums - update all
4. Dashboard JSON files are rich sources of SQL patterns
5. LLM-friendly comments are critical for Genie/AI/BI integration
6. Summary tables help maintain accuracy across large plans
7. Planning starts after data layers are complete - focus on consumption artifacts
8. **NEW:** Agents should NOT write SQL directly - use Genie Spaces as abstraction
9. **NEW:** Genie Spaces provide natural language understanding that agents leverage
10. **NEW:** Each specialized agent has a dedicated Genie Space (1:1 mapping)
11. **NEW:** Orchestrator agent uses Unified Genie Space for intent classification
12. **NEW:** Genie Spaces must be deployed BEFORE agents can be developed
13. **NEW:** Multi-agent workflows require correlation and synthesis in Orchestrator
14. **NEW:** Three-level testing ensures each layer works before next is built
15. **NEW:** General Instructions in Genie Spaces become agent system prompts

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/prashsub) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
