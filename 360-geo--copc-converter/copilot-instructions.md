## copc-converter

> A Rust CLI that converts LAS/LAZ point cloud files into [COPC](https://copc.io/) (Cloud-Optimized Point Cloud) files.

# COPC Converter

A Rust CLI that converts LAS/LAZ point cloud files into [COPC](https://copc.io/) (Cloud-Optimized Point Cloud) files.

## Agent rules

- **Never push, tag, or release without explicit user confirmation.** Always show what you intend to do and wait for approval before any action that affects the remote repository or creates a release.
- Run `cargo fmt` to fix formatting — don't manually edit whitespace.
- Do not add `Co-Authored-By` lines to commits.

## Spec & References

- COPC 1.0 specification: https://github.com/copcio/copcio.github.io/blob/main/copc-specification-1.0.pdf
- Reference implementations: [untwine](https://github.com/hobuinc/untwine), [LAStools lascopcindex64](https://github.com/LAStools/LAStools) (note: LAStools sometimes produces invalid files)

## Architecture

The public API is a typestate pipeline (`Pipeline<S>`) that enforces step ordering at compile time. Internal modules are `pub(crate)` — only `Pipeline`, `PipelineConfig`, and utility functions are public.

### Pipeline stages

```
Pipeline::scan(&files, config)?  -> Pipeline<Scanned>
  .validate()?                   -> Pipeline<Validated>
  .distribute()?                 -> Pipeline<Distributed>
  .build()?                      -> Pipeline<Built>
  .write(&output)?               -> ()
```

### Source files

| File | Purpose |
|---|---|
| `lib.rs` | Public API: `Pipeline<S>`, `PipelineConfig`, stage markers, utility functions |
| `main.rs` | CLI args (clap), calls the pipeline |
| `octree.rs` | `OctreeBuilder`, voxel key math, point distribution, octree construction |
| `validate.rs` | Input validation (CRS, point format, GPS time) |
| `writer.rs` | COPC file writer with parallel LAZ encoding |
| `copc_types.rs` | COPC-specific structs (header, VLRs, hierarchy entries, temporal index) |

### Key design decisions

- **Typestate pipeline**: compile-time enforcement of step ordering — can't distribute before validating, can't write before building
- **Minimal public API**: only `Pipeline`, `PipelineConfig`, stage markers, and two utility functions are public; all internal types are `pub(crate)`
- **Out-of-core**: points are written to per-voxel temp files during distribution to stay within a configurable memory budget (default 16 GB, applied with a 0.75 safety factor)
- **Temp cleanup**: `OctreeBuilder` implements `Drop` to remove the temp directory, ensuring cleanup even on error
- **Point formats**: automatically selects LAS point format 6, 7, or 8 based on input — uses the `las` crate for reading and `laz` for compression
- **Parallelism**: uses rayon throughout for reading, octree building, and LAZ compression
- **LOD thinning**: 128 grid cells per axis (matching untwine's CellCount) for good progressive rendering
- **Version in Cargo.toml**: kept as `0.0.0-dev`; CI patches it from the git tag at release time

## Development

```sh
cargo fmt            # format code
cargo clippy         # lint
cargo test           # run tests
```

CI runs all three on every push to `master` and on PRs. All must pass.

## Releasing

1. Commit and push to `master`
2. Create and push a git tag: `git tag vX.Y.Z && git push origin vX.Y.Z`
3. Create a GitHub Release: `gh release create vX.Y.Z --title "vX.Y.Z" --generate-notes`
4. CI triggers on the release event and automatically:
   - Builds binaries for linux (x86_64, aarch64), macOS (x86_64, aarch64), and Windows (x86_64)
   - Publishes to crates.io

**Important:** A git tag alone is not enough — the CI workflows trigger on `release: [published]`, so the GitHub Release (step 3) is required.

---
> Source: [360-geo/copc-converter](https://github.com/360-geo/copc-converter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
