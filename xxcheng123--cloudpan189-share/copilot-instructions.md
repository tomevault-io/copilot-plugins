## cloudpan189-share

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Memory Requirements

**IMPORTANT**: All communication and responses must be in Chinese (中文). Claude must use Chinese for all interactions with the user, including:
- 所有回复必须使用中文
- 代码注释和文档说明使用中文
- 错误信息和提示使用中文
- 任何交流都必须用中文进行

## Common Development Commands

### Building the Project
- `make build` - Build both frontend and backend
- `make build-frontend` - Build frontend only (Vue.js)
- `make build-backend` - Build backend only (Go binary)
- `make build-multi-arch` - Build for multiple architectures (Linux, Windows, macOS)

### Development
- `make dev` - Start development server (backend only)
- `go run ./cmd/main.go` - Direct backend development
- Frontend development: `cd fe && npm run dev` (starts Vite dev server on port 5173)

### Testing and Quality
- `make test` - Run Go tests
- `make lint` - Run golangci-lint
- Frontend linting: `cd fe && npm run lint` (ESLint + stylelint)
- Frontend formatting: `cd fe && npm run format` (Prettier)

### Docker
- `make docker-build` - Build Docker image
- `make docker-run` - Run Docker container
- `make docker-stop` - Stop Docker container

### Documentation
- `make swag-init` - Generate Swagger documentation
- `make swag-fmt` - Format Swagger comments

## Architecture Overview

This is a Go-based web application that provides WebDAV access to Cloud189 (天翼云盘) shared files with a Vue.js frontend.

### Backend Architecture (Go)

**Entry Point**: `cmd/main.go` - Main application entry

**Core Structure**:
- `internal/` - All internal packages
  - `bootstrap/` - Application bootstrap and service context
  - `configs/` - Configuration management
  - `framework/` - Custom framework components (HTTP context, task context)
  - `handler/` - Request handlers
    - `http/` - HTTP API handlers
    - `consumer/` - Message consumers for background tasks
  - `models/` - Data models and database entities
  - `router/` - Route definitions
  - `services/` - Business logic layer
  - `types/` - Type definitions and constants

**Key Services**:
- `cloudtoken` - Cloud189 authentication token management
- `cloudbridge` - Cloud189 API integration
- `virtualfile` - Virtual file system for Cloud189 shares
- `mediafile` - Media file STRM generation
- `mountpoint` - Storage mount point management
- `user`/`usergroup` - User and group management
- `autoingestplan` - Automated file ingestion scheduling

**Framework**: Built on Gin with custom framework abstractions for HTTP and task contexts.

### Frontend Architecture (Vue.js)

**Location**: `fe/` directory

**Tech Stack**:
- Vue 3 + TypeScript
- Vite build tool
- Pinia state management
- Naive UI component library
- Vue Router for navigation

**Structure**:
- `src/api/` - API client functions
- `src/components/` - Reusable Vue components
- `src/stores/` - Pinia stores
- `src/views/` - Page components
- `src/utils/` - Utility functions

### Database

- SQLite (via GORM) as primary database
- Supports MySQL and PostgreSQL as alternatives
- Database file location: `data/share.db`

### Key Features

1. **WebDAV Interface** - Provides standard WebDAV access to Cloud189 files
2. **Virtual File System** - Maps Cloud189 share links to directory structure
3. **Media STRM Generation** - Creates STRM files for media server compatibility
4. **Multi-tenant Architecture** - Supports users and groups with permissions
5. **Background Processing** - Automated file ingestion and sync tasks
6. **Media Streaming** - Built-in media player with HLS support

### Configuration

Configuration files in `etc/` directory:
- Main config: `etc/config.yaml`
- Database file: `data/share.db`
- Media directory: `media_dir/`
- Logs: `logs/share.log`

### Testing

- Go tests use standard testing package
- Test files follow `*_test.go` pattern
- Example: `internal/services/cloudbridge/fetch_files_test.go`

### API Endpoints

- `/api/user/*` - User management
- `/api/cloudtoken/*` - Token management
- `/api/storage/*` - Storage management
- `/api/setting/*` - System settings
- `/api/media/*` - Media management
- `/dav/*` - WebDAV interface

### Development Notes

- The project uses Go 1.25+ with modern Go features
- Frontend uses modern ES6+ and TypeScript
- Docker support for easy deployment
- Multi-architecture builds available
- Swagger documentation generation for APIs

---
> Source: [xxcheng123/cloudpan189-share](https://github.com/xxcheng123/cloudpan189-share) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
