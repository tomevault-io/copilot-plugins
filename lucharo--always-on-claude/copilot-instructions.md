## always-on-claude

> - **Don't install `mmdc` (mermaid-cli) to render mermaid diagrams.** GitHub renders mermaid in markdown natively, and every file in this repo is primarily consumed on GitHub. Write the mermaid block in markdown and stop — no local render step, no PNG export, no `npx @mermaid-js/mermaid-cli`.

# always-on-claude

## Diagrams

- **Don't install `mmdc` (mermaid-cli) to render mermaid diagrams.** GitHub renders mermaid in markdown natively, and every file in this repo is primarily consumed on GitHub. Write the mermaid block in markdown and stop — no local render step, no PNG export, no `npx @mermaid-js/mermaid-cli`.

## Upgrade / maintenance workflows

- For any Happier upgrade task (update CLI, restart daemon, sync versions across Mac + arch), use the `skills/happier-upgrade` skill. It captures the multi-machine traps learned the hard way (HAPPIER_CHANNEL env var scoping, SSH-flap-safe yarn builds, Node engine pins, etc.). Don't redo the investigation from scratch.

---
> Source: [lucharo/always-on-claude](https://github.com/lucharo/always-on-claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
