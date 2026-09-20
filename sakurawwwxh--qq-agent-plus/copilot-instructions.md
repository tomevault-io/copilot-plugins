## qq-agent-plus

> When adding or changing an experimental feature, follow

# Repository Instructions

When adding or changing an experimental feature, follow
[`docs/EXPERIMENTAL_FEATURE_STANDARD.md`](docs/EXPERIMENTAL_FEATURE_STANDARD.md).

In particular:

- treat each experimental feature as an independently owned module;
- keep the experiment settings page limited to enable/disable and graduation;
- give any feature with operational UI its own page;
- keep runtime enablement separate from the persistent `graduated` state;
- preserve data while disabled and never auto-retry unknown external writes.

---
> Source: [sakurawwwxh/qq-agent-plus](https://github.com/sakurawwwxh/qq-agent-plus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
