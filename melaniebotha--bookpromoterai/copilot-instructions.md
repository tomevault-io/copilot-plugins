## bump-app-version

> Bump app version in BookPromoterAI.csproj whenever shipping code changes


# Bump app version on every change

When you modify the BookPromoter AI app and commit or push, **always bump the version** in `BookPromoterAI/BookPromoterAI.csproj`:

```xml
<Version>1.28.1</Version>
<AssemblyVersion>1.28.1.0</AssemblyVersion>
<FileVersion>1.28.1.0</FileVersion>
<InformationalVersion>1.28.1</InformationalVersion>
```

## Version numbering (patch +1 only)

**Every release increments the patch number by exactly 1.** Do not skip numbers and do not bump minor/major for routine shipping.

- 1.28.0 → **1.28.1** → **1.28.2** → **1.28.3** …
- Same rule for any line: 1.24.4 → 1.24.5, not 1.25.0

Only bump **minor** (e.g. 1.28.9 → 1.29.0) when the user explicitly asks to start a new minor line. Only bump **major** when the user explicitly asks for a major release.

Also add a matching entry in `BookPromoterAI/ReleaseNotes.json` keyed by the new version string.

Do **not** edit hardcoded footer text. Version is read automatically from the assembly via `AppVersion.Display` in `AppVersion.cs` and shown in app/marketing footers and the Owner page.

Include the version bump in the same commit as the functional changes.

---
> Source: [MelanieBotha/BookPromoterAI](https://github.com/MelanieBotha/BookPromoterAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
