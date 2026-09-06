## streamlit-shadcn-ui

> 未经用户明确允许，不得修改或覆盖 shadcn 的默认样式。

# Repository instructions

## Preserve shadcn default styles

未经用户明确允许，不得修改或覆盖 shadcn 的默认样式。
Do not modify or override shadcn's default styles without the user's explicit
permission for the specific change.

- Use the preset and generated source pinned in
  `streamlit_shadcn_ui/frontend_v2/components.json` and
  `streamlit_shadcn_ui/frontend_v2/src/components/ui/` as the reference.
  Do not replace them with a different preset or newer registry to change
  appearance without permission.
- This rule applies to generated component source, registry data and generation
  scripts, Streamlit adapters, Elements renderers, shared CSS, theme tokens,
  and documentation examples. Leaving generated source untouched is not enough.
- Do not override control dimensions, padding, typography, text alignment,
  colors, borders, radii, shadows, focus rings, or interaction styles through
  `className`, `style`, CSS selectors, inherited styles, or theme variables.
- Compose the existing shadcn components and use their documented variants and
  size options. Do not redefine those options, change their defaults, or add
  cosmetic overrides. Keep surrounding page layout separate from control styles.
- A request to add a component, adapt it to Streamlit, improve touch usability,
  or publish a release does not itself authorize changing shadcn defaults.
  If a style change is necessary, explain the affected components, current
  defaults, and proposed changes, then obtain explicit permission. Reuse
  permission already given for that scope; do not ask again unnecessarily.

Before shipping component changes, inspect adapter and CSS overrides as well
as generated source. Compare the rendered component with the corresponding
shadcn controls, including dimensions and interaction states. Run the existing
generated-source checks and relevant tests. A passing generated-source check
alone does not prove that adapters preserve the default appearance.

## Release and documentation

The main repository is the source of truth for `Home.py`, `site_pages/`,
`pages/`, and `docs/components/`. Sync relevant changes to the documentation
repository and pin it to the published PyPI version. Verify the installed
package and the live documentation separately. Do not report an online update
as complete based only on a successful commit or push.

Use semantic commit messages and leave unrelated user changes out of commits.

---
> Source: [ObservedObserver/streamlit-shadcn-ui](https://github.com/ObservedObserver/streamlit-shadcn-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
