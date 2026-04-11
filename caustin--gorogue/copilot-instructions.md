## gorogue

> This document provides essential information for AI coding agents working on this Go-based roguelike game built with the Ebiten engine.

# Agent Instructions for gorogue

This document provides essential information for AI coding agents working on this Go-based roguelike game built with the Ebiten engine.

## Project Overview

- **Language**: Go 1.25.6
- **Module**: `github.com/caustin/rrogue`
- **Framework**: Ebiten v2.9.8 (2D game engine)
- **Structure**: Flat package structure (all code in `main` package)
- **Screen Size**: 1280x800 pixels (80x50 tiles at 16x16 each)

## Build, Run, and Test Commands

### Building and Running
```bash
# Build the binary
go build

# Run the game directly
go run .

# Build with optimizations
go build -ldflags="-s -w" -o rrogue

# Clean build artifacts
rm -f rrogue
```

### Testing
```bash
# Run all tests (when tests exist)
go test ./...

# Run tests with verbose output
go test -v ./...

# Run a single test
go test -run TestFunctionName ./...

# Run tests with coverage
go test -cover ./...

# Generate coverage report
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

### Linting and Formatting
```bash
# Format code (always run before committing)
go fmt ./...

# Vet code for common issues
go vet ./...

# Run golangci-lint (if installed)
golangci-lint run

# Check module dependencies
go mod tidy
go mod verify
```

## Code Style Guidelines

### Import Organization
```go
import (
    _ "image/png"      // Blank imports first (for side effects)
    "log"              // Standard library imports
    "fmt"
    
    "github.com/..."   // Third-party imports after blank line
)
```

### Naming Conventions
- **Types**: PascalCase (`Game`, `MapTile`, `GameData`)
- **Exported functions**: PascalCase (`NewGame`, `CreateTiles`, `GetIndexFromXY`)
- **Unexported functions**: camelCase (when needed)
- **Exported fields**: PascalCase (`PixelX`, `PixelY`, `Blocked`, `Image`)
- **Local variables**: camelCase (`gd`, `tile`, `wall`, `floor`)
- **Method receivers**: Single lowercase letter matching the type (`g` for `*Game`)
- **Constructor pattern**: Prefix with `New` (e.g., `NewGame`, `NewGameData`)

### Type Definitions
```go
// Struct types with exported fields
type Game struct {
    Tiles []MapTile
}

// Constructor functions
func NewGame() *Game {          // Return pointer for mutable types
    g := &Game{}
    g.Tiles = CreateTiles()
    return g
}

func NewGameData() GameData {   // Return value for immutable configs
    return GameData{
        ScreenWidth:  80,
        ScreenHeight: 50,
        TileWidth:    16,
        TileHeight:   16,
    }
}
```

### Error Handling
```go
// Standard pattern: check immediately after error-returning calls
wall, _, err := ebitenutil.NewImageFromFile("assets/wall.png")
if err != nil {
    log.Fatal(err)
}

// For critical errors in main/init, use log.Fatal
if err := ebiten.RunGame(g); err != nil {
    log.Fatal(err)
}

// Use blank identifier _ to explicitly ignore unused return values
_, ok := value.(Type)  // When you only need one value
```

### Documentation
- Add package-level documentation at the top of files
- Document all exported types, functions, and methods
- Use standard Go doc comment format:
```go
// GetIndexFromXY gets the index of the map array from a given X, Y tile coordinate.
// This coordinate is logical tile, not pixels.
func GetIndexFromXY(x int, y int) int {
    // ...
}
```

### Formatting Rules
- Use `gofmt` for all formatting (run before every commit)
- Use tabs for indentation (Go standard)
- Line length: No hard limit, but keep reasonable (~100-120 chars)
- Struct field alignment: Align field names and types for readability

## Ebiten Game Engine Patterns

### Required Interface Implementation
Every game must implement these three methods:
```go
func (g *Game) Update() error       // Game logic (60 ticks/second)
func (g *Game) Draw(screen *ebiten.Image)  // Rendering
func (g *Game) Layout(w, h int) (int, int) // Screen dimensions
```

### Drawing Images
```go
op := &ebiten.DrawImageOptions{}
op.GeoM.Translate(float64(x), float64(y))  // Position
screen.DrawImage(img, op)
```

### Asset Loading
```go
import _ "image/png"  // Required for PNG support

img, _, err := ebitenutil.NewImageFromFile("assets/image.png")
if err != nil {
    log.Fatal(err)
}
```

## Project-Specific Guidelines

### Game Data Management
- Use `NewGameData()` to get game configuration constants
- Centralize all game constants in `GameData` struct
- Don't hardcode dimensions; use `gd.ScreenWidth`, `gd.TileWidth`, etc.

### Tile Coordinate System
- Logical coordinates: Tile-based (0 to ScreenWidth-1, 0 to ScreenHeight-1)
- Pixel coordinates: Multiply logical by tile dimensions
- Use `GetIndexFromXY(x, y)` to convert 2D coordinates to 1D array index

### Known Issues to Avoid
- Bug in maptile.go:31 - uses `gd.TileWidth` instead of `gd.TileHeight` for PixelY calculation
- Asset images loaded in loop should be cached/reused for performance

## Development Workflow

1. **Before starting work**: Run `go mod tidy` to ensure dependencies are clean
2. **While coding**: Follow the style guidelines and naming conventions above
3. **Before committing**: Run `go fmt ./...` and `go vet ./...`
4. **Testing**: Write tests for new functionality (currently no tests exist)
5. **Asset changes**: Place all game assets in `assets/` directory

## Common Pitfalls

- Don't forget the `_ "image/png"` import for PNG support
- Remember that Ebiten's coordinate system has (0,0) at top-left
- `Update()` runs at 60 TPS regardless of frame rate
- Always use pointer receivers for methods that modify state
- Cache loaded images instead of loading them repeatedly

## Future Considerations

As the project grows, consider:
- Implementing unit tests for game logic
- Adding a configuration file for game constants
- Creating separate packages for different game systems
- Setting up CI/CD with linting and testing
- Adding player input handling and game state management

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/caustin)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/caustin)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
