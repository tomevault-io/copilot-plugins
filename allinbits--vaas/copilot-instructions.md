## vaas

> VAAS (Validator-as-a-Service) is a simplified Interchain Security (ICS) implementation for Cosmos blockchains, derived from [interchain-security](https://github.com/cosmos/interchain-security). It lets a provider chain lease its proof-of-stake security to consumer chains via automatic validator synchronization. All active validators validate all consumers — no opt-in/opt-out.

# AGENTS.md

## What This Is

VAAS (Validator-as-a-Service) is a simplified Interchain Security (ICS) implementation for Cosmos blockchains, derived from [interchain-security](https://github.com/cosmos/interchain-security). It lets a provider chain lease its proof-of-stake security to consumer chains via automatic validator synchronization. All active validators validate all consumers — no opt-in/opt-out.

Supports both IBC v1 (channel-based, ordered) and IBC v2 (client-based, out-of-order). IBC v2 uses application IDs `vaas/provider` and `vaas/consumer` instead of port IDs.

## Build & Test Commands

```bash
make build              # go build ./...
make test               # all unit tests (excludes e2e), 25m timeout
make lint               # golangci-lint via devdeps/go.mod
make lint-fix           # auto-fix lint issues
make mocks-gen          # regenerate mocks from x/vaas/types/expected_keepers.go
make vulncheck          # govulncheck

# Single test
go test ./x/vaas/provider/keeper -run TestValsetUpdateBlockHeight -v

# Protobuf (requires Docker)
make proto-gen          # generate Go code from .proto files
make proto-format       # clang-format
make proto-lint         # buf lint

# Localnet (3 terminals, or use localnet-start for all-in-one)
make localnet-start     # provider + consumer + Hermes relayer
make localnet-clean     # stop all, clean data

# E2E (Docker-based)
make docker-build-all   # build chain + Hermes images
make test-e2e           # run e2e suite
```

Dev tool dependencies are isolated in `devdeps/go.mod` and invoked via `go run -modfile devdeps/go.mod`.

## Architecture

### Module Structure

The core protocol lives in `x/vaas/` with two symmetric modules:

- **`x/vaas/provider/`** — runs on the provider chain. Manages consumer lifecycle, generates VSC (Validator Set Change) packets at epoch boundaries, handles key assignment, processes double-voting evidence.
- **`x/vaas/consumer/`** — runs on consumer chains. Receives VSC packets, maintains cross-chain validator set, reports evidence back to provider.
- **`x/vaas/types/`** — shared types, errors, constants, and `expected_keepers.go` (interfaces for external dependencies like staking/slashing).

Each module has: `keeper/` (business logic + state), `types/` (data structures + params), `client/cli/` (CLI commands), `module.go`, `ibc_module.go` (v1 callbacks), `ibc_module_v2.go` (v2 callbacks).

Two helper modules replace standard Cosmos modules to prevent automatic validator set updates on consumer chains:

- `x/vaas/no_valupdates_staking/` — staking without EndBlock valset exports
- `x/vaas/no_valupdates_genutil/` — genutil without gentx-based valset init

### Application Layer

- **`app/provider/`** — full Cosmos SDK app for the provider chain
- **`app/consumer/`** — full Cosmos SDK app for consumer chains
- **`app/cmd/`** — binary entry points (`provider` and `consumer` daemons)

Built with `make build-apps` into `build/`.

### Key Data Flow

1. Provider detects staking changes → queues VSC packets (every `blocks_per_epoch` blocks, default 600)
2. VSC packets sent via IBC to all registered consumers
3. Consumer receives packet → calls `ApplyCCValidatorChanges()` to update its validator set
4. Double-voting evidence flows consumer → provider, where per-consumer infraction parameters determine slash/jail

### Consumer Lifecycle

`REGISTERED → INITIALIZED → LAUNCHED → STOPPED → DELETED`

Managed via `MsgCreateConsumer`, `MsgUpdateConsumer`, `MsgRemoveConsumer` on the provider.

### State Management

Uses `cosmossdk.io/collections` for type-safe key-value storage. Each keeper defines a `Schema` with indexed maps. Key assignment state maps validator addresses to per-consumer consensus keys (`ValidatorConsumerPubKey`, `ValidatorByConsumerAddr`).

### Proto Definitions

Under `proto/vaas/`: `v1/` (shared wire types like `ValidatorSetChangePacketData`), `provider/v1/`, `consumer/v1/`. Generated Go code goes to corresponding `types/` packages.

## Testing

- **Unit tests**: alongside source in `*_test.go`. Use `testutil/keeper/unit_test_helpers.go` for in-memory keeper setup with `MockedKeepers` (gomock).
- **Mock generation**: `make mocks-gen` from `x/vaas/types/expected_keepers.go` → `testutil/keeper/mocks.go`
- **E2E tests**: `tests/e2e/` — Docker-based, spins up real chains with Hermes relayer.

## Lint / Import Ordering

Import groups (enforced by gci in `.golangci.yml`):

1. Standard library
2. Third-party
3. `github.com/cometbft/cometbft`
4. `github.com/cosmos/*`, `cosmossdk.io/*`, `github.com/cosmos/cosmos-sdk`
5. `github.com/atomone-hub/atomone`

Generated files (`*.pb.go`, `*.pb.gw.go`) and `tests/` directory are excluded from linting.

---
> Source: [allinbits/vaas](https://github.com/allinbits/vaas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
