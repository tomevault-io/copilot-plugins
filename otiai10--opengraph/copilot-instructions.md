## opengraph

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Build and Test
- **Run all tests**: `go test -v .`
- **Run tests with coverage**: `go test -race -coverprofile=coverage.txt -covermode=atomic`
- **Run a single test**: `go test -v -run TestName`
- **Format code**: `go fmt ./...`
- **Vet code**: `go vet ./...`
- **Tidy modules**: `go mod tidy`

### CLI Tool
- **Build CLI**: `go build -o ogp ./cmd/ogp`
- **Install CLI**: `go install github.com/otiai10/opengraph/v2/cmd/ogp@latest`
- **Run CLI**: `ogp -A [URL]` or `ogp --help`
- **Run with custom headers**: `ogp -H "User-Agent: MyBot/1.0" -H "Accept-Language: en-US" [URL]`

## Architecture

This is a Go library for parsing Open Graph Protocol (OGP) metadata from web pages. The core architecture:

### Main Components

1. **OpenGraph struct** (opengraph.go:28-54): Central data structure holding all OGP metadata including basic fields (title, type, image, URL), optional fields (audio, description, video), and additional metadata (favicon). Uses an Intent field to control parsing behavior.

2. **Intent struct** (intent.go): Configures parsing behavior including HTTP client, context, strict mode, trusted HTML tags, and custom HTTP headers. Controls whether non-standard tags are parsed and allows customization of request headers to bypass bot protection.

3. **Tag Parsing System**:
   - **MetaTag** (meta.go): Parses `<meta>` tags and extracts OGP properties
   - **LinkTag** (tags.go): Handles `<link>` tags for favicon extraction
   - **TitleTag** (tags.go): Falls back to `<title>` when no og:title exists (non-strict mode)

4. **Structured Properties** (structured.go): Type definitions for nested OGP properties (Image, Video, Audio, Favicon) with their respective sub-properties.

### Key Design Patterns

- **Walk Pattern**: The `Walk()` method (opengraph.go:132-161) recursively traverses HTML nodes to extract OGP data based on trusted tags.

- **Contribution Pattern**: Each tag type has a `Contribute()` method that updates the OpenGraph struct with parsed data. Meta tags handle complex property mapping including structured properties.

- **URL Resolution**: `ToAbs()` method converts relative URLs to absolute using the base URL from og:url or Intent.URL.

- **Flexible Parsing Modes**:
  - **Strict mode**: Only trusts `<meta>` tags with OGP properties
  - **Non-strict mode**: Falls back to standard HTML tags (`<title>`, `<meta name="description">`, favicon links)

- **Bot Protection Bypass**: Default browser-like headers (User-Agent, Accept, Accept-Language) are automatically added to HTTP requests to avoid bot detection. Custom headers can be specified via Intent.Headers for sites with specific requirements.

### Error Handling

- Functions return errors rather than panicking
- Content-Type validation ensures only HTML is parsed
- Graceful handling of missing or malformed properties

### Module Structure

This is a v2 module (`github.com/otiai10/opengraph/v2`) requiring Go 1.24+. Primary dependencies are `golang.org/x/net/html` for HTML parsing.

---
> Source: [otiai10/opengraph](https://github.com/otiai10/opengraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
