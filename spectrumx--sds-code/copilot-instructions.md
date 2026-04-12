## sds-code

> - Always define new Javascript functions inside separate .js files rather than inside of django template files.


# Frontend Development Rules

## Javascript Structure

- Always define new Javascript functions inside separate .js files rather than inside of django template files.
- Ask explicitly to create new Javascript files and explain their use and list the functions that would be created.
- When creating Javascript functions, they should be included as part of either: existing classes within `actions/`, `constants/`, `core/`, `dataset/`, `search/`, `share/`, etc. OR within NEW classes that may be added to follow the existing folder structure.
- New classes should be created in their OWN file inside a subfolder of `js/`, NOT the base `js/` folder.
- Classes should follow DRY principles and avoid duplicating the functionality of existing classes.
- Javascript added to existing templates should ONLY call existing classes to initialize them so their functions are available to run in the template.
- Ignore and NEVER update files in the `deprecated/` folder.

## Django Template Structure

- Django templates within higher folders should represent pages first, but the page components (large, nested elements) should be broken out into `partial` files (under a `partials/` folder) and those partials should then be loaded in the page template.
- Template files in `partials/` should NOT include script tags. Those should be loaded in at the page level.
- The `components/` folder is meant to house dynamically rendered HTML fragments
- Templates in `partials/` and `components/` should represent general, reusable design elements than can be used across multiple templates.
- Always ask before creating new template files and explain their use and why an existing one CANNOT be used.

## Dynamic HTML Rendering

- HTML fragments that may change throughout the page lifecycle should be created as django template files under `components/`
- Any variables within those components should be handled in the Javascript handlers/managers and passed as context objects to the Django view `RenderHTMLFragmentView` which renders the Django template files in `components/`using a context object passed into a Javascript `APIClient` call.
- Never try to render HTML fragments in Javascript.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/spectrumx)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/spectrumx)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
