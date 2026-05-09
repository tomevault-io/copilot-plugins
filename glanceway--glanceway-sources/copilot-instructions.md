## glanceway-sources

> Community repository for creating and distributing information sources for [Glanceway](https://glanceway.app), a macOS menu bar app that displays information items. Sources periodically fetch data from APIs/feeds and emit items for display.

# CLAUDE.md

## Project Overview

Community repository for creating and distributing information sources for [Glanceway](https://glanceway.app), a macOS menu bar app that displays information items. Sources periodically fetch data from APIs/feeds and emit items for display.

## Commands

```bash
npm install                                          # Install dependencies
npm run build-sources                                # Build all sources into dist/
npm run build-sources -- --source author/name        # Build a single source (also verifies compilation)
npm run generate-sources-json                        # Regenerate sources.json metadata
npm run generate-readme                              # Regenerate README.md
```

There is no test framework. Build the source to verify it compiles. There is no linter or formatter configured.

## Creating a Source

### Step 1: Create Source Directory

Create a `latest` directory with two files: `manifest.yaml` and `index.js`.

```
sources/<author>/<source-name>/latest/
├── manifest.yaml
└── index.js
```

To submit to this repository, place it under `sources/<author>/<source-name>/latest/` (e.g., `sources/myname/my-source/latest/`). When releasing a new version, update the files in `latest/` and bump the version in `manifest.yaml`. To preserve an older version, copy it to a version-numbered directory (e.g., `1.0.0/`) before updating `latest/`.

### Step 2: Implement index.js

Edit `index.js` to implement source logic. See the API Reference below and use existing sources as examples.

### Step 3: Update manifest.yaml Config

Add `config` entries for any values the user needs to provide (API tokens, usernames, etc.):

```yaml
config:
  - key: API_TOKEN
    name: API Token
    type: secret # string, number, boolean, secret, select, list, or multiselect
    required: true
    description: Your API token
  - key: USERNAME
    name: Username
    type: string
    required: false
    description: Optional username filter
  - key: SORT
    name: Sort Order
    type: select
    required: false
    default: hot
    description: Sort order for results
    options:
      - hot
      - new
      - top
  - key: CURRENCY
    name: Currency
    type: select
    required: false
    default: usd
    description: Display currency
    options:                              # options support label/value objects
      - label: US Dollar
        value: usd
      - label: Euro
        value: eur
  - key: CATEGORIES
    name: Categories
    type: multiselect                     # like select but allows multiple; stored as JSON array
    required: false
    description: Categories to show
    options:
      - label: Technology
        value: tech
      - label: Science
        value: science
      - label: Finance
        value: finance
```

### Step 4: Build and Verify

```bash
npm run build-sources -- --source myname/my-source
```

## Source Development Constraints

**NO external imports.** Sources cannot use `import` or `require` for external packages. All functionality is provided through the `api` parameter. Use `module.exports` to export the source function:

```javascript
module.exports = async (api) => {
  async function fetchData() {
    /* fetch, transform, emit */
  }

  // Start phase: initial fetch
  await fetchData();

  return {
    refresh: fetchData,
    stop() {
      /* optional cleanup */
    },
  };
};
```

## API Reference

All methods are available on the `api` parameter.

### api.emit(items)

Send items to Glanceway for display. Each item has these fields:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Unique identifier |
| `title` | string | Yes | Main display text |
| `subtitle` | string | No | Secondary text below title |
| `url` | string | No | Link opened on click |
| `timestamp` | Date / string / number | No | ISO string, Unix timestamp, or Date |
| `notify` | boolean | No | If `true`, content changes re-mark read items as unread so users notice updates (default: `false`) |
| `tickerText` | string | No | Text shown in the menu bar scrolling ticker; falls back to `title` if omitted |

Use `notify: true` for notification-style items where one record represents an ongoing thread (e.g. a PR or issue) — without it, once a user reads the item, subsequent content updates remain marked as read. Use `tickerText` when `title` is too long or not suited for the menu bar ticker; keep it short so it reads cleanly when scrolling.

### api.fetch(url, options?)

Make HTTP requests.

Options: `method` (default `"GET"`), `headers`, `body`, `timeout` (default 30000ms).

Response: `ok` (boolean), `status`, `headers`, `text`, `json` (parsed if valid).

Example:

```javascript
const response = await api.fetch("https://api.example.com/data", {
  headers: { Authorization: `Bearer ${token}` },
});
if (response.ok && response.json) {
  // use response.json
}
```

### api.config.get(key)

Get a user-configured value by key (defined in `manifest.yaml` config section). Returns the value with a type matching the config field type: `string` for `string`/`secret`/`select`, `number` for `number`, `boolean` for `boolean`, `string[]` for `list`/`multiselect`.

### api.config.getAll()

Get all user-configured values as a key-value map.

### api.storage.get(key)

Get a persisted value. Data survives between refreshes and app restarts.

### api.storage.set(key, value)

Store a value persistently.

### api.log(level, message)

Log messages for debugging. Levels: `"info"`, `"error"`, `"warn"`, `"debug"`.

### api.appVersion

Current Glanceway app version string (e.g., `"1.2.0"`).

### api.websocket.connect(url, callbacks)

Create a WebSocket connection for real-time data.

Callbacks: `onConnect(connection)`, `onMessage(data)`, `onError(error)`, `onClose(code)`.

The connection object has `send(message)` and `close()` methods.

## manifest.yaml Full Schema

The `id` field is required and must be declared in `manifest.yaml` as `<author>/<source-name>` (matching the directory layout). It uniquely identifies a package across versions — the Glanceway app uses it to detect re-imports of the same source and offer in-place upgrade (preserving items, read state, favorites, custom name, refresh interval, color tag, and config). The build script validates `id` and fails if it is missing or does not match `<author>/<source-name>`. **Never change a source's `id` after release — changing it breaks upgrade detection and re-imports become fresh installs.**

```yaml
id: authorname/source-name # Required: stable package identifier, must match <author>/<source-name>
version: 1.0.0 # Required: semantic version
name: Display Name # Required: shown in Glanceway
description: Brief description # Required
author: authorname # Required
author_url: https://... # Optional
category: Developer # Required: Developer | News | Social | Finance | Entertainment | Productivity | Other
tags: # Optional
  - tag1
min_app_version: 1.2.0 # Optional: minimum Glanceway app version required
config: # Optional: user-configurable values
  - key: API_TOKEN
    name: API Token
    type: secret # string, number, boolean, secret, select, list, or multiselect
    required: true
    description: Description shown to user
  - key: TAGS
    name: Tags
    type: list # list for string arrays (multiple values)
    required: false
    description: Tags to filter by
  - key: SORT
    name: Sort Order
    type: select # select requires options list
    required: false
    default: hot
    options:              # plain strings (label = value)
      - hot
      - new
      - top
  - key: CURRENCY
    name: Currency
    type: select
    required: false
    default: usd
    options:              # label/value objects (label shown in UI, value stored)
      - label: US Dollar
        value: usd
      - label: Euro
        value: eur
  - key: CATEGORIES
    name: Categories
    type: multiselect     # like select but allows picking multiple options; stored as JSON array
    required: false
    options:
      - label: Technology
        value: tech
      - label: Science
        value: science
```

## Source Lifecycle

JavaScript sources have two distinct phases:

1. **Start phase**: When the source is first loaded, the exported function (outer closure) runs. The app does **NOT** call `refresh()` at this point. Sources should perform their initial data fetch here by `await`ing their fetch function before returning.
2. **Refresh phase**: On each scheduled refresh interval, the app calls `refresh()`. This is the only time `refresh()` is invoked.

This separation allows sources to distinguish between initial load and periodic refresh, enabling different behavior if needed (e.g., full load on start vs. incremental update on refresh). For most sources, both phases do the same work.

### Standard Pattern

Extract the fetch logic into a named async function, call it in the outer closure for the start phase, and assign it as the `refresh` method:

```javascript
module.exports = async (api) => {
  const token = api.config.get("API_TOKEN");

  async function fetchData() {
    const res = await api.fetch(url);
    if (!res.ok || !res.json) {
      throw new Error(`Failed to fetch (HTTP ${res.status})`);
    }
    api.emit(toItems(res.json));
  }

  // Start phase: initial fetch
  await fetchData();

  return {
    refresh: fetchData,
  };
};
```

## Source Design Guidelines

- Always make full use of the `subtitle` field. If the API response contains summary, description, brief, or any descriptive text, map it to `subtitle` so users get maximum information at a glance.
- **Maximize items per fetch.** The app does not paginate, so each fetch should retrieve as many items as the API allows without hurting performance. The hard upper limit is **500 items** — never exceed this. For APIs with a configurable page size, set it to the API's maximum or 500, whichever is smaller. For sources that make N parallel sub-requests (e.g., Hacker News, xkcd), keep N reasonable to avoid excessive latency.

## Source Code Conventions

### File Structure Order

```javascript
// 1. Helper functions (pure utilities, no api dependency)
function stripHtml(html) {
  return html.replace(/<[^>]*>/g, "");
}

// 2. Module export
module.exports = async (api) => {
  // 3. Config reading (in outer closure; script reloads on config change)
  const token = api.config.get("API_TOKEN");

  // 4. Fetch function (fetch, transform, emit)
  async function fetchData() {
    // ...
  }

  // 5. Start phase: initial fetch (awaited before returning)
  await fetchData();

  return {
    refresh: fetchData,
  };
};
```

### Config Reading

Read config **in the outer closure** (before `return`), not inside the fetch function. When config changes, Glanceway reloads the entire script, so the outer closure always has fresh values.

```javascript
module.exports = async (api) => {
  const sort = api.config.get("SORT") || "hot";

  async function fetchData() {
    // use sort directly
  }

  await fetchData();

  return {
    refresh: fetchData,
  };
};
```

### Error Handling

Check `res.ok && res.json` before using response data. For the main/only request, throw on failure. For parallel sub-requests, skip failures silently.

```javascript
// Single request: throw on failure
const res = await api.fetch(url);
if (!res.ok || !res.json) {
  throw new Error(`Failed to fetch articles (HTTP ${res.status})`);
}
api.emit(toItems(res.json));
```

### Parallel Requests

Always use `Promise.allSettled` (never `Promise.all`) for parallel requests. Skip failed results instead of throwing.

```javascript
await Promise.allSettled(
  ids.map(async (id) => {
    const res = await api.fetch(`https://api.example.com/items/${id}`);
    if (res.ok && res.json) {
      items.push(res.json);
      api.emit(/* ... */);
    }
  }),
);
```

### Helper Functions

Define reusable mapping functions (e.g., `toItems`) **inside the fetch function** when they use closure variables. Define pure utility functions (e.g., `stripHtml`) **at the module top** before the export.

```javascript
// Module top: pure utility, no dependency on api or config
function stripHtml(html) {
  return html.replace(/<[^>]*>/g, "");
}

module.exports = async (api) => {
  async function fetchData() {
    // Inside fetch function: uses closure variables
    const toItems = (articles) =>
      articles.map((a) => ({
        id: a.id.toString(),
        title: a.title,
        subtitle: stripHtml(a.description),
        url: a.url,
        timestamp: a.published_at,
      }));
  }

  await fetchData();

  return {
    refresh: fetchData,
  };
};
```

## Auto-Generated Files (Do Not Commit)

The pre-commit hook blocks commits containing `dist/`, `README.md`, or `sources.json`. These are generated by CI on push to master.

---
> Source: [glanceway/glanceway-sources](https://github.com/glanceway/glanceway-sources) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
