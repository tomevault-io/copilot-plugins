## pixelport

> This repository contains a WinUI 3 desktop app (`SerialAssistant.csproj`) using an MVVM-style layout.

# Repository Guidelines

## Project Structure & Module Organization
This repository contains a WinUI 3 desktop app (`SerialAssistant.csproj`) using an MVVM-style layout.
- Root UI entry points: `App.xaml`, `MainWindow.xaml`, `MainWindow.xaml.cs`
- View models: `ViewModels/` (for example, `MainViewModel.cs`)
- Services: `Services/` (for example, `SerialPortService.cs`)
- Static resources: `Assets/` (icons, screenshots, sponsor images)
- App manifest/config: `app.manifest`, `SerialAssistant.csproj`

Keep UI behavior in XAML/code-behind, and business/device logic in `ViewModels` and `Services`.

## Build, Test, and Development Commands
Run from repository root (`SerialAssistant/`):
- `dotnet restore SerialAssistant.csproj` restores NuGet packages.
- `dotnet build SerialAssistant.csproj -c Debug` builds a local debug binary.
- `dotnet run --project SerialAssistant.csproj` launches the app for development.
- `dotnet publish SerialAssistant.csproj -c Release -r win-x64 --self-contained true` creates a release package (switch runtime to `win-x86` or `win-arm64` as needed).

## Coding Style & Naming Conventions
- Use 4-space indentation and standard C# formatting.
- Naming: `PascalCase` for types/methods/properties/XAML element names, `_camelCase` for private fields.
- Prefer `CommunityToolkit.Mvvm` attributes (`[ObservableProperty]`, `[RelayCommand]`) over manual boilerplate.
- Keep methods focused; place serial protocol and conversion logic in view model/service layers, not UI markup.

## Testing Guidelines
There is currently no dedicated automated test project in this repository.
- Minimum validation for each change: open/close port, Text/HEX send-receive, timed send, and log save.
- For new parsing or utility logic, add unit tests in a new `SerialAssistant.Tests` project (recommended: xUnit) with file names like `FeatureNameTests.cs`.

## Commit & Pull Request Guidelines
- Existing history uses concise, scoped subjects (for example, `docs: ...`).
- Prefer Conventional Commit prefixes: `feat:`, `fix:`, `docs:`, `refactor:`, `chore:`.
- Keep one logical change per commit.
- PRs should include: purpose summary, test evidence (commands or manual steps), linked issue (if any), and screenshots for UI changes.

## Security & Configuration Tips
- Do not commit build artifacts or local outputs (`bin/`, `obj/`, `Release/`, logs).
- Avoid hardcoding machine-specific serial settings; validate user-provided numeric values before applying them.

---
> Source: [leisure462/PixelPort](https://github.com/leisure462/PixelPort) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
