## app-struct

> - The [src/app.rs](mdc:src/app.rs) file defines the `App` struct, which represents an installed Android application.

# App Struct Reference

- The [src/app.rs](mdc:src/app.rs) file defines the `App` struct, which represents an installed Android application.
- Fields:
  - `package_name`: The unique package identifier of the app
  - `app_name`: The display name of the app
- The `App` struct is used throughout [src/adb_client.rs](mdc:src/adb_client.rs) for listing, selecting, and operating on installed apps.

---
> Source: [cesarferreira/dab](https://github.com/cesarferreira/dab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
