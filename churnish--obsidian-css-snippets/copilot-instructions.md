## obsidian-css-snippets

> - Do NOT update files in `archive/` to follow the rules below.

# snippets

- Do NOT update files in `archive/` to follow the rules below.
- Ensure a single blank line at file end.
- snippets/ is a git repo.

## File name

ALWAYS:

- Use kebab-case (all lowercase).
- Use format: `[add-on]-[faulty]-[device]-[target]-[mode]-name`.
	- `add-on` — plugin or theme name.
	- `faulty` — prefix for broken or non-functional snippets.
	- `device` — `desktop`, `mobile`, `phone`, or `tablet`.
	- `target` — UI area/element (e.g., `editor`, `sidebar`, `status-bar`).
	- `mode` — view mode or snippet type (e.g., `helper`, `reading-view`).
- ALL prefixes are optional.
- Target-only and name-only file names are valid.
- Multiple addons are permissible — order them alphabetically.

## Obsidian feature names

In comments (but not file names), ALWAYS use this exact capitalization:

- Live Preview
- Source view
- Reading view
- Quick switcher
- Command palette
- Bases
- Canvas
- Ribbon
- Properties [when referring to YAML frontmatter block at top of notes]

NEVER wrap these feature names in 'quotes'.

## Comments

- No articles (a/an/the).
- No contractions (e.g., 'do not' NOT 'don't').
- Drop linking verbs (is/are/was) when sentence remains clear without them.

## Metadata header

- Every CSS snippet MUST start with a single-line `/* <Description> */` comment summarizing what the snippet does.
- Add `<Add-on Name>:` prefix to description if plugin or theme is targeted. For multiple add-ons, join with ` + ` (e.g., `Baseline + Pane Relief:`).
- Optionally follow with `/* Source: ... */`, separated by a blank line.
- Add one blank line between header comments and CSS.
- When a comment starts with a lead-in followed by a colon (e.g., `Baseline: Set x to y`), the first word after the colon MUST be capitalized.
- Keep descriptions accurate and up-to-date — when edits change a snippet's behavior or scope, update the header to reflect it (without superfluous details).

## Snippet example

```css
/* Hide bookmark icon in tab title bar */

/* Source: https://github.com/churnish/obsidian-css-snippets */

.view-header .clickable-icon.view-action.mod-bookmark {
    display: none;
}
```

---
> Source: [churnish/obsidian-css-snippets](https://github.com/churnish/obsidian-css-snippets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
