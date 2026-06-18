## flclash-diversion-skills

> Use when a FlClash, Mihomo, or Clash Meta setup needs multiple profiles combined into one configuration with domain-specific proxy routing.


# FlClash Diversion Skills

Merge secondary FlClash/Mihomo profiles into a base profile with domain-specific
diversion groups and rules.

## Key Constraints

- FlClash activates one profile at a time; do not attempt to select two profiles.
- Merge a secondary config into a base config and load the single merged output.
- Rule order is first-match, so exact target rules go first.
- Use only confirmed target domains. Broad domains google.com, googleapis.com,
  gstatic.com are blocked by default.
- Real service usability matters more than latency.
- `select` for manually verified nodes; `url-test` only when every candidate is
  known to work.
- Never modify input profiles; write a new output file.

## Quick Start

```bash
python scripts/merge_flclash_configs.py \
  --base generic-main.yaml \
  --secondary generic-secondary.yaml \
  --output output.yaml \
  --main-group Main \
  --secondary-group Secondary-Main \
  --target-group NotebookLM \
  --target-domain notebooklm.google \
  --target-node Proxy-B
```

## Workflow

### 1. Inspect Configs

Read both YAML profiles. Identify the main group name in the base config and the
proxy names in the secondary config.

### 2. Choose Parameters

- **Main group** (`--main-group`): The top-level select group in the base config.
- **Secondary group** (`--secondary-group`): Name for the new group containing
  all secondary proxies. Created automatically.
- **Target group** (`--target-group`): Name for the diversion group routing
  specific domains. Created automatically.
- **Target domains** (`--target-domain`): Domains to divert. Repeatable.
- **Target nodes** (`--target-node`): Specific node names for the target group.
  Repeatable. Must exist in merged proxies.

### 3. Run the Script

Use the command pattern from Quick Start above, substituting your actual file
paths, group names, target domains, and target node names.

### 4. Import and Test

1. Open FlClash and go to Profiles.
2. Add the output YAML as a local config.
3. Enable the config.
4. Go to Proxies and verify nodes in each group.
5. Test that target domains route through the diversion group.

## Script Reference

### Required Arguments

| Argument | Description |
|---|---|
| `--base` | Base config YAML path |
| `--secondary` | Secondary config YAML path |
| `--output` | Output merged YAML path |
| `--main-group` | Name of the main select group in base config |
| `--secondary-group` | Name for the new secondary proxy group |
| `--target-group` | Name for the new diversion group |
| `--target-domain` | Domain to divert (repeatable) |
| `--target-node` | Specific node name for the target group (repeatable) |

### Optional Arguments

| Argument | Default | Description |
|---|---|---|
| `--target-group-type` | `select` | Group type: `select` or `url-test` |
| `--allow-broad-domain` | false | Allow broad domain rules |

### What the Script Does

1. Loads base and secondary YAML configs.
2. Appends secondary proxies to base (skips duplicates, skips meta-entries).
3. Creates the secondary group with all real secondary proxies.
4. Inserts the secondary group into the main group at position 0.
5. Validates every `--target-node` exists in merged proxies; exits nonzero if not.
6. Creates the target group with exactly the specified target nodes.
7. Removes existing DOMAIN-SUFFIX rules for target domains, then prepends new ones.
8. Blocks broad domain rules unless `--allow-broad-domain` is set.
9. Validates in memory before writing; re-reads and validates from disk after.

### Safety Features

- **Target node validation**: If any `--target-node` is not found in merged
  proxies, the script exits nonzero and does not write output.
- **Broad domain blocking**: Rules for google.com, googleapis.com, gstatic.com
  are blocked by default to prevent hijacking all Google traffic.
- **Post-write validation**: Output is re-read and validated; if validation
  fails, the output file is deleted.

## Troubleshooting

- If the script reports missing target nodes, verify node names against the
  secondary config's proxy list.
- If broad domain rules exist in the base config pointing to other groups, the
  script warns that they may override specific diversion rules.
- To roll back, simply reload the original profile in FlClash.

## Dependencies

- Python 3.8+
- `ruamel.yaml` (`python -m pip install ruamel.yaml`)

---
> Source: [yy3350008-lgtm/flclash-diversion-skills](https://github.com/yy3350008-lgtm/flclash-diversion-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
