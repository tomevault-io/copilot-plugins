## dbmate-deployer

> go build -o dbmate-deployer ./cmd/dbmate-deployer

# Claude Code Guidelines for dbmate-deployer

## Development

### Building

```bash
# Build the binary
make build

# Or directly with go
go build -o dbmate-deployer ./cmd/dbmate-deployer
```

### Running Locally

```bash
# Show help
./dbmate-deployer --help

# Run with local S3 (LocalStack) for testing
./dbmate-deployer watch \
  --database-url="postgres://user:pass@localhost:5432/db" \
  --s3-bucket=test-bucket \
  --s3-path-prefix=migrations/ \
  --s3-endpoint-url=http://localhost:4566
```

### Linting

```bash
# Run golangci-lint (install first: https://golangci-lint.run/usage/install/)
make lint
```

### Available Make Targets

```bash
make help           # Show all available targets
make build          # Build the binary
make test           # Run all tests (unit + integration)
make test-unit      # Run unit tests only (fast, no Docker)
make test-integration  # Run integration tests (requires Docker)
make lint           # Run golangci-lint
make clean          # Clean build artifacts
```

## Language Policy

All project artifacts must be written in **English**:

- **Commit messages**: Use conventional commit format (e.g., `feat:`, `fix:`, `docs:`, `refactor:`)
- **Pull request titles and descriptions**: Clear, concise English
- **Documentation**: README.md, code comments, and all user-facing docs
- **Code**: Variable names, function names, comments

## Documentation

- Do not include Kubernetes examples in README - keep deployment examples Docker-focused
- Focus on simplicity and common use cases

## Code Style

- Follow Go best practices and idiomatic Go code
- Use meaningful variable and function names
- Keep functions focused and single-purpose

## Testing

### Running Tests

```bash
# Run all tests (unit + integration)
make test

# Run unit tests only (fast, no Docker required)
make test-unit

# Run integration tests only (requires Docker for testcontainers)
make test-integration
```

### Running Integration Tests with Colima

If you're using [Colima](https://github.com/abiosoft/colima) as your Docker runtime on macOS, you need to set the following environment variables:

```bash
# Set Docker host for Colima
export DOCKER_HOST="unix://${HOME}/.colima/default/docker.sock"

# Disable Ryuk (testcontainers cleanup container) - it has issues with Colima
export TESTCONTAINERS_RYUK_DISABLED=true

# Run integration tests
make test-integration
```

Or as a one-liner:

```bash
DOCKER_HOST="unix://${HOME}/.colima/default/docker.sock" TESTCONTAINERS_RYUK_DISABLED=true make test-integration
```

### Test Organization

- **Unit tests**: Fast tests using mocks, no external dependencies
  - `internal/shared/*_test.go` - S3 operations, Slack notifications, migration validation
- **Integration tests**: End-to-end tests using [testcontainers-go](https://golang.testcontainers.org/)
  - `internal/once/once_integration_test.go` - Full migration workflow test
  - Requires Docker for running PostgreSQL container
  - S3 is mocked using [gofakes3](https://github.com/johannesboyne/gofakes3) (in-process, no container needed)
  - Tests are named with `Integration` suffix (e.g., `TestOnceIntegration`)

### Adding New Tests

- Place unit tests alongside the code they test
- Use the mock S3 client from `internal/shared/testhelpers` for unit tests
- For integration tests, use the `TestEnvironment` from `internal/shared/testhelpers/containers.go`

## Migration Files (dbmate reference)

This project uses dbmate for migrations. Migration files follow dbmate's naming convention:

```
YYYYMMDDHHMMSS_description.sql
```

Example:

```sql
-- migrate:up
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- migrate:down
DROP TABLE users;
```

For more details, see [dbmate documentation](https://github.com/amacneil/dbmate).

## Release Process

### Branch Protection

- The `main` branch is protected with repository rules
- Direct pushes to `main` are not allowed
- All changes must go through pull requests
- Required status checks must pass before merging

### Automated Release with tagpr

- This project uses [tagpr](https://github.com/Songmu/tagpr) for automated versioning
- On each merge to `main`, tagpr creates/updates a release PR
- When the release PR is merged, a git tag is automatically created
- Docker images and binary releases are automatically built and published

### Docker Image Publishing

Docker images are automatically built and published to GitHub Container Registry (ghcr.io) via GitHub Actions.

**GitHub Actions workflows**:
- `.github/workflows/docker.yml` - CI builds (PRs and main branch)
  - **Pull Requests**: Test build only (no push)
  - **Main branch**: Build and push development images with branch name and commit SHA tags
- `.github/workflows/tagpr.yml` - Version management
  - Creates/updates release PR on each merge to main
  - Creates git tag when release PR is merged
- `.github/workflows/release.yml` - Release builds (version tags)
  - **Binary releases**: Build binaries using GoReleaser
  - **Docker images**: Build and push release images with semantic version tags
  - Triggered automatically when tagpr creates a release tag

**Image tags**:
- `latest` - Latest stable release
- `v1.2.3` - Specific version
- `1.2`, `1` - Major/minor version aliases
- `main-<sha>` - Development builds from main branch

### Binary Releases

Binary releases are automatically built using [GoReleaser](https://goreleaser.com/) when a version tag is created.

**Supported platforms**:
- Linux AMD64 (`linux/amd64`)
- Linux ARM64 (`linux/arm64`)
- macOS ARM64 (`darwin/arm64`)

**Release artifacts**:
- Pre-built binaries for each platform
- Archive files (`.tar.gz`)
- Checksums file (`checksums.txt`)
- Automatically generated changelog

**Configuration**: `.goreleaser.yml`

### Making Container Images Public

After the first release, make the container image public:

1. Go to **Packages** in your GitHub repository
2. Click on the `dbmate-deployer` package
3. Go to **Package settings**
4. Scroll down to **Danger Zone**
5. Click **Change visibility** → **Public**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tokuhirom) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
