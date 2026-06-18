## elegoo-centauri-skill

> Control and monitor the Elegoo Centauri Carbon 3D printer via SDCP WebSocket protocol. Get status, temperatures, file list, start/stop/pause prints, set temperatures, control fans and lights, jog axes, and upload files.


# Elegoo Centauri Carbon - SDCP Control Skill

Control the Elegoo Centauri Carbon 3D printer via its native SDCP (Smart Device Control Protocol) WebSocket interface.

## Connection Details

- **Printer:** Elegoo Centauri Carbon
- **IP:** Configurable (default: `192.168.x.x`)
- **WebSocket:** `ws://<PRINTER_IP>:3030/websocket`
- **MainboardID:** Auto-discovered on first connection
- **Protocol:** SDCP V3.0.0

## Setup

1. Install the dependency:
```bash
pip install websockets
```

2. Configure your printer IP in `references/elegoo_centauri.py` or pass it to the constructor:
```python
async with ElegooCentauri(ip="192.168.1.100") as printer:
    status = await printer.get_status()
```

The MainboardID is auto-discovered on the first connection and cached for subsequent requests.

## Quick Usage

```python
from elegoo_centauri import ElegooCentauri

async with ElegooCentauri() as printer:
    # Get full status
    status = await printer.get_status()
    print(f"Nozzle: {status['TempOfNozzle']}°C")
    print(f"Bed: {status['TempOfHotbed']}°C")
    
    # Get attributes
    attrs = await printer.get_attributes()
    print(f"Firmware: {attrs['FirmwareVersion']}")
    
    # Get file list
    files = await printer.get_file_list()
    
    # Start a print
    await printer.start_print("/local/model.gcode")
    
    # Set temperatures
    await printer.set_nozzle_temp(200)
    await printer.set_bed_temp(60)
    await printer.set_chamber_temp(40)
    
    # Control fans (0-100)
    await printer.set_fan_speed("ModelFan", 100)
    await printer.set_fan_speed("AuxiliaryFan", 70)
    await printer.set_fan_speed("BoxFan", 50)
    
    # Light control
    await printer.set_light(True)
    
    # Jog axes (distance in mm)
    await printer.jog_axis("X", 10)  # +10mm on X
    await printer.jog_axis("Y", -5)  # -5mm on Y
    await printer.jog_axis("Z", 1)   # +1mm on Z
    
    # Home axes
    await printer.home_axes()
    
    # Print control
    await printer.pause_print()
    await printer.resume_print()
    await printer.stop_print()
    
    # Set print speed (percentage)
    await printer.set_print_speed(150)
```

## CLI Usage

The helper script also works as a CLI tool:

```bash
python elegoo_centauri.py status          # Get printer status
python elegoo_centauri.py info            # Get printer attributes
python elegoo_centauri.py files           # List files on printer
python elegoo_centauri.py start model.gcode  # Start a print
python elegoo_centauri.py pause           # Pause current print
python elegoo_centauri.py resume          # Resume paused print
python elegoo_centauri.py stop            # Stop current print
python elegoo_centauri.py nozzle 200      # Set nozzle temp to 200°C
python elegoo_centauri.py bed 60          # Set bed temp to 60°C
python elegoo_centauri.py chamber 40      # Set chamber temp to 40°C
python elegoo_centauri.py fan ModelFan 100  # Set fan to 100%
python elegoo_centauri.py light on        # Turn light on
python elegoo_centauri.py jog X 10        # Jog X axis +10mm
python elegoo_centauri.py home            # Home all axes
python elegoo_centauri.py speed 150       # Set print speed to 150%
python elegoo_centauri.py history         # Get print history
```

## SDCP Command Reference

| Command | Value | Direction | Description |
|---------|-------|-----------|-------------|
| GET_PRINTER_STATUS | 0 | Request | Get current temps, fans, light, print status |
| GET_PRINTER_ATTR | 1 | Request | Get printer attributes (name, firmware, MAC, etc.) |
| SEND_PRINTER_DISCONNECT | 64 | Send | Disconnect from printer |
| SEND_PRINTER_START_PRINT | 128 | Send | Start printing a file |
| SEND_PRINTER_SUSPEND_PRINT | 129 | Send | Pause current print |
| SEND_PRINTER_STOP_PRINT | 130 | Send | Stop/cancel current print |
| SEND_PRINTER_RESTORE_PRINT | 131 | Send | Resume paused print |
| GET_BLACKOUT_STATUS | 134 | Request | Check power outage recovery status |
| SEND_BLACKOUT_ACTION | 135 | Send | Respond to power outage recovery |
| SEND_PRINTER_EDIT_NAME | 192 | Send | Rename the printer |
| EDIT_PRINTER_FILE_NAME | 193 | Send | Rename a file on the printer |
| GET_PRINTER_FILE_LIST | 258 | Request | List files on printer storage |
| GET_PRINTER_FILE_DETAIL | 260 | Request | Get details/thumbnail for a file |
| GET_PRINTER_HISTORY_ID | 320 | Request | Get print history |
| GET_PRINTER_TASK_DETAIL | 321 | Request | Get task/print job details |
| DELETE_PRINTER_HISTORY | 322 | Send | Delete a print history entry |
| GET_PRINTER_HISTORY_VIDEO | 323 | Request | Get timelapse video |
| EDIT_PRINTER_VIDEO_STREAMING | 386 | Send | Control video streaming |
| EDIT_PRINTER_TIME_LAPSE_STATUS | 387 | Send | Control timelapse |
| EDIT_PRINTER_AXIS_NUMBER | 401 | Send | Jog/move an axis by distance |
| EDIT_PRINTER_AXIS_ZERO | 402 | Send | Home/zero axes |
| EDIT_PRINTER_STATUS_DATA | 403 | Send | Set temps, fans, lights, speed |

## Message Format

### Request (send to printer):
```json
{
  "Id": "",
  "Data": {
    "Cmd": <command_int>,
    "Data": <command_specific_data>,
    "RequestID": "<unique_hex_id>",
    "MainboardID": "<auto_discovered>",
    "TimeStamp": <unix_timestamp>,
    "From": 1
  }
}
```

### Response Topics:
- `sdcp/status/<MainboardID>` — Status updates (temps, print info)
- `sdcp/attributes/<MainboardID>` — Printer attributes
- `sdcp/response/<MainboardID>` — Command responses
- `sdcp/error/<MainboardID>` — Error messages

## Status Response Fields

```json
{
  "Status": {
    "CurrentStatus": [0],
    "TempOfHotbed": 23.6,
    "TempOfNozzle": 24.4,
    "TempOfBox": 23.5,
    "TempTargetHotbed": 0,
    "TempTargetNozzle": 0,
    "TempTargetBox": 0,
    "CurrenCoord": "0.00,0.00,0.00",
    "CurrentFanSpeed": {
      "ModelFan": 0,
      "AuxiliaryFan": 0,
      "BoxFan": 0
    },
    "ZOffset": 0,
    "LightStatus": {
      "SecondLight": 0,
      "RgbLight": [0, 0, 0]
    },
    "PrintInfo": {
      "Status": 0,
      "CurrentLayer": 0,
      "TotalLayer": 0,
      "CurrentTicks": 0,
      "TotalTicks": 0,
      "Filename": "",
      "TaskId": "",
      "PrintSpeedPct": 100,
      "Progress": 0
    }
  }
}
```

## EDIT_PRINTER_STATUS_DATA Subcommands

- **Set nozzle temp:** `{"TempTargetNozzle": 200}`
- **Set bed temp:** `{"TempTargetHotbed": 60}`
- **Set chamber temp:** `{"TempTargetBox": 40}`
- **Set fan speeds:** `{"TargetFanSpeed": {"ModelFan": 100, "AuxiliaryFan": 70, "BoxFan": 50}}`
- **Set light:** `{"LightStatus": {"SecondLight": 1}}`
- **Set print speed:** `{"PrintSpeedPct": 150}`

## EDIT_PRINTER_AXIS_NUMBER Data

```json
{"Axis": "X", "Distance": 10}
```
Axis can be "X", "Y", or "Z". Distance is in mm (positive or negative).

## EDIT_PRINTER_AXIS_ZERO Data

```json
{"AxisList": ["X", "Y", "Z"]}
```

## Print Status Values

| Value | Status |
|-------|--------|
| 0 | Idle / No print |
| 1 | Printing |
| 6 | Paused |
| 8 | Uploading file |
| 9 | Print complete |

## Pitfalls

- WebSocket connection is ephemeral — connect, query, disconnect for one-off commands
- The printer pushes status updates asynchronously; you may receive multiple messages
- `CurrentStatus` is an array, not a single value — check `includes()`
- Fan speeds are 0-100 (percentage), not 0-255
- Temperatures are floats (e.g., 23.61), not ints
- File paths start with `/local/` prefix
- The `RequestID` must be unique per request (use UUID hex)
- Upload uses 1MB chunks via a separate binary protocol
- The printer supports max 4 video streams simultaneously
- When sending `EDIT_PRINTER_STATUS_DATA`, send ALL fan values together, not individually
- The MainboardID is unique per printer — auto-discovered on first connection

---
> Source: [Flatsher/elegoo-centauri-skill](https://github.com/Flatsher/elegoo-centauri-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
