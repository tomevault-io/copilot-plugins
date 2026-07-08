## snmp-mqtt-bridge

> This project follows the [Conventional Commits](https://www.conventionalcommits.org/) specification.

# Claude Code Guidelines

## Conventional Commits

This project follows the [Conventional Commits](https://www.conventionalcommits.org/) specification.

### Commit Message Format

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

### Types

| Type | Description |
|------|-------------|
| `feat` | A new feature |
| `fix` | A bug fix |
| `docs` | Documentation only changes |
| `style` | Changes that do not affect the meaning of the code (formatting, etc.) |
| `refactor` | A code change that neither fixes a bug nor adds a feature |
| `perf` | A code change that improves performance |
| `test` | Adding missing tests or correcting existing tests |
| `build` | Changes that affect the build system or external dependencies |
| `ci` | Changes to CI configuration files and scripts |
| `chore` | Other changes that don't modify src or test files |

### Scopes

| Scope | Description |
|-------|-------------|
| `api` | REST API handlers |
| `mqtt` | MQTT client and discovery |
| `snmp` | SNMP polling and commands |
| `frontend` | Vue.js frontend |
| `config` | Configuration management |
| `db` | Database and repositories |
| `docker` | Docker and container setup |
| `profiles` | Device profiles |

### Examples

```
feat(mqtt): add SNMP SET command handler for outlet control

fix(snmp): preserve static OID values between poll cycles

docs: update project checklist with completed features

refactor(api): extract common response helpers

build: add GitHub Actions workflow for releases
```

## Project Commands

### Development

```bash
# Run backend
go run ./cmd/snmp-bridge

# Run frontend dev server
cd frontend && npm run dev

# Build frontend
cd frontend && npm run build

# Build complete application
go build -o snmp-bridge ./cmd/snmp-bridge
```

### Testing

```bash
# Run Go tests
go test ./...

# Run frontend tests
cd frontend && npm test
```

## Code Style

### Go

- Follow standard Go formatting (`gofmt`)
- Use meaningful variable names
- Keep functions focused and small
- Handle errors explicitly
- Use context for cancellation

### Vue.js/TypeScript

- Use Composition API with `<script setup>`
- Follow Vue.js style guide
- Use TypeScript for type safety
- Keep components small and reusable

## Architecture Notes

### Backend Structure

- `cmd/` - Application entry points
- `internal/` - Private application code
  - `api/` - HTTP handlers and server
  - `config/` - Configuration management
  - `domain/` - Business entities
  - `mqtt/` - MQTT client and HA discovery
  - `repository/` - Data access layer
  - `service/` - Business logic
  - `worker/` - Background workers

### Frontend Structure

- `src/views/` - Page components
- `src/components/` - Reusable components
- `src/stores/` - Pinia stores
- `src/api/` - API client

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `CONFIG_PATH` | Path to config file | `config.yaml` |
| `HTTP_PORT` | HTTP server port | `8080` |
| `MQTT_BROKER` | MQTT broker URL | `tcp://localhost:1883` |
| `DB_PATH` | SQLite database path | `data/snmp-bridge.db` |

---
> Source: [SPDG/snmp-mqtt-bridge](https://github.com/SPDG/snmp-mqtt-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
