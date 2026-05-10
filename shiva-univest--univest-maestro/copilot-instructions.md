## univest-maestro

> This is a Maestro E2E testing framework for the Univest Flutter mobile app (Android + iOS). It is a standalone test repository — not the Flutter app itself.

# CLAUDE.md - Project Context for Claude Code

## Project Overview
This is a Maestro E2E testing framework for the Univest Flutter mobile app (Android + iOS). It is a standalone test repository — not the Flutter app itself.

## Tech Stack
- **Test Framework**: Maestro (YAML-based mobile UI testing)
- **App Under Test**: Flutter (Android package: `com.univest.capp.uat`)
- **CI/CD**: GitHub Actions + Firebase Test Lab
- **Cloud**: Google Cloud (gcloud CLI, Firebase Test Lab)

## Project Structure
```
config.yaml          - Maestro global config (appId, screenshot settings)
flows/common/        - Shared reusable flows (launch)
flows/auth/          - Authentication flows (login with OTP)
flows/dashboard/     - Dashboard/home validation flows
test-suites/         - Composed test suites (smoke, regression)
scripts/             - Automation shell scripts
.github/workflows/   - CI pipeline definitions
artifacts/           - Test output directory (gitignored)
```

## Key Conventions
- All flows use YAML format following Maestro syntax
- Environment variables are defined in `config.yaml` under `env:` key, referenced as `${VAR_NAME}` in flows. Can be overridden with `-e KEY=VALUE`
- Flows are modular — use `runFlow` for composition, never duplicate steps
- Use `waitFor` and `waitForAnimationToEnd` for stability — never hardcoded `wait` durations
- Flutter widgets are targeted via `id` (maps to `ValueKey`), `text`, or `semanticsLabel` — avoid index-based selectors
- Test suites in `test-suites/` compose flows from `flows/` using `runFlow` with relative paths
- The `appId` is `com.univest.capp.uat` for all UAT flows

## Running Tests
```bash
# Local (requires running emulator/device) — env vars loaded from config.yaml
maestro test test-suites/smoke.yaml

# Single flow
maestro test flows/auth/login.yaml

# Override env inline
maestro test -e contact_number=9876543210 -e otp=0000 test-suites/smoke.yaml

# Via script (local)
./scripts/run_maestro_firebase.sh --local

# Via script (Firebase Test Lab)
./scripts/run_maestro_firebase.sh --firebase --project <firebase-project-id>
```

## When Adding New Flows
1. Create the flow YAML in the appropriate `flows/<category>/` directory
2. Follow existing patterns: include `appId` and `tags` in frontmatter
3. Use `runFlow: ../common/launch.yaml` if the flow needs to start the app
4. Reference env variables with `${VAR_NAME}` — add new vars to `config.yaml` under `env:`
5. Add the flow to `test-suites/regression.yaml` (and `smoke.yaml` if critical path)
6. Use `takeScreenshot` at key assertion points for debugging

## Common Gotchas
- Relative paths in `runFlow` are relative to the calling YAML file, not the project root
- `config.yaml` at project root sets defaults but flow-level `appId` takes precedence
- The `artifacts/` directory is gitignored — CI uploads artifacts separately
- Shell script requires `chmod +x` before first run
- Firebase Test Lab needs `gcloud` CLI authenticated and Test Lab API enabled

---
> Source: [shiva-univest/univest-maestro](https://github.com/shiva-univest/univest-maestro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
