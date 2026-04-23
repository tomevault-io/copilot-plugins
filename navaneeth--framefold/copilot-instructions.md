## framefold

> Framefold is a minimal photo and video organizer written in Go, designed to run efficiently on devices like Raspberry Pi. It organizes media files based on EXIF metadata into configurable folder structures.

# Framefold - Project Context

## Overview
Framefold is a minimal photo and video organizer written in Go, designed to run efficiently on devices like Raspberry Pi. It organizes media files based on EXIF metadata into configurable folder structures.

## Architecture

### Core Files
- **main.go** - CLI entry point, uses Go's standard `flag` package for argument parsing
- **pkg/framefold/processor.go** - Core processing logic containing all file operations (~442 lines)
- **pkg/framefold/lock.go** - Process lock management (creates `.framefold.lock` in home directory)
- **pkg/framefold/config.go** - Configuration handling with default values
- **pkg/framefold/stats.go** - Statistics tracking for processed files
- **pkg/framefold/version.go** - Version information embedded via ldflags

### Key Operations Flow
1. Acquire process lock (skipped in dry-run mode)
2. Create target directory
3. Scan source directory recursively using `filepath.Walk`
4. For each media file:
   - Read EXIF metadata using external `exiftool` command
   - Parse folder template (Go templates)
   - Check for duplicate files via SHA-256 hash
   - Copy file to target location
   - Optionally delete source file (if `--delete-source` flag)
5. Clean up empty directories (if deleting source files)
6. Write processed files list (if `--output` specified)
7. Display statistics summary

## Command Line Flags

| Flag | Type | Description | Default |
|------|------|-------------|---------|
| `--source` | string | Source directory containing photos (required) | - |
| `--target` | string | Target directory to organize photos (required) | - |
| `--config` | string | Path to configuration file | "" |
| `--delete-source` | bool | Delete source files after successful copy | false |
| `--dry-run` | bool | Simulate operations without making changes | false |
| `--output` | string | Path to output file for processed files list | "" |
| `--version` | bool | Show version information | false |

## Processor Struct Fields

```go
type Processor struct {
    config            Config
    stats             Stats
    sourceDir         string
    targetDir         string
    deleteSource      bool
    dryRun            bool            // Added for dry-run support
    processedDirs     map[string]bool // Track directories with processed files
    processedFiles    []string        // Track processed files for output
    outputPath        string
    lock              *processLock
    exiftoolChecked   bool
    exiftoolAvailable bool
}
```

## File System Operations

### Operations That Modify State
These are conditionally skipped in dry-run mode with `[DRY RUN] Would...` logging:

1. **Lock acquisition** (lines 98-109) - Skipped entirely in dry-run
2. **Target directory creation** (lines 111-120) - `os.MkdirAll()`
3. **Subdirectory creation** (lines 268-278 in processFile) - `os.MkdirAll()`
4. **File copying** (lines 318-348) - `copyFile()` → `os.Create()`
5. **Source file deletion** (lines 305-313, 332-335) - `os.Remove()`
6. **Empty directory cleanup** (lines 155-203) - `os.Remove()`
7. **Output file writing** (lines 70-93, 142-150) - `os.Create()`

### Read-Only Operations
These continue running in dry-run mode:

- File scanning (`filepath.Walk`)
- EXIF metadata reading (external `exiftool` calls)
- Duplicate detection (SHA-256 hash calculation)
- Template parsing
- Statistics tracking
- Logging

## Configuration

Default configuration (if no config file specified):

```json
{
    "folder_template": "{{.Year}}/{{.Month}}",
    "media_types": {
        "images": [".jpg", ".jpeg", ".png", ".gif", ".heic"],
        "videos": [".mp4", ".mov", ".avi"]
    },
    "use_original_filename": true,
    "logging": {
        "enabled": true,
        "level": "info"
    }
}
```

### Template Variables
- `{{.Year}}` - Four-digit year (e.g., "2025")
- `{{.Month}}` - Two-digit month (e.g., "04")
- `{{.Day}}` - Two-digit day (e.g., "24")
- `{{.Hour}}` - Two-digit hour in 24-hour format (e.g., "15")
- `{{.Minute}}` - Two-digit minute (e.g., "30")
- `{{.MediaType}}` - Type of media (e.g., "images", "videos")
- `{{.Extension}}` - File extension without dot (e.g., "jpg", "mp4")

## Build System

Uses Makefile with the following targets:

- `make` or `make install` - Build and install to `$GOPATH/bin` (or `$HOME/go/bin`)
- `make local` - Build binary in current directory
- `make release` - Build release tarballs for all platforms
- `make clean` - Remove build artifacts

Version information is embedded via ldflags:
```bash
-ldflags "-X 'framefold/pkg/framefold.CommitHash=$(git rev-parse --short HEAD)'
          -X 'framefold/pkg/framefold.Version=$(git describe --tags --always)'"
```

## Dependencies

- **Standard Library**: No external Go dependencies, uses only stdlib
- **External Tool**: Requires `exiftool` to be installed on the system for EXIF metadata extraction

## Recent Development

### Dry-Run Feature (Latest)
Added `--dry-run` flag that:
- Simulates all operations without making file system changes
- Shows preview with `[DRY RUN] Would...` log messages
- Skips lock acquisition (allows concurrent dry-runs)
- Maintains accurate statistics for preview
- Works correctly with `--delete-source` flag

### Implementation Notes
- Modified files: `main.go`, `pkg/framefold/processor.go`, `README.md`
- Added conditional logic around all `os.MkdirAll()`, `os.Create()`, `os.Remove()` calls
- Read-only operations preserved for accurate simulation

## Git Information

- **Main branch**: `main`
- **Recent commits**: exiftool usage, output command, locking, OOM fixes
- **Current version**: v1.0.2 (based on git tags)

## Usage Examples

```bash
# Basic copy operation
framefold --source ~/Pictures/Unsorted --target ~/Pictures/Organized

# Preview without changes (dry-run)
framefold --source ~/Pictures/Unsorted --target ~/Pictures/Organized --dry-run

# Move files (delete source)
framefold --source ~/Pictures/Unsorted --target ~/Pictures/Organized --delete-source

# With custom config
framefold --source ~/Pictures/Unsorted --target ~/Pictures/Organized --config config.json

# Save processed files list
framefold --source ~/Pictures/Unsorted --target ~/Pictures/Organized --output processed.txt
```

## Important Behaviors

1. **Default is safe**: Copies files without deleting sources
2. **Lock file prevents concurrent runs**: Only one instance can run unless in dry-run mode
3. **Duplicate detection**: Uses SHA-256 hash comparison (skips identical files)
4. **EXIF fallback**: Falls back to file modification time if EXIF not available
5. **Recursive scanning**: Processes all subdirectories in source
6. **Empty directory cleanup**: Only happens with `--delete-source` flag

## Testing Considerations

When testing new features, consider:
- Dry-run mode behavior (no file system changes)
- Interaction with `--delete-source` flag
- Duplicate file detection accuracy
- EXIF vs modification time fallback
- Lock file behavior (concurrent runs)
- Empty directory cleanup
- Statistics accuracy
- Logging output format

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/navaneeth) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
