## vphone-mcp

> MCP server for programmatic control of vphone-cli iOS VMs.

# vphone-mcp

MCP server for programmatic control of vphone-cli iOS VMs.

## Quick Reference

- **Install:** `uv sync`
- **Run:** `uv run vphone-mcp`
- **Test:** `uv run python -c "from vphone_mcp.client import VPhoneClient; print(VPhoneClient('/path/to/vm/vphone.sock').screenshot('/tmp/test.png'))"`

## Architecture

```
Claude Code / Claude Desktop
    ↓ MCP (stdio)
vphone-mcp (Python)
    ↓ Unix socket (JSON)
vphone-cli (Swift, vm/vphone.sock)
    ↓ Virtualization.framework
iOS VM
```

## Tool Layers

1. **Hardware keys** — `go_home`, `press_power`, `volume_up`, `volume_down`
2. **Screenshots** — `screenshot` (returns embedded image)
3. **Pre-mapped navigation** — `open_app`, `tap_back`, `scroll_down`, `scroll_up`, `open_notification_center`, `open_control_center`, `open_app_switcher`
4. **Raw interaction** — `tap(x,y)`, `swipe(x1,y1,x2,y2)` for app-specific UI

## Configuration

Set `VPHONE_SOCK` env var to point to the socket, or it auto-discovers from default paths.

## Coordinate System

All pixel coordinates are for the default 1290x2796 screen (3x scale). Use `screenshot()` to see the current display and derive coordinates for `tap()`.

---
> Source: [pluginslab/vphone-mcp](https://github.com/pluginslab/vphone-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
