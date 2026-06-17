## ansible-collection-vyos

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ansible Collection `gamersoutreach.vyos` — reusable roles and plugins for managing VyOS 1.4 devices. Consumed by multiple internal network projects (starting with `ansible-net-cloudrouter`). Inspired by [Arista Validated Design](https://avd.sh/en/stable/) patterns.

This collection owns no device inventory, host_vars, group_vars, or playbooks — those live in the consuming projects. Only roles, plugins, and their supporting scaffolding live here.

## Commands

All commands run via Docker (`ghcr.io/gamersoutreach/ansible-runner`).

```
make lint              # Run ansible-lint
```

## Architecture

### Roles

- **`vyos_cli_config_gen`** — Renders VyOS CLI configuration from host/group variables using Jinja2 templates. Templates cover all major VyOS 1.4 config sections (firewall, high-availability, interfaces, load-balancing, NAT, policy, protocols, QoS, service, system, VPN, VRF). Output is written to `{{ playbook_dir }}/configs/intended/<hostname>.cfg` and sorted into VyOS canonical order by the `vyos_sort_config` filter.
- **`vyos_config_deploy_ssh`** — Deploys rendered configuration via `vyos.vyos.vyos_config` over SSH. Has separate task files: `main.yml` (apply), `diff.yml`, and `rollback.yml`.

### Plugins

- **`vyos_defined`** (test plugin, `plugins/test/`) — Tests whether a variable is defined, not None, and optionally matches a specific value or type. Replacement for `arista.avd.defined`.
- **`vyos_sort_config`** (filter plugin, `plugins/filter/`) — Sorts generated config lines to match VyOS's canonical output order. IP-aware numeric sort for unquoted path nodes, string sort for quoted leaf values.

### Role variable contract

The `vyos_cli_config_gen` role consumes top-level variables that map directly to template sections. Variable names use underscores; these map to hyphens in VyOS CLI (e.g., `default_action` → `default-action`).

Top-level keys consumed: `firewall`, `high_availability`, `interfaces`, `load_balancing`, `nat`, `policy`, `protocols`, `qos`, `service`, `system`, `vpn`, `vrf`.

No `default_` / unprefixed merging is performed inside the collection. If a consumer wants recursive merging of group and host vars, they should do it in their own playbook via `set_fact` before invoking the role.

### Variable structure conventions

- **Dict keyed by name** — named collections (firewall rulesets, prefix lists, zones, interfaces by type)
- **Dict keyed by number** — numbered rules within a collection
- **List of dicts** — items with an identifying field (BGP neighbors)
- **Simple list** — flat value lists (addresses, name servers)
- **Boolean flags** — set to `true` to output the flag, omit or `false` to suppress

### Quoting rules (matched by templates)

VyOS config output has specific quoting conventions that templates must match for clean diffs:

- **Quoted** (single quotes): leaf values — `description 'text'`, `port '22'`, `state 'established'`
- **Unquoted**: path node keys — `next-hop 10.0.0.1`, `interface eth0`, `network 10.0.0.0/24`
- **No value**: boolean flags — `blackhole`, `disable`, `enable-default-log`

## Conventions

### FQCN everywhere

All internal references to this collection's plugins and roles use fully-qualified names:

- Test: `is gamersoutreach.vyos.vyos_defined`
- Filter: `| gamersoutreach.vyos.vyos_sort_config`
- Roles: `gamersoutreach.vyos.vyos_cli_config_gen`, `gamersoutreach.vyos.vyos_config_deploy_ssh`

No bare short names. The find-and-replace across ~820 template occurrences was done deliberately — short-name resolution via `collections:` keyword is avoided for explicitness and lint survivability.

### StrictUndefined and variable references

Ansible's `StrictUndefined` raises when resolving an undefined variable _before_ the test function is invoked — which defeats the `UndefinedError` handler in the `vyos_defined` plugin. Both dotted accesses _and_ bare top-level variable references in `{% if %}` guards must use `| default(none)` when the variable is optional:

```jinja2
{% if (firewall.group | default(none)) is gamersoutreach.vyos.vyos_defined %}       {# dotted access — correct #}
{% if (sysctl_parameter | default(none)) is gamersoutreach.vyos.vyos_defined %}     {# bare top-level — correct #}
{% if sysctl_parameter is gamersoutreach.vyos.vyos_defined %}                        {# WRONG — raises in strict mode #}
```

Loop variables (e.g., `for key, value in foo.items()`) don't need this guard since the iteration guarantees they're defined.

### Lint exceptions

Target profile: `production` (set in `.ansible-lint`). For legitimately-unfixable violations (e.g., variables that are part of the public role API and cannot be prefixed), use inline `# noqa: <rule>` on the task name line — never add to a blanket `skip_list`.

### Platform metadata

`roles/*/meta/main.yml` declares `platforms: [{name: Debian, versions: [all]}]` — **not** `VyOS`. VyOS isn't in ansible-lint's bundled Galaxy platform enum. Since VyOS 1.4 is Debian Bookworm-based, `Debian` is both accurate and lint-valid. This field is metadata-only; it doesn't affect role behavior.

## Development

### Testing changes against a consuming project

Consuming projects (e.g., `ansible-net-cloudrouter`) support a `VYOS_COLLECTION_DEV` environment variable that bind-mounts a local checkout of this collection over the installed version. From the consuming project:

```
export VYOS_COLLECTION_DEV=/path/to/ansible-collection-vyos
make plan
```

Edits in this repo are reflected immediately without reinstalling. The gold test for any change is a byte-for-byte comparison of `configs/intended/` before and after.

### Release process

1. Update `galaxy.yml` `version` and prepend an entry to `CHANGELOG.md`.
2. Commit and tag: `git tag vX.Y.Z && git push --tags`.
3. Consuming projects bump their `collections/requirements.yml` pin.

### Versioning

- Pre-1.0 (`0.x.y`): breaking changes allowed on minor bumps.
- 1.0.0 onwards: strict semver. Major = breaking to role interface or template output; minor = additive; patch = bug fix with identical template output.

## Dependencies

- Docker (lint runs in a container)
- `vyos.vyos` collection (declared in `galaxy.yml`, installed transitively when a consumer installs `gamersoutreach.vyos`)

---
> Source: [gamersoutreach/ansible-collection-vyos](https://github.com/gamersoutreach/ansible-collection-vyos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
