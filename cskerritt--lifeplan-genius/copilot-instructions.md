## lifeplan-genius

> Guidelines for working with Django templates and HTML files

## Django Template Coding Guidelines for HTML files

- Indent templates with two spaces.
- Use standard Django template syntax.
- Use translation markup, usually `translate` or `blocktranslate trimmed` with user-facing text.
  Don't forget to `{% load i18n %}` if needed.
- Use the Django `{% static %}` tag for loading images and external JavaScript / CSS files.
- Break re-usable template components into separate templates with `{% include %}` statements.
  These normally go into a `components` folder.
- Use DaisyUI styling markup for available components. When not available, fall back to standard TailwindCSS classes.
- Stick with the DaisyUI color palette whenever possible.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cskerritt) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
