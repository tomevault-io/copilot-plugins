## illumio-mcp-server

> Rules in Illumio are processed in this order:

# Illumio MCP Server

## Illumio Rule Processing Order

Rules in Illumio are processed in this order:

1. **Essential rules** (built-in, cannot be modified)
2. **Override Deny rules** (`override: true` on deny_rules endpoint) - deny traffic that overrides all allow rules. Used for emergency blocking scenarios.
3. **Allow rules** (normal rules in rulesets) - permit traffic
4. **Deny rules** (`override: false` on deny_rules endpoint) - block specific traffic
5. **Default action** - depends on enforcement mode:
   - **Selective mode**: default is **allow all** (only deny rules are actively enforced)
   - **Full enforcement**: default is **deny all** (only explicitly allowed traffic passes)

## Ringfencing Concepts

- Apps are identified by **app + env** label combination (unique app identity)
- Ringfence = coarse-grained segmentation controlling which apps can talk to each other on All Services
- **Standard ringfence**: intra-scope allow rule + extra-scope allow rules for known remote apps
- **Selective ringfence**: adds a **deny rule** (step 4) blocking all inbound, so in selective mode (where default=allow) the deny rule enforces the ringfence. Known remote apps get **allow rules** (step 3) which are processed before the deny rule.
- Override Deny is NOT used for ringfencing. It's for emergency scenarios where you need to block traffic that would otherwise be allowed.
- **Policy coverage**: Traffic flows have `policy_decision` (allowed, potentially_blocked, blocked). When creating ringfence rules, each remote app is tagged as `already_allowed` (all flows already covered by existing policy) or `newly_allowed` (at least one flow would be blocked). By default, rules are created for ALL observed traffic for documentation value. `skip_allowed=true` skips already-covered apps for minimal rulesets.

### Deny Consumer Flavors (deny_consumer parameter)

Illumio writes deny rules to the **source workload** (consumer side). The `deny_consumer` parameter controls where the deny rule is enforced:

- **`any`** (default): Consumer = IP list "Any (0.0.0.0/0)". Deny rule is only written to the **destination workloads** inside the scope. No impact on remote workloads. Safest option.
- **`ams`**: Consumer = All Workloads. Deny rule gets pushed to **every managed workload** outside the scope. Broader enforcement but wider blast radius.
- **`ams_and_any`**: Consumer = All Workloads + Any IP list. Maximum coverage — deny enforced at both managed source workloads and destination workloads.

## Infrastructure Service Identification

Infrastructure services should be policy'd first during segmentation rollouts. The `identify-infrastructure-services` tool builds an app-to-app communication graph from traffic flows and uses **dual-pattern scoring** to recognize two types of infrastructure:

- **Provider infra** (AD, DNS, shared DB) — consumed by many apps, high in-degree, low out-degree
- **Consumer infra** (monitoring, backup, log shipping) — connects out to many apps, high out-degree, low in-degree

**Scoring model:**
- **Provider score** = 40% in-degree centrality + 30% consumer ratio + 25% betweenness + 5% volume
- **Consumer score** = 40% out-degree centrality + 30% producer ratio + 25% betweenness + 5% volume
- **Final score** = max(provider, consumer)
- **Mixed-traffic dampening**: `score *= 1 / (1 + min(in, out) * 0.3)` — apps with both significant in AND out connections are business apps, not infra
- **Environment penalty**: Non-production environments get a 50% score reduction

Classification tiers: Core Infrastructure (>= 75), Shared Service (>= 50), Standard Application (< 50).

Key insight: infrastructure services have **strongly directional** traffic (either mostly inbound OR mostly outbound). Business apps have mixed bidirectional traffic.

## PCE API Notes

- Deny rules use `/deny_rules` endpoint on rulesets with `"override": true/false` in payload
- Deny rules API is NOT in the OpenAPI spec (undocumented)
- `pce.rule_sets.update()` returns None - must re-fetch after update
- `resolve_labels_as` must NOT be included in deny rule payloads (causes 406)
- Draft vs Active: mutations go to `/sec_policy/draft/`, reads can use either

## Project Structure

- `src/illumio_mcp/server.py` - Main MCP server with all tool definitions and handlers
- `src/illumio_mcp/__main__.py` - Entry point for `python -m illumio_mcp`
- `tests/test_mcp_tools.py` - Integration tests using MCP stdio_client
- `tests/conftest.py` - Test config
- `.env` - PCE credentials (not committed)

---
> Source: [alexgoller/illumio-mcp-server](https://github.com/alexgoller/illumio-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
