## pop3-downloader

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**pop3-downloader** is a Go CLI utility that downloads emails from a POP3S (secure POP3) server and stores them in mbox format. It supports duplicate detection via Message-ID headers, optional server-side deletion after download, and integration with mutt for reading.

## Common Commands

```bash
# Build
go build

# Run tests
go test ./...

# Run tests with verbose output
go test -v ./...

# Example usage
./pop3-downloader -host mail.example.com -username user@example.com
./pop3-downloader -host mail.example.com -username user@example.com -dryrun  # no deletion
./pop3-downloader -host mail.example.com -username user@example.com -read    # open in mutt
```

## Architecture

Single-package (`main`) Go application with logical file separation:

- **main.go** - Entry point, CLI flag parsing, config loading, workflow orchestration
- **pop3.go** - POP3S connection, TLS, authentication, message fetching/deletion
- **mbox.go** - mbox file writing, duplicate detection via Message-ID extraction

## Key Patterns

- **Error handling**: Standard Go error wrapping with `fmt.Errorf("context: %w", err)`
- **Testing**: Table-driven tests, mock POP3 server in `pop3_test.go`, fixtures in `testdata/`
- **Security**: Password in separate TOML config file (not CLI), mbox files created with 0600 permissions
- **Data safety**: Messages written to mbox BEFORE deletion from server

## Configuration

Password stored in `$HOME/.config/pop3-downloader-config.toml`:
```toml
password = "your-password-here"
```

## Dependencies

- `github.com/knadh/go-pop3` - POP3 client
- `github.com/emersion/go-mbox` - mbox format
- `github.com/BurntSushi/toml` - config parsing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/genesis32) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
