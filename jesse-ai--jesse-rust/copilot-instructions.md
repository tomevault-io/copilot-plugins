## jesse-rust

> The jesse-rust repository is a Rust project that provides high-performance implementations of technical indicators and other computationally intensive functions for the Jesse trading framework. It uses PyO3 to create Python bindings, allowing the main Jesse Python codebase to call these optimized Rust functions.

# Jesse-Rust Repository Guide for AI Agents

## Overview
The jesse-rust repository is a Rust project that provides high-performance implementations of technical indicators and other computationally intensive functions for the Jesse trading framework. It uses PyO3 to create Python bindings, allowing the main Jesse Python codebase to call these optimized Rust functions.

## Repository Purpose
- Provide performant implementations of technical indicators
- Speed up calculations that would be slow in pure Python
- Expose Rust functions to Python through PyO3 bindings

## Development Workflow

### Making Changes
When implementing new Rust code or modifying existing functions:

1. **Write the Rust code** in the `src/` directory
2. **Build locally** to test your changes:
   ```bash
   cd /Users/salehmir/Codes/jesse/dev-jesse/jesse-rust
   ./build-local.sh
   ```
3. **Update Jesse's source code** to use the new Rust functions
   - Never add checks to see if the Rust version exists
   - Always assume the Rust implementation is available
4. **Run Jesse's tests** to verify everything works:
   ```bash
   cd /Users/salehmir/Codes/jesse/dev-jesse/jesse
   pytest
   ```

### Build Scripts
- `build-local.sh` - Build and install locally for development/testing
- `build-quick.sh` - Quick build for rapid iteration
- `build-comprehensive.sh` - Comprehensive build with all checks
- `build-all-wheels.sh` - Build wheels for distribution

### Python Integration
The Python interpreter for testing is located at:
```
/Users/salehmir/miniconda3/envs/jesse3.12/bin/python
```

## Important Notes

### Performance
- **Always optimize for performance** - This repository exists to speed up calculations
- **Use release mode** when compiling - Always compile in release mode for maximum performance
- **Double-check your code** - Ensure the implementation is as efficient as possible

### Code Style
- Follow Rust best practices and idiomatic patterns
- Ensure type safety and proper error handling

### Testing Strategy
1. Build the Rust code locally first
2. Update the Python code in jesse/ to use the new Rust functions
3. Run pytest in the jesse/ repository to validate
4. Never assume the Rust code works without running Jesse's full test suite

## Common Tasks

### Adding a New Indicator
1. Implement the indicator in Rust (`src/lib.rs` or appropriate module)
2. Add PyO3 bindings to expose it to Python
3. Build locally with `./build-local.sh`
4. Update `jesse/indicators/` to use the new Rust implementation
5. Run tests with `cd ../jesse && pytest`

### Debugging
- Use `println!` or `eprintln!` for debugging in Rust
- Use `jh.debug()` in the Python code when debugging Jesse's integration
- Check build output for compilation errors or warnings

### File Structure
- `src/` - Rust source code
- `Cargo.toml` - Rust dependencies and project configuration
- `pyproject.toml` - Python project metadata
- `build-*.sh` - Various build scripts

---
> Source: [jesse-ai/jesse-rust](https://github.com/jesse-ai/jesse-rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
