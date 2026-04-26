## blazor-tailwind

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a .NET 10.0 Blazor Web Application with Tailwind CSS integration. It uses server-side interactive rendering with Razor components.

## Development Commands

```bash
# Run the application (starts on http://localhost:5172)
dotnet run

# Build the project
dotnet build

# Build for production
dotnet publish -c Release

# Clean and rebuild
dotnet clean && dotnet build
```

## Architecture

- **Rendering Mode**: Server-side interactive Blazor (`AddInteractiveServerComponents`)
- **Styling**: Tailwind CSS v4 with automatic compilation via `Tailwind.Hosting.Build`
  - Input: `wwwroot/tailwind.css`
  - Output: `wwwroot/app.css` (auto-generated, do not edit manually)

### Key Directories

- `Components/Pages/` - Routable page components with `@page` directives
- `Components/Layout/` - Layout components (MainLayout, NavMenu, ReconnectModal)
- `wwwroot/` - Static assets and CSS files

### Component Structure

- `App.razor` - Root HTML shell, includes CSS and Blazor script references
- `Routes.razor` - Routing configuration
- `Program.cs` - Application startup, DI configuration, middleware pipeline

## Tailwind CSS

Tailwind is integrated via NuGet packages (`Tailwind.Hosting` and `Tailwind.Hosting.Build`). The build system automatically:
- Watches for changes during development
- Compiles `tailwind.css` to `app.css` on build

Write Tailwind classes directly in Razor components. No manual CSS compilation step needed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aikiw2021325) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
