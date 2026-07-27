## rivian-python-api

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Python API and CLI tool for interacting with Rivian vehicles. It provides programmatic access to vehicle state, charging information, orders, and trip planning through Rivian's GraphQL API.

**Important Security Note**: Use a non-primary driver account for API access to avoid account lockouts. Rivian rate-limits API calls, and excessive polling can lock your account.

## Development Commands

### Setup
```bash
pip install -r requirements.txt
```

### Run CLI
```bash
bin/rivian_cli [options]
# or directly:
python src/rivian_python_api/rivian_cli.py [options]
```

### Common CLI Operations
```bash
# Login (requires RIVIAN_USERNAME and RIVIAN_PASSWORD in environment)
bin/rivian_cli --login

# Get vehicle state
bin/rivian_cli --state

# Poll vehicle state continuously
bin/rivian_cli --poll --poll_frequency 30

# Get vehicle orders
bin/rivian_cli --vehicle_orders

# Plan a trip (requires MAPBOX_API_KEY in .env)
bin/rivian_cli --plan_trip <soc>,<range_meters>,<origin_lat>,<origin_long>,<dest_lat>,<dest_long>

# Get charging sessions
bin/rivian_cli --charge_sessions

# See all options
bin/rivian_cli --help
```

## Code Architecture

### Core Components

**rivian_api.py** (src/rivian_python_api/rivian_api.py)
- Main API wrapper class (`Rivian`)
- Handles all GraphQL queries to Rivian's API endpoints
- Manages authentication flow including OTP/MFA
- Key endpoints:
  - `RIVIAN_GATEWAY_PATH`: Vehicle state, user info, trip planning
  - `RIVIAN_CHARGING_PATH`: Charging sessions and wallbox data
  - `RIVIAN_ORDERS_PATH`: Orders and retail purchases
  - `RIVIAN_TRANSACTIONS_PATH`: Transaction status and finance info

**rivian_cli.py** (src/rivian_python_api/rivian_cli.py)
- CLI interface for the API
- Wraps API calls with user-friendly output formatting
- Handles authentication state persistence via pickle files
- Supports metric/imperial unit conversion
- Privacy mode to hide PII (VIN, GPS, addresses)

**rivian_map.py** (src/rivian_python_api/rivian_map.py)
- Trip planning visualization using Mapbox
- Decodes polyline routes and displays charging stops
- Requires `MAPBOX_API_KEY` environment variable

### Authentication Flow

1. Call `create_csrf_token()` to get initial session tokens
2. Call `login(username, password)`
3. If OTP required (`otp_needed == True`), call `login_with_otp(username, otpCode, otpToken)`
4. Store tokens: `_access_token`, `_refresh_token`, `_user_session_token`
5. CLI persists tokens in `rivian_auth.pickle` for subsequent runs

### Key Data Flows

**Getting Vehicle State:**
1. CLI determines vehicle_id (from orders or user_information)
2. Calls `get_vehicle_state(vehicle_id, minimal=False)`
3. Returns comprehensive state including: power, battery, location, doors, climate, OTA status, etc.

**Polling Logic:**
- Minimal query mode reduces data fetched to avoid rate limits
- Calculates speed from odometer deltas between polls
- Optional sleep periods when vehicle inactive to allow sleep state

**Trip Planning:**
1. Call `plan_trip()` with origin/destination coords and starting battery state
2. Response includes polyline-encoded route and charging waypoints
3. `rivian_map.py` decodes and visualizes with Mapbox

### GraphQL Query Pattern

All API calls follow this pattern:
```python
headers = self.gateway_headers()  # or transaction_headers()
query = {
    "operationName": "OperationName",
    "query": "query OperationName { ... }",
    "variables": { ... }
}
response = self.raw_graphql_query(url=ENDPOINT, query=query, headers=headers)
return response.json()
```

Headers include session tokens (`A-Sess`, `U-Sess`, `Csrf-Token`) and unique client IDs.

## Important Conventions

- **Unit Conversion**: All functions accept `metric` boolean parameter. API returns meters/celsius, convert to miles/fahrenheit when `metric=False`
- **Privacy Mode**: Use `--privacy` flag to redact PII in CLI output
- **Verbose Mode**: Use `--verbose` to see raw API responses for debugging
- **Vehicle ID**: Most operations auto-detect first vehicle, or use `--vehicle_id` to specify
- **Timestamp Handling**: Use `get_local_time()` and `show_local_time()` helpers for timezone conversion

## Rate Limiting

Rivian aggressively rate-limits API access. Best practices:
- Use minimal queries when polling (`minimal=True`)
- Implement backoff/sleep periods during inactivity
- Never poll faster than every 30 seconds
- Use secondary account for API access to avoid locking primary account

## Environment Variables

Required:
- `RIVIAN_USERNAME`: Rivian account email
- `RIVIAN_PASSWORD`: Rivian account password

Optional:
- `RIVIAN_AUTHORIZATION`: Alternative auth format `access_token;refresh_token;user_session_token`
- `MAPBOX_API_KEY`: Required for trip planning visualization

## Vehicle Commands (Experimental)

Vehicle commands (lock, unlock, frunk, etc.) require HMAC signatures using phone enrollment keys. Implementation is incomplete (`send_vehicle_command()` has placeholder HMAC). Rivian limits to 2 enrolled phones per vehicle, so using API for commands costs a phone slot.

---
> Source: [the-mace/rivian-python-api](https://github.com/the-mace/rivian-python-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
