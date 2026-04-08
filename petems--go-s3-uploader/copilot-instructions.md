## go-s3-uploader

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Go S3 Uploader is an S3 uploader tool that speeds up uploads by caching MD5 sums of files locally. It only uploads files that have changed since the last run, making it ideal for large static sites with frequent small updates.

This is a fork of [alexaandru/go3up](https://github.com/alexaandru/go3up) maintained to add improvements and updates to the original implementation.

## Building and Testing

Build the binary with version information:
```bash
make build
```

The Makefile automatically injects version information from git (commit hash, tags, and build date).

Build directly with go (without version info):
```bash
go build
```

Run tests:
```bash
go test -race -coverprofile=coverage.txt -covermode=atomic ./...
```

Run tests with coverage report:
```bash
make cover
```

Run linter:
```bash
make lint
```

**Important**: Always run `make lint` in these situations:
- After completing implementation of a feature, fixing a bug, or making large code changes
- Before pushing code (unless you explicitly want to force things through)

Example run with test data:
```bash
./go-s3-uploader -bucket="s3.ungur.ro" -source=test/output -cachefile=test/.go3up.txt
```

## Architecture

### Core Components

**Options and Configuration** (opts.go, setup.go):
- Configuration is loaded via three-way merge: default values → config file (.go-s3-uploader.json) → command-line flags
- AWS authentication chain: shared profile → EC2 role → environment variables
- Configuration can be saved using the `-save` flag

**File Caching System** (main.go):
- Uses `github.com/alexaandru/utils` for file hashing and cache management
- `filesLists()` compares current directory state against cached MD5 sums to determine which files need upload
- Cache file stores file paths and their MD5 hashes
- Only changed files are uploaded on subsequent runs

**Upload Pipeline** (main.go):
- Concurrent upload system with worker goroutines (default: 2x CPU count)
- Failed uploads are retried with exponential backoff (100ms * 2^attempts)
- Maximum 10 retry attempts per file
- Recoverable errors (network issues, timeouts) trigger retries; non-recoverable errors are immediately rejected
- Results tracked in `syncedlist` (thread-safe list of rejected files)

**Headers and Compression** (source_file.go, setup.go):
- `customHeadersDef` maps file path patterns to HTTP headers (Content-Encoding, Cache-Control, encryption)
- Files matching gzip patterns are compressed on-the-fly during upload using io.Pipe
- Headers applied based on regex matching against file paths (first match wins)

**Logging** (utils.go):
- Three verbosity levels: verbose (all files), normal (progress indicators), quiet (errors only)
- `say()` function wraps logging, respecting verbosity flags
- In test environment, uses buffer instead of stdout

### Control Flow

1. Initialization (setup.go `init()`): Load config → parse flags → initialize AWS client
2. Validation: Check required flags (bucket, source directory exists)
3. File discovery: Build current file hash map, compare with cache to get diff
4. Upload phase: Spawn workers, distribute files via channel, handle retries
5. Cache update: Write successful uploads to cache file (excluding rejected files)

## Testing Notes

- Tests use `appEnv = "test"` to mock S3 operations and disable retry delays
- AWS initialization is skipped in test mode (detected via `-test.` flags)
- Test data in `test/` directory with sample cache files

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/petems)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/petems)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
