## vscode-team-kit

> Each top-level directory containing a `.plugin/plugin.json` is a plugin. Plugins are self-contained — skills, hooks, scripts, agents, and commands all live inside the plugin directory.

# Writing Plugins

Each top-level directory containing a `.plugin/plugin.json` is a plugin. Plugins are self-contained — skills, hooks, scripts, agents, and commands all live inside the plugin directory.

## Creating a New Plugin

1. Create a directory at the repo root (lowercase, hyphens ok, e.g. `my-plugin`).
2. Add `.plugin/plugin.json`:
   ```json
   {
     "name": "my-plugin",
     "version": "1.0.0",
     "description": "Brief description of what the plugin does.",
     "license": "MIT",
     "keywords": ["relevant", "tags"]
   }
   ```
3. Add a `CHANGELOG.md` with the initial release entry (see [Versioning](#versioning) below).
4. Add a `README.md` with: one-sentence description, skills table, how it works, and plugin structure tree.
5. Register the plugin in `marketplace.json` — add an entry to the `plugins` array with matching `name`, `version`, and `description`.
6. Add the plugin to the appropriate workflow section in the repo `README.md`.
7. Add your components (skills, commands, agents, hooks, scripts, etc.).

## Versioning

Every plugin uses [semantic versioning](https://semver.org/). Version numbers must stay in sync across three places:

| File | Field |
|---|---|
| `my-plugin/.plugin/plugin.json` | `"version"` |
| `marketplace.json` | `"version"` in the plugin's entry |
| `my-plugin/CHANGELOG.md` | Top entry heading |

**On every change to a plugin**, bump the version and add a changelog entry in the same commit. Don't batch version bumps across multiple PRs.

## Node Scripts

Dependencies are declared in the root `package.json`. Any Node.js script (hooks, MCP servers, utilities, etc.) should require the install helper at the top to ensure `node_modules` are available:

```js
import install from '<path-to-team-kit>/common/install-if-necessary.mts';

install().then(() => import('./impl.mts'));
```

This runs `npm install` if `package.json` or `package-lock.json` have changed since the last install.

## Edit Tools

The `common/edit-tools.mts` module describes the built-in file-editing tools (`replace_string_in_file`, `multi_replace_string_in_file`, `apply_patch`) and provides `extractEditInputs(toolName, toolInput)` to normalise their inputs into a common `EditInput[]` shape.

Use it in PreToolUse hook scripts to introspect which files and changes an edit tool is about to make:

```js
import { extractEditInputs, editedFilePaths, isEditTool } from '<team-kit>/common/edit-tools.mts';

// In a hook script, read the event from stdin:
const event = JSON.parse(await readStdin());
if (isEditTool(event.tool_name)) {
  const edits = extractEditInputs(event.tool_name, event.tool_input);
  const files = editedFilePaths(edits);
  // ... inspect edits or files
}
```

---
> Source: [microsoft/vscode-team-kit](https://github.com/microsoft/vscode-team-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
