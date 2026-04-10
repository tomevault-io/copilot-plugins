## sampoorna-sangathan

> 1. This is an ODOO project - all modules are Python Odoo addons, NOT Next.js/React

# AGENTS.md - SampoornaSangathan Odoo Custom Addons Guide

## Project: SampoornaSangathan (Unified Organisation OS)
## Platform: Odoo 17 Community Edition
## Working Directory: ~/odoo/custom-addons/
## Database: sampoorna-sangathan
## Odoo URL: http://localhost:8069

---

## Sacred Principles
1. This is an ODOO project - all modules are Python Odoo addons, NOT Next.js/React
2. ONE session = ONE Odoo module in ~/odoo/custom-addons/{module_name}/
3. NEVER modify Odoo core files - only custom-addons/
4. TEST via Odoo shell and module upgrade before committing
5. Each module MUST have proper __manifest__.py with dependencies
6. ROLLBACK if module install fails - never leave DB in broken state

---

## Environment Setup
```
Odoo Path: /home/eechie/odoo/community
Venv: /home/eechie/odoo/venv/bin/python
Config: /home/eechie/odoo/cfg/odoo.conf
Custom Addons: ~/odoo/custom-addons/
DB User: eechie
DB Name: sampoorna-sangathan
```

## Useful Commands
```bash
# Restart Odoo with module update
cd /home/eechie/odoo/community && /home/eechie/odoo/venv/bin/python odoo-bin -d sampoorna-sangathan -c /home/eechie/odoo/cfg/odoo.conf -u {module_name} --stop-after-init

# Install new module
cd /home/eechie/odoo/community && /home/eechie/odoo/venv/bin/python odoo-bin -d sampoorna-sangathan -c /home/eechie/odoo/cfg/odoo.conf -i {module_name} --stop-after-init

# Odoo Shell
cd /home/eechie/odoo/community && /home/eechie/odoo/venv/bin/python odoo-bin shell -d sampoorna-sangathan -c /home/eechie/odoo/cfg/odoo.conf --no-http

# Check module state
psql -U eechie -d "sampoorna-sangathan" -c "SELECT name, state FROM ir_module_module WHERE name LIKE 'sampoorna%' OR name LIKE 'jana%' OR name LIKE 'sambandha%' OR name LIKE 'kosha%' OR name LIKE 'bhandar%' OR name LIKE 'pariyojana%' OR name LIKE 'sutradhara%'"
```

---

## Module Map (22 Total - Phase-wise)

### Phase 1 (Core + 3 modules)
| Module | Directory | Depends On | Description |
|--------|-----------|------------|-------------|
| sampoorna_sangathan_core | sampoorna_sangathan_core/ | base | Branding, dashboard, shared utilities |
| jana_seva_hrms | jana_seva_hrms/ | hr, hr_holidays, hr_attendance, hr_expense | Extended HRMS |
| sambandha_path_crm | sambandha_path_crm/ | crm, sale, sale_management | Extended CRM |
| pariyojana_chakra | pariyojana_chakra/ | project, hr_timesheet | Project Management |

### Phase 2 (Finance + Inventory)
| Module | Directory | Depends On | Description |
|--------|-----------|------------|-------------|
| kosha_prabandhan | kosha_prabandhan/ | account, account_payment | Finance & Accounting |
| bhandar_griha | bhandar_griha/ | stock, purchase | Inventory & Warehouse |
| lekha_peethika | lekha_peethika/ | mail, calendar, contacts | Office Suite |

### Phase 3 (Advanced)
| Module | Directory | Depends On | Description |
|--------|-----------|------------|-------------|
| sutradhara_api | sutradhara_api/ | base | REST API endpoints |
| vidya_kendra | vidya_kendra/ | website, website_slides | eLearning/Knowledge Base |
| niti_rakshak | niti_rakshak/ | base | Compliance & Audit |

---

## Odoo Module Structure Template
```
{module_name}/
  __init__.py
  __manifest__.py
  models/
    __init__.py
    {model_name}.py
  views/
    {model_name}_views.xml
    menu_views.xml
  security/
    ir.model.access.csv
    security_groups.xml
  data/
    {module}_data.xml
  static/
    description/
      icon.png
    src/
      js/
      css/
      xml/
  wizard/
    __init__.py
  reports/
```

## __manifest__.py Template
```python
{
    'name': 'Module Display Name',
    'version': '17.0.1.0.0',
    'category': 'SampoornaSangathan',
    'summary': 'Brief description',
    'description': 'Detailed description',
    'author': 'SampoornaSangathan',
    'website': 'https://sampoorna-sangathan.vercel.app',
    'license': 'LGPL-3',
    'depends': ['base'],
    'data': [
        'security/ir.model.access.csv',
        'views/menu_views.xml',
    ],
    'installable': True,
    'application': True,
    'auto_install': False,
}
```

---

## Parallel Session Rules
- Each Claude Code session works on EXACTLY ONE module directory
- Sessions must NOT touch files outside their assigned module
- Core module (sampoorna_sangathan_core) must be installed FIRST
- Check dependency modules are installed before testing
- Use `--stop-after-init` flag to avoid port conflicts with running Odoo
- Git: commit to feature branch `feat/{module_name}`, merge to main after QA

## QA Checklist Per Module
1. `__manifest__.py` has correct depends list
2. `ir.model.access.csv` grants proper permissions
3. Module installs without error: `-i {module} --stop-after-init`
4. Module upgrades without error: `-u {module} --stop-after-init`
5. Views load in browser at localhost:8069
6. Menu items appear under SampoornaSangathan category
7. No Python import errors in Odoo logs
8. Security groups properly restrict access


---

## DUPLICATE PREVENTION & SESSION TRACKING

### How to Check if a Module is Already Built
BEFORE starting any session, Claude Code MUST run:
```bash
# Check if module directory already exists
ls -la ~/odoo/custom-addons/{module_name}/

# Check if module is installed in Odoo DB
psql -U eechie -d "sampoorna-sangathan" -c "SELECT name, state FROM ir_module_module WHERE name = '{module_name}'"

# Check __manifest__.py exists
cat ~/odoo/custom-addons/{module_name}/__manifest__.py 2>/dev/null
```

### Duplicate Prompt Rules
- If directory EXISTS and has __manifest__.py -> DO NOT recreate, ask user what to enhance/fix
- If directory EXISTS but module NOT installed -> fix and install
- If directory DOES NOT exist -> create from scratch
- NEVER overwrite existing working code without explicit user confirmation

### Session Status File (auto-updated)
Each Claude Code session MUST create/update a status file:
```bash
# At session START:
echo "STARTED $(date) by Session-{N}" > ~/odoo/custom-addons/{module_name}/.session_status

# At session END (success):
echo "COMPLETED $(date) - Module installed successfully" > ~/odoo/custom-addons/{module_name}/.session_status

# At session END (failure):
echo "FAILED $(date) - Error: {description}" > ~/odoo/custom-addons/{module_name}/.session_status
```

### Quick Status Check (run anytime)
```bash
for dir in ~/odoo/custom-addons/*/; do
  module=$(basename $dir)
  if [ -f "$dir/.session_status" ]; then
    echo "$module: $(cat $dir/.session_status)"
  elif [ -f "$dir/__manifest__.py" ]; then
    echo "$module: EXISTS (no status file)"
  else
    echo "$module: NOT STARTED"
  fi
done
```

---

## INSPIRATION REPOS - Leverage Existing Code

Claude Code MUST review these repos for patterns, models, and logic BEFORE building each module:

### Core & Dashboard
- `~/repos/antaryami-os/` or https://github.com/ganeshgowri-ASA/antaryami-os -> Enterprise Org AI OS architecture
- `~/repos/karmadhara/` or https://github.com/ganeshgowri-ASA/karmadhara -> Enterprise Portal, Workflow Authorization (SAP Fiori style)
- `~/repos/Karyadwar/` or https://github.com/ganeshgowri-ASA/Karyadwar -> Manufacturing Intranet Portal patterns

### HRMS (jana_seva_hrms)
- `~/repos/SarvePratibha/` or https://github.com/ganeshgowri-ASA/SarvePratibha -> Enterprise HRMS with HR, Payroll, Recruitment, Performance
- Extract: Employee models, leave management logic, attendance patterns, payroll calculations

### CRM (sambandha_path_crm)
- `~/repos/Krayavikrayam/` or https://github.com/ganeshgowri-ASA/Krayavikrayam -> AI-native CRM/ERP SaaS with Pipeline, Chatbots
- Extract: Lead scoring, pipeline stages, customer 360 view, email templates

### Office Suite (lekha_peethika)
- `~/repos/vidyalaya-office/` or https://github.com/ganeshgowri-ASA/vidyalaya-office -> AI-Native Office Suite (Docs, Sheets, Presentations)
- `~/repos/sahaayam/` or https://github.com/ganeshgowri-ASA/sahaayam -> IT Self-Service Portal
- Extract: Document models, collaboration patterns, template system

### Access & Security
- `~/repos/adhikarpath/` or https://github.com/ganeshgowri-ASA/adhikarpath -> Enterprise Access Management Portal
- Extract: Role-based access patterns, permission models

### Analytics Dashboards
- `~/repos/PhotonIQ/` or https://github.com/ganeshgowri-ASA/PhotonIQ -> Quality Dashboard, Analytics patterns
- `~/repos/SolarLabX/` or https://github.com/ganeshgowri-ASA/SolarLabX -> LIMS, QMS, Audit patterns
- Extract: Dashboard widget patterns, chart configurations, KPI calculation logic

### HOW TO USE Inspiration Repos
```bash
# Clone all inspiration repos locally (one-time setup)
mkdir -p ~/repos && cd ~/repos
git clone https://github.com/ganeshgowri-ASA/SarvePratibha.git
git clone https://github.com/ganeshgowri-ASA/Krayavikrayam.git
git clone https://github.com/ganeshgowri-ASA/vidyalaya-office.git
git clone https://github.com/ganeshgowri-ASA/karmadhara.git
git clone https://github.com/ganeshgowri-ASA/antaryami-os.git
git clone https://github.com/ganeshgowri-ASA/adhikarpath.git
git clone https://github.com/ganeshgowri-ASA/sahaayam.git
git clone https://github.com/ganeshgowri-ASA/PhotonIQ.git
git clone https://github.com/ganeshgowri-ASA/SolarLabX.git
git clone https://github.com/ganeshgowri-ASA/Karyadwar.git
```

For each module session, Claude Code should:
1. Read the mapped inspiration repo's models/schemas
2. Extract field definitions, business logic, validation rules
3. Translate from Next.js/Prisma/Express patterns to Odoo Python ORM
4. Preserve feature completeness while adapting to Odoo conventions
5. Add Odoo-specific features (workflows, security rules, reports) on top

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ganeshgowri-ASA)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ganeshgowri-ASA)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
