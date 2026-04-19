## investment-dashboard

> This document outlines the key components and organization of the Investment Dashboard project, which is designed to collect, analyze, and visualize investment-related data.

# Investment Dashboard Project Structure

This document outlines the key components and organization of the Investment Dashboard project, which is designed to collect, analyze, and visualize investment-related data.

## Core Application Structure

The application follows a layered architecture pattern with clear separation of concerns:

### Entry Point
- [app/main.py](mdc:app/main.py): Application entry point that initializes services and orchestrates the main workflow

### Key Components
1. **Services Layer** - [app/service/](mdc:app/service)
   - Contains core business logic
   - [macro_data_service.py](mdc:app/service/macro_data_service.py): Handles macroeconomic data collection and processing
   - Services should be stateless and focused on specific business domains

2. **Configuration** - [app/config/](mdc:app/config)
   - [db_config.py](mdc:app/config/db_config.py): Database connection and settings
   - [logging_config.py](mdc:app/config/logging_config.py): Logging configuration
   - Environment-specific settings should be managed through configuration files

3. **Client Layer** - `app/client/`
   - External API integrations and third-party service clients
   - Each client should handle a specific external service or API

4. **Controller Layer** - `app/controller/`
   - HTTP request handlers and API endpoints
   - Responsible for request validation and response formatting

5. **Repository Layer** - `app/repository/`
   - Data access layer
   - Handles database operations and data persistence
   - Abstracts storage implementation details from services

6. **Scheduler** - `app/scheduler/`
   - Background task scheduling and execution
   - Periodic data collection and updates
   - Task retry and error handling logic

## Project Configuration

- [pyproject.toml](mdc:pyproject.toml): Project metadata, build settings, and tool configurations
- [requirements.txt](mdc:requirements.txt): Python package dependencies with pinned versions

## Testing and Documentation

- `tests/`: Test suite organized by component
  - Unit tests for individual components
  - Integration tests for end-to-end workflows
  - Test fixtures and utilities
- `doc/`: Project documentation
  - Architecture diagrams
  - API documentation
  - Setup and deployment guides

## Development Guidelines

1. **Architecture Principles**
   - Follow the layered architecture pattern
   - Maintain separation of concerns between layers
   - Use dependency injection for better testability
   - Handle errors appropriately at each layer

2. **Code Organization**
   - Keep related functionality together in modules
   - Use clear, descriptive names for files and directories
   - Maintain a consistent project structure

3. **Best Practices**
   - Write unit tests for new functionality
   - Document public interfaces and important logic
   - Use type hints for better code clarity
   - Follow PEP 8 style guidelines

4. **Data Flow**
   - Controllers validate incoming requests
   - Services implement business logic
   - Repositories handle data persistence
   - Clients manage external communications

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hushenglang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
