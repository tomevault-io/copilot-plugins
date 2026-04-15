## androiddraw

> Draw monospace text on Android device screen via ADB by simulating finger strokes.

# AndroidDraw - Project Reference

Draw monospace text on Android device screen via ADB by simulating finger strokes.

## Quick Start

```bash
# Using IBM Plex Mono Thin (thinner lines, good for strict line thickness requirements)
dotnet run -c Release -- "Console.Beep();" --font "IBM" --wiggle ";" --x 250 --y 1200 --width 620 --height 300

# Using default JetBrains Mono
dotnet run -c Release -- "Hello" --x 100 --y 500 --width 800 --height 200

# Preview commands without executing
dotnet run -c Release -- "Test" --x 100 --y 500 --width 800 --height 200 --dry-run
```

## CLI Options

| Option | Description |
|--------|-------------|
| `--x`, `--y`, `--width`, `--height` | Target area (required) |
| `--font <name>` | `JetBrains Mono` (default), `IBM`, `Hershey` |
| `--thin` | Skip offset pass for thinner lines |
| `--wiggle <chars>` | Characters needing priming wiggles (e.g., `";"`) |
| `--simple` | Use built-in geometric/LCD font |
| `--delay <ms>` | Stroke delay (default: 50) |
| `--dry-run` | Print ADB commands without executing |
| `--device <serial>` | Target specific ADB device |

## Architecture

```
Program.cs              - Entry point, orchestration
CommandLineOptions.cs   - CLI argument parsing
FontPathExtractor.cs    - SkiaSharp font -> SKPath (per-character)
PathToStrokes.cs        - SKPath -> List<Stroke> with character tagging
BezierInterpolator.cs   - Curves -> line segments (30 per curve)
StrokeSimplifier.cs     - Ramer-Douglas-Peucker (epsilon=1.5)
CoordinateTransformer.cs - Scale/translate to target area
AdbController.cs        - Executes motionevent commands via ADB
Stroke.cs               - Stroke model (points + source character)
SingleStrokeFont.cs     - Built-in angular/geometric font
```

## Key Constants (AdbController.cs)

- `PixelsPerMoveEvent = 2f` - interpolation density
- `OffsetPassPixels = 1` - diagonal offset for second pass
- `DotRadius = 5` - spiral circle size for dots
- `SmallStrokeThreshold = 15f` - strokes smaller than this become dots

## Embedded Fonts

- `Fonts/JetBrainsMono-Regular.ttf` - default
- `Fonts/IBMPlexMono-Thin.ttf` - thinnest strokes
- `Fonts/AVHersheySimplexLight.ttf` - single-stroke style

## ADB Path

```
C:\Program Files (x86)\Android\android-sdk\platform-tools\adb.exe
```

## Dependencies

- SkiaSharp 3.119.1

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Memphizzz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Memphizzz)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
