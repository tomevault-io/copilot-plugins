## bibtex-importer

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Elixir library project called `bibtex_importer` that uses Rust NIFs (Native Implemented Functions) via Rustler to parse BibTeX files efficiently. The project combines Elixir for the API with Rust for high-performance parsing using the `biblatex` crate.

**Precompilation**: The project uses `rustler_precompiled` to provide precompiled NIFs for common platforms, eliminating the need for users to have Rust installed. During development, it falls back to compiling from source.

## Common Commands

### Development
- `mix compile` - Compile the project (includes Rust NIF compilation)
- `mix test` - Run all tests
- `mix test test/specific_test.exs` - Run a specific test file
- `mix format` - Format code according to .formatter.exs configuration
- `mix deps.get` - Fetch dependencies
- `mix deps.compile` - Compile dependencies

### Rust Development
- `cd native/bibtex_importer && cargo build` - Build Rust code separately
- `cd native/bibtex_importer && cargo test` - Run Rust tests
- `cd native/bibtex_importer && cargo check` - Check Rust code without building

### Testing
- `mix test --cover` - Run tests with coverage
- `mix test.watch` - Watch for file changes and run tests (if available)

## Project Structure

This is an Elixir library with Rust NIFs:

- `lib/bibtex_importer.ex` - Main Elixir module that exposes `parse_biblatex/1` function
- `native/bibtex_importer/src/lib.rs` - Rust NIF implementation using biblatex crate
- `native/bibtex_importer/Cargo.toml` - Rust dependencies and configuration
- `test/bibtex_importer_test.exs` - Elixir tests for the BibTeX parsing functionality
- `mix.exs` - Project configuration with rustler_precompiled and rustler dependencies
- `config/config.exs` - Configuration including force build during development
- `.github/workflows/release.yml` - GitHub Actions workflow for precompiled releases

## Code Architecture

The project uses a hybrid Elixir/Rust architecture:

1. **Elixir Layer**: Provides the public API through `BibtexImporter.parse_biblatex/1`
2. **Rust NIF Layer**: Handles actual BibTeX parsing using the `biblatex` crate
3. **Data Flow**: 
   - Input: BibTeX string from Elixir
   - Processing: Rust parses using biblatex crate
   - Output: Returns structured data as `Vec<(String, Vec<(String, Vec<String>)>)>`
   - Each entry contains: `(entry_key, [(field_name, [field_values])])`

## Development Notes

- Elixir version: ~> 1.18
- Rustler version: ~> 0.37.1 (latest)
- RustlerPrecompiled version: ~> 0.8.3
- Uses `biblatex` crate version 0.9.3 for Rust-based parsing
- Rust edition: 2021
- NIF functions handle the performance-critical parsing logic
- Tests cover both simple and complex BibTeX entries

## Precompilation

The project uses `rustler_precompiled` to provide precompiled NIFs for:
- Ubuntu x86_64 (GNU libc)
- Alpine x86_64 (musl libc) - via cross-compilation  
- macOS aarch64 (Apple Silicon)

During development, set `BIBTEX_IMPORTER_BUILD=1` environment variable to force compilation from source.

### Creating Releases

Release process for GitHub releases (NOT published to Hex):

1. **Create and push tag**: `git tag v0.1.0 && git push origin v0.1.0`
2. **Wait for builds**: GitHub Actions will build NIFs for all platforms
3. **Verify release**: Check GitHub releases page for precompiled artifacts
4. **Optional - Generate checksums**: If needed for local verification:
   ```bash
   mix rustler_precompiled.download BibtexImporter --all --print
   ```

**Note**: This project is NOT published to Hex.pm and should remain private/internal use only.

### Environment Variables

- `BIBTEX_IMPORTER_BUILD=1` - Force build from source during development

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kznconsulting) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
