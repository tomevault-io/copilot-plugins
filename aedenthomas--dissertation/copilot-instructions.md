## dissertation

> - Starting development environment for WebRTC screen sharing evaluation project

# Development Log

## 2025-09-07 - Initial Development Environment Setup

- Starting development environment for WebRTC screen sharing evaluation project
- Project compares P2P vs SFU architectures for screen sharing performance
- Key metrics: CPU utilization, Glass-to-Glass latency, Text Legibility Score
- Dead End: Docker Compose build failed - "rpc error: code = Unavailable desc = error reading from server: EOF"
  - Issue occurred during client build process after ~245 seconds
  - Likely resource constraints or Docker daemon issue during large build
- Success: Colima Docker daemon started successfully
- Success: Local development environment started successfully
  - Signaling server on port 3001 (healthy)
  - SFU server on port 3002 (mock mode, healthy)
  - React client running on port 3000
- Success: Fixed screen sharing connection issues
  - Dead End: WebRTC client connecting to Docker hostname instead of localhost
  - Dead End: Signaling server not handling specific WebRTC events (offer/answer/ice-candidate)
  - Success: Changed signaling URL from 'http://signaling-server:3001' to 'http://localhost:3001'
  - Success: Added specific WebRTC event handlers to signaling server (offer, answer, ice-candidate)
  - Success: Added comprehensive debugging statements throughout WebRTC service and signaling server
  - Result: P2P and SFU screen sharing is now working between presenter and viewers
  - Success: Signaling server WebSocket connections established successfully
  - Success: Session management working properly (preventing multiple presenters per session)
  - Success: Fixed macOS screen sharing permission dialog issue
    - Problem: Browser gets stuck waiting for screen selection after granting permission
    - Root cause: macOS has two-step permission process (browser permission + system screen selection)
    - Solution: Added macOS-specific user instructions and enhanced error handling
    - Added specific error messages for NotAllowedError, AbortError, NotFoundError, NotSupportedError
    - Added track 'ended' event listener to detect when user stops sharing via browser controls
  - Success: Fixed multiple permission dialog issue (4 clicks problem)
    - Root cause: React useEffect dependency cycle causing multiple startSharing calls
    - Issue 1: startSharing callback depended on isSharing state, creating re-render cycle
    - Issue 2: useEffect dependencies included callbacks that changed on every render
    - Solution: Added protection flags (isRequestingPermission, hasInitialized) to prevent multiple calls
    - Solution: Removed isSharing dependency from startSharing callback
    - Solution: Changed useEffect dependencies to only sessionId and role (stable values)
    - Added comprehensive debugging with call numbers and state tracking
  - Success: Fixed cross-device connectivity issue (Mac to iPhone)
    - Problem: iPhone viewers couldn't see Mac presenter's screen
    - Root cause: WebRTC service hardcoded signaling URL to 'http://localhost:3001'
    - Issue: When iPhone accesses Mac's IP (192.168.4.31:3000), it tried connecting to iPhone's localhost:3001
    - Solution: Made signaling URL dynamic using window.location.hostname
    - Implementation: URL now automatically becomes 'http://192.168.4.31:3001' when accessed from iPhone
    - Result: Cross-device screen sharing now works between Mac presenter and iPhone viewer

# Tech Stack

## Frontend

- React 18.x with TypeScript
- WebRTC APIs for P2P connections
- Custom hooks and services for metrics collection

## Backend

- Node.js signaling server with Socket.io
- Mediasoup SFU server for selective forwarding
- Express.js for REST API endpoints

## Testing & Analysis

- Puppeteer for automated browser testing
- Python 3.9+ for data analysis and visualization
- Tesseract.js for OCR-based text legibility scoring

## Infrastructure

- Docker and Docker Compose for orchestration
- Network traffic control (tc qdisc) for simulating conditions

# Architecture Overview

## Directory Structure

- `/client/` - React TypeScript frontend application
- `/signaling-server/` - WebRTC signaling coordination server
- `/sfu-server/` - Mediasoup SFU implementation
- `/test-framework/` - Automated testing with Puppeteer
- `/data-analysis/` - Python-based statistical analysis and visualization
- `/docker/` - Container orchestration files

## Entry Points

- Client: `client/src/index.tsx` - React application entry
- Signaling: `signaling-server/src/server.js` - Express + Socket.io server
- SFU: `sfu-server/src/server.js` - Mediasoup SFU server
- Tests: `test-framework/src/index.js` - Main test suite runner

## Configuration

- Docker Compose: `docker/docker-compose.yml` - Complete system orchestration
- Multiple Dockerfiles for different services
- Environment variables for service coordination

# Module Dependencies

## Component Relationships

- Client connects to signaling server for session coordination
- SFU server coordinates with signaling server for room management
- Test framework controls multiple browser instances via Puppeteer
- Data analysis processes results from test framework output

## Data Flow

- WebRTC signaling flow: Client -> Signaling Server -> SFU Server
- Metrics collection: Client -> Test Framework -> CSV/JSON results
- Analysis pipeline: Raw results -> Python analysis -> Visualization charts

## External Integrations

- WebRTC native browser APIs for media streaming
- Mediasoup for SFU functionality
- Tesseract.js for OCR text analysis
- TC (Traffic Control) for network condition simulation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AedenThomas)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AedenThomas)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
