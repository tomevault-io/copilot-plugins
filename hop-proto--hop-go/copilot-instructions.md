## hop-go

> See [ARCHITECTURE.md](./ARCHITECTURE.md) for more information about how the code

# Contributor Guide

## Code Structure

See [ARCHITECTURE.md](./ARCHITECTURE.md) for more information about how the code
is structured, and what this repository does.

## Dev Environment Tips

- Use go test <relative_path_to_package> to test a single package instead of everything, e.g. `go test ./certs`
- Don't do anything that relies on Docker
- Use `make format` to format code after making any changes
- Use `make lint` to catch other style and code errors

## Testing Instructions

- Find the CI plan in the .github/workflows folder, it is roughly equivalent to
  ensuring `make build`, `make lint`, and `make test` all pass.
- Ensure `make build`, `make lint`, and `make test` all pass before finishing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hop-proto)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hop-proto)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
