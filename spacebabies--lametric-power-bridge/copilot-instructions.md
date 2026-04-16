## lametric-power-bridge

> This document describes the architecture and development guidelines for future AI agents (Claude Code) and human developers.

# LaMetric Power Bridge - Developer Documentation

This document describes the architecture and development guidelines for future AI agents (Claude Code) and human developers.

---

## Architecture Overview

LaMetric Power Bridge is built with a **pluggable architecture** that clearly separates:

1. **Ingress (Sources)**: Data sources for power measurements
2. **Business Logic**: Data transformation and formatting
3. **Egress (Sinks)**: Output to LaMetric Time device

```
┌─────────────────────────────────────────────────────────────┐
│                    bridge.py (43 lines)                     │
│                     Orchestrator                            │
│                                                             │
│  1. Detect/select source (auto or --source CLI arg)        │
│  2. await source.connect()                                  │
│  3. async for reading in source.stream()                   │
│  4. await push_to_lametric(reading)                        │
└─────────────────────────────────────────────────────────────┘
           │                                  │
           ▼                                  ▼
┌──────────────────────┐          ┌──────────────────────┐
│   sources/           │          │   sinks/             │
│   (Ingress)          │          │   (Egress)           │
├──────────────────────┤          ├──────────────────────┤
│ base.py              │          │ lametric.py          │
│ - PowerReading       │─────────▶│ - push_to_lametric() │
│ - PowerSource        │          │ - HTTP POST          │
│   Protocol           │          │ - Icon selection     │
│                      │          │ - kW formatting      │
│ tibber.py            │          └──────────────────────┘
│ - TibberSource       │
│                      │          ┌──────────────────────┐
│ homewizard_v1.py     │          │   tests/             │
│ - HomeWizardV1Source │          ├──────────────────────┤
│   (v1 API - HTTP)    │          │ test_lametric.py     │
│                      │          │ test_tibber.py       │
│ homewizard_v2.py     │          │ test_homewizard_v1.py│
│ - HomeWizardV2Source │          │ conftest.py          │
│   (v2 API - WS)      │          └──────────────────────┘
│                      │
│ p1_serial.py         │
│ - P1SerialSource     │
│   (Serial)           │
└──────────────────────┘
```

---

## Core Concepts

### 1. PowerReading Dataclass

Uniform data contract between all sources and sinks:

```python
@dataclass
class PowerReading:
    power_watts: float      # Positive = consuming, negative = producing
    timestamp: str | None   # ISO8601, optional
```

### 2. PowerSource Protocol

Duck-typed interface (no ABC, no forced inheritance):

```python
class PowerSource(Protocol):
    async def connect(self) -> None:
        """Bootstrap: auth, discovery, setup"""
        ...

    async def stream(self) -> AsyncIterator[PowerReading]:
        """Yield PowerReading objects, handle auto-reconnect"""
        ...
```

**Why Protocol?**
- Explicit interface without boilerplate
- Duck typing continues to work
- Type checking with mypy/pyright
- Pythonic (no forced `class Foo(PowerSource)`)

### 3. Context Manager Pattern

All sources implement async context manager for automatic resource cleanup:

```python
async with HomeWizardV1Source(host="192.168.2.87") as source:
    async for reading in source.stream():
        await push_to_lametric(reading)
# Client automatically closed on exit
```

**Advantages**:
- Guaranteed cleanup (httpx client, websockets, etc)
- Pythonic async pattern
- No manual `aclose()` calls needed
- Also works with old `await source.connect()` pattern (backwards compatible)

**Implementation**:
```python
class YourSource:
    async def __aenter__(self):
        await self.connect()
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        # Cleanup resources (close clients, etc)
        if self.client:
            await self.client.aclose()
```

### 4. Single Responsibility

- **Sources**: Only fetch data and convert to PowerReading
- **Sinks**: Only format data and send
- **Bridge**: Only orchestration (no business logic)

---

## File Structure

```
lametric-power-bridge/
├── bridge.py                    # Orchestrator (43 lines)
├── sources/                     # Ingress modules
│   ├── __init__.py
│   ├── base.py                 # PowerReading + PowerSource Protocol
│   ├── tibber.py               # Tibber WebSocket implementation
│   ├── homewizard_v1.py        # HomeWizard P1 Meter (v1 HTTP API)
│   ├── homewizard_v2.py        # HomeWizard P1 Meter (v2 WebSocket)
│   └── p1_serial.py            # DSMR P1 Serial
├── sinks/
│   ├── __init__.py
│   └── lametric.py             # LaMetric formatting + HTTP push
├── tests/
│   ├── __init__.py
│   ├── conftest.py             # pytest fixtures
│   ├── test_lametric.py        # Sink tests (18 tests)
│   ├── test_tibber.py          # Tibber source tests (3 tests)
│   ├── test_homewizard_v1.py   # HomeWizard v1 tests (7 tests)
│   ├── test_homewizard_v2.py   # HomeWizard v2 tests (8 tests)
│   ├── test_p1_serial.py       # P1 Serial tests (11 tests)
│   └── test_bridge.py          # Bridge logic tests (5 tests)
├── lametric-power-bridge.env    # Configuration (all sources)
├── requirements.txt
├── requirements-dev.txt
└── README.md
```

---

## Multi-Source Selection

### Command Line Source Selection

Bridge.py requires explicit source selection via `--source` argument:

```bash
# Tibber (default if no --source specified)
python bridge.py
python bridge.py --source=tibber

# HomeWizard v1 API (HTTP Polling)
python bridge.py --source=homewizard-v1

# HomeWizard v2 API (WebSocket)
python bridge.py --source=homewizard-v2

# P1 Serial (DSMR via USB)
python bridge.py --source=p1-serial
```

**Important**:
- `--source` fails HARD if configuration is missing (no fallback)
- All sources use the same `.env` file (`lametric-power-bridge.env`)
- Default is `tibber` for backwards compatibility

### Implementation Guidelines

The current `get_source()` function in `bridge.py`:

```python
def get_source(source_name: str):
    """Initialize the selected power source with hard fail on misconfiguration"""
    if source_name == "tibber":
        token = os.getenv("TIBBER_TOKEN")
        if not token:
            logger.error("Tibber: TIBBER_TOKEN not configured in lametric-power-bridge.env")
            sys.exit(1)
        logger.info(f"Using source: Tibber")
        return TibberSource(token=token)
    elif source_name == "homewizard-v1":
        host = os.getenv("HOMEWIZARD_HOST")
        if not host:
            logger.error("HomeWizard P1: HOMEWIZARD_HOST not configured in lametric-power-bridge.env")
            sys.exit(1)
        logger.info(f"Using source: HomeWizard P1 (v1 API)")
        return HomeWizardV1Source(host=host)
    elif source_name == "homewizard-v2":
        host = os.getenv("HOMEWIZARD_HOST")
        token = os.getenv("HOMEWIZARD_TOKEN")
        # ... validation ...
        return HomeWizardV2Source(host=host, token=token)
    elif source_name == "p1-serial":
        return P1SerialSource(device="/dev/ttyUSB0")
    else:
        logger.error(f"Unknown source: {source_name}")
        sys.exit(1)
```

**Pattern for new sources:**
1. Add `elif source_name == "your-source"` case
2. Load config from environment variable (from `lametric-power-bridge.env`)
3. Hard fail with clear error if config is missing
4. Log which source is being used
5. Return initialized source instance

---

## Source Implementation Guide

### Adding a New Source

1. **Create `sources/your_source.py`**
2. **Implement class with PowerSource interface**:
   ```python
   class YourSource:
       async def connect(self) -> None:
           # Bootstrap logic (auth, discovery, etc)
           pass

       async def stream(self) -> AsyncIterator[PowerReading]:
           # Yield PowerReading objects
           while True:
               data = await self.fetch_data()
               yield PowerReading(
                   power_watts=data['power'],
                   timestamp=data.get('timestamp')
               )
   ```
3. **Write tests in `tests/test_your_source.py`**
4. **Update `get_source()` in bridge.py** (add elif case + argparse choices)
5. **Update `lametric-power-bridge.env.example`** with new configuration variables

### Existing Sources

#### Tibber (sources/tibber.py)

**Configuration** (`lametric-power-bridge.env`):
```bash
TIBBER_TOKEN=your_api_token_here
```

**Implementation**:
- `connect()`: HTTP POST to GraphQL API for WebSocket URL + home_id
- `stream()`: WebSocket with `graphql-transport-ws` protocol
- Auto-reconnect: Built-in in `async for websocket in websockets.connect()`

**API Details**:
- Endpoint: `https://api.tibber.com/v1-beta/gql`
- WebSocket subprotocol: `graphql-transport-ws`
- Auth: Bearer token in connection_init payload
- Data: `liveMeasurement.power` (in Watts)

#### HomeWizard P1 v1 API (sources/homewizard_v1.py)

**Configuration** (`lametric-power-bridge.env`):
```bash
HOMEWIZARD_HOST=192.168.2.87
```

**API Documentation**: https://api-documentation.homewizard.com/docs/v1/measurement

**Implementation**:
- `connect()`: Test connectivity with single HTTP GET, validate `active_power_w` field
- `stream()`: HTTP polling to `/api/v1/data` endpoint (1s interval default)
- Keep-alive: `httpx.AsyncClient` for persistent connections
- Error handling: Exponential backoff for device busy (429/503) states
- Data mapping: `active_power_w` → `PowerReading.power_watts`
- Timestamp: `None` (v1 API contains no timestamps)

**Features**:
- Retry logic with max retries (default: 3)
- Device busy detection (503 errors get longer retry delay)
- Graceful handling of missing fields (device initializing)
- Configurable poll interval and timeout

**Notes**:
- Device friendly: keep-alive prevents connection overhead
- v1 API is polling only (no push/WebSocket)
- All fields are optional in API response

#### HomeWizard P1 v2 API (sources/homewizard_v2.py)

**Configuration** (`lametric-power-bridge.env`):
```bash
HOMEWIZARD_HOST=192.168.1.xxx
HOMEWIZARD_TOKEN=your_local_token_here
```

**API Documentation**: https://api-documentation.homewizard.com/docs/v2/websocket

**Implementation**:
- `connect()`: Validates configuration (host and token)
- `stream()`: WebSocket to `/api/ws` endpoint (WSS on port 443)
- Authentication: Token-based challenge/response flow
- Data mapping: `power_w` → `PowerReading.power_watts`
- Timestamp: ISO8601 from API → `PowerReading.timestamp`

**Notes**:
- WebSocket protocol differs from Tibber (no GraphQL)
- Uses self-signed certificates (SSL verification disabled)
- Real-time push updates from device

#### P1 Serial Source (sources/p1_serial.py)

**Configuration**:
- `P1_SERIAL_DEVICE`: Serial device path (default: `/dev/ttyUSB0`)
- `P1_SERIAL_BAUDRATE`: Baudrate (default: `115200` for DSMR v4+, use `9600` for v2/v3)

**Implementation**:
- `connect()`: Open serial port (8N1), read test telegram
- `stream()`: Continuous read of DSMR telegrams
- Data mapping: Parse OBIS `1-0:1.7.0` (cons) and `1-0:2.7.0` (prod) → PowerReading
- Error handling: USB disconnect detection + CRC validation

**DSMR Parsing**:
- Minimal internal parser for power readings (no extra dependencies)
- CRC16 validation for telegram integrity
- Supports net power calculation (consumption - production)

---

## Testing Strategy

### Test Philosophy

- **Unit tests**: Mock all I/O (HTTP, WebSocket, Serial)
- **High-level tests**: Test business logic, not implementation details
- **No network**: `pytest-socket` blocks all network calls (see conftest.py)

### Test Coverage

**Minimum per source**:
1. `connect()` success scenario (mock bootstrap)
2. `connect()` failure scenario (missing config, API down)
3. `stream()` happy path (mock data → PowerReading)

**Sink tests**:
- Test all edge cases: positive/negative, kW conversion, rounding

### Fixtures (tests/conftest.py)

```python
@pytest.fixture
def sample_reading():
    """Standard PowerReading for tests"""
    return PowerReading(power_watts=1500.0, timestamp="2025-12-26T18:00:00")

@pytest.fixture
def mock_tibber_messages():
    """Example Tibber WebSocket messages"""
    return {
        "connection_ack": {"type": "connection_ack"},
        "next": {
            "type": "next",
            "payload": {
                "data": {
                    "liveMeasurement": {
                        "power": 1500,
                        "timestamp": "2025-12-26T18:00:00"
                    }
                }
            }
        },
        "error": {"type": "error", "payload": ["Error message"]}
    }
```

**Conventions**:
- Test functions: `test_<module>_<function>_<scenario>`
- Mock only what is necessary (no over-mocking)
- Use `mocker.patch()` for I/O, not `@patch` decorator

---

## Code Style & Conventions

### Python Style

- **Type hints**: Use where relevant (`PowerReading`, `AsyncIterator`, etc)
- **Docstrings**: Only for public API, not for obvious functions
- **Logging**: Use logger, not print()
- **Error handling**: Log errors, use sys.exit(1) for fatal errors

### Imports

```python
# Standard library
import asyncio
import logging
import os

# Third-party
import requests
import websockets
from dotenv import load_dotenv

# Local
from sources.base import PowerReading
from sinks.lametric import push_to_lametric
```

### Naming Conventions

- **Classes**: `PascalCase` (`TibberSource`, `PowerReading`)
- **Functions**: `snake_case` (`push_to_lametric`, `detect_source`)
- **Constants**: `UPPER_SNAKE_CASE` (`TIBBER_TOKEN`, `ICON_POWER`)
- **Private functions**: `_leading_underscore` (`_perform_http_request`)

### Async Conventions

- **Always await**: No `asyncio.create_task()` without await
- **Auto-reconnect**: Implement in source, not in bridge
- **Error handling**: Try/except in stream loop, await asyncio.sleep(5) after errors

---

## Deployment

### Systemd Service

**Location**: `/etc/systemd/system/lametric-power-bridge.service`

```ini
[Service]
ExecStart=/path/to/.venv/bin/python /path/to/bridge.py
# NO --source argument (auto-detect is default)
```

**Why no `--source` in service?**
- Auto-detect works out-of-the-box for most users
- With hardware changes (e.g. adding USB cable) it works automatically
- Explicit `--source` only for debugging/testing

### Environment Variables

**Location**: Single `.env` file for all configuration

- `lametric-power-bridge.env`: Contains all configuration variables
- Example file: `lametric-power-bridge.env.example`

**Loading in bridge.py**:
```python
# Load configuration from single .env file
load_dotenv("lametric-power-bridge.env")
```

**Variables**:
- `TIBBER_TOKEN`: Tibber API token (required for `--source=tibber`)
- `HOMEWIZARD_HOST`: HomeWizard P1 host IP (required for `--source=homewizard-v1`)
- `LAMETRIC_URL`: LaMetric Push URL (optional - auto-discovery via SSDP, only configure manually for 0 or 2+ devices)
- `LAMETRIC_API_KEY`: LaMetric API key (required for all sources)

---

## Refactoring History

This architecture is the result of step-by-step refactoring:

### STEP 1: Extract LaMetric Sink (commit 41da458)
- Separation of egress logic from bridge.py
- Created `sinks/lametric.py`
- bridge.py: 201 → 146 lines (-55)

### STEP 2: Introduce PowerReading (commit d876461)
- Uniform data contract between sources and sinks
- Created `sources/base.py` with PowerReading dataclass + PowerSource Protocol
- Updated all code to use PowerReading

### STEP 3: Extract Tibber Source (commit df40ac1)
- Isolated Tibber logic in `sources/tibber.py`
- bridge.py: 199 → 43 lines (-78%!)
- Bridge becomes source-agnostic orchestrator

### STEP 4: Add Tibber Tests (commit 5d1eb9c)
- 3 high-level tests for TibberSource
- Total: 9 tests (6 lametric + 3 tibber)

### STEP 5: CLI Source Selection (commits 46fa5a5, d02d186)
- Add `--source` CLI argument with argparse
- Implement `get_source()` factory function with config validation
- Multi-env loading (tibber.env + homewizard.env)
- Hard fail with clear errors on misconfiguration
- 3 new tests for bridge logic
- Total: 12 tests (6 lametric + 3 tibber + 3 bridge)

**Result**: From 259-line monolith to modular architecture with CLI support and full test coverage.

---

## Future Work (TODO)

### Prio 1: Multi-frame Support (optional)
- Extend PowerReading to MeterReading with gas/price data?
- Update LaMetric sink to send multiple frames
- Configurable: which measurements to show?

### Prio 2: Source Detection Improvements
- P1 Serial: Auto-scan `/dev/serial/by-id/*dsmr*` pattern
- HomeWizard: mDNS discovery as fallback
- Command: `python bridge.py --list-sources` to show available sources

---

## Troubleshooting

### Tests fail with SocketBlockedError

**Cause**: `pytest-socket` blocks all network calls (see `tests/conftest.py`).

**Solution**: Mock all HTTP/WebSocket calls with `pytest-mock`.

### Source not detected

**Debug**:
```bash
# Check which env vars are loaded
python -c "from dotenv import load_dotenv; load_dotenv('lametric-power-bridge.env'); import os; print(os.getenv('TIBBER_TOKEN'))"

# Check serial device
ls -la /dev/ttyUSB*

# Check HomeWizard host
python -c "from dotenv import load_dotenv; load_dotenv('lametric-power-bridge.env'); import os; print(os.getenv('HOMEWIZARD_HOST'))"
```

### Systemd service does not start

**Check logs**:
```bash
sudo journalctl -u lametric-power-bridge -f
```

**Common issues**:
- `.venv` path incorrect in service file
- `lametric-power-bridge.env` not readable by service user
- Config variables not correctly set in `lametric-power-bridge.env`

---

## AI Agent Guidelines

If you are an AI agent (Claude Code) working on this project:

### Development Workflow (MANDATORY) 🔄

**For EVERY code change, follow these steps:**

1. **Create a feature branch**
   ```bash
   git checkout -b feature/descriptive-name
   ```
   - NEVER work directly on main
   - Branch name: `feature/`, `fix/`, or `refactor/` prefix

2. **Make the changes**
   - Follow the architecture principles (see above)
   - **Commit early, commit often** on the feature branch
   - Keep commits small and focused (one feature/fix per commit)

3. **Write/update tests**
   - New features: Add tests (minimum 1 test)
   - Bug fixes: Add regression test
   - Refactoring: Existing tests must continue to work

4. **Run tests and commit**
   ```bash
   pytest tests/ -v
   git add -A
   git commit -m "descriptive message"
   ```
   - All tests MUST be green before commit
   - **COMMIT ON BRANCH = DESIRED** (this is not main!)
   - Use descriptive commit messages

5. **Ask user for review/merge**
   - Present the changes
   - Explain what you have done
   - Let the user merge the branch to main

**Why this workflow?**
- **Commits on branches are DESIRED**: this is how git works
- **NEVER commits on main**: user maintains control over main branch
- Test failures are caught early
- Git history remains clean and reviewable
- Easy to revert changes if necessary

**Git Policy Summary**:
- ✅ **DO**: Commit on feature branches (frequently and often!)
- ❌ **DON'T**: Commit directly on main branch (never!)

### Tone and Style 🎭

**IMPORTANT**: This project has a specific voice. Follow these guidelines:

**Prose (README, .env.example, documentation for users)**:
- ✅ **The Vibe:** Richard E. Grant meets Jimmy Carr. Haughty, hacker-friendly, anti-establishment, British sarcasm.
- ✅ **The Goal:** "How do I insult someone without actually using an insult?"
- ✅ Use mild annoyance and dry wit.
- ✅ Reference tone: Existing `README.md` and `.env.example` (perfect balance).
- ✅ Examples:
  - "An expensive pixel clock that has no business costing this much"
  - "It is almost 2026; please keep up"
  - "Do not let me catch you using nano"
  - "I am not your butler"
  - "If you push your secrets to GitHub, I cannot help you."
- ❌ No sycophantic/overly enthusiastic language.
- ❌ No American corporate speak ("amazing!", "awesome!", "delighted to help").

**Code Comments**:
- ✅ Neutral and technical.
- ✅ Clear and informative.
- ❌ **NO** sarcasm or humor in code comments.
- ❌ **NO** humor in docstrings.

**Commit Messages**:
- ✅ Professional and descriptive.
- ❌ No humor (unless subtly fitting and very brief).

**Example contrast**:
```python
# ✅ Code comment (neutral, technical)
# Create persistent HTTP client with keep-alive
self.client = httpx.AsyncClient(...)

# ✅ README prose (sarcastic, British)
"HTTP GET requests, sent every second, like a needy houseguest
checking if dinner is ready."
```

### DO's ✅

- **Commit early, commit often** on feature branches (after every logical step)
- **Write tests** for new sources (minimum 3 tests)
- **Follow PowerSource Protocol** for new sources
- **Mock all I/O** in tests (pytest-socket forces this)
- **Use type hints** where relevant
- **Use logging** for important events
- **Match tone**: Sarcasm in prose, neutral in code

### DON'Ts ❌

- **No over-engineering**: Keep it simple (KISS)
- **No ABCs**: Use Protocol for interfaces
- **No network in tests**: pytest-socket blocks this
- **No sys.exit() in sources**: Only in bootstrap (connect())
- **No work on main branch**: Always use feature branch
- **No sycophantic language**: README is dry/sarcastic, not enthusiastic
- **No humor in code comments**: Only in user-facing prose

### Best Practices

1. **Read the code first**: Understand the architecture before you change
2. **Follow existing patterns**: Tibber is the reference example
3. **Test your work**: `pytest tests/ -v` must always be green
4. **Small commits**: 1 feature/fix per commit
5. **Clear commit messages**: Describe what and why

---

## Resources

### Documentation
- **HomeWizard P1 API**: https://api-documentation.homewizard.com/docs/v2/websocket
- **Tibber GraphQL**: https://developer.tibber.com/docs/overview
- **DSMR Specification**: https://www.netbeheernederland.nl/dossiers/slimme-meter-15

### Dependencies
- **websockets**: AsyncIO WebSocket library
- **requests**: HTTP client for bootstrap calls
- **httpx**: Modern async HTTP client for polling sources (keep-alive support)
- **python-dotenv**: Environment variable loading
- **pytest-asyncio**: AsyncIO support in pytest
- **pytest-mock**: Mocking framework
- **pytest-socket**: Network blocking for tests

### Project Links
- **GitHub**: https://github.com/spacebabies/lametric-power-bridge
- **License**: GPLv3 (no commercial closed-source use)

---

## Changelog

- **2026-01-05**: Release v1.0.0 (First official release)
- **2026-01-02**: SSDP auto-discovery fixed - replaced `async-upnp-client` with custom `asyncio.DatagramProtocol` (library filtered responses from non-standard ports, LaMetric uses port 49153 instead of 1900, security tradeoff documented)
- **2026-01-01**: LaMetric Time SSDP auto-discovery added (LAMETRIC_URL now optional, handles DHCP lease renewals, 12 LaMetric tests total)
- **2025-12-28**: Consolidated .env files - all config in `lametric-power-bridge.env` (learning: separate files was over-engineering)
- **2025-12-28**: HomeWizard P1 v1 API added (`--source=homewizard-v1`, HTTP polling, 18 tests total)
- **2025-12-28**: CLI source selection added (STEP 5: `--source` argument, 12 tests total)
- **2025-12-26**: Stale data timeout monitoring added (60s timeout, "-- W" indicator)
- **2025-12-26**: Development Workflow added to CLAUDE.md (mandatory feature branches)
- **2025-12-26**: Refactoring to pluggable architecture (STEP 1-3)
- **2025-12-26**: Tibber tests added (9 tests total: 6 lametric + 3 tibber)
- **2025-12-26**: CLAUDE.md documentation written

---

*This document evolves with the project. Update on significant architecture changes.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/spacebabies) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
