## ui-icons

> Guidance for working in the `ui_icons` Drupal module (project `drupal/ui_icons`).

# CLAUDE.md

Guidance for working in the `ui_icons` Drupal module (project `drupal/ui_icons`).

## What this is

Generic icon manager for Drupal. It does not implement the icon system itself:
the Icon API lives in core since 11.1 (`Drupal\Core\Theme\Icon\*`, plugin
manager `plugin.manager.icon_pack`). This module adds the UI on top: a form
element to search and configure an icon, plus integrations (field, media, menu,
CKEditor 5, text filter, UI Patterns, Canvas).

Current branch line is **1.1.x**, Drupal 11.1+. The 1.0.x line shipped an API
backport, here `ui_icons_backport` and `ui_icons_iconify_api` are empty
placeholders kept only for upgrade paths. Do not add code to them.

Icon packs are declared by any extension in `*.icons.yml`. An icon id is
`pack_id:icon_id`, built with `IconDefinition::createIconId()`. Extractors
`path`, `svg`, `svg_sprite` are core, `font` is provided here.

## Layout

The root module is only the autocomplete form element. Everything else is a
submodule.

| Path | Content |
| --- | --- |
| `src/Element/IconAutocomplete.php` | The `icon_autocomplete` form element. Central piece, rebuilds itself and the extractor settings sub-form through ajax on `autocompleteclose change`. |
| `src/IconSearch.php` | Search service `ui_icons.search`, backs the autocomplete route. |
| `src/IconPreview.php`, `src/Template/IconPreviewTwigExtension.php` | Preview rendering, exposed to Twig as `icon_preview()`. |
| `src/Controller/` | Ajax endpoints, see `ui_icons.routing.yml` (`/ui-icons/ajax/autocomplete/icons`, `/ui-icons/ajax/preview/icons`). |
| `templates/icon-selector.html.twig` | Markup of the element. Provides the `.ui-icons-preview-icon` and `.ui-icons-settings-wrapper` hooks used by tests and themes. |
| `css/*.icon.autocomplete.css` | Per admin theme overrides (gin, dsfr, daisyui), picked by `ui_icons_preprocess_icon_selector()` via `_ui_icons_is_theme_active()`. |

Submodules worth knowing:

- `ui_icons_field`: field type `ui_icon`, widgets `icon_widget` / `icon_link_widget`, formatters `icon_formatter` / `icon_link_formatter`.
- `ui_icons_picker`: `icon_picker` element, a modal grid picker alternative to the autocomplete.
- `ui_icons_text`: `icon_embed` filter, turns `<drupal-icon data-icon-id data-icon-settings>` into rendered markup.
- `ui_icons_ckeditor5`: toolbar button and widget producing that `<drupal-icon>` tag. JS source in `js/ckeditor5_plugins/icon/src/`, bundle committed in `js/build/icon.js` (webpack, rebuild after editing source).
- `ui_icons_canvas`, `ui_icons_media`, `ui_icons_menu`, `ui_icons_patterns`, `ui_icons_font`, `ui_icons_library`: integrations, one concern each.

## Tests

PHPUnit tests live in `tests/src/{Unit,Kernel,Functional}` per module and carry
`#[Group('ui_icons')]`, which is what CI filters on. Fixtures come from
`tests/modules/ui_icons_test`, its packs (`test_path`, `test_svg`,
`test_settings`, ...) are declared in `ui_icons_test.icons.yml`. Templates and
setting defaults there are asserted verbatim by tests, so changing that file
breaks tests on purpose.

Make targets run from the project root, they wrap ddev:

```
make unit / kernel / functional / func-js    # phpunit testsuites
make phpunit F=path/to/Test.php              # single file
```

Playwright covers the browser flows. Specs in `tests/src/Playwright/Tests`,
fixtures and the `Drupal` page object in `tests/src/Playwright/{fixtures,objects}`.

```
npm run test          # firefox, @base tagged tests
npm run test:headed   # same, visible browser
npx playwright test -c playwright.local.config.ts tests/src/Playwright/Tests/ckeditor.spec.ts --project=chromium
```

The local config boots a PHP built-in server, and each worker installs its own
Drupal with `core/scripts/test-site.php` driven by `.env` (copy `.env.dist`).
Test setup is done per test with `drupal.installModules()` and
`drupal.drush('php:eval "..."')`, so keep it idempotent: tests in a worker share
one site.

CKEditor 5 icon coverage is Playwright only (`Tests/ckeditor.spec.ts`), the old
`FunctionalJavascript/IconPluginTest.php` was removed.

## QA

Config is local to the module: `.phpcs.xml` (Drupal + DrupalPractice + strict
types required), `phpstan.neon` (level 5, with a baseline), `.phpmd.xml`,
`.cspell.json`. From the project root: `make qa` runs the PHP and lint set,
`make phpcs F=... S=web/modules/custom/ui_icons`, `make phpstan`.

`phpstan.neon` excludes files depending on contrib that is not always installed
(ui_patterns, canvas, link_attributes). If you add such a file, add it there too.

## Conventions

- `declare(strict_types=1);` everywhere, enforced by phpcs.
- Root module uses procedural hooks in `ui_icons.module`, newer submodules use
  `#[Hook]` classes in `src/Hook/`. Follow whatever the submodule already does.
- Icon settings are always keyed by pack id (`[$pack_id => $settings]`) when
  passed as `#default_settings`. Flattening them silently empties the extractor
  sub-form.
- Never hand write markup for an icon, render through the icon pack template.

---
> Source: [Beyris-eu/ui-icons](https://github.com/Beyris-eu/ui-icons) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-15 -->
