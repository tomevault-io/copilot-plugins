## claude-calico

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Radio Calico is a live streaming radio application that provides 24-bit / 48 kHz lossless audio streaming with a modern web interface. The project consists of:

- **Backend**: Express.js server with SQLite database
- **Frontend**: Single-page HTML application with HLS.js for audio streaming
- **Stream Source**: CloudFront-hosted HLS stream with live metadata
- **Features**: Live audio playback, real-time metadata display, song rating system, recently played tracks

## Development Commands

### Starting the Application
- `npm start` - Starts the Express.js server on port 3000
- Server serves static files and provides API endpoints
- Main interface accessible at http://localhost:3000

### Available npm Scripts
- `npm start` - Run the production server
- `npm test` - Currently returns test error (no tests configured)

### Dependencies
- **express**: Web server framework (v5.1.0)
- **sqlite3**: Database for song ratings and metadata (v5.1.7)
- **hls.js**: Client-side HLS streaming (loaded via CDN)

## Architecture Overview

### Backend (server.js)
- **Express Server**: Handles HTTP requests and serves static files
- **SQLite Database**: In-memory database with three tables:
  - `tracks`: Basic track information and play history
  - `ratings`: User song ratings (thumbs up/down)
  - `song_stats`: Aggregated rating statistics
- **Metadata Fetching**: Polls CloudFront endpoint every 30 seconds for live stream metadata
- **Rating System**: MD5-based song and user identification for persistent ratings
- **API Endpoints**:
  - `GET /api/metadata` - Current track info with ratings
  - `POST /api/rate` - Submit song ratings
  - `GET /api/ratings/:songId` - Get specific song ratings

### Frontend (radio-player.html)
- **Audio Player**: HLS.js-powered streaming with fallback to native HLS
- **Metadata Display**: Real-time current track and recently played tracks
- **Rating Interface**: Thumbs up/down voting system
- **Responsive Design**: Mobile-friendly layout with custom CSS
- **Color Scheme**: Follows brand guidelines from RadioCalico_Style_Guide.txt

### Key Data Flow
1. Server fetches metadata from CloudFront every 30 seconds
2. Client polls server API every 10 seconds when playing
3. User ratings stored in SQLite with user identification
4. Album art served from CloudFront CDN

## File Structure

```
/
├── server.js                    # Main Express server
├── radio-player.html           # Complete frontend application
├── package.json                # Project dependencies and scripts
├── stream_URL.txt              # HLS stream endpoint
├── RadioCalico_Style_Guide.txt # Brand and design guidelines
├── RadioCalicoLogoTM.png       # Logo file
├── CLAUDE.md                   # This file
└── node_modules/               # Dependencies
```

## Brand Guidelines

The project follows specific brand guidelines defined in `RadioCalico_Style_Guide.txt`:

### Color Palette
- **Mint**: #D8F2D5 - Backgrounds and accents
- **Forest Green**: #1F4E23 - Primary buttons and headings
- **Teal**: #38A29D - Navigation and highlights
- **Calico Orange**: #EFA63C - Call-to-action elements
- **Charcoal**: #231F20 - Body text
- **Cream**: #F5EADA - Secondary backgrounds
- **White**: #FFFFFF - Text on dark backgrounds

### Typography
- **Headers**: Montserrat (weights 500-700)
- **Body**: Open Sans (weight 400-500)
- **Fallback**: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif

## Development Notes

### Database
- Uses in-memory SQLite database (data resets on server restart)
- User identification based on IP + User-Agent hash
- Song identification uses MD5 hash of title + artist

### Stream Configuration
- HLS stream URL: https://d3d4yli4hf5bmh.cloudfront.net/hls/live.m3u8
- Metadata endpoint: https://d3d4yli4hf5bmh.cloudfront.net/metadatav2.json
- Album art: https://d3d4yli4hf5bmh.cloudfront.net/cover.jpg

### Error Handling
- HLS playback includes automatic retry and recovery
- Metadata fetching gracefully handles network errors
- Rating submissions include validation and error responses

### Browser Compatibility
- Uses HLS.js for modern browsers
- Falls back to native HLS support (Safari)
- Responsive design for mobile devices

---
> Source: [benmiller169/claude_calico](https://github.com/benmiller169/claude_calico) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-12 -->
