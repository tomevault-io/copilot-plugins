## ponghub

> **Repository Summary**: PongHub is an open-source service status monitoring website written in Go. It performs zero-intrusion monitoring of services, supports multi-port detection, intelligent response validation, SSL certificate monitoring, and real-time status display. It generates static HTML reports deployed to GitHub Pages via automated CI/CD.

# PongHub Copilot Instructions

## High-Level Details

**Repository Summary**: PongHub is an open-source service status monitoring website written in Go. It performs zero-intrusion monitoring of services, supports multi-port detection, intelligent response validation, SSL certificate monitoring, and real-time status display. It generates static HTML reports deployed to GitHub Pages via automated CI/CD.

**Project Type**: Command-line application / static site generator  
**Primary Language**: Go  
**Go Version**: 1.24.5  
**Dependencies**: 
- `gopkg.in/yaml.v3 v3.0.1` (YAML configuration parsing)
- `github.com/google/uuid v1.6.0` (UUID generation for parameters)

**Repository Size**: ~100 files, small to medium-sized project with modular internal packages.

## Build Instructions

### Bootstrap

- Ensure Go 1.24.5 is installed (`go version` should return go1.24.5 or compatible).
- No additional setup required beyond Go installation.

### Build

- Command: `go build -o bin/ponghub ./cmd/ponghub`
- This creates a binary at `bin/ponghub` (or `bin/ponghub.exe` on Windows).
- Always run from the repository root directory.
- Build time: < 10 seconds on modern hardware.
- Precondition: Repository cloned and Go modules available.
- Postcondition: `bin/ponghub` executable exists and is runnable.

### Test

- Command: `go test ./...`
- Runs unit tests in packages that have them (currently `internal/common/params`, `internal/notifier`, `internal/notifier/channels`).
- Test execution time: < 5 seconds.
- Precondition: Repository cloned and dependencies downloaded (`go mod download` if needed).
- Postcondition: Test output shows "ok" for tested packages, "no test files" for others.

### Run

- Command: `./bin/ponghub` (after building)
- Executes the monitoring check, generates reports and logs.
- Runtime: 10-60 seconds depending on configured services and network conditions.
- Precondition: `config.yaml` exists in root, `data/` directory exists (created automatically if missing).
- Postcondition: Updates `data/index.html`, `data/ponghub_log.json`, `data/notify.txt`.
- Environment variables: Optional, used for SMTP/webhook auth if notifications configured.

### Lint

- No dedicated linting configuration found.
- Use `go fmt ./...` for formatting (standard Go formatting).
- Use `go vet ./...` for basic static analysis.
- No ESLint, golangci-lint, or similar configured.

### Validation Steps

- After build: Verify `bin/ponghub` exists and is executable.
- After run: Check `data/index.html` size > 0, `data/ponghub_log.json` is valid JSON.
- CI validation: PR workflow runs tests and build, deploy workflow runs on schedule.
- Always run tests before committing changes to ensure no regressions.

### Makefile Usage

- `make build`: Equivalent to `go build -o bin/ponghub cmd/ponghub/*.go` (but may fail on Windows without make).
- `make run`: Builds then runs the binary.
- `make test`: Runs `go test ./...`.
- `make clean`: Removes `bin/ponghub`.
- Note: Makefile uses `del` command, Windows-specific; on Unix use `rm`.

### Common Issues and Workarounds

- On Windows without make: Use `go build -o bin/ponghub ./cmd/ponghub` instead of `make build`.
- Missing data directory: Created automatically on first run.
- Network timeouts: Increase `timeout` in `config.yaml` if services are slow.
- SSL certificate errors: Services with expired certs will fail (expected behavior for monitoring).
- Build failures: Ensure Go 1.24.5, run `go mod tidy` if dependencies issue.

### Environment Setup

- No virtualenv, npm install, or similar required.
- GitHub Actions handles deployment; local runs generate static files.
- For notifications: Set environment variables like `SMTP_USERNAME`, `SMTP_PASSWORD` for email/webhook auth.

## Project Layout

### Major Architectural Elements

- `cmd/ponghub/main.go`: Entry point, orchestrates config loading, service checking, logging, reporting.
- `internal/checker/`: Service checking logic, HTTP requests, SSL validation.
- `internal/configure/`: Configuration loading from `config.yaml`.
- `internal/logger/`: Log reading/writing to `data/ponghub_log.json`.
- `internal/notifier/`: Notification handling (email, webhook, default GitHub Actions failure).
- `internal/reporter/`: HTML report generation using `templates/report.html`.
- `internal/types/`: Type definitions and default configurations.

### Configuration Files

- `config.yaml`: Service endpoints, timeouts, notification settings.
- `go.mod`: Go dependencies.
- `Makefile`: Build shortcuts (optional, direct `go` commands work).

### CI/CD Pipelines

- `.github/workflows/pr-build.yml`: Runs on PR to main/master, executes `go test`, build, run, verifies artifacts.
- `.github/workflows/deploy.yml`: Runs every 30 min, builds, runs, deploys to GitHub Pages.
- Validation: Ensures `data/index.html` and `data/ponghub_log.json` are generated correctly.

### Validation Steps for Changes

- Run `go test ./...` to ensure tests pass.
- Build and run locally: `go build -o bin/ponghub ./cmd/ponghub && ./bin/ponghub`
- Check generated files in `data/` for correctness.
- For UI changes: Verify `data/index.html` renders properly (open in browser).
- Notification changes: Test with sample config and env vars.

### Dependencies

- Pure Go, no external services required for core functionality.
- Notifications require SMTP server or webhook endpoint if configured.
- GitHub Pages for hosting (automatic via CI).

### Root Directory Files

- `README.md`: Project overview, quick start, configuration guide.
- `README_CN.md`: Chinese version of README.
- `config.yaml`: Example configuration.
- `go.mod`, `go.sum`: Go module files.
- `Makefile`: Build automation.
- `LICENSE`: MIT license.
- `SECURITY.md`, `CODE_OF_CONDUCT.md`, `CONTRIBUTING.md`: Standard docs.
- `CNAME`: Custom domain for GitHub Pages (optional).

### Key Source Files

- `cmd/ponghub/main.go`: Main function loads config, checks services, generates reports.
- `internal/configure/config.go`: YAML config parsing.
- `internal/checker/services.go`: HTTP checking with retries and SSL validation.
- `internal/reporter/report.go`: HTML generation from template.
- `templates/report.html`: HTML template for status page.
- `static/style.css`: CSS for the report page.

### Directory Structure

- `cmd/`: Executable source.
- `internal/`: Private packages (checker, configure, logger, notifier, reporter, types).
- `data/`: Generated output (index.html, ponghub_log.json, notify.txt).
- `templates/`: HTML templates.
- `static/`: Static assets (CSS, images).
- `imgs/`: Images for README.

Trust these instructions and only search further if information is incomplete or found incorrect. Always run builds/tests locally before committing to match CI behavior.

---
> Source: [DevXDojo/PongHub](https://github.com/DevXDojo/PongHub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-24 -->
