## epson-esc-proxy

> A proxy server that receives Epson ePOS XML/SOAP requests (typically from web-based POS systems) and forwards them to a thermal receipt printer via ESC/POS commands.

# epos-proxy - Epson ePOS Printer Proxy

A proxy server that receives Epson ePOS XML/SOAP requests (typically from web-based POS systems) and forwards them to a thermal receipt printer via ESC/POS commands.

## Project Structure

```
epos-proxy/
├── epos_proxy/
│   ├── __init__.py     # Package init
│   ├── cli.py          # Command-line interface and argument parsing
│   ├── config.py       # Configuration defaults and global state
│   ├── printer.py      # Printer connection, reconnection, ESC/POS commands
│   ├── server.py       # HTTP/HTTPS server and ePOS request handler
│   └── certs.py        # Self-signed SSL certificate generation
├── main.py             # Entry point for `python main.py`
├── pyproject.toml      # Project metadata and dependencies
└── CLAUDE.md           # This file
```

## Running

```bash
# Install dependencies
uv sync

# Run with defaults (HTTP on 0.0.0.0:8000, Linux USB printer)
uv run python main.py

# Or use the installed script
uv run epos-proxy

# Common options
uv run epos-proxy --https                      # Enable HTTPS
uv run epos-proxy --printer 192.168.1.87       # Network printer
uv run epos-proxy --printer /dev/usb/lp1       # Different USB device
uv run epos-proxy -H 192.168.1.5 -p 443        # Custom host/port
uv run epos-proxy --width 512                  # Set paper width for centering
```

## How It Works

1. **Receives** ePOS XML/SOAP requests via HTTP/HTTPS POST
2. **Parses** the XML for `<image>` tags (raster bitmap data) and `<pulse>` tags (drawer kick)
3. **Converts** to ESC/POS commands (GS v 0 for raster graphics, ESC p for drawer)
4. **Sends** to printer via network socket, USB device file, or serial port
5. **Returns** ePOS-compliant XML response to satisfy the client

## Printer Connection Types

- **Network**: `--printer 192.168.1.87` (uses port 9100 by default)
- **USB device file**: `--printer /dev/usb/lp0` (Linux)
- **USB by ID**: `--printer USB:0x04b8:0x0202` (vendor:product, requires libusb)
- **Serial**: `--printer COM3` (Windows) or `--printer /dev/ttyUSB0` (Linux)

## Key Features

- **Auto-reconnect**: Retries up to 3 times on connection failure
- **Centering**: Pads narrow images to center on paper (configurable via `--width`)
- **CORS support**: Allows requests from web browsers
- **HTTPS**: Self-signed certificate generation for secure contexts

## ESC/POS Commands Used

- `ESC @` (0x1B 0x40) - Initialize printer
- `GS v 0` (0x1D 0x76 0x30) - Print raster bit image
- `ESC d n` (0x1B 0x64 n) - Feed n lines
- `GS V` (0x1D 0x56) - Cut paper
- `ESC p` (0x1B 0x70) - Kick cash drawer

## Dependencies

- `python-escpos` - ESC/POS printer library (handles connection types)

## Development Notes

- The ePOS image data is already rasterized by the client; we just wrap it in ESC/POS commands
- Width/height come from XML attributes; if missing, defaults to `--width` and calculates height
- SSL certs are generated in the working directory (server.crt, server.key)
- Config is stored in a global dict in `config.py` for simplicity

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thearyadev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
