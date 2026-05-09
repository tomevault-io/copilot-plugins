## home-dashboard

> This application uses different services to collect data and display insights on a dashboard sent to an e-ink display. It renders the dashboard in HTML and CSS and then converts it to a 1-bit PNG image which is retrieved by the display every 10~ minutes. This project is meant to be open source and easy to follow, modular, and customizable.

# Agent Developer Guide

This application uses different services to collect data and display insights on a dashboard sent to an e-ink display. It renders the dashboard in HTML and CSS and then converts it to a 1-bit PNG image which is retrieved by the display every 10~ minutes. This project is meant to be open source and easy to follow, modular, and customizable.

## Architecture Overview

**Node and Express**
This application is built using Node.js and Express.js.

**Service-Based Architecture:**
All data services extend `BaseService` (`lib/BaseService.js`) which provides automatic caching, retry logic with exponential backoff, stale cache fallback, and status tracking. All services are located in `services/` directory.

**Data Flow:**
1. Enabled services are called in `lib/dataBuilder.js`.
2. Service data is either fetched from the API or retrieved from the cache using `lib/state.js`.
3. New service data is cached, each service has its own cache TTL.
4. Data is built in `lib/dataBuilder.js`.
5. Data is sent to the dashboard.

**State Management:**
Centralized in `lib/state.js` → `data/state.json`. All service caches and status are stored here. Use `getStateKey()`, `setStateKey()`, `updateState()`.

**Routes:**
All routes are defined in `routes.js`. Here are the key routes:

- `/dashboard` - Dashboard HTML used for development
- `/dashboard/image` - 1-bit PNG image sent to e-ink display
- `/admin` - Admin page for auth and monitoring
- `/api/dashboard` - Returns dashboard data in JSON format
- `/api/services/status` - Returns status of all services

**OAuth:**
Visit `/admin` to re-authenticate. Tokens are stored in `data/auth.json`.

**PM2:**
This application is run as a PM2 daemon. Use `npm start`, `npm stop`, `npm restart` to manage it.

**Views:**
Views are located in `views/` directory, built using EJS templates. Relevant styles are in `views/styles/` directory.

## Code Style

- Use best practices for JavaScript, CSS, and HTML. Follow the style of the existing code.
- Keep code DRY and avoid unnecessary code duplication or complex logic.
- Organize code into logical groupings, files, and directories as needed, but DO NOT over-engineer or over-architect.
- Use comments to make key logic and decisions clear for an open source audience.
- Update README.md as needed, do not create separate documentation.

## Key commands

```bash
# Test individual services
npm run test-service weather   # Visual Crossing Weather API
npm run test-service ambient   # Ambient Weather Station
npm run test-service calendar  # Google Calendar
npm run test-service vehicle   # Smartcar vehicle data
npm run test-service llm       # Claude AI insights

# View logs (use pm2 parameters to filter)
npx pm2 logs weather-dashboard

# Service management
npm start      # Start PM2 daemon
npm stop       # Stop daemon
npm restart    # Restart (reload .env)
npx pm2 list   # List processes
npx pm2 monit  # Monitor resources
```

## Adding a New Service

1. Understand how BaseService works, look at other service as an example, and create service class extending BaseService in `services/`, ensuring key methods are defined.
2. Add service data to `lib/dataBuilder.js` for data processing to send to dashboard.
3. Add to service list in `scripts/test-service.js` for testing

## Testing changes
- If you make changes to the server code, you may need to restart the server.
- You can also get the dashboard data or test a specific service to validate data flow and changes.
- You may need to clear the cache for a service to get fresh data.

## Debugging

**Test service:**
Check service status, see entire dashboard data, and also see what is being output from the service. Port defaults at 7272, but can be changed with PORT variable in `.env`.
```bash
curl http://localhost:<PORT>/api/services/status
curl http://localhost:<PORT>/api/dashboard
npm run test-service <service-name>
```

**Check the logs:**
Check logs and look for any errors or warnings.
```bash
pm2 logs weather-dashboard
```

**Clear service cache:**
If you need to get fresh data from a service, clear the cache for that service first.
```javascript
const { getStateKey, setStateKey } = require('./lib/state');
const caches = getStateKey('service_cache', {});
delete caches['<service-name>'];
setStateKey('service_cache', caches);
```

---
> Source: [kyleturman/home-dashboard](https://github.com/kyleturman/home-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
