## examples-yaml-sync

> Remind to update ALL example YAML configs when modifying hparams dataclasses


When modifying fields in hparams/ dataclasses:
- New fields: add to ALL example configs in examples/ with default value and `# Options:` comment
- Renamed fields: search-and-replace across ALL YAML files (old keys fail silently)
- Removed fields: clean up from ALL YAML files
See constraint #15 in .agents/knowledge/constraints.md.

---
> Source: [X-GenGroup/Flow-Factory](https://github.com/X-GenGroup/Flow-Factory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
