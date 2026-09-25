## antigravity-fusion-plugin

> Universal multi-model fusion plugin for agentic CLIs (Antigravity, Claude Code, Codex). It queries multiple models in parallel via background subprocesses of the host CLI, synthesizes the results, and acts on them.

# Fusion Development Guidelines

Universal multi-model fusion plugin for agentic CLIs (Antigravity, Claude Code, Codex). It queries multiple models in parallel via background subprocesses of the host CLI, synthesizes the results, and acts on them.

## 📁 Repository Layout

- `commands/` — Slash commands:
  - `fuse.md` — The core `/fusion:fuse <prompt>` command.
  - `setup.md` — The `/fusion:setup` command to configure models.
  - `config.md` — The `/fusion:config` command to view/set preferences.
- `scripts/` — Node.js background runner:
  - `fusion.mjs` — Zero-dependency ESM runner that handles parallel subprocess execution.
- `skills/` — Skills:
  - `fusion/SKILL.md` — Routing skill triggered by `/fusion` to delegate to `/fusion:fuse`.
  - `fusion-synthesis/SKILL.md` — Guidelines for the Judge model to perform synthesis on the panel responses.
- `plugin.json` — Metadata identifying the plugin name (`fusion`), version, and author.
- `README.md` — User documentation.

---

## ⚙️ Parallel Subprocess Engine (`fusion.mjs`)

Instead of trying to spawn subagents natively (which restricts them to the parent model's active configuration), `fusion.mjs` meta-orchestrates by launching parallel subprocesses of the active CLI.

- **Security & Safety**: Arguments are passed via an array to Node's `spawn`, which executes the subprocess directly at the OS level. There is no shell evaluation, protecting the system from prompt injection or shell metacharacter expansion.
- **Read-Only Advice**: Subagents are instructed via prompt decorators to behave as read-only panel advisors and run in print-mode (`--print` / `-p`) to output their response in a single turn without write permissions. Only the Judge has active write permissions to modify the workspace.
- **Preferences**: Models are listed in `~/.fusion_panel_prefs.txt` (one per line).

---

## 🔧 Troubleshooting and Validation

To validate the plugin structure for Antigravity:
```bash
agy plugin validate /path/to/antigravity-fusion-plugin
```

To install or update:
```bash
agy plugin install /path/to/antigravity-fusion-plugin
```

---
> Source: [ProxyAyush/antigravity-fusion-plugin](https://github.com/ProxyAyush/antigravity-fusion-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
