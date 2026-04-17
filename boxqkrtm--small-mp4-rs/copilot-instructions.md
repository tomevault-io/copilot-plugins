## small-mp4-rs

> Small MP4 is a video compression tool designed to reduce video file sizes for easy sharing and uploading. The tool supports both GUI (egui) and CLI interfaces with hardware acceleration capabilities.

# Small MP4 - Video Compression Tool

## Project Overview
Small MP4 is a video compression tool designed to reduce video file sizes for easy sharing and uploading. The tool supports both GUI (egui) and CLI interfaces with hardware acceleration capabilities.

## Critical Requirements
- **Target file size MUST NOT be exceeded** - This is for upload purposes where strict size limits apply
- Uses bitrate-based compression (CRF-based compression has been removed)
- Default target size is 10MB
- Supports hardware acceleration (NVIDIA NVENC, AMD VCE, Intel QuickSync, VAAPI)

## Technical Architecture

### Core Components
1. **Compression Engine** (`src/compression/`)
   - Bitrate-based compression using FFmpeg
   - Hardware acceleration support
   - Precise size targeting with safety margins

2. **GUI Interface** (`src/gui/`)
   - Built with egui (native Rust GUI)
   - Compact layout design
   - Korean/Japanese font support (Noto Sans CJK)

3. **CLI Interface** (`src/cli/`)
   - Command-line compression
   - Hardware encoder listing
   - Batch processing support

### Key Implementation Details

#### Bitrate Calculation
```rust
// Calculate bitrate to ensure file doesn't exceed target
let total_bits = target_mb * 8.0 * 1024.0 * 1024.0;
let audio_bits = 128.0 * 1024.0 * duration_seconds;
let container_overhead = total_bits * 0.02;
let available_video_bits = total_bits - audio_bits - container_overhead;
let safe_bitrate = (video_bitrate_kbps * 0.95) as u32; // 95% safety margin
```

#### Compression Settings
- Target sizes: 1MB, 5MB, 10MB (default), 30MB, 50MB
- Audio bitrate: 128 kbps (fixed)
- Container overhead: 2%
- Safety margin: 5%

## Recent Changes
1. **Removed CRF-based compression** - Now uses only bitrate-based compression for precise size control
2. **Default size changed** - From 50MB to 10MB
3. **Removed auto-size feature** - Simplified UI
4. **GUI layout made more compact** - Reduced spacing, smaller window size (500x320)
5. **Output folder selection improved** - Only shows when "Same folder" is unchecked

## Testing Infrastructure
- Test videos are auto-generated when running tests
- Test videos are gitignored (not tracked in repository)
- Run tests with: `./run_tests.sh`
- Test scenarios include various resolutions and durations

## Development Guidelines
1. Always ensure output files stay under target size
2. Test with different video types before committing
3. Use software encoding for consistent test results
4. Keep GUI compact and user-friendly

## Hardware Support
- NVIDIA: NVENC (H.264, H.265, AV1)
- AMD: VCE/AMF (H.264, H.265)
- Intel: QuickSync (H.264, H.265, AV1)
- Linux: VAAPI
- macOS: VideoToolbox
- Fallback: Software encoding (x264/x265)

## Build and Run
```bash
# Build
cargo build --release

# Run GUI
./target/release/small-mp4 gui

# Run CLI compression
./target/release/small-mp4 compress input.mp4 -o output.mp4 --size 10mb

# List hardware encoders
./target/release/small-mp4 list-hw

# Run tests
./run_tests.sh
```

## Important Notes
- Files must never exceed target size for upload compatibility
- Bitrate is calculated with multiple safety margins
- Hardware acceleration is auto-detected but can be disabled
- Test videos are generated on-demand and not stored in git

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/boxqkrtm) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
