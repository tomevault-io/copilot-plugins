## blkcache

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Vision

blkcache is a disk preservation toolkit that enables transparent, progressive caching of physical media. It's designed for archivists, digital preservationists, and anyone working with fragile or slow media who needs to minimize wear while maximizing data recovery.

The key insight: by inserting a transparent cache between applications and physical media, we can transform destructive, repeated reads into a one-time progressive archival process. Every read contributes to a complete preservation copy.

## Real-World Use Cases

- **Internet Archive Uploads**: Read a disk once while simultaneously uploading to IA, creating local backups, and generating checksums
- **Collaborative Disk Rescue**: Multiple tools can work on the same failing disk without re-reading sectors
- **Copy Protection Analysis**: Preserve raw sector data including intentional errors while tools analyze protection schemes
- **Sparse Disk Images**: Only store read sectors, perfect for partially-filled or damaged media
- **Retrocomputing**: Work with floppy disks and old CD-ROMs without wearing them out
- **Data Recovery**: Let multiple recovery tools examine a disk without repeated physical access

## Architecture

blkcache uses a composable, layered architecture that emphasizes clean abstractions:

### Core Layers

1. **Main Entry Point** (`main.py`): CLI interface that manages device monitoring and server lifecycle
2. **Server Layer** (`server.py`): Orchestrates nbdkit, nbdfuse, and FUSE mounting to create virtual block devices
3. **Backend Layer** (`backend.py`): nbdkit Python plugin that intercepts block device operations
4. **File Layer** (`file/`): Composable file abstraction system with automatic type detection

### Key Components

- **FileMap** (`file/filemap.py`): ddrescue-compatible mapfile for tracking read status and bad sectors
- **Cache Files**: Memory-mapped files storing actual sector data
- **Transitions**: Efficient representation of contiguous block states (unread → cached → error)

### Technical Approach

blkcache creates a virtual block device using:
- **nbdkit**: Provides the Network Block Device protocol
- **nbdfuse**: Exposes the NBD as a FUSE filesystem
- **Python Backend**: Handles read operations, caching, and error tracking

The implementation is transitioning from global state to a modular File-based architecture using composition over inheritance. This allows for clean separation of concerns and easy extension.

## Key Insight: Transparent Caching for Preservation

Traditional disk imaging is an all-or-nothing operation. If it fails, you start over. If you need the disk for something else mid-process, you interrupt the imaging.

blkcache inverts this model: every normal use of the disk contributes to its preservation. Mount a CD with blkcache, browse it, play music from it, run analysis tools - each operation permanently caches the sectors it reads. Over time, you build a complete image through natural use.

## Development Commands

```bash
# Create/activate virtual environment
make install

# Install dev dependencies
make dev

# Run tests
make test
# or directly: scripts/test.sh

# Run tests with coverage
make coverage

# Do not run make documentation unless asked.
# As this pushes to an external site.
make docs

# Build distribution
make dist

# Clean build artifacts
make clean
```

## Testing

Tests use pytest (functional style, not unittest OOP style). Test files mirror the source structure:
- Tests are in `tests/unit/`
- Test files are named `test_<module>.py`
- Tests should be short, well-named, and serve as executable documentation

## Key Technical Considerations

1. **Data Integrity**: This is a disk-level project - NEVER corrupt data. Better to crash with dignity than hide harmful bugs.
2. **Block Size**: Default is 4KB, but can be auto-detected based on device type or specified manually
3. **Caching**: Uses memory-mapped files for sector caching
4. **Disk Status Tracking**: Compatible with ddrescue format for tracking read status
5. **Transitions**: The FileMap uses a transition-based approach for efficiently representing block states

## Code Style

- Python 3.10+ with type hints where appropriate. i.e. dict[] rather than Dict[]
- Line length: 120 characters
- Use ruff for linting
- Modern pytest tests (functional style)
- Avoid unnecessary mocking - change code structure instead
- Keep logic flat and use descriptive variable names for conditions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bitplane)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bitplane)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
