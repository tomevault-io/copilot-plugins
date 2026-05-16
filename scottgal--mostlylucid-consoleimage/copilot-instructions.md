## mostlylucid-consoleimage

> A high-quality ASCII art renderer for converting images and GIFs to terminal output.

# ConsoleImage - ASCII Art Renderer

A high-quality ASCII art renderer for converting images and GIFs to terminal output.
Based on Alex Harri's shape-matching algorithm: https://alexharri.com/blog/ascii-rendering

## Project Structure

```
ConsoleImage/
├── ConsoleImage.Core/           # Core library (NuGet package)
│   ├── AsciiRenderer.cs         # Main ASCII rendering engine
│   ├── ColorBlockRenderer.cs    # Unicode block-based rendering (2x resolution)
│   ├── BrailleRenderer.cs       # Braille character rendering (2x4 dots per cell)
│   ├── MatrixRenderer.cs        # Matrix digital rain effect rendering
│   ├── UnifiedRenderer.cs       # Unified entry point for all render modes
│   ├── UnifiedPlayer.cs         # Unified animation player for all formats
│   ├── AsciiAnimationPlayer.cs  # GIF playback with DECSET 2026
│   ├── ConsoleImageDocument.cs  # JSON document format for saving/loading
│   ├── CompressedDocument.cs    # Compressed .cidz format with delta encoding
│   ├── StreamingDocumentWriter.cs # NDJSON streaming writer for long videos
│   ├── DocumentPlayer.cs        # Playback of saved JSON documents
│   ├── GifWriter.cs             # GIF output writer (text-to-image rendering)
│   ├── RenderOptions.cs         # All configuration options
│   ├── CharacterMap.cs          # Character shape analysis and matching
│   ├── StatusLine.cs            # Status display below rendered output
│   ├── SmartFrameSampler.cs     # Intelligent frame skipping via perceptual hashing
│   ├── MarkdownRenderer.cs      # Markdown/SVG/HTML export for ASCII art
│   ├── SecurityHelper.cs        # Input validation (paths, URLs, browser names)
│   ├── CalibrationHelper.cs     # Aspect ratio calibration with circle test pattern
│   ├── ConsoleHelper.cs         # ANSI support + cell aspect ratio auto-detection
│   ├── TerminalCapabilities.cs  # Terminal feature detection
│   └── Subtitles/               # SRT/VTT parsing, rendering, and transcription
├── ConsoleImage/                # CLI tool
│   ├── Program.cs               # Entry point + subcommands
│   ├── CliOptions.cs            # CLI option definitions
│   ├── Handlers/                # ImageHandler, VideoHandler, SlideshowHandler, etc.
│   └── Utilities/               # RenderHelpers, TimeParser
├── ConsoleImage.Transcription/  # Whisper AI transcription library
├── ConsoleImage.Video.Core/     # Video playback library (FFmpeg-based)
│   ├── VideoAnimationPlayer.cs  # Video streaming player with live subtitles
│   └── FFmpegService.cs         # FFmpeg process management
├── ConsoleImage.Mcp/            # MCP server for LLM integration
├── ConsoleImage.Player/         # Standalone zero-dependency player library
│   ├── ConsolePlayer.cs
│   └── PlayerDocument.cs
├── ConsoleImage.Core.Tests/
├── ConsoleImage.Video.Core.Tests/
├── ConsoleImage.Player.Tests/
└── docs/JSON-FORMAT.md          # JSON document format spec
```

## Key Architectural Concepts

### Render Modes
- **ASCII** - Classic characters via shape-matching against pre-computed vectors
- **Blocks** - Unicode half-block chars (▀▄█) with 24-bit fg/bg color
- **Braille** - 2x4 dot cells at 2x resolution; default since v3.0
- **Matrix** - Digital rain effect; always animates even on still images
- **Monochrome** - Braille without color

`UnifiedRenderer` selects among modes by `RenderMode` enum. CLI handlers still have their own switch logic — see Known Code Duplication below.

### ANSI Support
All renderers call `ConsoleHelper.EnableAnsiSupport()` in their constructors (idempotent). Callers that write raw ANSI strings directly (e.g. document replay) must call it once manually. `ConsoleImage.Player` has its own inline enabler (zero-dependency by design).

### Document Formats
- **`.cidz`** - GZip + delta encoding (P-frames); global color palette; default for animations
- **Standard JSON** - All frames in one object; for images/short GIFs
- **NDJSON** - One JSON object per line; streaming writes; auto-finalizes on Ctrl+C
Auto-detected on load by extension or magic bytes. See `docs/JSON-FORMAT.md`.

### Frame Timing / GIF Handling
GIFs can use partial frames (delta encoding). Always composite onto a full-size canvas before processing — do NOT use `CloneFrame()` directly as it returns the smaller partial image. `ImageFrame.Bounds()` returns local (0,0) coordinates, not canvas offsets.

### Subtitle Pipeline
`SubtitleResolver` in Core is the canonical source for subtitle discovery (local SRT/VTT → embedded → YouTube → Whisper). `ConsolePlayer` has an intentional inline duplicate parser (~90 lines) to stay zero-dependency.

## Common Issues

### Animation Adds Extra Lines Per Frame
Off-by-one in cursor positioning. After writing N lines without trailing newline, cursor is ON line N-1. Move up `maxHeight - 1` lines, not `maxHeight`.

### Inter-line Color Artifacts
ANSI color codes bleed between lines during animation. Reset colors (`\x1b[0m`) at end of each line before the newline character.

### Variable Frame Height in GIFs
GIFs with partial frames return smaller images from `CloneFrame()`. Composite onto full canvas at (0,0) first.

### Braille Solarization
Color averaging across all cell pixels causes solarization. Only sample colors from pixels where dots are actually displayed (i.e. above brightness threshold).

### Extra Background Fill in GIF Output
Use `--auto-bg` to auto-detect uniform background, or `--dark-bg-threshold` / `--bg-threshold` to suppress.

## Known Code Duplication

These are known issues — don't add more:

1. **Render mode selection** (`if useMatrix / elif useBraille / ...`) — repeated in `ImageHandler`, `VideoHandler`, `SlideshowHandler`. `UnifiedRenderer` exists but CLI handlers don't use it yet.
2. **Frame playback loop** — `RenderHelpers.PlayFramesAsync()` and `SlideshowHandler.PlayFramesInPlaceAsync()` share core logic. Should extract a shared engine.
3. **Subtitle loading** — `SlideshowHandler` and `VideoHandler` both do subtitle discovery. `SubtitleResolver` should be the single source.
4. **ConsolePlayer subtitle parsing** — Intentional duplicate for zero-dependency design.

## Build

```bash
dotnet build
dotnet run --project ConsoleImage -- image.jpg
```

AOT build scripts: `ConsoleImage/build-aot.ps1` (Windows), `ConsoleImage/build-aot.sh` (Linux/macOS).
**AOT note:** Use `string?` for CLI file arguments, not `FileInfo?` — AOT has path resolution issues with System.CommandLine's `FileInfo` type.

## Dependencies

- SixLabors.ImageSharp — image loading/processing
- System.CommandLine — CLI parsing
- System.Text.Json with source generation — AOT-compatible JSON
- Targets .NET 10.0

## Security

Input validation via `SecurityHelper`:
- `--cookies-from-browser` only accepts whitelisted browser names
- File paths passed to FFmpeg/yt-dlp validated for shell metacharacters
- Only http/https URLs accepted for streaming

## Calibration

Per-mode aspect ratios saved in `calibration.json`. `CalibrationHelper` generates a circle test pattern. Chain: CLI `--char-aspect` → saved calibration → auto-detect from font metrics → 0.5 default.

---
> Source: [scottgal/mostlylucid.consoleimage](https://github.com/scottgal/mostlylucid.consoleimage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
