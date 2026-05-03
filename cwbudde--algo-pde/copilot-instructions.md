## algo-pde

> - This repo is a Go library for fast spectral PDE tooling (target: Poisson/Helmholtz), built on `github.com/MeKo-Christian/algo-fft`.

# Copilot instructions (algo-pde)

## Big picture

- This repo is a Go library for fast spectral PDE tooling (target: Poisson/Helmholtz), built on `github.com/MeKo-Christian/algo-fft`.
- Current implemented “core building blocks”:
  - `r2r/`: DST-I + DCT-I plans implemented via FFT embedding
  - `fd/`: discrete Laplacian eigenvalues for `poisson.BCType`
  - `grid/`: `Shape`/stride/indexing + `LineIterator` for axis-wise line transforms
  - `poisson/`: boundary condition types, solver options, error types, transform interfaces (no full solver plan yet)

## Key conventions (read before editing)

- Grid layout is row-major, flattened as `[nx, ny, nz]` with missing dims set to `1` (see `grid.Shape` in `grid/grid.go`). Axis order is `0=x`, `1=y`, `2=z`.
- Line-wise transforms iterate using `grid.NewLineIterator(shape, axis)` and handle non-contiguous lines by copying into a temporary buffer (see `r2r/lines.go`).
- Transforms are plan-based and allocation-conscious:
  - Prefer `NewDSTPlan/NewDCTPlan` for repeated use; one-shot helpers allocate (`DST1`, `DCT1`).
  - `DSTPlan`/`DCTPlan` are **NOT thread-safe**; create one plan per goroutine (documented in `r2r/dst.go`, `r2r/dct.go`).
  - Keep the “separate input/output FFT buffers” behavior; it avoids known in-place FFT issues for some sizes.

## Boundary conditions + nullspace

- Boundary types: `poisson.Periodic`, `poisson.Dirichlet`, `poisson.Neumann` (see `poisson/bc.go`).
- Periodic + Neumann have a nullspace (constant mode). `poisson.Options` provides explicit handling via `NullspaceHandling` (see `poisson/options.go`).
- Errors are mostly sentinel `var`s (see `poisson/errors.go`, `r2r/errors.go`); prefer `errors.Is` when checking and wrap with `%w` when adding context.

## Workflows (use these commands)

- Tests: `just test` (or `go test ./...`); CI runs `go test -v -race -count=1 -coverprofile=coverage.txt ./...`.
- Lint: `just lint` (golangci-lint config in `.golangci.toml`; short math variable names are allowed).
- Format: `just fmt` / `just fmt-check` (treefmt runs `gofumpt` then `gci`, and `prettier` for Markdown).

## Compatibility note

- GitHub Actions uses Go `1.23` (see `.github/workflows/test-unit.yaml`). Keep changes compatible with that unless the repo updates CI.

---
> Source: [CWBudde/algo-pde](https://github.com/CWBudde/algo-pde) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
