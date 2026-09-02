## zcode-plugins

> This repository hosts the ZCode plugin marketplace. Use this file as the operating guide for creating plugins, modifying installable plugin content, and maintaining repository documentation. For field definitions, complete examples, and protocol details, load only the relevant section of [docs/PLUGIN_DEVELOPMENT.md](docs/PLUGIN_DEVELOPMENT.md).

# ZCode Plugin Marketplace: Agent Guide

This repository hosts the ZCode plugin marketplace. Use this file as the operating guide for creating plugins, modifying installable plugin content, and maintaining repository documentation. For field definitions, complete examples, and protocol details, load only the relevant section of [docs/PLUGIN_DEVELOPMENT.md](docs/PLUGIN_DEVELOPMENT.md).

## Sources of Truth

Derive the current contract from repository files, not memory or examples from other projects:

- The target plugin's `.zcode-plugin/plugin.json` manifest
- The target plugin's README documentation (`README.md` and `README_CN.md` when present)
- Its actual component directories: `commands/`, `skills/`, `agents/`, and `hooks/`, plus `.mcp.json` when present
- The matching entry in the root `marketplace.json`
- Repository validators and tests for machine-enforced behavior

## Workflow

1. Inspect the sources of truth and classify the task as a new plugin, an installable-content change, or a root-documentation-only change.
2. Load only the reference sections required for that task. Keep field tables, complete JSON examples, and Hook protocol details out of the working context until needed.
3. Make the smallest coherent change. Update behavior and manifests first, then synchronize the plugin's README documentation, the marketplace entry, and the plugin version as applicable.
4. Run the repository checks from the repository root. Treat command output, rather than static inspection, as the verification boundary.
5. Report the changed files, resulting plugin version, commands run, and their outcomes. State explicitly when installation or real ZCode runtime verification remains outstanding.

## Versioning Contract

Any change to a file included in the installable plugin package requires a new semantic version in both `.zcode-plugin/plugin.json` and the matching `marketplace.json` entry. This includes manifests, components, Hooks, MCP declarations, and plugin-level README files. Published versions are immutable and must never be reused.

Root-level documentation-only changes do not require a plugin version bump.

## Plugin Structure

```text
my-plugin/
├── .zcode-plugin/plugin.json    required manifest
├── commands/*.md                slash commands; use $ARGUMENTS for input
├── skills/<name>/SKILL.md       skills with precise name and description fields
├── agents/*.md                  sub-agent definitions and system prompts
├── hooks/hooks.json             automatically discovered Hook configuration
└── .mcp.json                    MCP server declarations
```

For a new plugin, start from `plugins/example-plugin/`, remove unused sample components, and retain at least one discoverable component. The directory name must be unique and kebab-case; the manifest `name` must satisfy `^[a-z0-9][a-z0-9._-]{0,127}$`.

## Validation and Definition of Done

Run the baseline checks from the repository root:

```shell
python3 scripts/validate.py
python3 scripts/build_dist.py
git diff --check
```

Plugin work is complete only when:

- The validator accepts the plugin directory and manifest.
- The manifest and marketplace entry agree on name, version, and localized descriptions; the marketplace entry contains a valid category.
- At least one component is discoverable, and the plugin's README documentation explains how a person or Agent can invoke it in every language provided.
- Relevant tests pass, including any focused tests required by changed scripts or runtime behavior.
- The primary capability has been exercised in ZCode, or the missing runtime check is recorded explicitly.

## Security and Reproducibility

Keep secrets, private endpoints, customer data, machine-specific paths, and unreproducible build artifacts out of the repository. Treat enabling a plugin as granting code-execution trust: review the provenance, permissions, network access, and side effects of every new Hook, script, dependency, and MCP server.

## Reference Routing

- Manifest, marketplace, command, or skill fields: [Section 2 — Field reference](docs/PLUGIN_DEVELOPMENT.md#2-field-reference)
- Complete manifest, marketplace, Hook, or MCP examples: [Section 3 — Complete JSON examples](docs/PLUGIN_DEVELOPMENT.md#3-complete-json-examples)
- Hook events, matchers, input/output contracts, or exit codes: [Section 4 — Hooks guide](docs/PLUGIN_DEVELOPMENT.md#4-hooks-guide)
- Documentation structure and Agent-facing writing: [Section 5 — Writing for people and Agents](docs/PLUGIN_DEVELOPMENT.md#5-writing-for-people-and-agents)
- Contribution and review requirements: [CONTRIBUTING.md](CONTRIBUTING.md)

---
> Source: [zai-org/zcode-plugins](https://github.com/zai-org/zcode-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
