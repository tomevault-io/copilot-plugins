## go-zap

> This project is a URL shortener service written in Go, originally named `url-shortener` and now renamed to `go-zap`.

# Gemini Project Context

## Overview
This project is a URL shortener service written in Go, originally named `url-shortener` and now renamed to `go-zap`.
It has been migrated from AWS (EC2) to Oracle Cloud Infrastructure (OCI).

## Tech Stack
- **Language:** Go 1.24.11
- **Database:** PostgreSQL
- **Infrastructure:** OCI (Oracle Linux, Podman)
- **Tools:** Docker, Docker Compose, Make, SQLC, Swagger, Migrate.

## Repository Rename (Jan 18, 2026)
The repository was renamed from `url-shortener` to `go-zap`.
- Updated `go.mod` module path.
- Updated all internal imports to `github.com/raythx98/go-zap`.
- Restructured `mocks/` directory.
- Updated `README.md` and `GEMINI.md`.

## Migration Analysis (Target: `../go-dutch`)

| Feature | Current (`go-zap`) | Target (`go-dutch`) | Planned Action |
| :--- | :--- | :--- | :--- |
| **Infrastructure** | OCI (Oracle Linux, Podman) | OCI (Oracle Linux, Podman) | Completed. |
| **Reverse Proxy** | Caddy (Dockerized Service) | Caddy (Dockerized Service) | Completed. |
| **Orchestration** | Docker Compose | Docker Compose | Completed. |
| **Container Image** | `distroless/static-debian12:nonroot` | `distroless/static-debian12:nonroot` | Completed. |
| **App Port** | 8080 | 8080 | Completed. |
| **Go Version** | 1.24.11 | 1.24.11 | Completed. |
| **CI/CD** | `.github/workflows/deploy.yml` | `.github/workflows/deploy.yml` | Completed. |

## Migration Plan (Status: Completed)

### 1. Cleanup
- [x] Remove `.nginx/` directory.

### 2. Configuration Files
- [x] **Create `docker-compose.yaml`**:
    - Define services: `caddy`, `app`, `migrate`, `db`.
    - Align volume and network configuration with `go-dutch`.
    - Use `APP_URLSHORTENER_*` environment variables.
- [x] **Create `Caddyfile`**:
    - Configure reverse proxy for the application.
    - Uses `129.150.49.141.sslip.io` as domain.

### 3. Build & Run Scripts
- [x] **Update `Dockerfile`**:
    - Change base image to `gcr.io/distroless/static-debian12:nonroot`.
    - Expose port `8080`.
    - Ensure `USER` is set to non-root.
- [x] **Update `Makefile`**:
    - Replace manual docker commands with `docker compose` commands (`up`, `down`, `logs`).
    - Simplify `run` and `db` targets.

### 4. CI/CD
- [x] **Update `.github/workflows/aws.yml`**:
    - Rename to `.github/workflows/deploy.yml`.
    - Update "Deploy" job to use OCI/Podman logic from `go-dutch`.

### 5. Documentation
- [x] **Update `README.md`**:
    - Update deployment instructions for OCI.
    - Update local development instructions to use Docker Compose.
    - Remove AWS-specific mentions and Nginx configuration details.

## Next Steps
- Monitor deployment health.
- Consider adding more tests for the new module structure.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/raythx98)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/raythx98)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
