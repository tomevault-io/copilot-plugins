## herdr-web

> This plugin runs as a persistent background process declared via `[[startup]]` in

# herdr-web-plugin — agent instructions

## Always restart the local install after code changes

This plugin runs as a persistent background process declared via `[[startup]]` in
`herdr-plugin.toml` (`node server.js`). That process is spawned once, when herdr's own server
starts — **not** on `herdr plugin enable`/`disable`. Confirmed empirically against a sibling
plugin (herdr 0.8.2, same startup mechanism): `herdr plugin disable`/`enable` reports success but
does not kill or respawn the running startup process. Code changes sit inert in an already-running
process until it's actually restarted.

Whenever you finish a change to this plugin's code (anything `server.js` loads), restart it
yourself as the last step, without being asked:

```bash
PLUGIN_DIR="$(cd "$(dirname "${BASH_SOURCE[0]:-$0}")" && pwd)"   # or hardcode the repo path
CONFIG_DIR="$(herdr plugin config-dir barnuri.herdr-web)"
pkill -f "node $PLUGIN_DIR/server\.js" 2>/dev/null
sleep 1  # give it a moment to actually exit before relaunching
HERDR_PLUGIN_CONFIG_DIR="$CONFIG_DIR" nohup node "$PLUGIN_DIR/server.js" >> "$CONFIG_DIR/server.log" 2>&1 &
disown
```

No restart needed for one-shot action scripts (`open`, `show-url`) or test/doc-only changes —
those run fresh on every invocation.

---
> Source: [barnuri/herdr-web](https://github.com/barnuri/herdr-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
