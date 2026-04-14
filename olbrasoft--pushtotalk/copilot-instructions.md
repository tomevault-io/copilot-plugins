## pushtotalk

> Instructions for AI agents working with this repository.

# AGENTS.md

Instructions for AI agents working with this repository.

## Project Overview

Push-to-Talk functionality for Linux voice assistant. Monitors mouse buttons (side buttons, middle click) and triggers speech-to-text recording via HTTP API calls.

## CI/CD Deployment

This repository is configured for **automatic deployment** using GitHub Actions with self-hosted runner.

### Deployment Workflow

1. **Push to `main` branch** → Triggers GitHub Actions
2. **Build & Test** (`.github/workflows/build.yml`)
   - Runs on GitHub-hosted runner (`ubuntu-latest`)
   - Restores dependencies
   - Builds project with version `1.0.${{ github.run_number }}`
   - Runs all tests
   - **MUST PASS** before deployment proceeds
3. **Deploy** (`.github/workflows/deploy.yml`)
   - Triggered only after successful build (workflow_run)
   - Runs on **self-hosted runner** (local machine)
   - Pulls latest code from `main`
   - Publishes to `/opt/olbrasoft/push-to-talk/app/`
   - Automatically uses same version as build

### Deployment Checklist

After pushing to GitHub:

1. ✅ **Check GitHub Actions status**
   ```bash
   gh run list --repo Olbrasoft/PushToTalk --limit 5
   ```
   - Both "Build and Test" and "Deploy to Production" must show ✅ success

2. ✅ **Verify application was deployed**
   ```bash
   ls -lh /opt/olbrasoft/push-to-talk/app/push-to-talk
   ```
   - Check timestamp matches deployment time

3. ✅ **Check running version**
   ```bash
   # Application is auto-deployed, no manual restart needed
   # Version format: 1.0.$RUN_NUMBER (e.g., 1.0.123)
   ```

### Production Deployment Path

**Target:** `/opt/olbrasoft/push-to-talk/app/`

The application is automatically restarted after deployment (if configured in systemd or launcher).

### Build Commands (Local Development)

```bash
dotnet build
dotnet test
dotnet publish -c Release -o ./publish
```

## Code Style

- Follow Microsoft C# naming conventions
- Use xUnit + Moq for testing
- Target .NET 10
- Namespace prefix: `Olbrasoft.PushToTalk`

## Important Paths

- Source: `src/`
  - `PushToTalk.Core/` - Core logic and interfaces
  - `PushToTalk.Linux/` - Linux-specific implementations
  - `PushToTalk.App/` - Desktop application
  - `PushToTalk.Service/` - Background service
- Tests: `tests/`
  - Each source project has its own test project
- Solution: `PushToTalk.sln`

## Architecture

- **Strategy Pattern** for mouse button monitoring (libevdev, udev, etc.)
- **SOLID principles** - especially Single Responsibility and Dependency Inversion
- Clean separation: Core (interfaces/models) -> Linux (platform) -> App/Service (UI/hosting)

## Testing Requirements

- Test naming: `[Method]_[Scenario]_[Expected]`
- Example: `Monitor_ButtonPressed_TriggersCallback`
- Framework: xUnit + Moq

## Secrets

Never commit secrets. Use:
- `dotnet user-secrets` for local development
- GitHub Secrets for CI/CD

## Related Projects

- VirtualAssistant (`~/Olbrasoft/VirtualAssistant/`) - Main voice assistant
- edge-tts-server - Text-to-speech service

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Olbrasoft) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
