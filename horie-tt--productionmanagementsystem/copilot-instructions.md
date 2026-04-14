## productionmanagementsystem

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Blazor WebAssembly application (ProductionManagementSystem) built with .NET 10.0. The application is a Progressive Web App (PWA) with multi-language support (Japanese, English, Chinese, Vietnamese) using cookie-based localization.

## Build and Run Commands

Build the project:
```bash
cd PMS.Client
dotnet build
```

Run the application (development server):
```bash
cd PMS.Client
dotnet run
```

Restore dependencies:
```bash
cd PMS.Client
dotnet restore
```

Build for production:
```bash
cd PMS.Client
dotnet build -c Release
```

Publish the application:
```bash
cd PMS.Client
dotnet publish -c Release
```

## Architecture

### Localization System

The application uses a custom client-side localization system that works via browser cookies:

- **Resource files**: Localization strings are stored in `.resx` files under `Resources/Localization/`
  - `AppLocalizationResource.resx` (default/fallback)
  - `AppLocalizationResource.ja.resx` (Japanese)
  - `AppLocalizationResource.en.resx` (English)
  - `AppLocalizationResource.vi.resx` (Vietnamese)

- **CultureService** (`Services/CultureService.cs`): Manages culture operations
  - Supported cultures: Japanese (ja), English (en), Chinese (zh)
  - Uses JavaScript interop to read/write culture cookies
  - Cookie format: `.AspNetCore.Culture=c={culture}|uic={culture}`

- **JavaScript manager** (`wwwroot/js/cultureManager.js`): Client-side cookie management
  - `getCurrentCulture()`: Reads culture from cookie
  - `setCulture(culture)`: Writes culture to cookie
  - `changeLanguage(culture)`: Changes culture and reloads the page

- **LanguageSwitcher component** (`Components/LanguageSwitcher.razor`): Dropdown UI for language selection

### Component Structure

- **App.razor**: Root component with Router configuration
- **MainLayout.razor**: Main layout with sidebar navigation
- **NavMenu.razor**: Navigation menu component with collapsible sidebar
- **Pages/**: Razor page components
  - `Home.razor`: Home page (route: "/")
  - `Counter.razor`: Counter example page (route: "/counter")
  - `Weather.razor`: Weather example page (route: "/weather")
  - `NotFound.razor`: 404 error page

### Service Registration

Services are registered in `Program.cs`:
- HttpClient is scoped with base address from hosting environment
- Note: CultureService registration is commented out but the service is used via dependency injection in components

### PWA Configuration

- Service worker files: `service-worker.js` and `service-worker.published.js`
- Manifest: `wwwroot/manifest.webmanifest`
- Icons: 192x192 and 512x512 PNG icons for mobile devices

## Adding New Localized Strings

1. Open `Resources/Localization/AppLocalizationResource.resx`
2. Add a new data entry with a key (e.g., "ButtonText")
3. Add the same key to all language-specific `.resx` files (ja, en, vi)
4. The Designer.cs file will auto-generate on build
5. Use in Razor pages: `@Loc[nameof(AppLocalizationResource.YourKey)]`

## Important Notes

- The project uses .NET 10.0, which requires the latest .NET SDK
- All localization uses the `IStringLocalizer<AppLocalizationResource>` injected as `Loc`
- Culture changes require a page reload due to the WebAssembly architecture
- The application is designed as a client-side only WebAssembly app (no server-side rendering)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/horie-tt)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/horie-tt)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
