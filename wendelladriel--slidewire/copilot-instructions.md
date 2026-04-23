## slidewire

> Run from package root:

# Guidelines

## Package Quality Gates

Run from package root:

```bash
composer lint
composer test
composer test:browser
```

What each command does:

- `composer lint`: Pint + Rector
- `composer test`: lint checks + type coverage + unit/feature tests (browser excluded)
- `composer test:browser`: browser suite (Playwright + Pest Browser)

## Browser Prerequisites

For browser tests, make sure Playwright browser is installed:

```bash
npm install
npx playwright install chromium
```

## Test Coverage

The test suite covers all documented package features:

| Feature | Test Type | Test File(s) |
|---------|-----------|-------------|
| Scaffold command (`make:slidewire`) | Feature | `MakeSlideCommandTest.php` |
| Presentation rendering | Feature | `PresentationDeckComponentTest.php`, `PresentationDeckExpandedTest.php` |
| Controls, progress, and fullscreen toggles | Feature | `ControlsToggleTest.php` |
| SlideWire helper and route compatibility | Feature | `SlideWireHelperTest.php` |
| Route macro registration | Feature | `RouteMacroTest.php` |
| Markdown slide rendering | Feature | `MarkdownComponentTest.php` |
| Code component rendering | Feature | `CodeComponentTest.php` |
| Diagram component rendering | Feature | `DiagramComponentTest.php` |
| Code highlighting and theme resolution | Feature/Unit | `CodeHighlightIntegrationTest.php`, `CodeHighlighterTest.php`, `CodeHighlighterExpandedTest.php`, `CodeHighlighterThemeResolutionTest.php` |
| Code block preprocessing | Unit | `CodeBlockPrecompilerTest.php` |
| Slide context resolution | Unit | `SlideContextTest.php` |
| Compiler: transitions and auto-slide | Unit | `PresentationCompilerTransitionAndAutoSlideTest.php` |
| Compiler: backgrounds | Unit | `PresentationCompilerBackgroundTest.php` |
| Compiler: vertical slides | Unit | `PresentationCompilerVerticalSlideTest.php` |
| Compiler: precedence | Unit | `PresentationCompilerPrecedenceTest.php` |
| Compiler: error handling and edge cases | Unit | `PresentationCompilerExpandedTest.php` |
| Path resolution | Unit | `PresentationPathResolverTest.php`, `PresentationPathResolverExpandedTest.php` |
| Effective settings resolution | Unit | `EffectiveSettingsResolverTest.php` |
| Theme and typography resolution | Unit | `ThemeResolverTest.php` |
| Theme schema structure | Unit | `ThemeSchemaTest.php` |
| Config validation | Unit | `ConfigValidatorTest.php` |
| Architecture constraints | Unit | `ArchitectureTest.php` |
| Browser navigation | Browser | `PresentationNavigationBrowserTest.php` |
| Keyboard navigation | Browser | `KeyboardNavigationBrowserTest.php` |
| Auto-slide behavior | Browser | `AutoSlideBrowserTest.php` |

## Validation Flow

For a full validation of all features, run in order:

```bash
composer lint
composer test
composer test:browser
```

All three must pass. Fix any failures and rerun until green.

## Troubleshooting

- If updates do not appear, clear caches:

```bash
php artisan optimize:clear
```

- If browser tests fail locally, verify Playwright install and Chromium binaries.

---
> Source: [WendellAdriel/slidewire](https://github.com/WendellAdriel/slidewire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
