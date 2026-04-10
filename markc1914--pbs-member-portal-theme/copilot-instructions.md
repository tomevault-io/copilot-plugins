## pbs-member-portal-theme

> > For Claude, Codex, Gemini, Copilot, and other AI assistants

# AI Agent Instructions

> For Claude, Codex, Gemini, Copilot, and other AI assistants

## Quick Start

**Main CSS file:** `package/pbs-theme.css`

**What this is:** CSS theme for iMIS 20.2 member portal (Phi Beta Sigma Fraternity)

**Constraint:** CSS-only. No HTML/JS access.

## Essential Rules

1. **Always use `!important`** - Required to override iMIS base styles
2. **Edit only `package/pbs-theme.css`** - This is the deployed file
3. **Mobile styles go in `@media (max-width: 767px)` block**
4. **Commit to git after changes** - User deploys from GitHub

## Brand Colors

```css
--pbs-blue: #164F90;      /* Primary */
--pbs-blue-dark: #0d3a6a;
--pbs-blue-light: #2066ac;
--pbs-blue-accent: #6a9ac9;
--pbs-white: #FFFFFF;
```

## Key Selectors

| Element | Selector |
|---------|----------|
| Navigation | `.RadMenu`, `.RadMenu_Austin`, `.rmLink` |
| Tabs | `.RadTabStrip`, `.rtsLink` |
| Buttons | `.TextButton`, `.PrimaryButton` |
| Disabled | `.aspNetDisabled` |
| Panels | `.PanelHead`, `.PanelBody` |
| Sidebar | `#masterSideBarPanel` |
| Content | `#yui-main`, `#masterContentArea` |

## File Structure

```
package/
├── pbs-theme.css      ← EDIT THIS
├── ThemeSettings.xml
└── images/
    └── pbs-header2.png
```

## Responsive Breakpoints

- **991px** - Tablet
- **767px** - Mobile (stack columns)
- **480px** - Small mobile

## Common Fixes

### Remove bullet points
```css
#hd li, .NavigationListItem {
    list-style: none !important;
    padding-left: 0 !important;
}
```

### Full width on mobile
```css
@media (max-width: 767px) {
    #masterSideBarPanel, #yui-main {
        float: none;
        width: 100%;
    }
}
```

## Don't Do

- Don't use overly broad selectors like `#hd *` (breaks things)
- Don't forget `!important` (styles won't apply)
- Don't edit files outside `package/` folder

## See Also

- `CLAUDE.md` - Full documentation
- `testingScreenshots/` - Current state screenshots
- `PBSTheme_Current_Production.css` - Reference backup

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/markc1914)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/markc1914)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
