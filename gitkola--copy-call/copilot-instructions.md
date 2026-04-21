## copy-call

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Copy Call is a serverless, copy-paste WebRTC application for video calls that works as a static site without any backend infrastructure. It uses a unique peer-to-peer connection flow where offers and answers are exchanged through URL fragments and manual copy-paste.

## Development Commands

- **Development server**: `bun run dev` or `bun index.html` - serves the HTML file with hot reload
- **Production build**: `bun build index.html` - builds the static site
- **Install dependencies**: `bun install`
- **Type checking**: Use TypeScript compiler directly via `bun tsc --noEmit`

## Architecture

### Core Application Structure
- **Single-file architecture**: The entire WebRTC application is contained within `index.html` with embedded CSS and JavaScript
- **No bundling required**: Bun serves the HTML file directly with built-in hot module replacement
- **State management**: The `MobileWebRTC` class manages all WebRTC peer connection state, UI state, and media streams

### WebRTC Flow
1. **Initiator Mode**: Creates offer, generates shareable URL with encoded offer data in fragment
2. **Responder Mode**: Parses offer from URL fragment, creates answer, generates base64-encoded answer code  
3. **Connection**: Initiator pastes answer code to establish peer connection
4. **Connected Mode**: Both parties see dual video streams with media controls

### Key Components
- **URL-based signaling**: Uses URL fragments with optimized compression instead of signaling server
- **Advanced compression**: Two-stage optimization (SDP aliasing + gzip) achieving 79% size reduction
- **Mobile-optimized UI**: Responsive design with touch-friendly controls and orientation handling
- **Media controls**: Camera and microphone toggle functionality in both modes
- **Connection state management**: Handles peer connection states and UI transitions

## Development Guidelines

### Bun-Specific Patterns
- Use `bun <file>` instead of `node <file>` 
- Bun automatically loads .env files
- HTML imports work directly with Bun's bundler - no need for separate bundler configuration

### WebRTC Implementation Notes
- **STUN servers**: Uses Google's public STUN servers for NAT traversal
- **Media constraints**: Optimized for mobile with 640x480 video resolution, with fallback to basic constraints
- **ICE gathering**: 3-second timeout for connection establishment
- **Error handling**: Comprehensive error handling with specific error types and user-friendly messages
- **Browser compatibility**: Automatic detection and warnings for unsupported features
- **HTTPS detection**: Automatic warnings and guidance for HTTP vs HTTPS access
- **Permission management**: Proactive permission checking and retry mechanisms
- **Device enumeration**: Automatic detection of available cameras and microphones

### Compression System
- **SDP Optimization**: Field aliasing reduces verbose WebRTC session descriptions by 27%
  - Extension URIs: `urn:ietf:params:rtp-hdrext:ssrc-audio-level` → `@1`
  - ICE patterns: `typ host generation 0` → `h g0` (conservative aliases to prevent parsing errors)
  - Codec names: `opus/48000/2` → `op48`
- **Gzip Compression**: Native browser CompressionStream API provides additional 62% reduction
- **Total Performance**: 79% size reduction (10k+ chars → ~2.1k chars)
- **Telegram Compliance**: URLs fit comfortably within 4,096 character limit
- **Browser Compatibility**: Handles Chrome's verbose SDPs and Safari's compact SDPs equally well
- **Backward Compatibility**: Automatic detection and fallback for uncompressed formats with priority order
- **Error Recovery**: Robust SDP parsing with detailed error messages and fallback mechanisms

### UI/UX Considerations
- **Mobile-first design**: Optimized for touch interfaces and mobile browsers
- **Orientation handling**: Dynamic layout switching between portrait/landscape
- **Copy-paste UX**: Simple clipboard operations for offer/answer exchange with compressed URLs
- **Visual feedback**: Status indicators, button state changes, and compression ratio logging
- **Performance indicators**: Console logs show SDP optimization and compression statistics
- **Error messaging**: User-friendly error messages with specific instructions for resolution
- **Debug information**: Comprehensive system information logging for troubleshooting
- **Retry mechanisms**: Automatic retry buttons for recoverable errors like permission denials
- **HTTPS warnings**: Prominent warnings and solutions for security-related camera access issues

## Error Handling & Troubleshooting

### Camera/Microphone Access Issues
The application includes comprehensive error handling for media access failures:

- **NotAllowedError**: Permission denied - provides instructions for enabling browser permissions
- **NotFoundError**: No devices found - guidance on device connectivity
- **NotReadableError**: Device busy - instructions to close other applications
- **OverconstrainedError**: Constraint not supported - fallback to basic constraints
- **SecurityError**: Security restrictions - HTTPS requirement warnings
- **Browser compatibility**: Detection of unsupported browsers with recommendations

### SDP Compression Issues
- Conservative SDP optimization to prevent parsing errors
- Automatic fallback between compressed and uncompressed formats
- Detailed error logging for WebRTC connection failures
- Priority order: uncompressed → compressed → error with detailed diagnostics

### Development Tools
- Real-time debug panel with system information
- Device enumeration and permission state logging
- Compression ratio and performance statistics
- Test page (`test.html`) for isolating camera access issues

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gitkola) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
