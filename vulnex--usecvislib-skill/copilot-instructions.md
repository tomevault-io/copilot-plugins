## usecvislib-skill

> Expert guidance for generating security visualizations with the USecVisLib library ecosystem


# USecVisLib Security Visualization Skill

Expert guidance for generating security visualizations with the USecVisLib library ecosystem.

---

## 1. Visualization Selection Guide

Match the security question to the right visualization type and tools.

| Security Question | Visualization Type | Primary Tool | Analysis Tools |
|---|---|---|---|
| What are the attack paths to a target? | Attack Tree | `generate_attack_tree` | `validate_attack_tree`, `get_attack_tree_stats`, `build_attack_tree_from_spec` |
| How do attackers traverse the network? | Attack Graph | `generate_attack_graph` | `validate_attack_graph`, `get_attack_graph_stats`, `find_attack_paths`, `analyze_critical_nodes`, `find_chokepoints`, `analyze_centrality` |
| What threats exist in data flows? | Threat Model (DFD) | `generate_threat_model` | `validate_threat_model`, `analyze_stride_threats`, `get_threat_model_stats` |
| What does the binary look like inside? | Binary Analysis | `analyze_binary_entropy` | `analyze_binary_distribution`, `analyze_binary_heatmap`, `analyze_binary_all`, `get_binary_stats` |
| What's the cloud architecture? | Cloud Diagram | `generate_cloud_diagram` | `validate_cloud_diagram`, `get_cloud_diagram_stats`, `search_cloud_icons`, `list_cloud_providers`, `list_cloud_icons` |
| Where are trust boundary violations? | Privilege Gradient | `generate_privilege_gradient` | `validate_privilege_gradient`, `get_privilege_gradient_stats`, `detect_privilege_inversions`, `analyze_zone_influence` |
| What's the system architecture? | Component Diagram | `generate_component_diagram` | `validate_component_diagram`, `get_component_diagram_stats` |
| How do modules depend on each other? | Dependency Graph | `generate_dependency_graph` | `validate_dependency_graph`, `get_dependency_graph_stats` |
| Custom topology or flowchart? | Custom Diagram | `generate_custom_diagram` | `validate_custom_diagram`, `get_custom_diagram_stats`, `list_shapes` |

**Additional tools**: `render_mermaid` (render Mermaid syntax to image), `convert_to_mermaid`, `calculate_cvss_score`, `detect_config_type`, `export_data`, `generate_report`, `batch_process`, `compare_visualizations`. See [REFERENCE.md](REFERENCE.md) for the complete 49-tool catalog with parameters and return types.

---

## 2. Configuration Best Practices

### Attack Trees

Structure: hierarchical goal decomposition with AND/OR gates.

- **Root**: Single top-level attacker goal
- **Sub-goals**: 3-5 levels of decomposition (depth 3-5 optimal)
- **Gates**: Use AND for multi-step attacks, OR for alternatives
- **Leaves**: Attach CVSS scores for quantitative risk
- **Config keys**: `tree` (name, root, params), `nodes` (styling), `edges` (connections with labels)
- **Optional fields**: `params.rankdir` (layout direction), node `fontcolor`, `style`, `penwidth`

```json
{
  "tree": {"name": "...", "root": "Root Goal", "params": {"rankdir": "TB"}},
  "nodes": {"Root Goal": {"shape": "oval", "fillcolor": "#e74c3c"}},
  "edges": {"Root Goal": [{"to": "Sub Goal", "label": "OR"}]}
}
```

See [examples/attack_tree_web_app.json](examples/attack_tree_web_app.json) for a full web application attack tree.

### Attack Graphs

Structure: network topology with hosts, vulnerabilities, exploits.

- **Hosts**: Include IP addresses and zone assignments (external/dmz/internal)
- **Vulnerabilities**: Link to specific hosts, include CVE IDs and CVSS scores
- **Exploits**: Define preconditions (vulnerabilities) and postconditions (privileges)
- **Services**: Map to hosts with port/protocol
- **Config keys**: `graph`, `hosts`, `vulnerabilities`, `exploits`
- **Optional fields**: `privileges` (privilege definitions), `services` (port/protocol mappings per host), `network_edges` (explicit network topology links between hosts)

*Note: IP addresses in examples are fictional (RFC 1918 private ranges) and for illustration only.*

```json
{
  "graph": {"name": "..."},
  "hosts": [
    {"id": "web", "label": "Web Server", "ip": "10.0.1.10", "zone": "dmz"},
    {"id": "app", "label": "App Server", "ip": "10.0.2.20", "zone": "internal"},
    {"id": "db", "label": "Database", "ip": "10.0.2.30", "zone": "internal"}
  ],
  "vulnerabilities": [
    {"id": "v1", "label": "RCE - CVE-2024-1234", "cvss": 9.8, "affected_host": "web"},
    {"id": "v2", "label": "SQLi - CVE-2024-5678", "cvss": 8.6, "affected_host": "app"}
  ],
  "services": [
    {"id": "s1", "host": "web", "port": 443, "protocol": "https"},
    {"id": "s2", "host": "app", "port": 8080, "protocol": "http"}
  ],
  "exploits": [
    {"id": "e1", "label": "Exploit RCE", "preconditions": ["v1"], "postconditions": ["priv_web_shell"]},
    {"id": "e2", "label": "Exploit SQLi", "preconditions": ["v2", "priv_web_shell"], "postconditions": ["priv_db_read"]}
  ],
  "network_edges": [
    {"from": "web", "to": "app", "label": "HTTP 8080"},
    {"from": "app", "to": "db", "label": "SQL 3306"}
  ]
}
```

See [examples/attack_graph_network.json](examples/attack_graph_network.json) for a full network topology example.

### Threat Models

Structure: Data Flow Diagram with STRIDE analysis.

- **Format**: Dict-of-dicts (not lists) for processes, datastores, externals, dataflows
- **Processes**: Include `sanitizesInput`, `hasAccessControl` boolean properties
- **Datastores**: Include `isEncrypted`, `storesPII` properties
- **Dataflows**: Must reference valid `from`/`to` element IDs, include `isEncrypted`
- **Boundaries**: Group elements into trust zones
- **Config keys**: `model`, `processes`, `datastores`, `externals`, `dataflows`
- **Optional fields**: `boundaries` (trust zone groupings that cluster elements visually)

```json
{
  "model": {"name": "...", "description": "..."},
  "processes": {"web_app": {"label": "Web App", "sanitizesInput": true}},
  "datastores": {"db": {"label": "Database", "isEncrypted": true}},
  "externals": {"user": {"label": "User", "isTrusted": true}},
  "dataflows": {"f1": {"from": "user", "to": "web_app", "isEncrypted": true}}
}
```

See [examples/threat_model_microservice.json](examples/threat_model_microservice.json) for a full microservice DFD.

### Binary Analysis

Pipeline: entropy -> distribution -> heatmap for triage.

- **Input**: Path to binary file on disk
- **Entropy**: Reveals packed/encrypted sections (high entropy regions)
- **Distribution**: Shows byte frequency patterns
- **Heatmap**: 2D visualization of byte patterns across file offset
- **All-in-one**: `analyze_binary_all` generates all three plus stats

See [examples/binary_analysis_sample.json](examples/binary_analysis_sample.json) for a workflow config showing the analysis pipeline.

### Cloud Diagrams

Structure: nodes with provider icons, edges, optional clusters.

- **Providers**: AWS, GCP, Azure, etc. (use `list_cloud_providers` to discover)
- **Icons**: Provider-specific icon paths (use `search_cloud_icons` or `list_cloud_icons`)
- **Clusters**: Group nodes by VPC, subnet, region
- **Direction**: TB (top-bottom), LR (left-right)
- **Config keys**: `diagram`, `nodes` (with `icon` field), `edges`
- **Optional fields**: `clusters` (group nodes by VPC, subnet, or region with nested node lists)

```json
{
  "diagram": {"title": "...", "direction": "TB"},
  "nodes": [{"id": "lb", "icon": "aws.network.ELB", "label": "Load Balancer"}],
  "edges": [{"from": "lb", "to": "web", "label": "HTTPS"}]
}
```

See [examples/cloud_aws_three_tier.json](examples/cloud_aws_three_tier.json) for a full AWS three-tier architecture.

### Privilege Gradient

Structure: trust zones with components and cross-zone influences.

- **Zones**: Define trust levels (0 = untrusted, higher = more trusted)
- **Components**: Assign each to a zone
- **Influences**: Cross-zone data flows (from/to component IDs)
- **Inversions**: `detect_privilege_inversions` finds flows from higher to lower trust
- **Config keys**: `gradient`, `zones`, `components`, `influences`

```json
{
  "gradient": {"name": "...", "description": "..."},
  "zones": {"external": {"label": "External", "level": 0}},
  "components": {"cdn": {"label": "CDN", "zone": "external"}},
  "influences": [{"from": "cdn", "to": "web_app", "label": "HTTPS"}]
}
```

See [examples/privilege_gradient_web.json](examples/privilege_gradient_web.json) for a full web application privilege gradient.

### Component Diagrams

Structure: layered architecture with ordered components and typed connections.

- **Layers**: Define presentation, logic, and data tiers with `order` (rendering priority) and `color`
- **Components**: Assign each to a layer; include `type` (service, database, queue, cache, etc.)
- **Connections**: Typed links between components (`sync`, `async`, `data`, `event`)
- **Validation**: `validate_component_diagram` checks for orphan components and missing layers
- **Config keys**: `diagram`, `layers` (with `order` and `color`), `components` (with `layer` and `type`), `connections`

```json
{
  "diagram": {"name": "...", "description": "..."},
  "layers": {
    "presentation": {"label": "Presentation", "order": 1, "color": "#3498db"},
    "logic": {"label": "Business Logic", "order": 2, "color": "#2ecc71"}
  },
  "components": {
    "web_ui": {"label": "Web UI", "layer": "presentation", "type": "service"},
    "api": {"label": "API Server", "layer": "logic", "type": "service"}
  },
  "connections": [
    {"from": "web_ui", "to": "api", "label": "REST", "type": "sync"}
  ]
}
```

See [examples/component_diagram_layered.json](examples/component_diagram_layered.json) for a full three-tier example.

### Dependency Graphs

Structure: modules with dependency relationships and metadata.

- **Modules**: Include `sloc` (size), `language`, and optional `version` for sizing/coloring nodes
- **Dependencies**: Typed relationships (`import`, `runtime`, `dev`, `optional`) between modules
- **Cycles**: `validate_dependency_graph` detects circular dependencies
- **Stats**: `get_dependency_graph_stats` returns module count, edge count, and connectivity metrics
- **Config keys**: `graph`, `modules` (with `sloc`, `language`), `dependencies` (with `type`)

```json
{
  "graph": {"name": "...", "description": "..."},
  "modules": {
    "core": {"label": "Core Library", "sloc": 2400, "language": "python"},
    "api": {"label": "API Layer", "sloc": 1800, "language": "python"}
  },
  "dependencies": [
    {"from": "api", "to": "core", "type": "import", "label": "core models"}
  ]
}
```

See [examples/dependency_graph_modules.json](examples/dependency_graph_modules.json) for a full multi-module example.

### Custom Diagrams

Structure: freeform nodes, edges, and optional groups for any topology or flowchart.

- **Nodes**: Define `shape` (box, oval, diamond, hexagon, cylinder, parallelogram), `fillcolor`, `label`
- **Edges**: Connect nodes with optional `label`, `style` (solid, dashed, dotted), and `color`
- **Groups**: Cluster nodes into named subgraphs with `label` and `color`
- **Layout**: Set `rankdir` (TB, BT, LR, RL) in the diagram params
- **Validation**: `validate_custom_diagram` checks for dangling edge references
- **Config keys**: `diagram`, `nodes`, `edges`
- **Optional fields**: `groups` (cluster nodes into named subgraphs with `label`, `color`, and `nodes` list)

```json
{
  "diagram": {"name": "...", "params": {"rankdir": "TB"}},
  "nodes": {
    "start": {"label": "Start", "shape": "oval", "fillcolor": "#2ecc71"},
    "decision": {"label": "Valid?", "shape": "diamond", "fillcolor": "#f39c12"},
    "process": {"label": "Process", "shape": "box", "fillcolor": "#3498db"}
  },
  "edges": [
    {"from": "start", "to": "decision", "label": "input"},
    {"from": "decision", "to": "process", "label": "yes", "style": "solid"}
  ],
  "groups": [
    {"id": "validation", "label": "Validation Phase", "nodes": ["start", "decision"]}
  ]
}
```

See [examples/custom_diagram_flowchart.json](examples/custom_diagram_flowchart.json) for a full incident response flowchart.

---

## 3. Style Recommendations

Choose styles based on the target audience:

| Audience | Recommended Styles | Rationale |
|---|---|---|
| Executive / Board | `*_corporate` | Clean, professional, muted colors |
| Security Team | `*_default`, `*_dark` | Standard technical detail, dark theme for SOC screens |
| Pentest Report | `*_neon` | High contrast, emphasizes severity |
| Academic / Research | `*_blueprint` | Technical blueprint aesthetic |

Style ID format: `{module_prefix}_{style}` (e.g., `at_corporate`, `ag_dark`, `tm_neon`).

To discover all available styles, call the generation tool with an invalid style ID — the error response lists valid options. See [REFERENCE.md](REFERENCE.md) for the full style discovery methods per integration.

---

## 4. Workflow Orchestration

### Workflow 1: Comprehensive Security Assessment

Full threat analysis pipeline from DFD to actionable report.

1. **Threat Model DFD**: `generate_threat_model` with all processes, datastores, externals, dataflows, boundaries
2. **STRIDE Analysis**: `analyze_stride_threats` to identify threats per element
3. **Attack Trees**: `generate_attack_tree` for each high-risk STRIDE threat — decompose into sub-goals
4. **Attack Graph**: `generate_attack_graph` mapping exploits across the network
5. **Chokepoint Analysis**: `find_chokepoints` + `analyze_centrality` to prioritize defensive controls
6. **Report**: `generate_report` on the attack graph for a multi-format deliverable

### Workflow 2: Binary Triage Pipeline

Rapid binary analysis for malware/firmware review.

> **Safety note:** This workflow performs **static analysis only** — it reads byte patterns and metadata without executing the binary. However, always handle suspicious binaries in an isolated environment (VM/sandbox) and never execute them on production systems.

1. **Stats**: `get_binary_stats` for file size, type, section overview
2. **Entropy**: `analyze_binary_entropy` to identify packed/encrypted regions
3. **Distribution**: `analyze_binary_distribution` for byte frequency anomalies
4. **Heatmap**: `analyze_binary_heatmap` for spatial byte pattern visualization
5. Or simply: `analyze_binary_all` for all four in one call

### Workflow 3: Cloud Security Review

Architecture documentation with security analysis.

1. **Provider Discovery**: `list_cloud_providers` + `list_cloud_icons` to find correct icon paths
2. **Cloud Diagram**: `generate_cloud_diagram` with proper provider icons, VPC clusters
3. **Privilege Gradient**: Map cloud resources to trust zones, `generate_privilege_gradient`
4. **Inversion Detection**: `detect_privilege_inversions` to find misconfigured trust boundaries
5. **Threat Model**: `generate_threat_model` for the cloud data flows

### Workflow 4: Attack Surface Mapping

Network-centric attack surface analysis.

1. **Attack Graph**: `generate_attack_graph` with full host/vulnerability/exploit data
2. **Chokepoints**: `find_chokepoints` — critical defensive positions
3. **Centrality**: `analyze_centrality` — infrastructure bridge nodes
4. **Privilege Gradient**: Map network zones to trust levels, `generate_privilege_gradient`
5. **Inversions**: `detect_privilege_inversions` for trust boundary violations
6. **Comparison**: `compare_visualizations` to diff against previous assessment

---

## 5. Export and Output Guidance

### Data Export (`export_data`)

| Format | Best For | Notes |
|---|---|---|
| `json` | API integration, archives | Returns string directly |
| `yaml` | Human-readable configs | Returns string directly |
| `csv` | Spreadsheets, tabular analysis | Writes file, returns path + row count |
| `md` | Documentation, reports | Returns markdown table string |

### Report Generation (`generate_report`)

- Accepts any Pattern A visualization type (`attack_tree`, `attack_graph`, `threat_model`, `privilege_gradient`, `component_diagram`, `dependency_graph`)
- Produces reports in multiple formats simultaneously
- Default formats: `["json", "csv", "md"]`
- Returns a directory with all generated files

### Batch Processing (`batch_process`)

- Process multiple configs of the same type in parallel
- **Maximum 100 configs per batch** — split larger sets into multiple calls
- Set `max_workers` based on system resources (default: 4, max: 8)
- Each config is subject to the same size limits as individual tool calls (10 MB max)
- Returns summary with success/failure counts per file
- Supported types: `attack_tree`, `attack_graph`, `threat_model`, `privilege_gradient`, `component_diagram`, `dependency_graph`

### Mermaid Export (`convert_to_mermaid`)

- Converts attack trees, attack graphs, and threat models to Mermaid diagram syntax
- Useful for embedding in Markdown documentation
- Auto-detects visualization type from config structure

---

## 6. Auto-Detection Guidance

### Config Type Detection (`detect_config_type`)

Pass any config dict to automatically identify the visualization type. Confidence levels:

- **high**: Unique structural markers found (e.g., `tree.root` for attack trees)
- **medium**: Probable match based on field patterns
- **low**: Ambiguous, may need manual specification

### CVSS Version Detection (`calculate_cvss_score`)

Automatically detects CVSS version from vector prefix:
- `CVSS:3.0/...` → CVSS 3.0
- `CVSS:3.1/...` → CVSS 3.1
- `CVSS:4.0/...` → CVSS 4.0

Returns score, severity rating (None/Low/Medium/High/Critical), and parsed vector components.

### Style Fallback

If a specified style ID is not found, the library falls back to the module's default style. See [REFERENCE.md](REFERENCE.md) for style discovery methods to list valid style IDs before generation.

---

## 7. Validation and Error Handling

Always validate configs before generation to catch structural issues early.

### Using Validation Tools

Each visualization type has a `validate_*` tool. Call it before `generate_*`:

```python
# Validate first
result = validate_attack_tree(config)
# result: {"valid": false, "errors": ["Missing required field: tree.root"], "warnings": ["Node 'x' has no edges"]}

# Fix errors, then generate
image_path = generate_attack_tree(config)
```

### Common Validation Errors

| Error | Cause | Fix |
|---|---|---|
| `Missing required field: tree.root` | Attack tree config lacks root node | Add `"root": "Goal Name"` to `tree` object |
| `Dangling edge reference: 'node_x'` | Edge references a node ID that doesn't exist | Check `from`/`to` values match defined node IDs |
| `Invalid dataflow: 'f1' references unknown element 'x'` | Dataflow `from` or `to` doesn't match a process/datastore/external ID | Ensure all element IDs in dataflows are defined |
| `Circular dependency detected: a -> b -> a` | Dependency graph has a cycle | Remove or restructure the circular dependency |
| `Unknown icon: 'aws.foo.Bar'` | Cloud diagram references a non-existent provider icon | Use `search_cloud_icons` to find the correct icon path |
| `Zone 'x' referenced by component but not defined` | Privilege gradient component points to undefined zone | Add the zone to the `zones` object or fix the zone reference |
| `Orphan component: 'x' has no connections` | Component diagram has isolated components | Add connections or remove the orphan component |

### Warnings vs Errors

- **Errors** prevent generation — the config is structurally invalid
- **Warnings** allow generation but flag potential issues (e.g., orphan nodes, missing optional fields)

Always resolve errors before calling the generation tool. Warnings can be reviewed and addressed as needed.

---

## 8. Performance Guidance

### Config Size Limits

| Constraint | Limit | Notes |
|---|---|---|
| Max config size | 10 MB | Per individual tool call |
| Max batch size | 100 configs | Per `batch_process` call; split larger sets |
| Max attack graph nodes | ~500 hosts | Beyond this, rendering becomes slow; consider filtering by zone |
| Max attack tree depth | ~10 levels | Deep trees become unreadable; flatten sub-trees into separate diagrams |
| Max dependency graph modules | ~200 | Large graphs benefit from filtering by dependency type |

### Processing Expectations

- **Simple configs** (< 20 nodes): sub-second generation
- **Medium configs** (20-100 nodes): 1-5 seconds
- **Large configs** (100-500 nodes): 5-30 seconds depending on edge density
- **Binary analysis**: scales with file size — a 10 MB binary takes ~2-5 seconds for entropy analysis
- **Batch processing**: set `max_workers` based on available CPU cores (default: 4, max: 8)

### Memory Considerations

- Attack graphs with dense `network_edges` consume more memory than sparse topologies
- Binary heatmap generation loads the entire file into memory — avoid files > 100 MB
- SVG output is generally larger than PNG but allows lossless scaling
- PDF output embeds fonts and is the largest format

### Tips for Large Visualizations

- **Filter by zone/layer**: Generate separate diagrams per network zone or architectural layer rather than one massive diagram
- **Use `max_paths`/`max_depth`**: When calling `find_attack_paths`, limit results to avoid combinatorial explosion
- **Use `top_n`**: For `analyze_critical_nodes`, `find_chokepoints`, and `analyze_centrality`, use `top_n` to limit output
- **Split batch jobs**: Keep batches under 50 configs for predictable performance

---

## 9. Troubleshooting

### MCP Server Connection

| Symptom | Likely Cause | Solution |
|---|---|---|
| `usecvislib-mcp: command not found` | MCP server package not installed | Run `pip install usecvislib-mcp` |
| `Connection refused` on MCP | Server not running or wrong port | Check MCP config points to correct command; restart the agent |
| Tools appear but return errors | USecVisLib library not installed | Run `pip install usecvislib` (the MCP server depends on it) |
| `ModuleNotFoundError: graphviz` | Graphviz system package missing | Install via `brew install graphviz` (macOS), `apt install graphviz` (Linux), or `choco install graphviz` (Windows) |

### Generation Failures

| Symptom | Likely Cause | Solution |
|---|---|---|
| Empty or blank image output | Config is valid but contains no renderable elements | Check that nodes/components are defined, not just metadata |
| `FileNotFoundError` on binary analysis | Binary file path is wrong or inaccessible | Use an absolute path; verify the file exists and is readable |
| Style not applied | Invalid style ID, fell back to default | Use style discovery to list valid IDs (see [REFERENCE.md](REFERENCE.md)) |
| `PermissionError` writing output | Output directory is not writable | Check directory permissions; ensure the working directory is writable |
| Mermaid rendering fails | `render_mermaid` requires a Mermaid CLI (`mmdc`) | Install via `npm install -g @mermaid-js/mermaid-cli` |

### Config Debugging Checklist

1. **Run the validator first** — `validate_*` catches most structural issues before generation
2. **Check ID references** — every `from`/`to` in edges/dataflows/dependencies must match a defined element ID
3. **Use `detect_config_type`** — if unsure which type a config matches, let auto-detection confirm
4. **Start from examples** — copy a working config from `examples/` and modify incrementally
5. **Check JSON syntax** — trailing commas and unquoted keys are common JSON errors (use a linter)

---
> Source: [vulnex/usecvislib-skill](https://github.com/vulnex/usecvislib-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
