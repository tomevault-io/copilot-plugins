## apple-eventkit-obsidian-plugin

> Obsidian plugin that integrates with Apple Calendar and Reminders on macOS via a Swift CLI bridge.

# Apple EventKit Plugin

Obsidian plugin that integrates with Apple Calendar and Reminders on macOS via a Swift CLI bridge.

## Architecture

The plugin has two parts:

1. **`eventkitcli`** — A standalone Swift CLI tool that wraps Apple's EventKit framework. Bundled alongside the plugin in its directory. All commands output JSON to stdout.
2. **`src/`** — The Obsidian plugin (TypeScript). Calls `eventkitcli` via `child_process.execFile` through a bridge layer (`bridge.ts`).

## Building

- **Plugin**: `npm run build` produces `main.js` in the project root.
- **CLI**: `cd eventkitcli && bash build.sh` builds a universal binary (arm64 + x86_64), codesigns it, and copies it to the plugin directory.

## Key conventions

- The bridge layer (`bridge.ts`) is the only file that calls `child_process`. All CLI interaction goes through it.
- The agenda view separates data/lifecycle (`agenda-view.ts`) from DOM rendering (`agenda-renderer.ts`). Renderer functions are pure — they take a container element and data, and return nothing.
- Event notes are linked to calendar events via `event-id` in frontmatter. Note lookup scans `app.metadataCache`.
- Settings support moment.js date tokens in `noteFolderPath` (e.g. `YYYY/MM`). Empty path means vault root.

## Testing the CLI standalone

```sh
eventkitcli list-calendars
eventkitcli list-events --from 2026-02-14 --to 2026-02-14
eventkitcli list-reminder-lists
eventkitcli --help
```

## Testing the plugin

Copy the built plugin files (`main.js`, `manifest.json`, `styles.css`, `eventkitcli-bin`) into a vault's `.obsidian/plugins/apple-eventkit/` directory and enable it in Obsidian settings.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cvanes)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cvanes)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
