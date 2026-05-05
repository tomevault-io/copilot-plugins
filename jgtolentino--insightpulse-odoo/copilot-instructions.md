## insightpulse-odoo

> - **ERP**: Odoo 18 CE + OCA (NOT Enterprise, NOT Odoo 19)

# Cursor AI Rules – InsightPulse Odoo Monorepo

## Project Context
- **ERP**: Odoo 18 CE + OCA (NOT Enterprise, NOT Odoo 19)
- **Target**: Enterprise Finance Shared Service Center (Philippines)
- **Compliance**: BIR tax forms (1601-C, 2550Q, 1702-RT), immutable accounting
- **Multi-tenant**: Per-legal-entity isolation (company_id), NOT department routing
- **Cost Objective**: Replace $60K+/year in SaaS with <$300/year self-hosted

## Mandatory Rules

### 1. Odoo Version (CRITICAL)
- ✅ Use **Odoo 18 CE** APIs and documentation ONLY
- ❌ NEVER reference Odoo 19 or Enterprise features
- ✅ Link to: https://www.odoo.com/documentation/18.0/
- ✅ OCA modules: Use 18.0 branches from github.com/OCA

### 2. BIR Compliance (Philippines)
- ✅ Immutable accounting: Use reversals, NOT in-place edits
- ✅ Audit trail: All state changes logged via chatter
- ✅ Forms: 2307, 2316, 1601-C, 2550Q, 1702-RT
- ❌ NEVER allow direct modification of posted journal entries

### 3. Multi-Tenancy
- ✅ Use `company_id = fields.Many2one('res.company', required=True)`
- ✅ Separate legal entities = separate DB or strict company isolation
- ❌ NEVER use agency_id or department_id for tenancy
- ✅ Add RLS rules: `@api.depends('company_id')`

### 4. Code Standards (OCA-aligned)
- ✅ License: AGPL-3 for all custom modules
- ✅ Python: Type hints, docstrings, no raw SQL with user input
- ✅ XML: Explicit comments for XPaths, proper state transitions
- ✅ Tests: Unit tests + HttpCase for all new modules (>80% coverage)
- ✅ Manifests: Correct `depends`, semantic versioning

### 5. Security
- ✅ Secrets: Use env vars or `ir.config_parameter`, NEVER hardcode
- ✅ Controllers: Add `x-atomic` and `x-role-scopes` in OpenAPI
- ✅ Authentication: OAuth via `/auth_oauth/signin` (Odoo standard)
- ❌ NEVER commit credentials, API keys, or database URLs

### 6. Module Structure
```
addons/custom/<module_name>/
├── __init__.py
├── __manifest__.py          # AGPL-3, correct depends
├── models/
│   ├── __init__.py
│   └── <model_name>.py      # Inherit with _inherit or create with _name
├── views/
│   └── <model_name>_views.xml
├── security/
│   ├── ir.model.access.csv
│   └── <module_name>_security.xml
├── tests/
│   ├── __init__.py
│   └── test_<model_name>.py  # HttpCase or TransactionCase
└── README.md
```

### 7. Deployment
- ✅ CI/CD: GitHub Actions handles deployments, NOT manual SCP
- ✅ Rollback: Git tags + module upgrade with `--stop-after-init`
- ✅ Health checks: Prometheus + Grafana + Superset
- ❌ NEVER push directly to production without staging validation

### 8. SaaS Replacement Strategy
| SaaS Product | Replacement | Annual Savings |
|--------------|-------------|----------------|
| SAP Concur | Odoo Expense | $15,000 |
| SAP Ariba | Odoo Procurement | $12,000 |
| Tableau | Apache Superset | $8,400 |
| Odoo Enterprise | Odoo CE + OCA | $4,728 |

## Development Workflow

### Before Writing Code
1. Check `docs/PRD_ENTERPRISE_SAAS_PARITY.md` for epic scope
2. Check `docs/ROADMAP.md` for current wave/sprint
3. Review `claude.md` sections 0-10 for constraints
4. Validate OCA module exists before reinventing

### Writing Code
1. Use `_inherit` for extending existing models
2. Use `_name` for new models
3. Add `mail.thread` for audit trail
4. Add tests (unit + integration)
5. Document in module README.md

### After Writing Code
1. Run tests: `pytest odoo/tests -q`
2. Run linters: `black . && flake8 . && pylint addons/`
3. Update CHANGELOG.md
4. Create PR with epic reference

## Evals (Must Pass)
1. **OpenAPI contract**: All new controllers have `x-atomic` + `x-role-scopes`
2. **OAuth block present**: `/web/login` renders Google OAuth buttons
3. **Expense intake idempotency**: Duplicate POST returns `idempotent: true`
4. **OCR health**: `GET /health` returns `{ok: true}`
5. **Warehouse view**: `vw_expense_fact` exists and accessible

## Common Pitfalls (Avoid)
- ❌ Using Odoo 19 or Enterprise APIs
- ❌ Mutable finance records (edit posted entries)
- ❌ Hardcoding secrets/URLs
- ❌ Missing tests
- ❌ Gating by agency instead of company
- ❌ Manual deployments via SCP

## High-Leverage Questions (Ask When Unsure)
1. "Is this per-company or cross-company?"
2. "What BIR artifact is affected?"
3. "Which OCA module covers 80% of this?"
4. "Does this need reversals or can we edit directly?" (Answer: reversals)

## References
- Odoo 18.0 Docs: https://www.odoo.com/documentation/18.0/
- OCA GitHub: https://github.com/OCA
- BIR Forms: https://www.bir.gov.ph/
- Apache Superset: https://superset.apache.org/

## Repository Structure
```
insightpulse-odoo/
├── addons/              # Custom Odoo 18 CE modules
├── docs/                # Architecture, PRD, guides
├── .claude/             # Claude Code config (skills, commands)
├── claude.md            # AI assistant contract (canonical)
├── .cursorrules         # This file (Cursor AI rules)
├── ROADMAP.md           # Product roadmap
├── PRD_ENTERPRISE_SAAS_PARITY.md  # Requirements
├── CHANGELOG.md         # Version history
├── TASKS.md             # Current sprint tasks
└── PLANNING.md          # Sprint/milestone planning
```

## Success Metrics
- Performance: <200ms CRUD, <3s dashboards (p95)
- Quality: >80% test coverage
- Compliance: BIR docs generatable, immutable posted moves
- Uptime: >99.9%, error rate trending down

---

**Version**: cursor-rules@2025-11-08
**Odoo Target**: 18 CE (consistent, no mixing)
**License**: AGPL-3.0
**Maintainer**: InsightPulse AI Team

---
> Source: [jgtolentino/insightpulse-odoo](https://github.com/jgtolentino/insightpulse-odoo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
