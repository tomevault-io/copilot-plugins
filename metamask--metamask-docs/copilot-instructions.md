## metamask-docs

> Product-specific guidance for Snaps documentation, including naming, content areas, and generated-content constraints.


# Snaps

## Naming

- **Snaps** (capital S) refers to the platform or ecosystem.
- **a Snap** (capital S, singular) refers to an individual extension.
- Do not use lowercase "snaps" or "snap" in prose (only in code identifiers or file paths).

## Content areas

Snaps uses a content structure that differs slightly from other products:

| Folder          | Type                  | Description                                   |
|-----------------|-----------------------|-----------------------------------------------|
| `learn/`        | Explanation           | Conceptual content, best practices, tutorials |
| `how-to/`       | How-to                | Procedural step-by-step guides                |
| `features/`     | Feature reference     | Feature-centric documentation                 |
| `reference/`    | API reference         | Snaps API and Keyring API methods             |
| `get-started/`  | Onboarding            | First-time setup and onboarding               |

## Generated reference pages

Reference pages under `snaps/reference/snaps-api/` are **generated** from a JSON schema by the
`plugin-snaps-docs.ts` build plugin. Do not edit these files directly. To update their content,
edit the source schema or the plugin in `src/plugins/plugin-snaps-docs.ts`.

## Experimental features

The `flaskOnly` admonition keyword is available in Snaps docs for features that require MetaMask
Flask. Use it like any other admonition:

```markdown
:::flaskOnly
This feature is experimental and only available in MetaMask Flask.
:::
```

## Sidebar

The sidebar configuration is in [snaps-sidebar.js](snaps-sidebar.js).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MetaMask)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MetaMask)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
