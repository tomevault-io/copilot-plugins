## practical-aspnetcore

> This guide helps a coding agent work efficiently with the **practical-aspnetcore** repository from the very first interaction.

# Coding Agent Onboarding Guide

This guide helps a coding agent work efficiently with the **practical-aspnetcore** repository from the very first interaction.

---

## What This Repository Is

A large, community-driven collection of **practical ASP.NET Core code samples** organized by topic. Each sample is intentionally small and focused on demonstrating one concept clearly. The repository is used as a learning resource for developers at all levels.

---

## Technology Stack

| Item | Value |
|------|-------|
| Runtime | .NET 10 (RC – see `global.json`) |
| SDK rollForward | `major` with `allowPrerelease: true` |
| Web SDK | `Microsoft.NET.Sdk.Web` |
| Language version | `preview` (`<LangVersion>preview</LangVersion>`) |
| Implicit usings | Enabled (`<ImplicitUsings>true</ImplicitUsings>`) |
| Target framework | `net10.0` |

The `global.json` at the repository root pins the SDK version. Any newer major SDK will also work because `rollForward` is set to `major`.

---

## Directory Layout

```
practical-aspnetcore/
├── projects/                  # All sample projects live here
│   ├── minimal-api/           # One directory per topic category
│   │   ├── README.md          # Index of samples in this category
│   │   ├── build.bat          # Optional batch helper
│   │   ├── hello-world/       # One sub-directory per sample
│   │   │   ├── Program.cs     # ALL application code goes here
│   │   │   ├── README.md      # Description of this sample
│   │   │   └── hello-world.csproj
│   │   └── ...
│   ├── mvc/
│   ├── blazor-wasm/
│   └── ...                    # 60+ topic categories
├── exercises/                 # Learning exercises (separate from samples)
├── scripts/                   # Utility scripts (e.g., upgrade-to-net10.ps1)
├── .github/
│   └── FUNDING.yml            # GitHub Sponsors config – no CI workflows
├── global.json                # SDK version pinning
├── README.md                  # Main index of all samples (update when adding)
├── CONTRIBUTING.md            # Contribution guidelines (read before adding samples)
├── CODE_OF_CONDUCT.md
└── skills-checklist.md        # WIP skills checklist
```

---

## Running a Sample

```bash
cd projects/<category>/<sample-name>
dotnet watch run
```

Open `http://localhost:5000` in a browser (or the port shown in the terminal).

There is **no solution file** and no global build command. Every sample is an independent project that is built and run individually.

---

## How to Add a New Sample

Follow these steps every time a new sample is created:

### 1. Create the sample directory

```
projects/<category-name>/<sample-name>/
```

Use lowercase kebab-case for both the category name and the sample name.

### 2. Create the `.csproj` file

Name the file `<sample-name>.csproj`. Use this minimal template:

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <ImplicitUsings>true</ImplicitUsings>
    <LangVersion>preview</LangVersion>
  </PropertyGroup>
</Project>
```

Add `<PackageReference>` entries inside `<ItemGroup>` only if the sample requires NuGet packages.

### 3. Write all code in `Program.cs`

**All application code must live in `Program.cs`.** This is the most important convention in this repository. Do not create additional `.cs` files, controllers, or class files unless it is absolutely impossible to demonstrate the concept in a single file (e.g., a Razor Pages `.cshtml` + code-behind pair).

Typical structure:

```csharp
// using statements if not covered by implicit usings

var builder = WebApplication.CreateBuilder(args);
// Register services
var app = builder.Build();
// Configure middleware / map routes
app.Run();
```

For the simplest samples `WebApplication.Create()` is enough (no explicit builder needed):

```csharp
WebApplication app = WebApplication.Create();
app.Run(async context =>
{
    await context.Response.WriteAsync("Hello world");
});
await app.RunAsync();
```

### 4. Create a `README.md` for the sample

Keep it concise. Include:
- A one-sentence description of what the sample demonstrates.
- Any notable code snippets if useful for comprehension.
- Links to relevant official documentation when helpful.

### 5. Update the category `README.md`

Add a line for the new sample to the category-level `README.md` (e.g., `projects/minimal-api/README.md`).

### 6. Update the root `README.md`

- Find the relevant section in the table and **increment the sample count**.
- Add a bullet point describing the sample in the appropriate section further down the file.

---

## Conventions & Rules (from `CONTRIBUTING.md`)

1. **All code in `Program.cs`** – makes it easy to read online without chasing types across files.
2. **Keep samples small and specific** – one concept per sample.
3. **No sample is too small** – if it shows one useful thing, it belongs.
4. **Update the README** and increment the sample count when adding a new sample.

---

## Project File Conventions

- **SDK**: Always `Microsoft.NET.Sdk.Web`
- **TargetFramework**: `net10.0`
- **ImplicitUsings**: `true` (so `Microsoft.AspNetCore.*` and other common namespaces are available without explicit `using` directives)
- **LangVersion**: `preview`
- **No `<Nullable>enable</Nullable>`** – samples typically omit this for brevity
- Project file is named after the sample directory (e.g., `hello-world.csproj` in `hello-world/`)

---

## No CI/CD Pipelines

There are **no GitHub Actions workflows** in this repository. The `.github/` directory contains only `FUNDING.yml`. Validation is done manually by running each sample locally with `dotnet watch run`.

---

## Common Pitfalls & Known Issues

| Issue | Workaround |
|-------|-----------|
| SDK version mismatch | `global.json` has `rollForward: major` so any .NET 10+ SDK works. If you only have .NET 8/9, some samples targeting `net10.0` won't build. |
| `allowPrerelease: true` required | The SDK version is a release-candidate. Ensure your local SDK installation includes preview/RC versions. |
| No solution file | There is no `.sln` file. Load individual `.csproj` files in your IDE or run `dotnet watch run` directly inside the sample directory. |
| Samples reference older APIs | A few older samples still use `IHostBuilder` patterns. New samples should use the modern `WebApplication` / `WebApplicationBuilder` pattern. |
| Implicit usings | Many `using` statements (e.g., `Microsoft.AspNetCore.Builder`, `Microsoft.AspNetCore.Http`) are provided by implicit usings and should not need to be explicitly listed in new samples, though older samples may still include them. |

---

## Useful Reference Samples

| Goal | Sample location |
|------|----------------|
| Simplest possible web app | `projects/minimal-api/hello-world/` |
| Using builder pattern | `projects/minimal-hosting/` |
| Razor Pages | `projects/razor-pages/` |
| MVC controllers | `projects/mvc/` |
| gRPC | `projects/grpc/` |
| Blazor WASM | `projects/blazor-wasm/` |
| Blazor SSR | `projects/blazor-ssr/` |
| HTMX integration | `projects/htmx/` |
| Generic Host (non-web) | `projects/generic-host/` |
| Dependency Injection | `projects/dependency-injection/` |

---

## Summary Checklist for Adding a Sample

- [ ] Create `projects/<category>/<sample-name>/` directory
- [ ] Add `<sample-name>.csproj` with `net10.0` target
- [ ] Put ALL code in `Program.cs`
- [ ] Add `README.md` describing the sample
- [ ] Update `projects/<category>/README.md`
- [ ] Increment count and add bullet in root `README.md`
- [ ] Verify with `dotnet watch run` inside the sample directory

---
> Source: [dodyg/practical-aspnetcore](https://github.com/dodyg/practical-aspnetcore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
