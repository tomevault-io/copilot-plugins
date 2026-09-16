## file-compressor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A file compression/decompression utility written in Rust using the Zstandard (zstd) compression algorithm. Provides both CLI and GUI interfaces.

**Language**: CLI and library use Italian for all user-facing text (error messages, help text, comments). GUI auto-detects system locale and supports both Italian and English.

## Build and Development Commands

```bash
# Build
cargo build                    # Debug build
cargo build --release          # Release build (optimized)

# Run CLI
cargo run -- compress <file> --livello 10
cargo run -- decompress <file.zst>
cargo run -- multi-compress file1 file2 --output archive.tar.zst
cargo run -- batch "*.log" --livello 5
cargo run -- verifica <file.zst>
cargo run -- riduci scansione.pdf --max-size 2MB   # lossy, output -ridotto.pdf

# Run GUI
cargo run --bin file_compressor_gui

# Install desktop entry + binaries per-user on Linux (no sudo; --uninstall to remove)
./install-linux.sh

# Test
cargo test                     # Run all tests
cargo test -- --nocapture      # With output
cargo test <test_name>         # Specific test
cargo test -- <name1> <name2>  # Multiple filters need `--` (bare args error out)

# Lint and format
cargo clippy
cargo fmt

# Validate workflow YAML (python3-pyyaml is installed; ruby is not)
python3 -c "import yaml; yaml.safe_load(open('.github/workflows/ci.yml'))"
```

**Binary outputs:**

- CLI: `target/{debug,release}/file_compressor`
- GUI: `target/{debug,release}/file_compressor_gui`

## Architecture

### Module Structure

| File | Purpose |
|------|---------|
| [src/lib.rs](src/lib.rs) | Core compression library with all compression/decompression logic |
| [src/main.rs](src/main.rs) | CLI application with progress bars |
| [src/gui.rs](src/gui.rs) | egui-based GUI application |
| [src/shrink.rs](src/shrink.rs) | "Riduci" feature: lossy in-format re-compression (PDF/JPEG/PNG/DOCX) to fit upload size limits |
| [src/updater.rs](src/updater.rs) | In-app update: GitHub release check, per-channel apply (AppImage/installer/bundle) |

### Core Library (lib.rs)

Key types:

- `CompressOptions` / `DecompressOptions` - Builder pattern for operation configuration
- `CompressOutcome` - `Compressed(CompressionResult)` or `Skipped { reason }`; skipping an already-compressed file is a normal outcome, never an `Err`
- `CompressionResult` - Stores input/output sizes
- `VerifyResult` - File integrity verification result
- `CancelFlag` (`Arc<AtomicBool>`) - Checked once per I/O block, so cancellation takes effect *within* a large file, not just between files

Key functions:

- `compress_file()` / `decompress_file()` - Single file operations with progress callbacks
- `compress_directory()` - Creates tar.zst from directory
- `compress_multiple_files()` - Bundles files into tar.zst archive; renames duplicates (`report.txt` → `report-2.txt`) since the archive is flat
- `verify_zst()` - Validates file integrity
- `declared_decompressed_size()` - Reads the original size from the zstd frame header (written by `set_pledged_src_size`), giving progress bars a real total instead of an estimate
- `estimate_compressed_size()` / `estimate_directory_compressed_size()` - Real-compression size estimates (whole file ≤4MB, three 512KB samples above; directories capped at 20 files/8MB). The GUI consumes them from a background worker with a (path, level, smart) cache — never call them on the render thread.
- `cleanup_temp_files()` - Removes in-flight temp files; call from signal handlers, which skip destructors

### CLI Commands (main.rs)

| Command | Description |
|---------|-------------|
| `compress` | Single file/directory compression (level 1-21, default 3) |
| `decompress` | Decompress .zst or extract .tar.zst |
| `multi-compress` | Create tar.zst from multiple files (clap derives kebab-case from `MultiCompress`) |
| `batch` | Compress files matching glob pattern (e.g., `*.log`, `**/*.txt`) |
| `verifica` | Verify .zst file integrity |
| `riduci` | Lossy in-format shrink of already-compressed files (`--max-size 2MB`, `--preset leggera/media/aggressiva`); output is always a new `-ridotto` file |

### Dependencies

| Crate | Purpose |
|-------|---------|
| zstd | Zstandard compression |
| clap | CLI argument parsing |
| indicatif | Progress bars |
| tar | TAR archive handling |
| rayon | Parallel batch processing |
| glob | Pattern matching for batch operations |
| eframe/rfd | GUI framework and file dialogs |
| sys-locale | GUI system locale detection |
| ctrlc | Ctrl+C signal handling for graceful interruption |
| ureq | HTTP client for update checks/downloads (blocking, rustls) |
| serde_json | GitHub API response parsing |
| semver | Version comparison for updates |
| dirs | Cross-platform state dir for update-check throttle |
| winreg | (Windows) detect Inno-installed copies via uninstall registry key |
| lopdf | PDF parsing/rewriting for the shrink feature |
| image | JPEG/PNG decode + re-encode (shrink) |
| imagequant | PNG palette quantization (shrink) |
| png | Indexed PNG writing (shrink) |
| zip | DOCX/XLSX/PPTX re-zip (shrink) |

## Design Patterns

- **Streaming I/O**: Adaptive buffer sizes (256KB for files <10MB, 1MB for larger) - never loads entire files into memory
- **Auto-parallel**: Files ≥1MB automatically use multi-threaded compression; explicit `--parallel` flag also available
- **Large file optimizations**: Files ≥10MB enable WindowLog(24) and long-distance matching for better compression
- **Progress callbacks**: All operations accept `Box<dyn Fn(u64) + Send + Sync>` for progress tracking
- **Force flag**: Requires `--force` to overwrite existing files
- **Output naming**: `file.txt` → `file.txt.zst`, directories → `dirname.tar.zst`

## Gotchas

- **Italian text in CLI/lib**: All error messages, help text, comments, and test assertion messages must be in Italian. GUI handles both Italian/English via sys-locale.
- **Adding a GUI string takes three edits**: a field in `Strings`, plus a value in *both* `STRINGS_IT` and `STRINGS_EN`.
- **Tests require temp files**: Many tests create/delete files in temp directories; ensure cleanup on failure.
- **lib.rs tests use fixed temp-file names** in `std::env::temp_dir()`: two checkouts/worktrees running `cargo test` concurrently collide with spurious, non-reproducible failures. Rerun alone to confirm; real fix is pid-unique names.
- **`FileCompressor-x86_64.AppImage` and `appimagetool` are tracked binaries** (committed before the .gitignore rules): `./build-appimage.sh` overwrites both. `git restore` them afterwards — never commit locally built copies.
- **Testing the AppImage on FUSE2-less distros** (Fedora 40+): it fails with "error loading libfuse.so.2" — launch with `--appimage-extract-and-run`. `$APPIMAGE` is still set, so the updater's channel detection works normally.
- **Timing/interruption tests need incompressible input** (`/dev/urandom`, not repeated text): zstd crushes repetitive data in well under a second, so the interruption never lands and the test silently proves nothing.
- **Estimation-accuracy tests need uniformly semi-compressible data** (64-symbol alphabet over pseudo-random bytes, see `semi_compressible_bytes` in lib.rs tests): on repetitive/cyclic data, whole-file compression exploits long-range matches the 512KB samples can't see, so estimate-vs-real drifts for data reasons, not code bugs.
- **GUI colors live in `mod palette` (gui.rs)**: never hardcode `Color32::from_rgb` in UI code — add or reuse a named constant.
- **Shrink ("Riduci") is lossy and format-preserving**: it lives in `src/shrink.rs`, never overwrites the input (always writes `nome-ridotto.ext`), and must REFUSE signed PDFs (`/Type /Sig`, `/ByteRange`, `.p7m`) and encrypted PDFs — re-compressing them would invalidate the signature. Target-size mode walks `TARGET_LADDER` presets until the file fits.
- **`[profile.dev.package."*"] opt-level = 2` is required**: the image-decoding crates (zune-jpeg & co.) are 10-50x slower unoptimized — without it, shrinking a scanned PDF in a debug build takes minutes instead of seconds and the shrink tests crawl.
- **egui's default font lacks `→` and `▸`/`▾`**: they render as tofu boxes (□). Use ASCII (`->`, `>`/`v`) or glyphs covered by the embedded emoji font (📁 ✅ 🔄 ✖ 📥 …), which are known to render.
- **Malicious-archive fixtures**: the `tar` crate rejects `append_data` with `..` in the path; write the GNU header name bytes directly (see `test_decompress_tar_zst_rejects_path_traversal`).
- **Verifying the GUI on macOS**: `screencapture` fails ("could not create image") without Screen Recording permission. Assert on a headless `egui::Context` instead (see `test_dark_theme_and_custom_style_survive`).
- **zsh eats bare `=`-prefixed words**: `echo ===` fails with "== not found" (zsh `=word` path expansion) and aborts the rest of an `&&` chain — quote separator strings (`echo "==="`).
- **Windows builds**: `build.rs` embeds Windows resources (icon, metadata) - requires `winresource` build dependency.
- **`NbWorkers` needs the zstd `zstdmt` feature**: `set_parameter(CParameter::NbWorkers(..))` returns `Unsupported parameter` unless `Cargo.toml` declares `zstd = { version = "0.13", features = ["zstdmt"] }`. Since `auto_parallel` defaults to true above 1MB, dropping that feature makes every file ≥1MB fail to compress. Guarded by `test_compress_file_over_1mb_auto_parallel`.
- **`entry.unpack()` bypasses tar sanitization**: `decompress_tar_zst` unpacks each entry to a manually joined path, so `tar`'s built-in traversal protection (which only applies to `Archive::unpack`) does not run. Entry paths are validated explicitly against `..`, absolute paths, and symlink escapes — do not remove those checks. Guarded by `test_decompress_tar_zst_rejects_path_traversal`.
- **All file output goes through `AtomicOutput`**: writes land on a `.tmp-<pid>-<n>` file that is renamed only on success, so an interrupted run never leaves a truncated `.zst` that looks valid. Note `panic = "abort"` in the release profile means destructors do not run on panic; `cleanup_temp_files()` covers the signal path.
- **GUI reads file metadata once**: `SelectedEntry` caches name/size/is_dir at selection time. Calling `fs::metadata()` inside the render loop costs one syscall per file per frame.
- **`with_app_id("file-compressor")` must match the `.desktop` basename**: GNOME (Wayland and X11) matches a window to its menu entry via the app_id / `StartupWMClass`. Renaming either side in `src/gui.rs`, `install-linux.sh`, or `build-appimage.sh` silently downgrades the dock to a generic icon — nothing errors.
- **GUI theme is pinned, not reapplied**: `ctx.set_theme(ThemePreference::Dark)` runs once at startup. Calling `set_visuals()` every frame (the previous approach) silently discarded the custom widget styling from `setup_custom_style`. Guarded by `test_dark_theme_and_custom_style_survive`.
- **Release asset names are a contract with `src/updater.rs`**: `FileCompressor-Linux-x86_64.AppImage`, `FileCompressor-Setup-<version>.exe`, `FileCompressor-macOS-app.tar.zst`. Renaming them in `release.yml` silently breaks in-app updates (users just get the release-page fallback). Same for the Inno `AppId`: `windows_install_location()` hardcodes the `{C7E8F9A0-...}_is1` registry key.
- **Auto-check is skipped under `cfg!(test)`**: `CompressorApp::default()` would otherwise fire a real HTTP request from every GUI test. `FILE_COMPRESSOR_UPDATE_URL` overrides the API base for local end-to-end testing (see the E2E task in the update plan).
- **The updater never touches the network in unit tests**: mini HTTP servers on `127.0.0.1` via `std::net::TcpListener` (see `servi_una_risposta` in updater.rs tests).

## Release Builds

```bash
cargo build --release              # Full LTO, optimized (slower build)
cargo build --profile release-fast # Thin LTO, faster build
```

Multi-platform releases are automated via GitHub Actions (`.github/workflows/release.yml`) triggered by version tags (`v*`):

- Linux: AppImage
- Windows: ZIP archive + Inno Setup installer (`installer/windows-installer.iss`, version injected from the tag)
- macOS: Universal binary DMG (Intel + Apple Silicon) + `FileCompressor-macOS-app.tar.zst` consumed by the in-app updater

End-user install instructions live in `INSTALLAZIONE.md`. Cross-platform tests run on every push via `.github/workflows/ci.yml` (Ubuntu/Windows/macOS).

**Tag and Cargo.toml version must match**: the `check-version` job in release.yml fails any `v*` tag that differs from Cargo.toml (past tags v0.2.1/v0.3.0 drifted; the in-app updater compares `CARGO_PKG_VERSION` against the remote tag, so drift would break it). Bump Cargo.toml first, then tag. Feature commits have regressed the version before (`add shrink feat` committed a stale 0.5.0 over the released 0.6.0): at release prep, compare `git show HEAD:Cargo.toml` against the latest tag's Cargo.toml, not just the working tree. Check history with `git tag -l` and `gh release list` (`gh` is installed and authenticated).

---
> Source: [edocico/file_compressor](https://github.com/edocico/file_compressor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
