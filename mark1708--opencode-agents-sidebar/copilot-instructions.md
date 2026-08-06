## opencode-agents-sidebar

> **Version:** 1.0.7 | **npm:** `opencode-agents-sidebar` | **Plugin ID:** `agents-sidebar:tui`

# AGENTS SIDEBAR PLUGIN

**Version:** 1.0.7 | **npm:** `opencode-agents-sidebar` | **Plugin ID:** `agents-sidebar:tui`

## OVERVIEW

Displays OhMyOpenAgent agent lifecycle categories, collapsible sections, model info, and team status in sidebar. Reads agent config from `oh-my-openagent.json`.

## STRUCTURE

```
agents-sidebar/
├── src/
│   ├── index.ts    # Plugin API stub (definePlugin)
│   ├── tui.ts      # Re-exports from all modules + render.ts default
│   ├── render.ts   # TUI entry -- element/render/poll loop (uses @opentui/solid)
│   ├── agents.ts   # Agent discovery, category mapping, group building
│   ├── config.ts   # Reads ~/.config/opencode/oh-my-openagent.json
│   ├── defaults.ts # DEFAULTS object with all config fallbacks
│   ├── format.ts   # Agent name/model/category formatting
│   └── types.ts    # Core types: AgentEntry, CategoryGroup, OmoConfig, SidebarConfig
├── assets/         # Screenshots for README
├── .gitlab-ci.yml  # CI: validate -> build -> publish
└── package.json
```

## WHERE TO LOOK

| Task | File |
|------|------|
| Add agent fields | `src/types.ts` (`AgentEntry`) + `src/agents.ts` (mapping logic) |
| Change category logic | `src/agents.ts` -- `categorizeAgents()`, `DEFAULT_CATEGORY` |
| Display formatting | `src/format.ts` -- agent row rendering, name truncation |
| Config options | `src/types.ts` (`SidebarConfig`) + `src/defaults.ts` (`DEFAULTS`) |
| Config parsing | `src/config.ts` -- `mergeConfig()`, type-safe value extractors |
| TUI rendering | `src/render.ts` -- sidebar layout, sections, keybind handling |

## CONVENTIONS (THIS REPO)

- **Config source**: Single file `~/.config/opencode/oh-my-openagent.json` with `tui` sub-key for sidebar config
- **Type-safe parsing**: `isRecord()`, `stringValue()`, `numberValue()`, `booleanValue()`, `enumValue()` -- no `as` casts
- **Agent categories**: Defined in `defaults.ts` (`DEFAULT_CATEGORY_ORDER`), users override via `tui.category_order`
- **Alias maps**: `provider_aliases`, `model_aliases`, `variant_aliases` for display name overrides
- **Model display modes**: `"inline"` (default), `"details-only"`, `"hidden"`
- **Show disabled modes**: `"hidden"`, `"dimmed"`, `"grouped"`
- **Config fingerprint**: `configFingerprint()` for change detection in poll loop

## ANTI-PATTERNS

- NEVER add runtime dependencies beyond peer deps (`@opencode-ai/plugin`, `@opentui/solid`, `solid-js`)
- NEVER import `@opentui/solid` outside `render.ts`
- NEVER mutate `DEFAULTS` object -- always spread into new objects
- NEVER use `text()` for interactive elements -- use `box()` with `onMouseDown`
- All format output MUST be single-line, truncate-safe

---
> Source: [Mark1708/opencode-agents-sidebar](https://github.com/Mark1708/opencode-agents-sidebar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
