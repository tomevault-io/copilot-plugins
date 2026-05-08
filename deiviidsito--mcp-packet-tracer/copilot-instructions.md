## mcp-packet-tracer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Packet Tracer MCP Server** — a Model Context Protocol server that enables LLMs to create, configure, validate, and **deploy in real-time** network topologies to Cisco Packet Tracer. The server provides **35 MCP tools** and 5 MCP resources, including live deployment via HTTP bridge, topology intelligence, config templates, and scenario presets.

**Tech Stack:** Python 3.11+, Pydantic 2.0+, MCP (FastMCP), Jinja2, Streamable HTTP
**Transport:** `http://127.0.0.1:39000/mcp` (streamable-http) | `--stdio` for legacy
**Version:** 0.5.0

## Common Commands

### Run the MCP Server
```bash
# Streamable HTTP on :39000 (default)
python -m src.packet_tracer_mcp

# Stdio transport (debug/legacy clients)
python -m src.packet_tracer_mcp --stdio

# Custom port via env var
PT_MCP_PORT=8080 python -m src.packet_tracer_mcp
```

### Install/Reinstall
```bash
# Production
pip install -e .

# Development (includes pytest, ruff, mypy)
pip install -e ".[dev]"
```

### Run Tests
```bash
# All tests (129 tests)
python -m pytest tests/ -v

# Single test file
python -m pytest tests/test_full_build.py -v

# Specific test
python -m pytest tests/test_full_build.py::TestFullBuild::test_basic_2_routers -v

# With coverage
python -m pytest tests/ --cov=src/packet_tracer_mcp --cov-report=term-missing
```

### Lint & Format
```bash
# Lint
ruff check src/

# Auto-fix
ruff check src/ --fix

# Format
ruff format src/

# Type check
mypy src/
```

### Configuration

**Claude Code** (`.mcp.json` in project root):
```json
{
  "mcpServers": {
    "packet-tracer": {
      "type": "http",
      "url": "http://127.0.0.1:39000/mcp"
    }
  }
}
```

**VS Code** (`.vscode/mcp.json`):
```json
{
  "servers": {
    "packet-tracer": {
      "url": "http://127.0.0.1:39000/mcp"
    }
  }
}
```

**Claude Desktop** (`claude_desktop_config.json`):
```json
{
  "mcpServers": {
    "packet-tracer": {
      "url": "http://127.0.0.1:39000/mcp"
    }
  }
}
```

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `PT_MCP_PORT` | `39000` | Server HTTP port |
| `PT_MCP_LOG_LEVEL` | `INFO` | Logging level (DEBUG, INFO, WARNING, ERROR) |

## Architecture

This project follows **Clean Architecture / Domain-Driven Design** with clear layer separation:

```
src/packet_tracer_mcp/
├── adapters/mcp/          # MCP protocol layer
│   ├── tools/             # Split by domain concern (9 modules)
│   │   ├── catalog_tools.py       # pt_list_devices, pt_list_templates, pt_get_device_details
│   │   ├── planning_tools.py      # pt_estimate_plan, pt_plan_topology
│   │   ├── validation_tools.py    # pt_validate_plan, pt_fix_plan, pt_explain_plan,
│   │   │                          # pt_validate_config, pt_validate_topology
│   │   ├── generation_tools.py    # pt_generate_script, pt_generate_configs, pt_full_build
│   │   ├── deploy_tools.py        # pt_export, pt_deploy, pt_list_projects, pt_load_project,
│   │   │                          # pt_export_documentation
│   │   ├── bridge_tools.py        # pt_live_deploy, pt_bridge_status, pt_ping_bridge,
│   │   │                          # pt_undo_last_action, pt_load_last_plan,
│   │   │                          # pt_query/delete/rename/move/send_raw
│   │   ├── topology_tools.py      # pt_analyze_topology, pt_suggest_improvements,
│   │   │                          # pt_calculate_addressing
│   │   ├── preset_tools.py        # pt_list_presets, pt_load_preset
│   │   ├── template_tools.py      # pt_list_config_templates, pt_apply_template
│   │   └── _bridge_helpers.py     # HTTP helpers, bridge singleton, retry, persistence
│   ├── tool_registry.py  # Thin coordinator (delegates to tools/)
│   └── resource_registry.py
├── application/           # Use cases + DTOs (requests/responses)
├── domain/                # Core business logic
│   ├── models/           # TopologyPlan, DevicePlan, LinkPlan, errors, TopologyAnalysis
│   ├── services/         # Orchestrator, IPPlanner, Validator, AutoFixer, Explainer,
│   │                     # Estimator, TopologyAnalyzer, TemplateEngine, Presets
│   └── rules/            # Validation rules (devices, cables, IPs)
├── infrastructure/        # External concerns
│   ├── catalog/          # Device catalog, cable types, templates, aliases
│   ├── generator/        # PTBuilder (JS) + CLI config generators
│   ├── execution/        # HTTP bridge, live executor, deploy, manual export
│   └── persistence/      # Project repository (save/load)
├── shared/               # Enums, constants, utilities, logging
│   └── templates/        # 8 Jinja2 IOS config templates
├── server.py             # MCP server entry point
├── settings.py           # Server config (v0.5.0)
└── __main__.py           # python -m module entry
```

### Key Data Flow

1. **Request** → `TopologyRequest` (domain/models/requests.py)
2. **Orchestration** → `plan_from_request()` (domain/services/orchestrator.py)
3. **Validation** → `validate_plan()` (domain/services/validator.py)
4. **Generation** → PTBuilder script + CLI configs (infrastructure/generator/)
5. **Deploy** → Live via HTTP bridge (infrastructure/execution/live_executor.py) OR export to files

### Live Deploy Architecture

Python HTTP bridge (`live_bridge.py`) on `127.0.0.1:54321` ↔ PTBuilder QWebEngine webview polls `GET /next` every 500ms ↔ `$se('runCode', cmd)` executes in PT Script Engine.

- **PTCommandBridge**: HTTP server with GET /next, GET /ping, POST /result, POST /queue
- **LiveExecutor**: Converts TopologyPlan → executable JS commands → sends via bridge
- **Bootstrap**: One-liner JS pasted in Builder Code Editor starts the polling loop
- **Retry logic**: Exponential backoff (3 retries, 0.5s base delay) on bridge HTTP calls
- **Plan persistence**: Last deployed plan saved to `projects/.last_plan.json` for recovery
- **Undo support**: Command history with pop-based undo for addDevice commands

### Core Domain Models

- **TopologyRequest**: Input parameters (routers, pcs_per_lan, has_wan, routing, etc.)
- **TopologyPlan**: Complete validated plan with devices, links, IPs, DHCP pools, routes
- **DevicePlan**: Device with name, model, category, coordinates, interfaces
- **LinkPlan**: Connection between two devices with ports and cable type
- **TopologyAnalysis**: NLP-parsed topology with sites, suggested models, subnets
- **AddressingPlan**: IPv4/IPv6 dual-stack addressing with VLANs and loopbacks

### Device Catalog

Located in `infrastructure/catalog/`. Contains 11 device models (routers: 1941, 2901, 2911, ISR4321; switches: 2960-24TT, 3560-24PS; PC-PT, Server-PT, Laptop-PT, Cloud-PT, AccessPoint-PT) with verified port definitions. **No router has serial ports by default** — serial requires HWIC modules.

## MCP Tools (35)

**Catalog:** `pt_list_devices`, `pt_list_templates`, `pt_get_device_details`
**Estimation:** `pt_estimate_plan` (dry-run)
**Planning:** `pt_plan_topology`
**Validation:** `pt_validate_plan`, `pt_fix_plan`, `pt_explain_plan`, `pt_validate_config`, `pt_validate_topology`
**Generation:** `pt_generate_script`, `pt_generate_configs`
**Pipeline:** `pt_full_build` (complete workflow)
**Deploy:** `pt_deploy` (clipboard), `pt_live_deploy` (real-time HTTP bridge), `pt_bridge_status`, `pt_ping_bridge`
**Recovery:** `pt_undo_last_action`, `pt_load_last_plan`
**Topology interaction:** `pt_query_topology`, `pt_delete_device`, `pt_rename_device`, `pt_move_device`, `pt_delete_link`, `pt_send_raw`
**Intelligence:** `pt_analyze_topology`, `pt_suggest_improvements`, `pt_calculate_addressing`
**Templates:** `pt_list_config_templates`, `pt_apply_template`
**Presets:** `pt_list_presets`, `pt_load_preset`
**Export/Projects:** `pt_export`, `pt_list_projects`, `pt_load_project`, `pt_export_documentation`

## MCP Resources (5)

- `pt://catalog/devices` — All devices with ports
- `pt://catalog/cables` — Cable types
- `pt://catalog/aliases` — Model aliases
- `pt://catalog/templates` — Topology templates
- `pt://capabilities` — Server capabilities

## Live Deploy Setup

Bootstrap script (paste in PT Builder Code Editor and click Run):
```javascript
/* PT-MCP Bridge */ window.webview.evaluateJavaScriptAsync("setInterval(function(){var x=new XMLHttpRequest();x.open('GET','http://127.0.0.1:54321/next',true);x.onload=function(){if(x.status===200&&x.responseText){$se('runCode',x.responseText)}};x.onerror=function(){};x.send()},500)");
```

## Key Services

- **Orchestrator** (`domain/services/orchestrator.py`): Main pipeline, transforms requests into plans
- **IPPlanner** (`domain/services/ip_planner.py`): Assigns IP addresses to LANs (/24) and inter-router links (/30)
- **Validator** (`domain/services/validator.py`): Validates plans with typed error codes
- **AutoFixer** (`domain/services/auto_fixer.py`): Auto-corrects cables, upgrades routers, reassigns ports
- **Explainer** (`domain/services/explainer.py`): Generates natural language explanations
- **Estimator** (`domain/services/estimator.py`): Dry-run estimation without full plan generation
- **TopologyAnalyzer** (`domain/services/topology_analyzer.py`): NLP-lite topology parsing, improvement suggestions, IPv4/IPv6 addressing
- **TemplateEngine** (`domain/services/template_engine.py`): Jinja2-based IOS config template rendering
- **Presets** (`domain/services/presets.py`): 8 scenario presets (small_office, branch_hq, dmz_network, etc.)

## Jinja2 Config Templates (8)

Located in `shared/templates/`. Generate IOS CLI configs for:
- `ospf_basic` — OSPF with areas and passive interfaces
- `eigrp_named` — EIGRP named mode
- `vlan_trunk` — VLAN creation + trunk ports
- `hsrp_pair` — HSRP active/standby
- `nat_overload` — NAT overload (PAT)
- `acl_dmz` — Extended ACL for DMZ
- `dhcp_server` — DHCP pool with exclusions
- `stp_rapid` — Rapid PVST+ with root bridge

## Error Taxonomy

15 error codes with messages, affected devices, and fix suggestions. Categories: device, link, IP, DHCP, routing, template. See `domain/models/errors.py`.

## Testing

Tests are in `tests/`. Currently **129 tests** across 15 test files covering:
- IP planning, validation, auto-fixing, explanation, estimation
- Code generation (PTBuilder JS + CLI configs)
- Full build integration (8 scenarios)
- RIP and EIGRP routing protocols (14 tests)
- Project persistence (save/load/list)
- Resource/catalog data validity (11 tests)
- **Topology intelligence** (analyze, suggest, addressing — 20 tests)
- **Jinja2 config templates** (all 8 templates — 11 tests)
- **Scenario presets** (discovery + loading — 4 tests)
- **Validation upgrades** (config lines, deep topology — 12 tests)
- **Bridge recovery** (command history, plan persistence — 4 tests)
- Runtime regressions

## Supported Routing

- **static**: Complete (generates `ip route` commands, supports floating routes with AD=254)
- **ospf**: Complete (generates `router ospf` configs with router-id and area support)
- **rip**: Complete (generates `router rip` v2 configs with no auto-summary)
- **eigrp**: Complete (generates `router eigrp` configs with wildcard masks and AS number)
- **none**: No routing

## IP Addressing

- **LANs**: `192.168.0.0/16` base, /24 prefixes (254 hosts per LAN)
- **Inter-router links**: `10.0.0.0/16` base, /30 prefixes (2 hosts per link)
- Gateway is always `.1`, PCs get sequential IPs from `.2`
- **IPv6 dual-stack**: Available via `pt_calculate_addressing` (fd00::/48 ULA base)

## Logging

Structured logging via Python's built-in `logging` module. Configure with `PT_MCP_LOG_LEVEL` env var.
Logger factory: `from shared.logging import get_logger; logger = get_logger(__name__)`

---
> Source: [Deiviidsito/MCP_Packet_Tracer](https://github.com/Deiviidsito/MCP_Packet_Tracer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
