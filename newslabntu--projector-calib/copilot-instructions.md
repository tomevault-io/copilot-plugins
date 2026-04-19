## projector-calib

> **projector-calib** is a Rust-based interactive calibration tool for measuring projector intrinsic parameters and lens distortion coefficients. The tool uses an **inverse distortion workflow** where users adjust parameters representing physical lens distortion, and the program computes the exact mathematical inverse using OpenCV's iterative Newton-Raphson solver.

# projector-calib - Claude Code Context

## Project Overview

**projector-calib** is a Rust-based interactive calibration tool for measuring projector intrinsic parameters and lens distortion coefficients. The tool uses an **inverse distortion workflow** where users adjust parameters representing physical lens distortion, and the program computes the exact mathematical inverse using OpenCV's iterative Newton-Raphson solver.

**Target Resolution**: 1920x1080 (hardcoded)

**License**: MIT (Copyright 2025 Lin Hsiang-Jui)

**Repository**: https://github.com/NEWSLabNTU/projector-calib

## Key Architectural Decisions

### Inverse Distortion Approach

The tool applies **inverse distortion** (undistortion) as pre-warp for projector calibration:

1. User adjusts distortion parameters (k1, k2, k3, p1, p2) to match the physical lens
2. Program computes the exact inverse using `opencv::calib3d::init_undistort_rectify_map`
3. Ideal pattern is pre-warped with inverse distortion
4. Projector lens applies forward distortion, canceling out the pre-warp
5. Result: geometrically correct projection

**Why inverse?** The Brown-Conrady distortion model is NOT self-inverse. Forward distortion approximations (using `-k1` for lens with `+k1`) fail for large distortions. OpenCV's `init_undistort_rectify_map` uses iterative Newton-Raphson to compute the exact inverse.

### Joint Calibration

The tool performs **joint calibration** of camera matrix and distortion coefficients:
- Scale (`s`) and translation (`tx`, `ty`) are user-adjustable
- Camera matrix is computed: `fx = fy = s × width`, `cx = width/2 + tx`, `cy = height/2 + ty`
- Physical measurement (`physical_size_meters`) is optional and set via config file

## Code Structure

```
src/
  main.rs           - Main loop, key handling, tracing initialization
  config.rs         - CalibrationParams struct, JSON serialization
  display.rs        - Display management, help overlay, hints
  distortion.rs     - Inverse distortion using OpenCV (ImageWarper)
  pattern.rs        - Pattern generation (chessboard, etc.)

docs/
  math.org          - Mathematical formulation
  system_design.org - Architecture and workflow
  roadmap.org       - Development phases and testing

Makefile            - Build targets (build, run, test, check, format, lint, clean)
Cargo.toml          - Dependencies and build profile
README.md           - User documentation
LICENSE             - MIT License
```

## Important Implementation Details

### Distortion Engine (src/distortion.rs)

Uses OpenCV's `init_undistort_rectify_map` for exact inverse computation:

```rust
calib3d::init_undistort_rectify_map(
    &camera_matrix,      // [fx, 0, cx; 0, fy, cy; 0, 0, 1]
    &dist_coeffs,        // [k1, k2, p1, p2, k3]
    &no_array(),         // R (identity)
    &no_array(),         // New camera matrix (same as input)
    Size::new(width, height),
    CV_32FC1,
    &mut map_x,
    &mut map_y,
)?;
```

**Critical**: User provides LENS distortion parameters. OpenCV computes the inverse for pre-warping.

### Configuration File (calibration.json)

Auto-created on first run. Contains:
- User-adjustable: `scale`, `tx`, `ty`, `k1`, `k2`, `k3`, `p1`, `p2`
- Computed: `fx`, `fy`, `cx`, `cy`
- Optional: `physical_size_meters`

External edits are detected via file modification time. User is prompted to press `L` to reload.

### Key Bindings

- **Scale/Translation**: `+/-`, arrow keys
- **Distortion (k1-k3)**: `W/S`, `E/D`, `R/F`
- **Distortion (p1-p2)**: `T/G`, `Y/H`
- **Step multiplier**: `[/]` (adjust step size)
- **Cell size**: `,/.` (pattern cell size)
- **Pattern cycling**: `TAB`, `Shift+TAB`
- **Actions**: `I` (toggle help), `L` (reload config), `0` (reset), `SPACE` (save), `ESC` (quit)

### Logging

Uses `tracing` crate for structured logging:
- `info!()` for user-facing messages
- `debug!()` for detailed debugging (step changes, pattern switches)
- `error!()` for errors

Default level: `info`. Override with `RUST_LOG=debug` or `RUST_LOG=trace`.

## Code Conventions

### Rust Edition

Uses **Rust 2024** features:
- Let-chain syntax for flattening nested if-let: `if let Ok(x) = foo() && let Some(y) = bar() { ... }`
- Context structs to reduce function parameter counts

### Linting

Must pass `make lint` without warnings:
- `cargo +nightly fmt --check` (formatting)
- `cargo clippy --profile release-with-debug -- -D warnings` (clippy)

Use `#[allow(dead_code)]` only for intentionally unused items kept for future use.

### Build Profile

Uses `release-with-debug` profile (defined in Cargo.toml):
- Release optimizations for performance
- Debug symbols for debugging
- No stripping

## Development Workflow

### Building and Running

```bash
make build    # Build with release-with-debug profile
make run      # Build and run in fullscreen
```

### Testing and Quality

```bash
make test     # Run tests
make check    # Run cargo check
make format   # Format code with rustfmt
make lint     # Check formatting and run clippy
make clean    # Clean build artifacts
```

### Common Tasks

**Adding a new keyboard command:**
1. Add case to `process_key()` in `src/main.rs`
2. Update help overlay in `src/display.rs`
3. Update README.md controls section
4. Document in `docs/system_design.org`

**Modifying distortion computation:**
- Edit `compute_distortion_maps()` in `src/distortion.rs`
- Update `docs/math.org` with mathematical changes
- Update test cases in `docs/roadmap.org`

**Changing patterns:**
- Edit `PatternGenerator` in `src/pattern.rs`
- Add new `PatternType` enum variant
- Update pattern cycling logic in `src/main.rs`

## Current Limitations

1. **Fixed resolution**: 1920x1080 hardcoded in main.rs
2. **Single projector**: No multi-projector support
3. **Manual adjustment**: No automatic camera-based optimization
4. **Brown-Conrady only**: No fisheye or other distortion models
5. **Limited patterns**: Currently chessboard only (pattern cycling exists but not fully implemented)

## Documentation Guidelines

### For Code Changes

Always update:
1. Inline code comments for complex logic
2. `docs/system_design.org` for architectural changes
3. `docs/roadmap.org` for completed features
4. README.md for user-facing changes

### Org-mode Format

Documentation in `docs/` uses Emacs Org-mode (`.org` files):
- Headers: `* Level 1`, `** Level 2`, etc.
- Code blocks: `#+BEGIN_SRC rust ... #+END_SRC`
- Lists: `-` for unordered, `1.` for ordered
- Bold: `*text*`, Italic: `/text/`, Code: `~text~`

## Testing Strategy

### Phase 1 (MVP) - COMPLETED

- [x] Project setup with Rust 2024 and OpenCV
- [x] Pattern generation (chessboard)
- [x] Inverse distortion using OpenCV
- [x] Display management with help overlay
- [x] Key handling with context struct
- [x] Configuration save/load with external edit detection
- [x] Structured logging with tracing
- [x] Lint compliance (clippy, rustfmt)

### Phase 2 (Future)

- [ ] Additional pattern types (grid lines, circles)
- [ ] Undo/redo for parameter changes
- [ ] Named presets for different projectors
- [ ] Visual quality metrics

### Phase 3 (Future)

- [ ] Camera-based automatic calibration
- [ ] Export to OpenCV YAML and ROS camera_info formats

## Important Files to Preserve

**Never modify without explicit user request:**
- `LICENSE` - MIT License, Copyright Lin Hsiang-Jui
- `Cargo.toml` edition field (must stay "2024")
- OpenCV features in dependencies (highgui, imgproc, calib3d)

**Always update together:**
- `src/main.rs` key bindings + `src/display.rs` help text + README.md controls
- `src/distortion.rs` implementation + `docs/math.org` formulation
- Cargo.toml version + README.md version references

## Common Gotchas

1. **Forward vs Inverse Distortion**: Always use inverse (undistortion) for projector pre-warp
2. **Let-chains require Rust 2024**: Don't downgrade edition in Cargo.toml
3. **OpenCV feature flags**: `calib3d` is required for `init_undistort_rectify_map`
4. **File modification detection**: Uses `SystemTime`, handle errors gracefully
5. **Arrow key codes**: Platform-specific (81=Left, 82=Up, 83=Right, 84=Down)

## External Dependencies

- **OpenCV 4.x**: Requires system installation with development headers
- **Rust nightly**: For `cargo fmt` (formatting)
- **Rust stable 2024+**: For compilation

## Questions to Ask Before Code Changes

1. Does this affect the inverse distortion computation? → Update `docs/math.org`
2. Does this add/change keyboard bindings? → Update help overlay and README
3. Does this change user-facing behavior? → Update README.md
4. Does this complete a roadmap item? → Mark as completed in `docs/roadmap.org`
5. Will this pass `make lint`? → Test before committing

## Project Philosophy

This tool prioritizes:
1. **Mathematical correctness** over approximations (hence exact inverse)
2. **Real-time feedback** over batch processing
3. **Manual control** over automatic algorithms (for fine-tuning)
4. **Simplicity** over feature bloat (single projector, fixed resolution)
5. **Documentation** over implicit knowledge (extensive docs/ directory)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NEWSLabNTU) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
