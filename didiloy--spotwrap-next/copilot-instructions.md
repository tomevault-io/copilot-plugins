## spotwrap-next

> This is a Spotify release tracking application built with Wails (Go + Web frontend). The application allows users to track their favorite artists and get notified of new releases.

# Project Overview

This is a Spotify release tracking application built with Wails (Go + Web frontend). The application allows users to track their favorite artists and get notified of new releases.

## Main Components

- [app.go](mdc:app.go): The main application entry point that handles core functionality
- [api/spotify.go](mdc:api/spotify.go): Spotify API integration and rate limiting
- [database/database.go](mdc:database/database.go): Database operations and schema
- [notifications/notifications.go](mdc:notifications/notifications.go): Desktop notification system

## Key Features

1. Artist tracking and release monitoring
2. Rate-limited Spotify API integration
3. Desktop notifications for new releases
4. Background checking for new releases
5. Token management and automatic refresh

## Architecture

The application follows a clean architecture pattern:
- Backend (Go) handles business logic and API calls
- Frontend (Web) provides the user interface
- Database stores artist information and credentials
- Background processes handle periodic checks

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Didiloy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
