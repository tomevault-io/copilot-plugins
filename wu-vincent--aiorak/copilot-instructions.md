## aiorak

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

```bash
# Run full test suite (use -n auto to parallelize via pytest-xdist)
python -m pytest -n auto

# Run a single test file
python -m pytest tests/test_reliability.py

# Run a single test class or function
python -m pytest tests/test_reliability.py::TestOrdering::test_ordering_in_order

# Lint and format
ruff check src/ tests/
ruff format src/ tests/
```

No build step required - pure Python package with zero runtime dependencies. Uses hatchling for packaging. Python 3.10+.

## Architecture

aiorak is a RakNet reliable UDP implementation for Python asyncio. It is wire-compatible with RakNet 4.x peers, written from scratch using the C++ source as reference (not a direct port).

### Layer stack (bottom to top)

1. **`_transport.py`** - Thin asyncio `DatagramProtocol` wrapper. Dispatches raw UDP datagrams to a callback.
2. **`_bitstream.py`** - Bit-level serialization matching RakNet's `BitStream` class. All wire encoding goes through this.
3. **`_wire.py`** - Datagram header, message frame, and range list encode/decode. Three datagram types: ACK, NAK, and Data (containing message frames).
4. **`_congestion.py`** - TCP-like sliding window congestion control. Tracks RTT, cwnd, and RTO estimation.
5. **`_reliability.py`** - Core reliability layer per connection: send/receive queues, ACK/NAK generation, ordering heaps (32 channels), sequencing, split-packet reassembly, resend buffer. Driven by periodic `update()` calls (~10ms) and `on_datagram_received()`.
6. **`_connection.py`** - Per-peer state machine (`DISCONNECTED -> CONNECTING -> CONNECTED -> DISCONNECTING -> DISCONNECTED`). Owns a `ReliabilityLayer` + `CongestionController`. Handles the 6-message handshake (3 offline + 3 reliable).
7. **`_server.py` / `_client.py`** - High-level async API. Server spawns handler coroutines per peer. Client provides `send()`/`recv()`/`async for`.

### Handshake flow

**Offline phase** (raw UDP, identified by OFFLINE_MAGIC): OpenConnectionRequest1 -> Reply1 -> Request2 -> Reply2 (establishes MTU).
**Reliable phase** (through ReliabilityLayer): ConnectionRequest -> ConnectionRequestAccepted -> NewIncomingConnection.

### Key design points

- `_constants.py` holds protocol constants (message IDs, MTU limits, timing). Protocol version, MTU bounds, and discovery sizes are configurable per Server/Client instance.
- `_types.py` defines `Reliability` enum (8 modes) with `is_reliable`/`is_sequenced`/`is_ordered` properties used throughout the reliability layer.
- Sequence numbers are 24-bit (`SEQ_NUM_MAX = 0xFFFFFF`).
- The receive-side uses a sliding window (`_ReceivedWindow`, size 512) instead of an unbounded set.
- Split trackers have timeout-based eviction matching C++ behavior.

## Testing

- `pytest-asyncio` with `asyncio_mode = "auto"` - all async test functions run automatically.
- `conftest.py` provides `server_factory`, `client_factory`, and `server_and_client` fixtures for integration tests.
- Unit tests instantiate internal classes directly (e.g., `ReliabilityLayer`, `BitStream`).
- Integration tests use real UDP loopback connections.
- Ruff config: line-length 120, import sorting enabled.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wu-vincent) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
