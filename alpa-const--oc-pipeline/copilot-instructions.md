## oc-pipeline

> Quality module - Deficiencies, inspections, and punch lists


# MODULE: QUALITY MANAGEMENT

## 📋 OVERVIEW

The Quality module tracks quality control activities, deficiencies, and punch lists.

**Key Entities:** Deficiencies, Inspections, Punch Lists, NCRs

---

## 🗄️ DATABASE TABLES

| Table | Purpose |
|-------|---------|
| `deficiencies` | Quality issues |
| `inspections` | QC inspections |
| `inspection_items` | Checklist items |
| `punch_lists` | Punch sets |
| `punch_items` | Individual items |
| `ncrs` | Non-conformance |
| `quality_plans` | QC plans |
| `test_reports` | Material tests |

---

## 🔗 API ENDPOINTS

```
GET    /quality/deficiencies
POST   /quality/deficiencies
GET    /quality/inspections
POST   /quality/inspections
GET    /quality/punch-lists
```

---

## 🤖 ATLAS AGENT

**Agent ID:** `quality-001`

**Capabilities:**
- Deficiency pattern detection
- Inspection scheduling
- Quality trend analysis

$END$

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ALPA-Const) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
