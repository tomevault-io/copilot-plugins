## swiftmail-project

> SwiftMail project-specific coding standards and organization rules

 # SwiftMail Project Rules

## Code Organization

- Protocol conformances for types should always be in a separate file named `Type+Protocol.swift`
- Group extensions next to their model definitions in the file structure
- Move files on the file system rather than generating them unnecessarily
- Maintain a clear directory structure:
  - `Sources/SwiftMail/IMAP/` - Code related to IMAP
  - `Sources/SwiftMail/SMTP/` - Code related to SMTP
  - `Sources/SwiftMail/Core/` - Code common to IMAP and SMTP implementations
  - `Demos` - Demo Apps
  - `Tests` - Test files

---
> Source: [Cocoanetics/SwiftMail](https://github.com/Cocoanetics/SwiftMail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
