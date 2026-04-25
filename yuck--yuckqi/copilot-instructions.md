## yuckqi

> Keep NuGet package readme.md files in sync with implementation changes


# NuGet Package Readme Convention

Every project under `src/` that sets `<GeneratePackageOnBuild>true</GeneratePackageOnBuild>` is a NuGet package and **must** have a `readme.md` in its project directory. This readme is packed into the NuGet package via `<PackageReadmeFile>readme.md</PackageReadmeFile>`.

## When to Update

Update the project's `readme.md` when any of the following changes are made:

- A public type is added, removed, or renamed
- A public type's purpose or behavior changes significantly
- A project dependency is added or removed (NuGet package or project reference)
- The project's target framework changes
- The package `<Description>` in the `.csproj` changes

Do **not** update the readme for internal/private changes, bug fixes that don't alter the public API, or minor refactors.

## Readme Structure

Follow this consistent structure (omit sections that don't apply):

```
# {PackageName}

{One-to-two sentence description matching the csproj <Description>.}

## Key Types

{Bulleted list of public interfaces, classes, and other notable types with a brief description of each.}

## Dependencies

{Links to NuGet package dependencies, both project references and external packages.}

## Installation

{dotnet add package command.}
```

## New Project Checklist

When creating a new NuGet package project:

1. Create a `readme.md` following the structure above
2. Add `<PackageReadmeFile>readme.md</PackageReadmeFile>` to the `<PropertyGroup>` in the `.csproj`
3. Add `<None Include="readme.md" Pack="true" PackagePath="\" />` to an `<ItemGroup>` in the `.csproj`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Yuck) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
