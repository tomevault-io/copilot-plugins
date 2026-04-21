## photos

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository contains a Docker Compose setup for Immich, a self-hosted photo and video management solution. The deployment is configured to run behind a Traefik reverse proxy with SSL termination and serves photos at `photos.waugze.com`.

## Architecture

### Core Components
- **Immich Server**: Main application container handling API requests and web interface
- **Immich Machine Learning**: Handles photo/video analysis, face detection, and AI features
- **PostgreSQL Database**: Stores metadata and application data using Immich's custom postgres image with vector extensions
- **Redis/Valkey**: Caching layer for improved performance
- **Traefik Integration**: Reverse proxy with SSL termination and load balancing

### Network Configuration
- **proxy**: External network for Traefik integration
- **immich**: Internal network for service communication

### Storage Configuration
- **Photo/Video Storage**: Configurable upload location (default: `./library`)
- **Database Storage**: Configurable database location (default: `./postgres`)
- **Model Cache**: Docker volume for ML model caching

## Environment Configuration

The application uses environment variables defined in `.env` (create from `.env.example`):

### Required Variables
- `UPLOAD_LOCATION`: Path for photo/video storage
- `DB_DATA_LOCATION`: Path for PostgreSQL data
- `DB_PASSWORD`: Database password
- `DB_USERNAME`: Database username (default: postgres)
- `DB_DATABASE_NAME`: Database name (default: immich)
- `TZ`: Timezone setting
- `IMMICH_VERSION`: Immich version to deploy (default: release)

### Performance Tuning
- `MACHINE_LEARNING_WORKERS`: Number of ML workers (default: 1)
- `MACHINE_LEARNING_WORKER_TIMEOUT`: ML worker timeout in seconds (default: 120)

## Common Commands

### Deployment
```bash
# Start all services
docker-compose up -d

# Stop all services
docker-compose down

# View logs
docker-compose logs -f

# View logs for specific service
docker-compose logs -f immich-server
```

### Environment Setup
```bash
# Create environment file from example
cp .env.example .env

# Edit environment variables
nvim .env
```

### Maintenance
```bash
# Update to latest version
docker-compose pull
docker-compose up -d

# Restart specific service
docker-compose restart immich-server

# Check service status
docker-compose ps
```

### Troubleshooting
```bash
# Check container logs
docker-compose logs immich-server
docker-compose logs immich-machine-learning

# Access container shell
docker-compose exec immich-server sh

# Check database connection
docker-compose exec database psql -U postgres -d immich
```

## Important Notes

### Traefik Configuration
- Configured for `photos.waugze.com` domain
- Uses Let's Encrypt for SSL certificates
- Includes timeout middleware for large file uploads
- Requires external `traefik` network to be created

### Storage Considerations
- Database should be stored on SSD for optimal performance
- Upload location can be on slower storage (HDD acceptable)
- Ensure proper permissions for mounted volumes

### Security
- Database password should be changed from default
- Consider network isolation for production deployments
- SSL/TLS handled by Traefik reverse proxy

### Performance
- ML workers can be increased for faster processing on multi-core systems
- Model cache volume persists between container restarts
- Database includes vector extensions for AI/ML features

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Bparsons0904) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
