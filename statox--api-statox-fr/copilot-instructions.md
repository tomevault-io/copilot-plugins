## api-statox-fr

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal TypeScript/Express API backend serving multiple features (HomeTracker, Chords, Clipboard, Cookbook, WebWatcher, etc.) with type-safe routes, JSON schema validation, and an auto-generated SDK for the frontend.

## Development Commands

### Local Development

These commands are not for you to run, the user should have started them already:

```bash
npm run env              # Start docker environment (MySQL, LocalStack S3)
./src/tools/init-db.sh   # Initialize database tables
npm run watch            # TypeScript watcher
npm run serve            # Start server on port 3000 (watches dist/)
```

### Testing

```bash
npm run tests                    # Run route & periodic task tests
npm run tests:framework          # Run framework tests
npm run tests:packages           # Run package tests
npm run tests:all                # Run all test suites
```

To run a single test, use the npm script which invokes mocha. Use the `-f` option to filter on the name of the test.

```bash
npm run tests -- -f 'personalTracker/getAll'
```

Adding `debug=true` before the test command trigger more verbose logs useful for debugging:

```bash
debug=true npm run tests -- -f 'personalTracker'
```

### Code Quality

```bash
npm run check                    # Run lint + prettier
npm run lint                     # ESLint
npm run lint:fix                 # Fix ESLint issues
npm run prettier                 # Check formatting
npm run prettier:fix             # Fix formatting
```

### SDK Generation

```bash
npm run generate:sdk             # Generate TypeScript SDK from routes
```

The SDK is generated from route definitions and synced to the frontend codebase.

### Deployment

This must be run exclusively by the user directly on their terminal.

```bash
npm run heroku:deploy            # Deploy to production (runs checks + tests)
npm run heroku:deploy:skip-tests # Deploy without running tests
npm run heroku:ssh               # SSH into production
```

### User Management

```bash
npm run user:create              # Create local user interactively
```

## Architecture

### Route System

Routes are the core of the API. Each route is defined as a strongly-typed object in `src/libs/routes/[module]/[endpoint].ts`:

```typescript
export const route: GetRoute<InputType, OutputType> = {
    method: 'get' | 'post',
    path: '/module/endpoint',
    handler: async ({ input, loggableContext, authenticatedUser }) => { ... },
    authentication: 'none' | 'apikey-iot' | 'user2',
    scope: 'public' | 'admin' | 'homeTracker' | 'personalTracker', // Required for user2 auth
    inputSchema: { ... },      // JSON schema (POST only)
    outputSchema: { ... }      // JSON schema
};
```

Routes are registered in `src/libs/routes/index.ts` by importing and adding them to the `routes.list` array. The route system automatically:

- Validates input against JSON schema (POST routes)
- Applies authentication middleware based on `authentication` field
- Validates user scopes (for `user2` authentication)
- Handles errors via `errorHandler` middleware
- Generates OpenAPI definitions
- Powers SDK generation

### Authentication

Three authentication modes:

1. **none**: Public endpoints
2. **apikey-iot**: IoT devices using API key headers (validated by `validateAPIKeyHeader`)
3. **user2**: Session-based auth using PassportJS with local strategy
    - Login: `/auth/login` validates credentials and creates session
    - Session stored in MySQL `sessions` table
    - Cookie contains session ID + signature (validated with session secret)
    - Subsequent requests validated via `validatePassportSession` middleware
    - Scopes enforced via `validateEndpointScope` middleware

See `DEV.md` for detailed auth flow.

### Middleware Pipeline

Each route goes through middleware in `src/app.ts` (lines 72-108):

1. `loggingHandler` - Request logging
2. `multipartHandler` - File upload handling
3. Authentication middleware (based on route.authentication)
4. Input validation (POST routes only, via express-json-validator-middleware)
5. `apiPipeline` - Main route handler wrapper
6. `errorHandler` - Global error handler

### Directory Structure

- **src/libs/routes/**: Route definitions organized by module (auth, chords, clipboard, cookbook, gravitrips, health, homeTracker, personalTracker, reactor, webWatcher)
- **src/libs/middleware/**: Express middleware (auth, errors, logging, multipart, apiPipeline)
- **src/libs/modules/**: Business logic modules (ajv, auth, chords, clipboard, cookbook, ephemerides, gravitrips, homeTracker, logging, meteofrance, notifier, openapi, personalTracker, reactor, s3files, webWatcher)
- **src/libs/databases/**: Database connections (MySQL, Elasticsearch, S3)
- **src/libs/PeriodicTasks/**: Background tasks started in production
- **src/packages/**: Reusable packages (config, suncalc)
- **src/tools/**: CLI utilities (DB init, user creation, ELK tools, OpenAPI generation)
- **tests/**: Test files mirroring src structure (routes, framework, helpers, periodicTasks)

### Configuration

Configuration is managed by the `src/packages/config/` package:

- **sources/**: Environment variable readers (one file per service)
- **services/parseConfig.ts**: Gathers config from sources, validates with JSON schema
- Config is type-safe and validated at startup (app fails if invalid)

### Databases

- **MySQL**: User sessions, application data (dev: docker, test: docker on different port, prod: self-hosted VPS)
- **Elasticsearch**: Logging via `slog` module + HomeTracker sensor data (dev: stdout only, test: mocked, prod: self-hosted VPS)
- **S3/R2**: File storage via AWS SDK (dev: LocalStack, test: mocked, prod: Cloudflare R2)

Tables are defined in `src/tools/tables/[table-name].sql` and created via `./src/tools/init-db.sh`.

### Logging

Custom logger `slog` in `src/libs/modules/logging/`:

- Dev/test: stdout
- Prod: Elasticsearch cluster
- Usage: `slog.log(component, message, data?)`
- Logs include `loggableContext` injected by middleware

**IMPORTANT: Correct slog usage**

The `slog.log()` function has strict typing to prevent Elasticsearch mapping conflicts:

```typescript
slog.log(component: AppLogComponent, message: string, data?: LogObject)
```

1. **Component** (first argument): Must be one of the predefined strings in `AppLogComponent` type (`src/libs/modules/logging/types.ts`):
    - Check `src/libs/modules/logging/types.ts` to find the possible options.
    - Do NOT use arbitrary strings like 'personal-tracker' or 'my-module'
    - Use 'periodic-tasks' for all periodic task logging
    - Use the most specific component that matches your context
    - If you are creating logs for a new component suggest editing the file to add the new component to the list.

2. **Data** (third argument): Must only contain properties defined in `LoggableProperties` type:
    - Check `src/libs/modules/logging/types.ts` to find the possible options.
    - Do NOT use arbitrary property names like `dateUnix`, `myCustomField`, etc.
    - Common available properties: `userId`, `error`, `timestamp`, `taskName`, `status`, `url`, `sensorName`, `eventTS`, etc.
    - If a property you need doesn't exist, add it to `LoggableProperties` type first

**Examples:**

```typescript
// ✅ CORRECT
slog.log('periodic-tasks', 'Reminder sent', { userId: 2 });
slog.log('home-tracker', 'Sensor offline', { sensorName: 'kitchen', lastSyncDateUnix: 1234567890 });

// ❌ WRONG
slog.log('a-random-component', 'Reminder sent', { userId: 2, dateUnix: 1234567890 });
//       ^^^^^^^^^^^^^^^^^^^ Not in AppLogComponent type     ^^^^^^^^ Not in LoggableProperties
```

### Notifications

The `notifier` module (`src/libs/modules/notifier/`) provides two notification channels:

**1. Slack Notifications (`slackNotifier`)**

- Uses Slack webhooks via `@slack/webhook` package
- Config: `config.slack.webhookUrl` and `config.slack.userId`
- Usage: `slackNotifier.notifySlack({ message?, error?, directMention? })`
- Behavior:
    - Dev: logs to console
    - Test/Prod: sends to Slack webhook
    - Can include error stack traces
    - Supports `directMention: true` to @mention user (requires `config.slack.userId`)
- Used for:
    - Critical errors in error middleware
    - Authentication issues
    - App shutdown notifications
    - Periodic task failures
    - WebWatcher monitoring alerts
    - HomeTracker data ingestion errors

**2. Push Notifications (`pushNotifier`)**

- Uses ntfy.sh service for mobile push notifications
- Config: `config.ntfy_sh.topicUrl`
- Usage: `pushNotifier.notify({ message, title? })`
- Default title: `'api.statox.fr'`
- Used for:
    - WebWatcher content change alerts
    - HomeTracker sensor status (offline/online/no data)
- Errors are logged via `slog` but don't throw

Both notifiers wrap errors internally - failed notifications don't interrupt normal execution flow.

### WebSocket Support

WebSocket server runs on same port as HTTP server (lines 116-117 in src/app.ts). Routes for WebSocket connections are defined in `src/libs/routes/index.ts` under `routesWS.list` and implemented in route files like `src/libs/routes/gravitrips/ws_game.ts`.

### Periodic Tasks

Background tasks defined in `src/libs/PeriodicTasks/` run only in production (started in `src/app.ts:121`).

## Testing

Tests use Mocha + Chai + Supertest. Three test suites with separate Mocha configs in `.mocha/`:

1. **packages** (`npm run tests:packages`): Package-level unit tests
2. **framework** (`npm run tests:framework`): Framework/infrastructure tests
3. **routes** (`npm run tests`): Route integration tests

Route tests require database initialization (`src/tools/init-db.sh --tests`) and run against test DB.

### Test Helpers Framework

The `tests/helpers/` directory contains a custom helper framework that simplifies test writing by providing:

- Automatic setup/teardown via Mocha hooks
- Utilities for seeding test data
- Assertions for verifying database state, logs, and external service calls
- Mocking/stubbing for external dependencies

All helpers are exported through a centralized `th` object in `tests/helpers/index.ts`.

**Available helpers:**

1. **MySQL** (`th.mysql`):
    - `fixture(data)` - Seeds database tables with test data
    - `checkContains(data)` - Verifies tables contain expected rows (supports partial matching, custom matchers, and `aroundNowSec` for timestamps)
    - `checkDoesNotContain(data)` - Verifies tables don't contain specified rows
    - `checkTableLength(table, length)` - Verifies exact row count in a table
    - `dumpTables(tables)` - Prints table contents for debugging
    - `aroundNowSec` - Special matcher for timestamp assertions (within 1 second of now)
    - `nowSec()` - Returns current Unix timestamp in seconds
    - **Hook**: `beforeEach` clears all tables automatically

2. **Auth2** (`th.auth2`):
    - `getPassportSessionCookie(username?)` - Returns session cookie for authenticated requests (default user: 'user' with admin scope)
    - `setupAuth2User({ username, password, scopes })` - Creates a user with custom scopes
    - **Hook**: `beforeEach` creates default 'user' with admin scope

3. **S3** (`th.s3`):
    - `checkNbCalls({ nbCalls })` - Verifies number of S3 operations
    - `checkCall({ commandType, input })` - Verifies S3 command was called with specific parameters
    - **Hook**: `beforeEach` resets S3 mock (note: keys containing 'should_fail' will simulate failures)

4. **ELK** (`th.elk`):
    - `fixture(data)` - Seeds Elasticsearch indices with test documents
    - `checkDocumentCreated(index, document)` - Verifies document was indexed
    - `flush()` - Manually clears indices (only use when necessary - expensive operation)
    - `dumpIndex(index, size?)` - Prints index contents for debugging
    - **Hooks**: Sets up spy on `elk.index`, mocks search to auto-refresh indices

5. **Slog** (`th.slog`):
    - `checkLog(component, message, data?)` - Verifies log entry was created (supports sinon matchers in data)
    - `checkNoLogs()` - Verifies no logs were created
    - **Hooks**: `beforeEach` sets up spy, `afterEach` restores

6. **Time** (`th.time`):
    - `fakeSinonDateTimeNow(timestampSec)` - Mocks Luxon's `DateTime.now()` to return a specific time
    - `restoreDateTimeNow()` - Restores real time (call manually when done)
    - `isAroundNowSec(timestamp, maxDelaySec?)` - Asserts timestamp is within maxDelay seconds of now (default: 2)

7. **Slack Notifier** (`th.slack`):
    - `checkNotificationSent(params?)` - Verifies Slack notification was sent
    - `checkNoNotifications()` - Verifies no notifications were sent
    - **Hooks**: Automatically stubs `slackNotifier.notifySlack`

8. **Push Notifier** (`th.push`):
    - `checkNotificationSent(params?)` - Verifies push notification was sent
    - `checkNoNotifications()` - Verifies no notifications were sent
    - **Hooks**: Automatically stubs `pushNotifier.notify`

**Using helpers in tests:**

```typescript
import request from 'supertest';
import { app } from '../../../src/app.js';
import { th } from '../../helpers/index.js';

describe('myModule/myRoute', () => {
    it('should create an entry', async () => {
        // Seed test data
        await th.mysql.fixture({
            MyTable: [{ id: 1, name: 'test', createdAt: th.mysql.nowSec() }]
        });

        // Make authenticated request
        await request(app)
            .post('/myModule/myRoute')
            .set('Cookie', th.auth2.getPassportSessionCookie())
            .send({ data: 'value' })
            .expect(200);

        // Verify database state
        await th.mysql.checkContains({
            MyTable: [
                {
                    name: 'test',
                    createdAt: th.mysql.aroundNowSec,
                    // Custom matcher function
                    status: (value) => value === 'active'
                }
            ]
        });

        // Verify logs
        th.slog.checkLog('myModule', 'entry created', { name: 'test' });

        // Verify S3 calls
        th.s3.checkNbCalls({ nbCalls: 1 });
    });
});
```

**Creating a new helper:**

1. Create a directory in `tests/helpers/` (e.g., `tests/helpers/myService/`)
2. Create `index.ts` with a class extending `TestHelper`:

```typescript
import { TestHelper } from '../TestHelper.js';
import sinon from 'sinon';
import { myService } from '../../../src/libs/modules/myService/index.js';

let myServiceStub: sinon.SinonStub;

const setupStub = async () => {
    myServiceStub = sinon.stub(myService, 'doSomething').resolves('mocked');
};

const restoreStub = async () => {
    myServiceStub.restore();
};

class TestHelper_MyService extends TestHelper {
    constructor() {
        super({
            name: 'MyService',
            hooks: {
                beforeEach: setupStub,
                afterEach: restoreStub
            }
        });
    }

    checkCalled = () => {
        sinon.assert.called(myServiceStub);
    };
}

export const testHelper_MyService = new TestHelper_MyService();
```

3. Export from `tests/helpers/index.ts`:

```typescript
import { testHelper_MyService } from './myService/index.js';
export const th = {
    // ... existing helpers
    myService: testHelper_MyService
};
```

4. Add to mocha wrapper in `tests/helpers/mocha/routesMochaWrapper.ts` (order matters - dependencies must come first):

```typescript
const helpers: TestHelper[] = [
    testHelper_Mysql,
    // ... other helpers
    testHelper_MyService // Add here
];
```

**Important notes:**

- Helpers run in the order defined in `routesMochaWrapper.ts` - MySQL runs first to clear tables
- `th.mysql.fixture()` automatically clears tables before each test via `beforeEach` hook
- Use `th.elk.flush()` sparingly - it's expensive (deletes and recreates indices)
- For timestamp assertions, prefer `th.mysql.aroundNowSec` over exact values
- Custom matcher functions in `checkContains()` receive the column value and should return boolean
- Test helpers are automatically initialized by Mocha root hooks - no manual setup needed in test files

## Important Patterns

### Adding a New Route

1. Create route file in `src/libs/routes/[module]/[endpoint].ts`
2. Define input/output JSON schemas
3. Implement typed handler
4. Import and add to `routes.list` in `src/libs/routes/index.ts`
5. Run `npm run generate:sdk` to update frontend SDK
6. Create test file in `tests/routes/[module]/[endpoint].test.ts`

### Adding Database Tables

1. Create SQL file in `src/tools/tables/[table-name].sql`
2. Run `./src/tools/init-db.sh` locally
3. Run `./src/tools/init-db.sh --prod` for production (requires heroku login)

### OpenAPI Generation

OpenAPI definitions are auto-generated from route schemas in `src/tools/openapi/generate_openapi_definition.ts` and served at `/openapi/definition`.

## Environment Notes

- **Timezone**: App requires UTC timezone (validated at startup in src/app.ts:43-48)
- **Node version**: 24.x (specified in package.json engines)
- **TypeScript**: Code is compiled to `dist/` via `tsc`. Both `watch` (for dev) and `postinstall` (for prod) compile the code.
- **ES Modules**: Project uses `"type": "module"` - all imports must use `.js` extensions even for `.ts` files

## CI/CD

- Dependabot auto-merge workflow at `.github/workflow/dependabot-auto-merge.yml`
- Tests run in CI via `npm run tests:ci` (outputs JSON reports for GitHub Actions)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/statox) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
