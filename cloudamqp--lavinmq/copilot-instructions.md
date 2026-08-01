## lavinmq

> Message queue server implementing the AMQP 0-9-1 and MQTT 3.1.1 protocols, built with Crystal.

# CLAUDE.md

Message queue server implementing the AMQP 0-9-1 and MQTT 3.1.1 protocols, built with Crystal.

## Commands

- `make test SPEC=spec/foo_spec.cr` — run a single spec
- `make test` — run all specs
- `make lint` — run linter

## Development Workflow

- Run `make test` and `make lint` before considering work complete.

## Running locally

- `make bin/lavinmq CRYSTAL_FLAGS=` - Build debug version of the main server binary
- `bin/lavinmq --data-dir ./tmp/data --debug` - Run server with specified data directory in debug logging mode

Standard library sources: `crystal env CRYSTAL_PATH` shows compiler search paths.

## Key Patterns

- Avoid allocations in hot paths (message publishing and delivery)
- All code changes must have corresponding specs. New features need specs, bug fixes need regression specs.

---
> Source: [cloudamqp/lavinmq](https://github.com/cloudamqp/lavinmq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
