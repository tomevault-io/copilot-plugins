## nulua

> The following tools are used for NuLua development:

# CONTRIBUTING.md

## Dependencies

The following tools are used for NuLua development:

- [Visual Studio Code](https://code.visualstudio.com/) / [Rider](https://www.jetbrains.com/rider/)
- [Unity Editor](https://unity3d.com/unity/editor) (optional)
- .NET SDK and runtimes (11.0 or later)
- [CSharpier](https://github.com/belav/csharpier) (formatter)

## Project Structure

### `src/`

All C# source code is located in the `src/` directory. Packages containing binaries for each runtime are in `src/NuLua.Runtime.*`, bindings for each runtime are in `src/NuLua.Interop.*`, and common abstractions and high-level APIs are in `src/NuLua`.

### `native/`

All bindings are automatically generated using [bindgen](https://github.com/rust-lang/rust-bindgen) and [csbindgen](https://github.com/Cysharp/csbindgen). Do not edit the generated files directly.

### `submodules/`

This directory contains the source code of Lua runtimes used for generating bindings and building native binaries.

### `tools/CodeGen/`

To standardize the `ILuaState` implementation across multiple versions, most of the implementation is automatically generated from templates using [Scriban](https://github.com/scriban/scriban). Do not edit the generated files directly.

## Build & Test

```bash
dotnet build -c Release
dotnet test -c Release
```

## Issue/Pull Request Guidelines

- Do not create a pull request that includes multiple changes. Create a separate pull request for each change or fix.
- We accept corrections for typos, but please do not submit pull requests solely to tweak minor phrasing unless there is a critical misunderstanding of the meaning.
- 

## Publishing Packages

Publishing to NuGet is handled by a workflow via GitHub Actions. This will run whenever you push a `v.*.*.*` tag. Before this runs, make sure that the versions of `Directory.Build.props`. Otherwise, the CI will fail.

---
> Source: [nuskey8/NuLua](https://github.com/nuskey8/NuLua) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-23 -->
