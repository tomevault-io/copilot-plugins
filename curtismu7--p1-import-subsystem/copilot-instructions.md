## p1-import-subsystem

> The PingOne Import Tool is a comprehensive web-based application for managing PingOne user data. It provides functionality for importing, exporting, and modifying user data with real-time progress tracking and detailed logging.


# PingOne Import Tool

## Product Overview
The PingOne Import Tool is a comprehensive web-based application for managing PingOne user data. It provides functionality for importing, exporting, and modifying user data with real-time progress tracking and detailed logging.

## Core Functionality
- **User Import**: Bulk import users from CSV files with validation and error handling
- **User Export**: Export users from specific populations with customizable options
- **User Modification**: Update existing user attributes and data
- **Population Management**: Create, delete, and manage user populations
- **Real-time Progress**: Live progress tracking with detailed status updates
- **Comprehensive Logging**: Detailed logs for debugging and audit trails

## Advanced Features
- **Token Management**: Automatic token refresh and validation with status indicators
- **Error Handling**: Robust error handling with detailed error messages
- **File Validation**: CSV format validation and data integrity checks
- **Responsive UI**: Modern, mobile-friendly interface with Ping Identity styling
- **Settings Management**: Centralized configuration management
- **API Testing**: Built-in API connection testing tools
- **Swagger UI Integration**: Interactive API documentation
- **Authentication Subsystem**: Standalone, isolated authentication mechanism

# Project Structure

## Directory Organization

```
pingone-import/
├── auth-subsystem/         # Authentication subsystem (isolated)
│   ├── client/            # Client-side auth components
│   └── server/            # Server-side auth components
├── public/                # Static assets and client-side code
│   ├── css/              # Stylesheets
│   ├── js/               # Client-side JavaScript
│   ├── vendor/           # Third-party libraries
│   └── swagger/          # Swagger UI files
├── routes/                # Express route handlers
│   └── api/              # API route handlers
├── server/                # Server-side utilities
├── test/                  # Test files
│   ├── api/              # API tests
│   ├── integration/      # Integration tests
│   ├── frontend/         # Frontend tests
│   ├── unit/             # Unit tests
│   ├── ui/               # UI tests
│   └── mocks/            # Test mocks and fixtures
├── data/                  # Configuration data and exports
├── docs/                  # Documentation
│   ├── api/              # API documentation
│   ├── features/         # Feature documentation
│   └── deployment/       # Deployment guides
├── logs/                  # Application logs
└── scripts/               # Utility scripts
```

## Key Files

- `server.js`: Main application entry point
- `package.json`: Project dependencies and scripts
- `swagger.js`: Swagger/OpenAPI configuration
- `.env`: Environment variables (not in repo)
- `data/settings.json`: Application settings

## Code Organization Patterns

### Server-Side

1. **Modular Routes**: Routes are organized by feature area in the `routes/` directory
2. **Middleware Pattern**: Express middleware for authentication, logging, etc.
3. **Service Layer**: Business logic separated from route handlers
4. **Utility Modules**: Shared utilities in the `server/` directory

### Client-Side

1. **Module Pattern**: Client-side code uses ES modules
2. **Event-Driven Architecture**: Event listeners for UI interactions
3. **API Client Pattern**: Structured API clients for server communication
4. **Component-Based**: UI components are organized by feature

## Authentication Architecture

The authentication subsystem is isolated in the `auth-subsystem/` directory:

- `auth-subsystem/server/`: Server-side authentication API
- `auth-subsystem/client/`: Client-side authentication utilities
- `auth-subsystem/server/pingone-auth.js`: PingOne authentication service
- `auth-subsystem/server/credential-encryptor.js`: Secure credential storage

## Testing Organization

Tests are organized by type in the `test/` directory:

- `test/unit/`: Unit tests for individual functions
- `test/integration/`: Integration tests for API endpoints
- `test/api/`: Tests for external API interactions
- `test/frontend/`: Tests for client-side code
- `test/ui/`: Tests for UI components

## Logging Structure

Logs are organized by type in the `logs/` directory:

- `logs/access.log`: HTTP access logs
- `logs/error.log`: Error logs
- `logs/application.log`: Application logs
- `logs/combined.log`: Combined logs
- `logs/performance.log`: Performance metrics

## Documentation Structure

Documentation is organized by type in the `docs/` directory:

- `docs/api/`: API documentation
- `docs/features/`: Feature documentation
- `docs/deployment/`: Deployment guides

# Technical Stack & Build System

## Core Technologies
- **Runtime**: Node.js (v14+)
- **Server**: Express.js
- **Frontend**: Vanilla JavaScript with Browserify bundling
- **Authentication**: Custom PingOne authentication subsystem
- **Real-time Updates**: Socket.IO with WebSocket fallback
- **API Documentation**: Swagger/OpenAPI
- **Logging**: Winston with rotating file logs
- **Testing**: Jest with various testing utilities
- **Security**: Helmet, CORS, rate limiting, encryption

## Key Dependencies
- **Express**: Web server framework
- **Socket.IO**: Real-time bidirectional event-based communication
- **Winston**: Logging library
- **Axios**: HTTP client
- **CSV-Parse**: CSV parsing
- **Bcryptjs**: Password hashing
- **Joi**: Data validation
- **JWT**: JSON Web Token implementation
- **Swagger**: API documentation

## Build System
- **Bundling**: Browserify with Babel for transpilation
- **Transpilation**: Babel for ES6+ support
- **Testing**: Jest for unit, integration, and API tests
- **Development**: Nodemon for auto-reloading

## Common Commands

### Installation
```bash
# Install dependencies
npm install

# Set up environment variables
cp env.example .env
```

### Development
```bash
# Start development server
npm start

# Start with auto-reloading
npm run dev

# Build client-side bundle
npm run build:bundle
```

### Testing
```bash
# Run all tests
npm test

# Run specific test categories
npm run test:unit
npm run test:integration
npm run test:api
npm run test:frontend

# Run tests with coverage
npm run test:coverage
```

### Server Management
```bash
# Stop server
npm run stop

# Restart server
npm run restart

# Restart server safely
npm run restart:safe

# Check port status
npm run check:port
```

### Updates & Maintenance
```bash
# Check for package updates
npm run update:check

# Auto-update packages
npm run update:auto

# Check for update conflicts
npm run update:conflicts

# Safe update (check conflicts then update)
npm run update:safe
```

## Environment Configuration
The application uses environment variables for configuration, which can be set in a `.env` file:

- `PINGONE_CLIENT_ID`: PingOne API client ID
- `PINGONE_CLIENT_SECRET`: PingOne API client secret
- `PINGONE_ENVIRONMENT_ID`: PingOne environment ID
- `PINGONE_REGION`: PingOne region code
- `PORT`: Server port (default: 4000)
- `NODE_ENV`: Environment (development, test, production)
- `AUTH_SUBSYSTEM_ENCRYPTION_KEY`: Encryption key for auth subsystem

## Deployment
The application is configured for deployment on Render with the `render.yaml` configuration file.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/curtismu7) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
