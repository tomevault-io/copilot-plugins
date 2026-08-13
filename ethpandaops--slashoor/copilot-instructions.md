## slashoor

> Ethereum beacon chain slasher for detecting and submitting attester and proposer slashings.

# Slashoor

Ethereum beacon chain slasher for detecting and submitting attester and proposer slashings.

## Pre-commit Checks

Before committing changes, always run these checks:

```bash
# Format code
gofmt -s -w .

# Run vet
go vet ./...

# Run staticcheck
staticcheck ./...

# Run linter
golangci-lint run ./...

# Run tests with race detector
go test -race ./...

# Build
go build ./...
```

Or run all checks in one command:

```bash
gofmt -s -w . && go vet ./... && staticcheck ./... && go test -race ./... && go build ./...
```

## Project Structure

```
slashoor/
├── main.go                 # Entry point
├── cmd/                    # CLI commands (cobra)
├── pkg/
│   ├── beacon/            # Beacon node client (multi-node, failover)
│   ├── indexer/           # Lazy slasher indexing (m/M algorithm)
│   ├── detector/          # Attester slashing detection + proof generation
│   ├── proposer/          # Proposer slashing detection (double proposals)
│   ├── submitter/         # Slashing submission to beacon nodes
│   ├── dora/              # Dora explorer integration
│   └── coordinator/       # Orchestrates all services
```

## Configuration

See `config.yaml` for configuration options. Key settings:

- `beacon.endpoints` - List of beacon node URLs
- `dora.enabled` - Enable Dora integration for historical scanning
- `dora.scan_on_startup` - Scan all orphaned blocks on startup
- `submitter.dry_run` - Log slashings without submitting

## Running

```bash
./slashoor --config config.yaml --log-level info
```

---
> Source: [ethpandaops/slashoor](https://github.com/ethpandaops/slashoor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
