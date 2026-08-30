## stratusscan-cli-azure

> A Python CLI tool that exports Azure resource inventories to Excel workbooks.

# StratusScanCLI-Azure — Agent Guide

## What This Is

A Python CLI tool that exports Azure resource inventories to Excel workbooks.
Sibling to `stratusscan-cli` (AWS). No shared runtime code — shared philosophy and patterns.

Primary use cases: infrastructure audits, FedRAMP evidence collection, cost optimization.

---

## Critical Design Constraints

Read these before touching any code. They are non-negotiable.

### 1. CloudShell-first
Must work in a fresh Azure Cloud Shell session with no extra setup beyond `pip install`.
If a proposed change breaks this, reject it.

### 2. subprocess architecture
`stratusscan.py` launches exporters as subprocesses. It never calls Azure APIs directly.
Exporters in `scripts/` are fully independent — they can also run directly.

### 3. No print() in utils.py
`utils.py` is a shared library. It must not emit console output. Return structured data.
Only the CLI scripts (stratusscan.py, configure.py, exporter scripts) print to console.

### 4. CI mode
`STRATUSSCAN_AUTO_RUN=1` bypasses all interactive prompts.
`STRATUSSCAN_SUBSCRIPTIONS=sub-id-1,sub-id-2` sets subscriptions non-interactively.
Every interactive prompt must check `utils.is_auto_run()` before displaying.

### 5. TUI-ready design
The North Star is a Textual TUI (v0.3.0+). Design as if something will consume exporter output.
No arbitrary prints in shared code.

---

## Project Structure

```
stratusscan.py              # main menu — launches exporters as subprocesses
configure.py              # subscription/environment config wizard
utils.py                  # shared library — imported by every exporter
scripts/                  # all exporters — flat directory, no subdirs
  subscriptions_export.py
  resource_groups_export.py
  virtual_machines_export.py
  managed_disks_export.py
  aks_clusters_export.py
  app_service_export.py
  function_apps_export.py
  virtual_networks_export.py
  subnets_export.py
  network_security_groups_export.py
  public_ips_export.py
  load_balancers_export.py
  application_gateway_export.py
  azure_firewall_export.py
  firewall_policy_rules_export.py
  route_tables_export.py
  vnet_peerings_export.py
  storage_accounts_export.py
  azure_sql_export.py
  cosmos_db_export.py
  key_vault_export.py
  role_assignments_export.py
  policy_assignments_export.py
  management_groups_export.py
  defender_scores_export.py
  defender_assessments_export.py
  advisor_export.py
  metric_alerts_export.py
  action_groups_export.py
  log_analytics_export.py
output/                   # all .xlsx exports land here
logs/                     # per-run log files (14-day retention)
policies/                 # Azure RBAC read-only role definitions
tests/
pyproject.toml
```

---

## Authentication

`DefaultAzureCredential` from `azure-identity`. Auto-detects: Cloud Shell → Azure CLI → Service Principal → Managed Identity.

Never instantiate `azure-mgmt-*` clients directly. Always use `utils.get_azure_client(service_name, subscription_id)`.

For government cloud: set `AZURE_ENVIRONMENT=AzureUSGovernment` or `environment: government` in `config.json`.

---

## Exporter Pattern

Every script in `scripts/` must follow this structure exactly:

```python
try:
    import utils
except ImportError:
    sys.path.append(str(Path(__file__).parent.parent))
    import utils

utils.setup_logging("my-service-export")

environment = utils.detect_environment()
if not utils.is_service_available_in_environment("myservice", environment):
    sys.exit(0)

client = utils.get_azure_client("myservice", subscription_id)
resources = list(client.resource_type.list())

filename = utils.create_export_filename(subscription_name, "my-service", "all")
utils.save_dataframe_to_excel(df, filename)
```

Azure SDK `.list()` methods return lazy iterators — wrap in `list()` to materialize.

---

## Key utils.py Functions

| Function | Purpose |
|---|---|
| `get_azure_client(service, sub_id)` | Client factory — never call mgmt clients directly |
| `detect_environment()` | Returns `'public'` or `'government'` |
| `is_service_available_in_environment(svc, env)` | Guard for gov-restricted services |
| `setup_logging(script_name)` | Call once at script start |
| `get_current_timestamp()` | Returns `MM.DD.YYYY` |
| `create_export_filename(sub_name, resource_type, suffix)` | Builds output path |
| `save_dataframe_to_excel(df, filename)` | Single-sheet write + column autofit |
| `save_multiple_dataframes_to_excel(sheets, filename)` | Multi-sheet write |
| `get_config()` | Thread-safe config.json singleton |
| `is_auto_run()` | CI mode check |
| `get_auto_subscriptions()` | Reads STRATUSSCAN_SUBSCRIPTIONS env var |
| `list_subscriptions()` | Lists all accessible subscriptions |

---

## Output Format

Filename: `{SUBSCRIPTION-NAME}-{resource-type}-{suffix}-export-{MM.DD.YYYY}.xlsx`
All files land in `output/`.

---

## Conventions

- Script filenames: `lowercase_underscored.py`
- Timestamps: `MM.DD.YYYY` via `utils.get_current_timestamp()`
- No comments on what code does — name things well instead
- Output dir: always via `utils.create_export_filename()`, never hardcoded
- Subscription name in filenames: always from config/arg, never hardcoded

---

## Version Roadmap

| Version | Scope |
|---|---|
| v0.1.0 (current) | 21 exporters, Public + Government environments, subscription selector |
| v0.2.0 | Resource Graph Smart Scan, Entra ID exporters (Graph SDK), pricing data |
| v0.3.0+ | Multi-tenant scanning, Textual TUI |

---

## Explicitly Out of Scope for v0.1.0

- Identity/Entra ID exporters (requires Microsoft Graph SDK — separate integration)
- Resource Graph / Smart Scan engine
- Pricing data / cost columns
- Multi-tenant scanning
- Textual TUI

---
> Source: [ColonelPanicX/stratusscan-cli-azure](https://github.com/ColonelPanicX/stratusscan-cli-azure) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
