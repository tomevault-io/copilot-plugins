## claude-code-test

> This file provides guidance to Claude Code (claude.ai/code) when working with this repository.

# e6 Capital Matrix Terminal

This file provides guidance to Claude Code (claude.ai/code) when working with this repository.

## Project Overview

A Matrix-themed AI assistant interface for e6 Capital, featuring:
- Terminal-style UI with Matrix aesthetic
- n8n webhook integration for AI responses
- Real-time status indicators and metrics
- Responsive design optimized for financial consultations

## Architecture

- **Frontend**: Pure HTML/CSS/JavaScript (no frameworks)
- **Styling**: Matrix-themed terminal interface with green glow effects
- **Backend**: n8n webhook integration for AI processing
- **Deployment**: Vercel with environment variable configuration

## Build and Development Commands

```bash
# Local development
npm run dev        # Start local server on port 3000

# Production build
npm run build      # Build for deployment (creates dist/ folder)
npm start          # Alias for npm run build
```

## Environment Configuration

### Required Environment Variables
- `N8N_WEBHOOK_URL`: Your n8n webhook endpoint URL
- `GA_TRACKING_ID`: (Optional) Google Analytics tracking ID
- `API_BASE_URL`: (Optional) Additional API endpoints

### Configuration Files
- `.env.example`: Template for environment variables
- `env-config.js`: Environment variable injection template
- `build.js`: Build script that processes environment variables

## Deployment

### Vercel Deployment
1. Environment variables are configured in Vercel dashboard
2. Build process injects variables into `env-config.js`
3. Static files are served from `dist/` directory

### GitHub Repository
- Repository: https://github.com/e6inc/claude-code-test
- Excludes development files via `.gitignore`
- Includes all production assets

## File Structure

```
├── index.html          # Main application entry point
├── style.css           # Matrix terminal styling
├── script.js           # Application logic and n8n integration
├── e6-logo.svg         # Company logo
├── env-config.js       # Environment variable template
├── build.js            # Build script
├── package.json        # Project configuration
├── vercel.json         # Vercel deployment settings
├── .env.example        # Environment variable template
├── .gitignore          # Git ignore rules
└── CLAUDE.md           # This file
```

## Development Guidelines

### Security
- No hardcoded API keys or sensitive URLs in source code
- Environment variables for all external endpoints
- Client-side architecture (webhook URL will be public)

### Code Style
- Pure JavaScript (ES6+) without frameworks
- Matrix theme with green terminal aesthetic
- Responsive design for desktop and mobile

### Testing
- Use `test_webhook.py` for webhook endpoint testing
- Use `webhook_server.py` for local development server
- Both files excluded from production deployment

---
> Source: [jeffsavoie/claude-code-test](https://github.com/jeffsavoie/claude-code-test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
