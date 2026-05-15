## financial-services

> Cowork plugins and Claude Managed Agent templates for financial services. Each named agent ships two ways from one source.

# Financial Services Plugins

Cowork plugins and Claude Managed Agent templates for financial services. Each named agent ships two ways from one source.

## Repository Structure

```
├── plugins/
│   ├── agent-plugins/               #   named agents — one self-contained plugin each
│   │   └── <slug>/
│   │       ├── .claude-plugin/plugin.json
│   │       ├── agents/<slug>.md     #   ← canonical system prompt (one source, two wrappers)
│   │       └── skills/              #   ← bundled copies, synced from vertical-plugins/
│   ├── vertical-plugins/            #   FSI verticals — skill sources, commands, MCPs
│   │   └── <vertical>/
│   │       ├── .claude-plugin/plugin.json
│   │       ├── commands/
│   │       ├── skills/
│   │       └── .mcp.json
│   └── partner-built/               #   partner plugins (LSEG, S&P Global)
├── managed-agent-cookbooks/         # CMA cookbooks (one dir per named agent)
│   └── <slug>/
│       ├── agent.yaml               #   system + skills → ../../plugins/agent-plugins/<slug>/...
│       ├── subagents/*.yaml         #   depth-1 leaf workers
│       ├── steering-examples.json
│       └── README.md                #   security tier + handoff notes
├── claude-for-msft-365-install/     # admin tooling for the Microsoft 365 add-in (separate from FSI plugins)
└── scripts/                         # deploy-managed-agent.sh, check.py, validate.py, orchestrate.py, sync-agent-skills.py
```

Run `python3 scripts/check.py` before committing — it lints every manifest, verifies all `system.file` / `skills.path` / `callable_agents.manifest` references resolve, and fails if any `agent-plugins/<slug>/skills/` copy has drifted from its `vertical-plugins/` source. **Edit skills in `vertical-plugins/`**, then run `python3 scripts/sync-agent-skills.py` to propagate into the agent bundles.

## Key Files

- `marketplace.json`: Marketplace manifest - registers all plugins with source paths
- `plugin.json`: Plugin metadata - name, description, version, and component discovery settings
- `commands/*.md`: Slash commands invoked as `/plugin:command-name`
- `skills/*/SKILL.md`: Detailed knowledge and workflows for specific tasks
- `*.local.md`: User-specific configuration (gitignored)
- `mcp-categories.json`: Canonical MCP category definitions shared across plugins

## Development Workflow

1. Edit markdown files directly - changes take effect immediately
2. Test commands with `/plugin:command-name` syntax
3. Skills are invoked automatically when their trigger conditions match

---
> Source: [anthropics/financial-services](https://github.com/anthropics/financial-services) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
