## engineermeetupannouncementgas

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development
```bash
npm install              # Install dependencies
npm run build           # Clean, bundle and prepare for deployment
npm run test            # Run Jest tests
npm run lint            # Run ESLint and license checks
npm run format          # Auto-fix formatting issues with Prettier
```

### Deployment
```bash
npm run deploy          # Deploy to development environment
npm run deploy:prod     # Deploy to production environment
npm run open            # Open the deployed script in browser
npm run open:prod       # Open the production script in browser
```

### Testing
```bash
npm test                # Run all tests
npm run test:watch      # Run tests in watch mode
npm run test:coverage   # Run tests with coverage report
```

## High-Level Architecture

This is a Google Apps Script (GAS) project that automates announcements for the "エンジニア作業飲み集会" (Engineer Working & Drinking Meetup) across multiple platforms.

### Core Architecture Principles

1. **Separation of Concerns**: GAS-dependent code is isolated in the `src/API/gas/` directory, while business logic remains in `src/models/`
2. **Platform Integrations**: Each external platform (Discord, Twitter, VRChat, Cluster) has its own module in `src/API/`
3. **Type Safety**: The entire codebase uses TypeScript for better maintainability
4. **Deployment**: Uses Rollup to bundle TypeScript into a single ES module for GAS compatibility

### Directory Structure

```
src/
├── API/                     # Platform-specific integrations
│   ├── cluster/            # Cluster event posting
│   ├── discord/            # Discord webhook messaging
│   ├── gas/                # Google Apps Script APIs
│   │   ├── form/          # Google Forms handling
│   │   ├── spreadsheet/   # Google Sheets data management
│   │   └── triggers/      # Time-based triggers
│   ├── twitter/            # Twitter/X API integration
│   └── vrchat/             # VRChat group announcements
├── models/                  # Business logic
│   ├── events/             # Event management logic
│   └── hackathons/         # Hackathon-specific features
├── index.ts                # Main entry point with trigger functions
├── postEventAnnouncement.ts # Core announcement orchestration
└── constants.ts            # Configuration and constants
```

### Key Components

1. **Event Management**: Tracks event schedules, counts, and status in Google Sheets
2. **Multi-Platform Posting**: Coordinates announcements across Discord, Twitter, VRChat, and Cluster
3. **Scheduled Triggers**: Uses GAS time-based triggers for automated announcements
4. **Hackathon Support**: Special features for managing hackathon events including timetables and participant tracking

### Data Flow

1. Event data is stored in Google Sheets (configured via `SHEET_ID` in constants)
2. Time-based triggers execute announcement functions
3. The system checks event status and composes appropriate messages
4. Messages are posted to all configured platforms
5. Results are logged back to Google Sheets

### Environment Configuration

The project uses Google Apps Script properties for sensitive configuration:
- Discord webhook URLs
- API credentials for Twitter and VRChat
- Google Sheets IDs
- Timezone is set to 'Asia/Tokyo'

### Testing Strategy

- Unit tests for business logic using Jest
- Mock implementations for GAS-specific APIs
- Integration tests can be run using the development deployment

### Deployment Process

1. Code is bundled using Rollup into a single file
2. License headers are automatically added
3. Clasp uploads the bundle to Google Apps Script
4. Separate development and production deployments are supported

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yuni-shinogami)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yuni-shinogami)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
