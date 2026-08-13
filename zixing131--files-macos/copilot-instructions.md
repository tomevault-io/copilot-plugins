## files-macos

> Files_MacOS is a C#/.NET 10 desktop file manager for macOS. It uses Uno Platform for the UI and a native Objective-C/AppKit bridge for macOS integration.

# Files_MacOS Development Guidelines

Files_MacOS is a C#/.NET 10 desktop file manager for macOS. It uses Uno Platform for the UI and a native Objective-C/AppKit bridge for macOS integration.

- Follow `.editorconfig` and keep changed text files in CRLF format.
- Protect context usage. Prefer targeted searches and cap commands with potentially large output.
- Never edit generated files under `bin` or `obj`.
- Keep changes scoped and avoid unrelated formatting or dependency churn.
- Treat file operations, permissions, drag/drop, clipboard, archives, trash, native menus, signing and localization as high-risk areas.
- For UI work, use existing XAML resources, controls, converters and localization resources.
- For native integration, extend `Native/FilesMacOSBridge.m` and `Interop/MacOSNativeMethods.cs` instead of adding separate ad hoc native libraries.
- Prefer Rider for C# and XAML. Use Xcode only when native bridge debugging or Apple signing tools require it.
- Use `Files.MacOS.slnx`; the repository does not contain or build the former Windows application.

## Structure

```text
src/Files.App.MacOS/
├── Controls/       Reusable UI controls
├── Converters/     XAML value converters
├── Interop/        Managed declarations for the native bridge
├── Models/         Application state and file models
├── Native/         Objective-C/AppKit bridge
├── Packaging/      App icon, plist and entitlements
├── Services/       File system and macOS services
├── Strings/        English and Simplified Chinese resources
└── ViewModels/     Browser and window view models
```

## Build

Build the current Mac architecture:

```bash
dotnet build src/Files.App.MacOS/Files.App.MacOS.csproj \
  -c Debug \
  -v:quiet \
  -clp:ErrorsOnly
```

Build a specific architecture:

```bash
dotnet build src/Files.App.MacOS/Files.App.MacOS.csproj \
  -c Debug \
  -r osx-arm64 \
  -v:quiet \
  -clp:ErrorsOnly
```

Do not run build commands in parallel. There is currently no separate automated test project; validate changes with the focused build and relevant runtime behavior.

## Commit

Before committing, run:

```bash
git status --short
git diff --check
```

Do not revert or stage unrelated user changes. Use a concise commit message describing the behavior change.

---
> Source: [zixing131/Files_MacOS](https://github.com/zixing131/Files_MacOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
