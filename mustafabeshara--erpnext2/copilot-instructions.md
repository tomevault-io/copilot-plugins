## erpnext2

> This is a medical device distribution ERP system built on ERPNext v15.

# BesharaMed ERPNext Project Rules

## Project Context
This is a medical device distribution ERP system built on ERPNext v15.
- **Company**: BesharaMed (Kuwait)
- **Industry**: Medical Device Distribution
- **Currency**: KWD (Kuwaiti Dinar)
- **Server**: 192.168.68.50:8080 (Docker on Windows)

## Code Style Rules

### Python (Frappe/ERPNext)
- Use snake_case for functions and variables
- Use PascalCase for class names
- Always check if doctype/document exists before creating: `if not frappe.db.exists(...)`
- Commit after batch operations: `frappe.db.commit()`
- Use `frappe.get_doc()` for creating documents
- Handle exceptions gracefully with try/except

### ERPNext Specific
- Custom fields use `custom_` prefix
- Workflow states need corresponding Workflow State documents
- Workflow actions need Workflow Action Master documents
- doc_status: 0=Draft, 1=Submitted, 2=Cancelled
- Submitted documents cannot transition back to draft states

### Deployment
- Scripts go in `/DEPLOYMENT/` folder
- Use `bench --site frontend execute module.function` pattern
- Clear cache after deployments: `bench --site frontend clear-cache`
- Docker container: `erpnext-backend-1`

## Approval Thresholds
- Purchase Orders > 5,000 KWD: GM approval required
- Payments > 5,000 KWD: Finance Manager approval
- Journal Entries > 10,000 KWD: Finance Manager approval
- Low-margin quotations (<15%): GM approval

## Medical Device Requirements
- All items require batch tracking (has_batch_no=1)
- Most items require expiry tracking (has_expiry_date=1)
- FEFO (First Expiry First Out) enforcement
- Mandatory QC inspection for incoming goods
- Quarantine workflow for failed QC

## File Structure
```
ERPNEXT Management/
├── DEPLOYMENT/           # Deployment scripts
│   ├── comprehensive_workflows.py
│   ├── import_master_data.py
│   └── deploy_*.sh
├── TRAINING/             # User training documents
└── 02_MASTER_DATA_TEMPLATES/  # CSV templates
```

## SSH Access
- Host: 192.168.68.50
- User: musta
- Use sshpass for automation

## Common Commands
```bash
# Execute script
bench --site frontend execute frappe.module_name.function_name

# Clear cache
bench --site frontend clear-cache

# MariaDB query
bench --site frontend mariadb -e "SQL QUERY"

# Restart workers
bench --site frontend restart
```

## Workflows Active (21 total)
Sales: Lead Qualification, Quotation Approval, Sales Order Fulfillment, Tender Approval
Procurement: Material Request Approval, Purchase Order Approval, Supplier Quotation Evaluation
Inventory: Purchase Receipt QC, Delivery Note Dispatch, Stock Entry Approval, Quality Inspection
Finance: Sales Invoice Collection, Purchase Invoice Processing, Payment Entry, Journal Entry
Projects: Task Management, Project Lifecycle
Operations: RMA Request Process, FEFO Override Approval

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Mustafabeshara) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
