## 01-project-overview

> description: Overview of the project structure and quick start instructions for running the application in different environments.

description: Overview of the project structure and quick start instructions for running the application in different environments.

# Tarot Project Overview

This is a tarot application with a Vue.js frontend and Python FastAPI backend.

## Project Structure

- [frontend/](mdc:frontend) - Vue.js frontend application
- [backend/](mdc:backend) - Python FastAPI backend application
- [docker-compose.yml](mdc:docker-compose.yml) - Production Docker configuration
- [docker-compose.dev.yml](mdc:docker-compose.dev.yml) - Development Docker configuration
- [docker-compose.local.yml](mdc:docker-compose.local.yml) - Local development configuration

## Quick Start

1. For development: `docker-compose -f docker-compose.dev.yml up`
2. For local development: `docker-compose -f docker-compose.local.yml up`
3. For production: `docker-compose up`

---
> Source: [AHTOOOXA/gramkit](https://github.com/AHTOOOXA/gramkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
