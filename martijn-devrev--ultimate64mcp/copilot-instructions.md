## ultimate-mcp-tools

> Guide for using the Commodore 64 Ultimate MCP Server tools to control real C64 hardware


# Commodore 64 Ultimate MCP Server - Tool Guide

This MCP server controls real Commodore 64 hardware via the Ultimate 64/II+ REST API.

## Connection Management

### Setting Up Connection
Always ensure connection is established before using other tools:

```
ultimate_set_connection: Set the C64's IP address
  - hostname: "192.168.1.64" (required)
  - port: 80 (optional, default)

ultimate_get_connection: Verify current connection
ultimate_version: Test API connectivity
```

## Running Programs

### For files ON the Ultimate device (USB/SD):
```
ultimate_run_program:
  - file: "/Usb0/Games/game.prg"  # Path on Ultimate filesystem
```

### For files NOT on the device (remote upload):
```
ultimate_run_prg_binary:
  - prg_data_base64: "AQgLCJ4A..."  # Base64-encoded PRG
  - url: "https://example.com/demo.prg"  # OR download from URL
  - file_path: "/local/path/demo.prg"  # OR local server file (path on the SERVER filesystem)
  
  Use ONE of the above parameters, not multiple.
```

### Direct Upload (Recommended for Local Files)
If you have a PRG file on your local machine and want to run it via the MCP server (bypassing MCP protocol limits or path issues):

```bash
curl -X POST <mcp-server-url>/upload-prg \
  -H "Content-Type: application/octet-stream" \
  --data-binary @path/to/demo.prg
```

### PRG File Format
PRG files start with 2-byte load address (little-endian):
- Byte 0: Low byte of load address
- Byte 1: High byte of load address
- Remaining bytes: Program data

Example: `01 08 ...` = Load at $0801 (BASIC start)

## Playing Music

### SID Files (C64 native music)
```
ultimate_play_sid:
  - file: "/Usb0/Music/Commando.sid"
  - song_number: 1  # Optional, for multi-song SIDs
```

### MOD Files (Amiga tracker music)
```
ultimate_play_mod:
  - file: "/Usb0/Music/track.mod"
```

## Memory Operations

### Reading Memory
```
ultimate_read_memory:
  - address: "D020"  # Hex address (border color)
  - length: 16       # Bytes to read (max 256)
```

### Writing Memory
```
ultimate_write_memory:
  - address: "D020"  # Hex address
  - data: "00"       # Hex data (blue border)
  
ultimate_write_memory_binary:
  - address: "C000"  # Hex address
  - file_path: "/path/to/data.bin"
```

### Common Memory Addresses
| Address | Description |
|---------|-------------|
| `D020` | Border color (0-15) |
| `D021` | Background color (0-15) |
| `0400` | Screen RAM start (40x25 chars) |
| `D800` | Color RAM start |
| `D000` | Sprite 0 X position |
| `D001` | Sprite 0 Y position |
| `D015` | Sprite enable register |
| `D418` | SID volume (low nibble) |

## Disk Operations

### Mounting Disk Images
```
ultimate_mount_disk:
  - drive: "A"  # A, B, C, or D
  - file: "/Usb0/Disks/game.d64"

ultimate_unmount_disk:
  - drive: "A"
```

### Creating Disk Images
```
ultimate_create_d64:
  - path: "/Usb0/Disks/newdisk.d64"
  - tracks: 35  # or 40
  - diskname: "MY DISK"  # Optional

ultimate_create_d71:  # Double-sided
  - path: "/Usb0/Disks/newdisk.d71"

ultimate_create_d81:  # 3.5" format
  - path: "/Usb0/Disks/newdisk.d81"
```

### Drive Control
```
ultimate_turn_drive_on:
  - drive: "A"

ultimate_turn_drive_off:
  - drive: "A"

ultimate_set_drive_mode:
  - drive: "A"
  - mode: "1541"  # or "1571", "1581"
```

## Machine Control

### Reset Operations
```
ultimate_reset_machine:    # Standard C64 reset
ultimate_soft_reset:       # Load empty program
ultimate_reboot_device:    # Reboot Ultimate device
ultimate_power_off:        # Power off (careful!)
```

### Machine State
```
ultimate_get_machine_info:   # Hardware info
ultimate_get_machine_state:  # Current state
```

## Configuration

### Reading Configuration
```
ultimate_get_config_categories:  # List all categories
ultimate_get_config_category:
  - category: "C64 and Cartridge Settings"
  
ultimate_get_config_item:
  - category: "C64 and Cartridge Settings"
  - item: "SID Type"
```

### Writing Configuration
```
ultimate_set_config_item:
  - category: "C64 and Cartridge Settings"
  - item: "SID Type"
  - value: "8580"

ultimate_save_config:   # Save to flash
ultimate_load_config:   # Load from flash
ultimate_reset_config:  # Factory defaults
```

## Streaming (Ultimate 64 Only)

```
ultimate_start_stream:
  - stream: "video"  # or "audio", "debug"
  - ip: "192.168.1.100:6789"

ultimate_stop_stream:
  - stream: "video"
```

## Best Practices

### Before Running Programs
1. Check connection: `ultimate_get_connection`
2. Consider reset first: `ultimate_reset_machine`
3. Wait briefly after reset before loading

### Memory Access Patterns
- Read before write if preserving state
- Use hex addresses WITHOUT $ prefix: "D020" not "$D020"
- Maximum read is 256 bytes per call

### File Paths on Ultimate
- USB drives: `/Usb0/`, `/Usb1/`
- SD card: `/Sd/`
- Case sensitivity varies

### Uploading Custom Programs
For AI-generated demos or assembled code:
1. Assemble to PRG format (with load address header)
2. Base64 encode the binary
3. Use `ultimate_run_prg_binary` with `prg_data_base64`

### Color Values (C64 Palette)
| Value | Color |
|-------|-------|
| 0 | Black |
| 1 | White |
| 2 | Red |
| 3 | Cyan |
| 4 | Purple |
| 5 | Green |
| 6 | Blue |
| 7 | Yellow |
| 8 | Orange |
| 9 | Brown |
| 10 | Light Red |
| 11 | Dark Grey |
| 12 | Grey |
| 13 | Light Green |
| 14 | Light Blue |
| 15 | Light Grey |

## Error Handling

If tools return errors:
- "No C64 host configured" → Use `ultimate_set_connection` first
- Connection refused → Check IP address and network
- File not found → Verify path on Ultimate device
- Timeout → Device may be busy, try again

## Example Workflows

### Quick Demo Test
1. `ultimate_set_connection` with device IP
2. `ultimate_version` to verify connection
3. `ultimate_run_prg_binary` with base64-encoded demo

### Play Music Collection
1. `ultimate_set_connection`
2. `ultimate_play_sid` with file path
3. Change songs with `song_number` parameter

### Change Screen Colors
1. `ultimate_set_connection`
2. `ultimate_write_memory` address "D020" data "06" (blue border)
3. `ultimate_write_memory` address "D021" data "00" (black background)

---
> Source: [Martijn-DevRev/Ultimate64MCP](https://github.com/Martijn-DevRev/Ultimate64MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
