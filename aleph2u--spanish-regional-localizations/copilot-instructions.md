## spanish-regional-localizations

> This is a Flutter package that provides Material Design localizations for Spanish regional languages:

# Spanish Regional Localizations - Claude Context

## Project Overview
This is a Flutter package that provides Material Design localizations for Spanish regional languages:
- **Euskera (Basque)** - locale code: `eu`
- **Galician (Gallego)** - locale code: `gl`

## Project Structure
```
spanish_regional_localizations/
├── lib/
│   ├── spanish_regional_localizations.dart  # Main library export file
│   └── src/
│       ├── material_localizations_eu.dart   # Euskera translations
│       ├── material_localizations_gl.dart   # Galician translations
│       └── delegates.dart                   # Localization delegate
├── example/
│   └── main.dart                           # Example app demonstrating usage
├── test/
│   └── spanish_regional_localizations_test.dart  # Unit tests
├── pubspec.yaml                            # Package configuration
├── README.md                               # User documentation
├── CHANGELOG.md                            # Version history
└── LICENSE                                 # MIT License
```

## Key Implementation Details

### MaterialLocalizations Implementation
Both `MaterialLocalizationEu` and `MaterialLocalizationGl` extend `MaterialLocalizations` and implement:
- All UI strings (buttons, dialogs, tooltips, etc.)
- Date/time formatting methods
- Month and weekday names in native languages
- Number formatting and pluralization
- Keyboard key labels

### Important Methods with Parameters
These methods require parameters (not getters):
- `aboutListTileTitle(String applicationName)`
- `licensesPackageDetailText(int licenseCount)`
- `pageRowsInfoTitle(int firstRow, int lastRow, int rowCount, bool rowCountIsApproximate)`
- `remainingTextFieldCharacterCount(int remaining)`
- `selectedRowCountTitle(int selectedRowCount)`
- `tabLabel({required int tabIndex, required int tabCount})`
- `dateRangeEndDateSemanticLabel(String formattedDate)`
- `dateRangeStartDateSemanticLabel(String formattedDate)`
- `scrimOnTapHint(String modalRouteContentName)`
- `formatDecimal(int number)`
- `timeOfDayFormat({bool alwaysUse24HourFormat = false})`

### Translation Guidelines
- **Euskera**: Uses traditional Basque UI terminology, uppercase for main action buttons (ADOS, UTZI, GORDE)
- **Galician**: Uses standard Galician UI terminology, uppercase for main action buttons (ACEPTAR, CANCELAR, GARDAR)
- Both languages use Monday as the first day of the week
- Date formats follow European convention (dd/mm/yyyy)

## Development Commands

### Run Tests
```bash
flutter test
```

### Run Example App
```bash
cd example
flutter run
```

### Publish Package (when ready)
```bash
flutter pub publish --dry-run  # Test publication
flutter pub publish            # Actual publication
```

## Translation Resources
- **Euskera**: Microsoft Language Portal (eu-es), euskera.eus
- **Galician**: Xunta de Galicia translator, galego.org

## Dependencies
- flutter_localizations (from SDK)
- intl: ^0.20.2

## Notes for Future Development
- The package is ready for publication but needs the homepage URL in pubspec.yaml to be updated with the actual GitHub repository
- All tests are passing
- The example app demonstrates language switching and common UI elements
- Consider adding Catalan (ca) and Asturian (ast) in future versions

---
> Source: [aleph2u/spanish_regional_localizations](https://github.com/aleph2u/spanish_regional_localizations) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
