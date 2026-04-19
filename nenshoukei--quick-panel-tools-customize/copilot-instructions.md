## quick-panel-tools-customize

> - Update `info.json` version number following semantic versioning.


# Version Control and Migrations

## Version Management

- Update `info.json` version number following semantic versioning.
- Document breaking changes in changelog.
- Use migration files for data structure changes.

## Changelog

- Maintain a `changelog.txt` file documenting all changes. It has strict format rules.
- Each version section starts with a line of 99 dashes.
- The following line must be a version line, `Version: x.y.z`. It cannot be empty.
- Follow semantic versioning for version numbers.
- Date `Date: ` line is optional. Date format can be any format, but official example uses `24. 12. 2012` format. (`DD. MM. YYYY`)
- Category `  Category:` line must start with 2 spaces and have a category name. The line ends with a colon.
    - Some categories are recognized by the game and used for sorting: `Major Features`, `Features`, `Minor Features`, `Graphics`, `Sounds`, `Optimizations`, `Balancing`, `Combat Balancing`, `Circuit Network`, `Changes`, `Bugfixes`, `Modding`, `Scripting`, `Gui`, `Control`, `Translation`, `Debug`, `Ease of use`, `Info`, `Locale`, `Compatibility`
- Entry `   -` line must start with 4 spaces followed by a dash followed by another space.
- Document breaking changes, new features, and bug fixes.

### Example

```
---------------------------------------------------------------------------------------------------
Version: 1.1.60
Date: 06. 06. 2022
  Features:
    - This is an entry in the "Features" category.
    - This is another entry in the "Features" category.
    - This general section is the 1.1.60 version section.
  Balancing:
    - This is a multiline entry in the "Balancing" category.
      There is some extra text here because it is needed for the example.
      Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
  Bugfixes:
    - Fixed that canceling syncing mods with a save would exit the GUI.
    - Fixed a desync when fast-replacing burner generators.
---------------------------------------------------------------------------------------------------
Version: 1.1.59
Date: 06. 05. 2022
  Bugfixes:
    - This general section is the 1.1.59 version section.
    - This is an entry in the "Bugfixes" category.
    - Fixed grenade shadows.
---------------------------------------------------------------------------------------------------
Version: 0.1.0
Date: 24. 12. 2012
  Major Features:
    - Initial release.
    - This general section is the 0.1.0 version section.
```

## Migrations

- Create migration scripts in `migrations/` directory.
- Name migration files with version numbers: `migration-1.0.0.lua`.
- Use `script.on_configuration_changed()` for non-destructive updates.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nenshoukei) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
