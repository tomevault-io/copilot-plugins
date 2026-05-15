## claude-stack-dotnet

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ClaudeStack.net is a .NET 10.0 RC 2 template project with integrated Claude Code AI assistance, demonstrating full-stack architecture with centralized package management. The project includes ASP.NET Core MVC and Minimal API applications, each with corresponding unit tests (MSTest) and end-to-end tests (Playwright).

## Build & Test Commands

### Building
```bash
# Build entire solution
dotnet build

# Build specific project
dotnet build src/ClaudeStack.Web/ClaudeStack.Web.csproj
dotnet build src/ClaudeStack.API/ClaudeStack.API.csproj
```

### Running Applications
```bash
# Run MVC application
dotnet run --project src/ClaudeStack.Web/ClaudeStack.Web.csproj

# Run API application
dotnet run --project src/ClaudeStack.API/ClaudeStack.API.csproj
```

### Running Tests
```bash
# Run all tests
dotnet test

# Run specific test project (using Microsoft.Testing.Platform)
dotnet run --project tests/ClaudeStack.Web.Tests/ClaudeStack.Web.Tests.csproj
dotnet run --project tests/ClaudeStack.API.Tests/ClaudeStack.API.Tests.csproj

# Run Playwright tests
dotnet run --project tests/ClaudeStack.Web.Tests.Playwright/ClaudeStack.Web.Tests.Playwright.csproj
dotnet run --project tests/ClaudeStack.API.Tests.Playwright/ClaudeStack.API.Tests.Playwright.csproj

# Run a single test (using test filter)
dotnet test --filter FullyQualifiedName~TestMethod1
```

### Playwright Setup
After creating a new Playwright test project, install browsers:
```powershell
pwsh -Command "cd tests/ClaudeStack.Web.Tests.Playwright/bin/Debug/net10.0; ./playwright.ps1 install"
```

## Architecture

### Project Structure
- **src/ClaudeStack.Web**: ASP.NET Core MVC application with Razor runtime compilation enabled
- **src/ClaudeStack.API**: ASP.NET Core Web API using Minimal APIs with OpenAPI/Swagger
- **tests/ClaudeStack.Web.Tests**: MSTest unit tests for MVC application
- **tests/ClaudeStack.Web.Tests.Playwright**: Playwright end-to-end tests for MVC application
- **tests/ClaudeStack.API.Tests**: MSTest unit tests for API application
- **tests/ClaudeStack.API.Tests.Playwright**: Playwright end-to-end tests for API application

### Configuration Files

#### Directory.Build.props
Defines shared MSBuild properties for all projects:
- **TargetFramework**: net10.0
- **Nullable**: disabled (explicit using statements required)
- **ImplicitUsings**: disabled (explicit using statements required)
- **TreatWarningsAsErrors**: true

#### Directory.Packages.props
Centralized NuGet package version management (CPM). All package versions are defined here, and project files reference packages without versions.

To add a new package:
1. Add `<PackageVersion Include="PackageName" Version="x.y.z" />` to Directory.Packages.props
2. Add `<PackageReference Include="PackageName" />` (without Version) to the project file

#### global.json
Pins the .NET SDK version and configures the test runner:
```json
{
  "sdk": {
    "version": "10.0.100-rc.2.25502.107",
    "rollForward": "latestFeature"
  },
  "test": {
    "runner": "Microsoft.Testing.Platform"
  }
}
```

### Testing Framework

This project uses **MSTest with Microsoft.Testing.Platform** (the new test runner, not the legacy VSTest). Test projects require:
```xml
<PropertyGroup>
  <EnableMSTestRunner>true</EnableMSTestRunner>
  <OutputType>Exe</OutputType>
</PropertyGroup>
```

Tests are configured for method-level parallelization in MSTestSettings.cs:
```csharp
[assembly: Parallelize(Scope = ExecutionScope.MethodLevel)]
```

## Important Notes

### Creating New Test Projects

**CRITICAL**: When creating new MSTest projects, do NOT use the `--test-runner` flag. The test runner is already configured in global.json, and using the flag will overwrite the entire global.json file, removing the SDK version configuration.

```bash
# WRONG - will overwrite global.json
dotnet new mstest -o tests/NewProject --test-runner Microsoft.Testing.Platform

# CORRECT - test runner inherited from global.json
dotnet new mstest -o tests/NewProject
```

After creating the test project, manually add to the .csproj:
```xml
<PropertyGroup>
  <EnableMSTestRunner>true</EnableMSTestRunner>
  <OutputType>Exe</OutputType>
</PropertyGroup>
```

### Implicit Usings Disabled

Since ImplicitUsings is disabled, all C# files must include explicit using statements. Common namespaces needed:
- `using System;`
- `using System.Linq;`
- `using System.Threading.Tasks;`
- `using Microsoft.VisualStudio.TestTools.UnitTesting;` (for tests)
- `using Microsoft.Playwright.MSTest;` (for Playwright tests)
- ASP.NET Core namespaces (Microsoft.AspNetCore.Builder, Microsoft.Extensions.DependencyInjection, etc.)

### Package References

Always reference packages without Version attributes in .csproj files. Versions are centrally managed in Directory.Packages.props.

```xml
<!-- CORRECT -->
<PackageReference Include="Microsoft.AspNetCore.OpenApi" />

<!-- WRONG -->
<PackageReference Include="Microsoft.AspNetCore.OpenApi" Version="10.0.0" />
```

## Claude Code Infrastructure

This project uses Claude Code infrastructure for enhanced development workflow.

**Environment:** Running on WSL2 (Ubuntu) - all hooks and scripts use Linux/bash conventions.

### Installed Components

**Hooks:**
- **skill-activation-prompt** (UserPromptSubmit) - Auto-suggests relevant skills based on prompts and file context
- **post-tool-use-tracker** (PostToolUse) - Tracks file changes for context management

**Skills:**
- **skill-developer** - Meta-skill for creating and managing Claude Code skills
- **azure-devops** - Azure DevOps automation using az CLI with azure-devops extension
- **.NET 10 Project-Specific Skills:**
  - **mstest-testing-platform** - MSTest with Microsoft.Testing.Platform (new test runner)
  - **dotnet-centralized-packages** - Centralized Package Management with Directory.Packages.props
  - **playwright-dotnet** - E2E testing with Playwright for .NET
  - **dotnet-minimal-apis** - ASP.NET Core Minimal APIs with OpenAPI
  - **dotnet-cli-essentials** - Essential .NET CLI commands for this project
  - **aspnet-configuration** - ASP.NET Core configuration and options pattern

**Agents:**
- **code-architecture-reviewer** - Reviews code for best practices and architectural consistency
- **code-refactor-master** - Handles comprehensive code refactoring tasks
- **documentation-architect** - Creates and enhances documentation
- **plan-reviewer** - Reviews development plans before implementation
- **refactor-planner** - Analyzes code and creates refactoring plans
- **web-research-specialist** - Researches technical issues and solutions online

**Dev Docs System:**
- **Slash commands:** `/dev-docs` (create new docs), `/dev-docs-update` (update existing docs)
- **Location:** `dev/active/` directory
- **Pattern:** Three-file structure (plan.md, context.md, tasks.md) for complex tasks

### Configuration
- `.claude/` directory contains skills, hooks, agents, and configuration
- `.claude/hooks/` - TypeScript/bash hooks with npm dependencies
- `.claude/settings.json` - Hook registration and settings
- `.claude/skills/skill-rules.json` - Skill trigger definitions
- `dev/active/` - Development documentation for complex tasks

### Usage
Skills activate automatically based on your prompts and file context. See `.claude/README.md` for details.

### Creating Additional .NET-Specific Skills
Use skill-developer to create skills tailored to this .NET 10 project:
- ASP.NET Core MVC patterns
- Minimal API best practices
- MSTest with Microsoft.Testing.Platform
- .NET 10 specific guidance

Start with: "I want to create a skill for [ASP.NET Core/testing/etc]"

## GitHub Integration

This project includes comprehensive GitHub Actions automation for pull request validation and quality assurance.

### PR Validation Pipeline

Every pull request goes through a **6-step automated validation pipeline**:

#### 1️⃣ Authorization
- Verifies contributor authorization
- Checks against authorized users list and organization membership
- Emergency circuit breaker capability for security incidents

**Configuration:** `.github/claude-authorized-users.yml`

#### 2️⃣ PR Guardrails
- Ensures PR has adequate description (minimum 50 characters)
- Checks PR size (warns if >2000 lines changed)
- Validates required template sections are completed

#### 3️⃣ Quality Checks
- **Code Formatting:** Runs `dotnet format --verify-no-changes`
- **Build Verification:** Runs `dotnet build` on entire solution
- **Test Execution:** Runs `dotnet test` for all test projects
- Reports detailed results in PR comments

#### 4️⃣ Code Review (Optional)
- **Claude Code automated review** with OIDC authentication
- Reviews for .NET 10 best practices and project compliance
- Checks for:
  - C# coding conventions and naming standards
  - Async/await patterns
  - ImplicitUsings compliance (must be disabled)
  - Centralized Package Management compliance (no Version in .csproj)
  - MSTest with Microsoft.Testing.Platform patterns
  - Security best practices
  - Breaking changes
  - Documentation completeness

**Setup Required:** Install Claude Code GitHub App (recommended) or add `CLAUDE_CODE_OAUTH_TOKEN` secret

#### 5️⃣ Security Review
- **GitLeaks:** Scans for exposed secrets (API keys, tokens, passwords)
- **.NET Security Analyzers:** Roslyn security analyzers with `RunAnalyzers=true`
- **NuGet Vulnerability Scanner:** Detects vulnerable package versions (CVEs)
- **Path Security Scanner:** Finds unsafe file operations, path traversal, command injection risks

**Scripts:**
- `.github/scripts/check-dotnet-vulnerabilities.ps1`
- `.github/scripts/check-path-security.ps1`

#### 6️⃣ .NET Validation
- **.csproj Structure:** Validates project files for CPM compliance, MSTest configuration, ImplicitUsings
- **CPM Compliance:** Validates `Directory.Packages.props` structure and semantic versioning
- **global.json:** Validates SDK version, test runner configuration, rollForward policy

**Scripts:**
- `.github/scripts/check-csproj-structure.ps1`
- `.github/scripts/check-cpm-compliance.ps1`
- `.github/scripts/check-global-json.ps1`

### Validation Results

Each step posts detailed results as PR comments that **update in place** (no spam):
- ✅ **Pass** - Step succeeded
- ⚠️ **Warning** - Issues found but non-blocking
- ❌ **Failed** - Critical issues must be fixed
- ⏭️ **Skipped** - Step skipped (e.g., no Claude Code token)

### Setting Up Claude Code Integration

**Option 1: GitHub App (Recommended)**

1. **Install via Claude Code CLI:**
   ```bash
   # Open Claude Code
   claude

   # Run installation command
   /install-github-app
   ```

2. **Or install manually:**
   - Visit https://github.com/apps/claude
   - Click **Install** and select this repository
   - Grant permissions and complete setup

3. **Verify:** Create test PR and check Step 4 executes

**Benefits:** No manual token management, automatic authentication, fine-grained permissions

**Option 2: Manual OAuth Token**

1. **Get OAuth Token:**
   ```bash
   claude auth token
   ```

2. **Add to Repository:**
   - GitHub → Settings → Secrets and variables → Actions
   - New repository secret: `CLAUDE_CODE_OAUTH_TOKEN`
   - Paste token value

3. **Verify:** Create test PR and check Step 4 executes

### Local Validation

Run validation checks locally before pushing:

```bash
# Code formatting
dotnet format

# Build
dotnet build

# Tests
dotnet test

# Security scans
pwsh -File .github/scripts/check-dotnet-vulnerabilities.ps1
pwsh -File .github/scripts/check-path-security.ps1

# .NET validation
pwsh -File .github/scripts/check-csproj-structure.ps1
pwsh -File .github/scripts/check-cpm-compliance.ps1
pwsh -File .github/scripts/check-global-json.ps1
```

### Documentation

- **[.github/workflows/README.md](.github/workflows/README.md)** - Workflow documentation
- **[.github/scripts/README.md](.github/scripts/README.md)** - Script documentation
- **[CONTRIBUTING.md](CONTRIBUTING.md)** - Contribution guidelines
- **[SECURITY.md](SECURITY.md)** - Security policy

### Emergency Circuit Breaker

Disable all automated GitHub actions in case of security incident:

```yaml
# .github/claude-authorized-users.yml
emergency:
  enabled: true
  reason: "Investigating security issue"
```

This blocks all Claude Code mentions and PR automation until disabled.

## CI/CD Pipeline

This project includes a comprehensive continuous integration and deployment pipeline that builds, tests, and publishes applications across multiple platforms.

### Build & Test

The CI/CD pipeline automatically builds and tests on:
- **Linux** (Ubuntu latest)

**Triggers:**
- Push to `main` or `develop` branches
- Pull requests to `main` or `develop`
- Manual workflow dispatch

### Code Coverage

Automated test coverage collection and reporting:

**Features:**
- XPlat Code Coverage collection
- HTML and Cobertura report generation
- PR comments with coverage summary
- Codecov integration (optional)
- Coverage threshold checking

**Setup Codecov:**
```bash
# Add CODECOV_TOKEN secret to repository
# Get token from codecov.io after adding repository
```

**View Coverage:**
- Check PR comments for summary
- Download HTML report from workflow artifacts
- View dashboard at codecov.io (if configured)

### Docker Images

Automated Docker image builds for both applications:

**Images:**
- `ghcr.io/{owner}/{repo}/example-web` - MVC application
- `ghcr.io/{owner}/{repo}/example-api` - Minimal API

**Triggers:**
- Push to `main` branch only
- Tagged releases

**Pull Images:**
```bash
docker pull ghcr.io/{owner}/{repo}/claudestack-web:main
docker pull ghcr.io/{owner}/{repo}/claudestack-api:main

# Run containers
docker run -p 8080:8080 ghcr.io/{owner}/{repo}/claudestack-web:main
docker run -p 8080:8080 ghcr.io/{owner}/{repo}/claudestack-api:main
```

**Local Docker Build:**
```bash
# Build images
docker build -f src/ClaudeStack.Web/Dockerfile -t claudestack-web:local .
docker build -f src/ClaudeStack.API/Dockerfile -t claudestack-api:local .

# Run locally
docker run -p 8080:8080 claudestack-web:local
docker run -p 8080:8080 claudestack-api:local
```

### NuGet Package Publishing

Automated NuGet package publishing on tagged releases:

**Trigger:** Tag push with pattern `v*` (e.g., `v1.0.0`)

**Destinations:**
- NuGet.org (if `NUGET_API_KEY` configured)
- GitHub Packages (automatic)

**Create Release:**
```bash
# Tag and push
git tag -a v1.0.0 -m "Release 1.0.0"
git push origin v1.0.0

# Pipeline automatically:
# 1. Builds solution
# 2. Packs NuGet packages
# 3. Publishes to NuGet.org and GitHub Packages
# 4. Creates GitHub Release with artifacts
```

**Setup NuGet Publishing:**
1. Get API key from [nuget.org](https://www.nuget.org/account/apikeys)
2. Add as repository secret: `NUGET_API_KEY`

### Artifacts

The CI/CD pipeline produces various artifacts:

**Build Artifacts** (7-day retention):
- `web-app-linux-x64` - Published Web application
- `api-app-linux-x64` - Published API application

**Coverage Artifacts** (30-day retention):
- `coverage-report` - HTML coverage report

**Release Artifacts** (permanent):
- NuGet packages
- Published binaries
- Attached to GitHub Releases

### CI/CD Workflow Diagram

```
┌─────────────────────────────────────────────────┐
│ Push/PR to main/develop                         │
└──────────────┬──────────────────────────────────┘
               │
       ┌───────┴────────┐
       │                │
   ┌───▼───┐       ┌───▼───┐
   │ Build │       │ Coverage│
   │ Test  │       │ Report │
   │(Ubuntu)│      │        │
   └───┬───┘       └───┬────┘
       │               │
       │          ┌────▼────┐
       │          │ PR      │
       │          │ Comment │
       │          └─────────┘
       │
┌──────▼────────┐
│ Push to main  │
└──────┬────────┘
       │
   ┌───▼───┐
   │ Docker│
   │ Build │
   └───┬───┘
       │
  ┌────▼────┐
  │ GHCR    │
  │ Publish │
  └─────────┘

┌──────────────┐
│ Tag Push (v*)│
└──────┬───────┘
       │
   ┌───▼────┐
   │ NuGet  │
   │ Publish│
   └───┬────┘
       │
  ┌────▼─────┐
  │ GitHub   │
  │ Release  │
  └──────────┘
```

### Optional Secrets

| Secret | Purpose | Required | Setup |
|--------|---------|----------|-------|
| `CODECOV_TOKEN` | Upload coverage to Codecov | Optional | [codecov.io](https://codecov.io) |
| `NUGET_API_KEY` | Publish to NuGet.org | Optional | [nuget.org](https://www.nuget.org/account/apikeys) |

### Monitoring

**View CI/CD Status:**
- Actions tab → .NET CI/CD workflow
- Check individual job logs
- Download artifacts

**Typical Run Times:**
- Build & Test: 2-5 minutes
- Coverage Collection: 2-3 minutes
- Docker Build: 3-5 minutes
- Total (parallel): ~5-8 minutes

---
> Source: [NotMyself/claude-stack-dotnet](https://github.com/NotMyself/claude-stack-dotnet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
