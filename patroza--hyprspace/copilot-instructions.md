## hyprspace

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HyprSpace is an i3-like tiling window manager for macOS written in Swift. The project uses Swift Package Manager (SPM) for building and dependency management, with Xcode project generation for App Bundle creation.

## Architecture

### Core Components

1. **Client/Server Architecture**:
   - `hyprspace` CLI binary (client) - Located in `Sources/Cli/`
   - `HyprSpace.app` (server) - Located in `Sources/AppBundle/`
   - Communication via UNIX socket at `/tmp/barut.hyprspace-{username}.sock`

2. **Tree-based Window Management**:
   - Window hierarchy managed as a tree structure in `Sources/AppBundle/tree/`
   - Workspaces, containers, and windows organized hierarchically
   - Supports tiling layouts: tiles, accordion, dwindle

3. **Command System**:
   - Commands defined in `Sources/AppBundle/command/`
   - Shared argument parsing in `Sources/Common/cmdArgs/`
   - Each command has corresponding implementation and argument parsing

4. **Configuration**:
   - TOML-based configuration in `~/.hyprspace.toml` (release) or `~/.hyprspace-debug.toml` (debug)
   - Config parsing in `Sources/AppBundle/config/`

5. **Package Targets** (from `Package.swift`):
   - `Common`: Shared code with argument parsing
   - `PrivateApi`: C wrapper for `_AXUIElementGetWindow` private API
   - `AppBundle`: Core window manager logic (library)
   - `HyprSpaceApp`: Main app executable (SPM builds)
   - `Cli`: hyprspace CLI binary
   - `AppBundleTests`: Test suite

## Build Commands

### Development
```bash
# Build debug version (uses SPM, outputs to .debug/)
./build-debug.sh

# Run tests (comprehensive: build, tests, format, generation checks)
./run-tests.sh

# Run only Swift unit tests
./run-swift-test.sh

# Run debug app
./run-debug.sh

# Run CLI in debug mode
./run-cli.sh [args]

# Format code (SwiftFormat + SwiftLint)
./format.sh

# Generate files (xcodeproj, parser, cmd help)
./generate.sh

# Generate parser from ANTLR grammar
./generate-shell-parser.sh

# Reset accessibility permissions for debug app
./script/reset-accessibility-permission-for-debug.sh

# Clean project completely
./script/clean-project.sh
```

### Release
```bash
# Build release version (uses Xcode via XcodeGen, outputs to .release/)
./build-release.sh

# Install from sources as hyprspace-dev brew cask
./install-from-sources.sh

# Build brew cask package
./script/build-brew-cask.sh
```

### Documentation & Completion
```bash
# Build documentation and man pages
./build-docs.sh

# Build shell completion (requires bash 5+)
./build-shell-completion.sh
```

### Makefile Support
```bash
# For vim :make integration
make build      # Runs ./build-debug.sh
make test       # Runs ./run-tests.sh
make swift-test # Runs ./run-swift-test.sh
make format     # Runs ./format.sh
```

## Testing

- Unit tests: `swift test` or `./run-swift-test.sh`
- Full test suite: `./run-tests.sh` (includes build, tests, format check, and generation verification)
- Test files located in `Sources/AppBundleTests/`
- Single test: `swift test --filter TestName`

## Development Setup

1. **Version Requirements**:
   - Swift 6.1.2 (via `.swift-version`, managed with swiftly)
   - Ruby 3.3.4 (via `.ruby-version`, for documentation)
   - Xcode (from App Store, for release builds only)

2. **Dependencies** (auto-installed by `./script/install-dep.sh`):
   - SwiftFormat 0.56.4 (code formatting)
   - SwiftLint 0.59.1 (linting with strict mode)
   - XcodeGen 2.43.0 (generates Xcode project from `project.yml`)
   - complgen (shell completion from BNF grammar)
   - ANTLR4 tools (parser generation from `.g4` files)
   - asciidoctor 2.0.23 (documentation)

3. **Code Signing** (for App Bundle):
   - Create self-signed certificate in Keychain Access
   - Name: `hyprspace-codesign-certificate`
   - Type: Code Signing

4. **IDE Setup**:
   - Xcode: Open `Package.swift` (not the .xcodeproj)
   - Set Xcode scheme console to Terminal for accessibility permissions
   - LSP support available via sourcekit-lsp for other editors
   - EditorConfig enforces: 120 char lines, 4-space indentation

## Key Subsystems

### Tree Model (`Sources/AppBundle/tree/`)
- `TreeNode`: Base node type for the window hierarchy
- `Workspace`: Virtual workspace implementation
- `TilingContainer`: Container for tiled windows
- `MacWindow`/`Window`: Window abstractions
- Frozen tree structure for immutability

### Layout System (`Sources/AppBundle/layout/`)
- Layout algorithms for tiles, accordion, dwindle modes
- Normalization of container structures
- Focus management and window placement

### Command Processing (`Sources/AppBundle/command/`)
- Individual command implementations
- Command validation and execution
- Integration with tree model for window manipulation

### Configuration (`Sources/AppBundle/config/`)
- TOML parsing via TOMLKit dependency
- Keybinding management via HotKey dependency
- Workspace-to-monitor assignments
- Gap and padding configurations

### Parser Generation (`grammar/`)
- `ShellLexer.g4` / `ShellParser.g4`: ANTLR4 grammar definitions
- `commands-bnf-grammar.txt`: BNF for shell completion
- Generated code in `ShellParserGenerated/` (excluded from formatting)

## Build Outputs

- `.debug/`: Debug build outputs (hyprspace CLI + HyprSpaceApp)
- `.release/`: Release build outputs from Xcode
- `.site/`: Generated documentation site
- `.man/`: Generated man pages
- `.shell-completion/`: Shell completion scripts
- `.deps/`: Downloaded dependencies and tools
- `.xcode-build/`: Xcode build directory

## Important Notes

- Uses macOS Accessibility API (no SIP disabling required)
- Single private API usage: `_AXUIElementGetWindow`
- Emulates virtual workspaces instead of using macOS Spaces
- Thread-per-application architecture for non-blocking AX API calls
- Debug builds use `~/.hyprspace-debug.toml` configuration
- Bundle IDs: `barut.hyprspace.debug` (Debug), `barut.hyprspace` (Release)
- Runs as agent app (LSUIElement) without Dock icon
- Sandbox disabled for Accessibility API access

## Common Development Tasks

### Adding a New Command
1. Define command arguments in `Sources/Common/cmdArgs/`
2. Implement command logic in `Sources/AppBundle/command/`
3. Update documentation in `docs/hyprspace-*.adoc`
4. Add shell completion in `grammar/commands-bnf-grammar.txt`
5. Run `./generate.sh` to update generated files

### Modifying Tree Structure
- Core tree logic in `Sources/AppBundle/tree/`
- Ensure normalization rules are maintained
- Update frozen tree structure if needed
- Test with multiple monitor setups

### Debugging
- Use Accessibility Inspector.app for window properties
- Test multiple monitors with DeskPad or BetterDisplay 2
- Check `.debug/` directory for debug builds
- Logs and debugging available through CLI commands
- Reset accessibility: `./script/reset-accessibility-permission-for-debug.sh`

---
> Source: [patroza/HyprSpace](https://github.com/patroza/HyprSpace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
