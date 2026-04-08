## miaro-scheduler-api

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Go/Gin API tracking a 10-day rotating work schedule. Serves a French-language HTML dashboard (3 switchable themes) and JSON endpoint showing current shift, work status, next working day, and monthly calendar.

## Build / Run / Test

```bash
go run .                          # Run locally (port 8081)
go build -o miaro-scheduler-api . # Build binary
go test ./...                     # Run all tests
go test -v -race ./...            # Tests with race detection (CI uses this)
go test -v -run TestName ./pkg    # Run a single test in pkg/
go test -v -run TestName .        # Run a single test in main (integration tests)
go fmt ./...                      # Format code
```

Module name: `miaro-scheduler-api` (imports use `miaro-scheduler-api/pkg`).

## Architecture

```
Request -> Gin router (main.go) -> Handler -> CalculateSchedule() -> FormatScheduleBeautified() -> Render
```

**Routes:** `GET /miaro` (HTML), `GET /miaro/json` (JSON), `GET /health` (health check).

**Team resolution** (HTML handler): `?team=` param > `miaro-team` cookie > default (1). `?team=1` redirects to `/miaro` (canonical). Cookie set with `Secure; SameSite=Lax; Path=/miaro`.

**Schedule calculation** (`pkg/schedulerService.go`): Epoch is Aug 27, 2024 midnight Europe/Paris. Computes `daysSinceEpoch % 10` to determine shift:

| Day index | Shift | Hours |
|-----------|-------|-------|
| 0-1 | MORNING | 06:00-14:00 |
| 2-3 | AFTERNOON | 14:00-22:00 |
| 4-5 | NIGHT | 22:00-06:00 |
| 6-9 | FREE | Off duty |

`CalculateSchedule(date time.Time, team int)` computes the schedule for a given time and team. Tests inject specific dates; handlers pass `time.Now()`.

### Teams

5 teams share the same 10-day rotation, each offset by 2 days. Team 1 (Miaro, `MiaroTeam` constant) is the default. Formula: `epoch + (team-1)*2 days`. Team selection via `?team=N` query param with cookie persistence (`miaro-team`). Constants and helpers in `pkg/team.go`.

**Formatting** (`pkg/template.go`): Converts schedule to French text, determines real-time work status by checking hour against shift windows, finds next working day, generates `CalendarDay` slice for the monthly calendar grid.

**Template** (`templates/miaroSchedule.tmpl`): Single embedded HTML file with 3 themes (Neumorphism, Bento, Terminal) toggled via CSS/JS. Theme persists in `miaro-theme` cookie (migrated from localStorage). Templates are embedded at compile time via `//go:embed templates/*`.

## Design Decisions

- **No database** -- schedule is deterministic from epoch date via modulo arithmetic.
- **Europe/Paris timezone** loaded once at package init into `parisLoc` var. All schedule code uses `parisLoc` -- never call `time.LoadLocation` again in pkg.
- **All UI text in French.** The `title` template function provides French-compatible title casing.
- **Structured logging** via `slog` with JSON handler. Tests use `slog.LevelError` to suppress noise.
- **Graceful shutdown** on SIGINT/SIGTERM with 5-second drain.

## Configuration

Environment variables (all optional):

| Variable | Default | Description |
|----------|---------|-------------|
| `PORT` | `8081` | Server listen port |
| `TIMEZONE` | `Europe/Paris` | Validated at startup but not used for schedule calc (hardcoded to Europe/Paris) |
| `ENABLE_CORS` | `false` | Enable CORS with allow-all-origins |

## Deployment

- **Service:** `miaro-scheduler-api` (systemd)
- **Binary:** `/home/dev/projects/miaro-scheduler-api/main` (service ExecStart points here)
- **Public URL:** `https://disqt.com/miaro` (nginx reverse proxy)
- **VPS access:** `ssh dev`

Deploy steps (on VPS):
```bash
ssh dev "cd ~/projects/miaro-scheduler-api && git pull && go test ./... && CGO_ENABLED=0 go build -o main . && sudo systemctl restart miaro-scheduler-api"
```

Verify: `ssh dev "systemctl is-active miaro-scheduler-api && curl -s http://localhost:8081/health"`

Note: `scripts/deploy.sh` copies to `/opt/` which is stale -- the service runs from `~/projects/`. The binary must be named `main`.

## CI

GitHub Actions on push/PR to `main` and `dev`:
1. Test (`go test -v -race -coverprofile=coverage.txt -covermode=atomic ./...`) + Codecov
2. Lint (`golangci-lint`, continue-on-error)
3. Build (depends on test + lint)
4. Docker image build (push to main/dev only, no registry push)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/disqt)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/disqt)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
