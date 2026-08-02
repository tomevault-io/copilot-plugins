## pytak

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install in editable mode (default make target)
python3 -m pip install -e .

# Install test dependencies
python3 -m pip install -r requirements_test.txt

# Run all tests
python3 -m pytest

# Run a single test file
python3 -m pytest tests/test_classes.py

# Run a single test by name
python3 -m pytest tests/test_classes.py::test_function_name

# Run tests with coverage
python3 -m pytest --cov=pytak --cov-report term-missing

# Lint
pylint --max-line-length=88 src/pytak/*.py
flake8 --max-line-length=88 --extend-ignore=E203 src/pytak/*.py
black .
```

## Architecture

PyTAK is an asyncio-based Python library for building TAK (Team Awareness Kit) network clients and gateways. Applications built on PyTAK subclass its workers and plug into its queue pipeline.

### Worker pipeline

The core pattern is a producer→queue→consumer pipeline running under `asyncio`:

```
QueueWorker.run()  →  tx_queue  →  TXWorker.run()  →  network writer
                                                              ↕
                       rx_queue  ←  RXWorker.run()  ←  network reader
```

- **`Worker`** (`classes.py`) — abstract base; holds `queue` and `config`; provides `fts_compat()` sleep, queue-full handling, and the `run()` loop that calls `run_once()` forever.
- **`TXWorker(Worker)`** — dequeues bytes and writes to a network `writer`; handles XML→Protobuf conversion when `TAK_PROTO > 0` via the optional `takproto` package.
- **`RXWorker(Worker)`** — reads from a network `reader` using `readcot()` (stream boundary detection on `</event>`), puts raw CoT bytes onto `rx_queue`.
- **`QueueWorker(Worker)`** — application-level producer; subclass this and implement `handle_data()` to produce CoT events; use `put_queue()` to enqueue.
- **`CLITool`** — top-level harness; holds `tx_queue` / `rx_queue`, creates workers via `create_workers()`, and drives `asyncio.wait()` on all running tasks.

### Transport layer

`protocol_factory()` in `client_functions.py` dispatches on `COT_URL` scheme and returns `(reader, writer)`:

| Scheme | Transport |
|---|---|
| `tcp://`, `tcp+wo://`, `tcp+ro://` | `asyncio.open_connection` |
| `tls://`, `ssl://`, `tls+wo://`, `tls+ro://`, … | `create_tls_client()` → `get_ssl_ctx()` |
| `udp://`, `udp+wo://`, `udp+ro://`, `udp+broadcast://`, `udp+multicast://` | bundled `asyncio_dgram` |
| `log://` | stdout/stderr buffer |
| `file://` | binary file |

TLS supports PKCS#12 (`.p12`) certs: `convert_cert()` in `crypto_functions.py` extracts PEM files. If `PYTAK_TLS_CERT_ENROLLMENT_USERNAME` + `PYTAK_TLS_CERT_ENROLLMENT_PASSWORD` are set, `CertificateEnrollment` in `crypto_classes.py` performs a full CSR→PKCS#12 enrollment flow (requires `cryptography` and `aiohttp`).

### Configuration

All configuration flows through `configparser.SectionProxy`. The `cli()` function in `client_functions.py` is the standard entry point: it merges environment variables, `config.ini`, and optional ATAK pref packages (`.zip`) into a single config, then calls `asyncio.run(main(...))`. Key env vars:

- `COT_URL` — destination URL (default: `udp+wo://239.2.3.1:6969` ATAK multicast). Append `+wo` or `+ro` to `tcp`/`tls`/`udp` for write-only or read-only (`tls+wo://` drains inbound without enqueueing; useful for send-only gateways)
- `TAK_PROTO` — `0` for XML (default), `1` for Protobuf (requires `takproto`)
- `PYTAK_TLS_CLIENT_CERT` / `PYTAK_TLS_CLIENT_KEY` / `PYTAK_TLS_CLIENT_CAFILE` — TLS identity
- `PYTAK_TLS_DONT_VERIFY` / `PYTAK_TLS_DONT_CHECK_HOSTNAME` — TLS verification bypass
- `FTS_COMPAT` — enable random inter-send sleep for FreeTAKServer compatibility
- `DEBUG` — verbose logging

### CoT data model

`SimpleCOTEvent` and `COTEvent` (dataclasses in `classes.py`) hold lat/lon/uid/stale/type and render via `cot2xml()` → `gen_cot_xml()` in `functions.py`. `TAKDataPackage` generates TAK Data Package zip files.

### Optional dependencies

- `takproto` — TAK Protobuf v1 encode/decode (install: `pip install pytak[with_takproto]`)
- `cryptography` + `aiohttp` — TLS cert enrollment via `CertificateEnrollment`

### How downstream tools use PyTAK

Downstream tools (ADSBCOT, AISCOT, etc.) follow this pattern:

```python
# In the tool's module:
def create_tasks(config, clitool):
    return set([MyQueueWorker(clitool.tx_queue, config)])

# Entry point:
def main():
    pytak.cli("my_tool_module")
```

`pytak.cli()` handles everything else: arg parsing, config loading, TLS setup, worker wiring, and the asyncio event loop.

### Transport additions (recent)

- **`tak://` onboarding** — `resolve_tak_url()` in `client_functions.py` parses TAK enrollment deep-links, checks `~/.pytak/certs/` for a cached cert, re-enrolls via `CertificateEnrollment` if needed, then rewrites `COT_URL` to `tls://`. Requires `aiohttp` + `cryptography`.
- **`marti://` REST API** — `marti_txworker_factory()` / `marti_rxworker_factory()` create `MartiTXWorker` / `MartiRXWorker` (in `classes.py`) that POST/poll CoT via the TAK Server Marti HTTP API. `marti://` uses TLS; `marti+http://` uses plain HTTP. Requires `aiohttp`.

---

## Documentation

The docs live in `docs/` and are built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/). The site is hosted at **pytak.rtfd.io** via Read the Docs.

### Build and preview locally

```bash
pip install -r docs/requirements.txt
mkdocs serve          # live-reload at http://127.0.0.1:8000
mkdocs build          # static output to site/
```

### Doc structure

| File | Purpose |
|---|---|
| `docs/index.md` | Home — includes `README.md` via `{!README.md!}` |
| `docs/quickstart.md` | Zero-to-CoT in 5 minutes |
| `docs/installation.md` | Debian pkg, pip, source, Windows |
| `docs/configuration.md` | All env vars / config params |
| `docs/examples.md` | Runnable code examples |
| `docs/compatibility.md` | Supported TAK clients, protocols, Python version |
| `docs/clients.md` | Known downstream PyTAK-based tools |
| `docs/troubleshooting.md` | Common errors and fixes |
| `docs/changelog.md` | Includes `CHANGELOG.md` via `{!CHANGELOG.md!}` |

### Guidelines for future agents

- **No placeholder text.** Never leave `TK`, `TK TK TK`, or `FIXME` in documentation.
- **Fix RST-style double-backticks.** This project uses MkDocs (Markdown), not Sphinx (RST). Replace `` ``code`` `` with `` `code` ``.
- **Keep examples runnable.** Every code block in `docs/examples.md` should work as-is (no personal paths, no dummy credentials that look real). Use `takserver.example.com` as the hostname placeholder.
- **Keep config params in sync.** When adding a new config constant to `src/pytak/constants.py`, add a corresponding entry to `docs/configuration.md`.
- **Use admonitions for warnings.** Use `!!! warning`, `!!! note`, and `!!! tip` callouts rather than inline parentheticals for important caveats.
- **Test the build.** After editing docs, run `mkdocs build` to confirm there are no broken includes or YAML errors.
- **`{!path!}` syntax** is the include-markdown plugin's delimiter (not RST). Use it to include files like `{!examples/send.py!}`.
- **Nav lives in `mkdocs.yml`.** Add new pages there under the `nav:` key; they won't appear in the sidebar otherwise.

---
> Source: [snstac/pytak](https://github.com/snstac/pytak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
