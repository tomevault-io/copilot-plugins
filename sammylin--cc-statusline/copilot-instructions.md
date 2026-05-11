## cc-statusline

> Bump **all three** version fields together. They feed different consumers and

# cc-statusline — Notes for Claude

## Release checklist

Bump **all three** version fields together. They feed different consumers and
drift causes silent breakage:

- `package.json` → `version`
- `.claude-plugin/plugin.json` → `version`
- `.claude-plugin/marketplace.json` → `plugins[0].version`

Why: Claude Code caches installed plugins by version under
`~/.claude/plugins/cache/cc-statusline/cc-statusline/<version>/`. If the
manifest version doesn't change, `claude plugin update` doesn't refresh the
cache — users keep running whatever they first installed.

## Hook paths

In `hooks/hooks.json` (plugin-mode registration): use
`${CLAUDE_PLUGIN_ROOT}/hooks/<name>.js`. Never write `~/.claude/hooks/` here —
that directory only exists under the legacy manual-install flow documented in
README Option B, and under plugin mode it will always `MODULE_NOT_FOUND`.

README **Option B — manual** is the one place `~/.claude/hooks/*.js` paths are
correct, because that flow literally copies files there. Don't "fix" those.

## CI

`test/release-invariants.test.js` enforces both rules above. If you edit
versions or hook paths, run `npm test` before pushing.

---
> Source: [SammyLin/cc-statusline](https://github.com/SammyLin/cc-statusline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
