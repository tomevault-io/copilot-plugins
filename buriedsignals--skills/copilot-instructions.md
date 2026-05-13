## skills

> marketplace.json        # Marketplace catalog

# Skills Marketplace

## Repo layout

```
.claude-plugin/
  marketplace.json        # Marketplace catalog
osint/                    # Plugin: OSINT toolkit
  .claude-plugin/
    plugin.json           # Plugin manifest (name, description, version)
  skills/
    osint/
      SKILL.md            # Skill content (becomes /osint command)
      references/
storytelling/             # Plugin: Scrollytelling toolkit
  .claude-plugin/
    plugin.json
  skills/
    scrolly/
      SKILL.md            # Becomes /scrolly command
      references/
```

## Adding a new plugin

1. Create the plugin directory at repo root (e.g. `my-plugin/`)
2. Add `.claude-plugin/plugin.json` inside it:
   ```json
   {
     "name": "my-plugin",
     "description": "What it does",
     "version": "1.0.0"
   }
   ```
3. Add skills under `my-plugin/skills/<skill-name>/SKILL.md`
4. Add an entry to `.claude-plugin/marketplace.json`:
   ```json
   {
     "name": "my-plugin",
     "source": "./my-plugin",
     "description": "What it does",
     "author": { "name": "Buried Signals" },
     "category": "category-name"
   }
   ```
5. Bump the version in `plugin.json` for updates to propagate

## Important conventions

- Plugin dirs MUST live at repo root — NOT under a `plugins/` subdirectory (this broke installs for 7 attempts)
- The `name` in marketplace.json, the `name` in plugin.json, the directory name, and the `source` path must ALL match (e.g. name: `"osint"`, source: `"./osint"`, directory: `osint/`)
- Source paths must start with `./` (e.g. `"./my-plugin"`)
- The skill directory name becomes the slash command (e.g. `skills/osint/` → `/osint`)
- Bump `version` in plugin.json to trigger auto-updates for installed users

## Verifying a new plugin

After committing and pushing:

1. `/plugin marketplace remove buriedsignals`
2. `/plugin marketplace add buriedsignals/skills`
3. Restart Claude Code (marketplace index caches per session)
4. `/plugin install <name>@buriedsignals`
5. Restart Claude Code again to load the plugin
6. `/<skill-name>` should appear in available skills

## User install experience

```
/plugin marketplace add buriedsignals/skills
/plugin install osint@buriedsignals
/plugin install storytelling@buriedsignals
```

---
> Source: [buriedsignals/skills](https://github.com/buriedsignals/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
