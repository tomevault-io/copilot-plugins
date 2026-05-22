## splitflap

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a split-flap display simulator implemented as a single HTML file (`flipboard.html`). It creates an authentic airport/train station style flip display with realistic animations and sound effects.

## Architecture

### Single-file Application
- **flipboard.html**: Complete standalone application with embedded CSS and JavaScript
- No build process, dependencies, or external files required
- Can be opened directly in any modern web browser

### Core Components

**Display System** (`flipboard.html:430-488`):
- 6 display lines, each with 16 character positions
- Character set: `' ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789:-./`
- Flip animation system with top/bottom halves for realistic mechanical movement

**Animation Engine** (`flipboard.html:442-471`):
- CSS-based 3D flip animations using `rotateX` transforms
- Staggered character updates (80ms delay between characters)
- Visual clicking effects and sound synchronization

**Audio System** (`flipboard.html:331-388`):
- Web Audio API implementation for realistic mechanical clicking sounds
- Multi-oscillator synthesis for complex click sound
- User-controllable sound toggle

**User Interface** (`flipboard.html:304-321`):
- Text input with 16-character limit
- Preset buttons for common airport messages
- Demo mode with automated sequence
- German language interface

## Common Development Tasks

### Testing the Application
```bash
# Open directly in browser
open flipboard.html
# Or serve locally for testing
python3 -m http.server 8001
```

### Key Functions to Understand
- `flipCharacter()` - Core animation logic for individual character flips
- `updateLine()` - Updates entire display line with staggered animations
- `playClickSound()` - Generates realistic mechanical sound effects
- `startDemo()` - Automated demonstration sequence

### Character Animation Timing
- Individual flip duration: 600ms (300ms down + 300ms up)
- Character stagger delay: 80ms between positions
- Line stagger delay: 300ms between lines in multi-line updates

### Display Constraints
- 16 characters per line maximum
- 6 display lines total
- Automatic uppercase conversion and padding
- Only supports defined character set (see CHARACTERS constant)

## API Control

The display is designed for API control with hidden manual controls. Multiple API methods are available:

### HTTP API Server
Start the web server:
```bash
python3 simple_server.py 8001
# or
./start_server.sh
```

**API Endpoints:**
```bash
# Get server status
GET /api/status

# Set display content
POST /api/display
Content-Type: application/json
{
  "line1": "HELLO",
  "line2": "WORLD",
  "line3": "",
  "line4": "",
  "line5": "",
  "line6": ""
}

# Alternative format with array
POST /api/display
Content-Type: application/json
{
  "lines": ["HELLO", "WORLD", "", "", "", ""]
}

# Enable/disable datetime mode
POST /api/datetime
Content-Type: application/json
{
  "enable": true
}

# Clear display
POST /api/clear

# Start demo
POST /api/demo
```

**Examples:**
```bash
# Set display via curl
curl -X POST http://localhost:8001/api/display \
  -H "Content-Type: application/json" \
  -d '{"line1":"FLUGHAFEN MÜNCHEN","line2":"TERMINAL 2","line3":"ABFLÜGE","line4":"LH 441 FRANKFURT","line5":"GATE A15","line6":"12:30 PÜNKTLICH"}'

# Enable datetime mode
curl -X POST http://localhost:8001/api/datetime \
  -H "Content-Type: application/json" \
  -d '{"enable": true}'

# Clear display
curl -X POST http://localhost:8001/api/clear

# Start demo
curl -X POST http://localhost:8001/api/demo

# Set display with per-line colors
curl -X POST http://localhost:8001/api/display \
  -H "Content-Type: application/json" \
  -d '{"line1":"ANKUNFT","line2":"LH 441","line3":"FRANKFURT","line4":"GATE B7","line5":"12:30","line6":"PÜNKTLICH","color1":"blau","color2":"hellblau","color3":"gruen","color4":"orange","color5":"gelb","color6":"gruen"}'

# Alternative array format with colors
curl -X POST http://localhost:8001/api/display \
  -H "Content-Type: application/json" \
  -d '{"lines":["ZEILE 1","ZEILE 2","ZEILE 3","ZEILE 4","ZEILE 5","ZEILE 6"],"colors":["rot","gruen","blau","gelb","orange","violett"]}'
```

### JavaScript API (window.splitflapAPI)
```javascript
// Set all six lines at once
window.splitflapAPI.setDisplay('LINE 1', 'LINE 2', 'LINE 3', 'LINE 4', 'LINE 5', 'LINE 6');

// Set all six lines with colors
window.splitflapAPI.setDisplay(
  'LINE 1', 'LINE 2', 'LINE 3', 'LINE 4', 'LINE 5', 'LINE 6',
  'rot', 'gruen', 'blau', 'gelb', 'orange', 'violett'
);

// Set individual line (0-5)
window.splitflapAPI.setLine(1, 'HELLO WORLD');

// Set individual line with color
window.splitflapAPI.setLine(1, 'HELLO WORLD', 'rot');

// Set color for existing line
window.splitflapAPI.setLineColor(0, 'blau');

// Get available colors
window.splitflapAPI.getColors();
// Returns: ['blau', 'hellblau', 'rot', 'gruen', 'hellgruen', 'orange', 'violett', 'rosa', 'gelb', 'weiss']

// Clear all lines
window.splitflapAPI.clear();

// Start demo sequence
window.splitflapAPI.demo();

// Enable/disable datetime mode
window.splitflapAPI.datetime(true);

// Get current state
window.splitflapAPI.getCurrentDisplay(); // Returns array of current text
window.splitflapAPI.isAnimating(); // Returns boolean

// ====== TEMPLATES ======

// List all available templates
window.splitflapAPI.listTemplates();
// Returns: [{ id: 'airport-arrival', name: 'Airport Arrival', description: '...' }, ...]

// Display using a template
window.splitflapAPI.displayTemplate('airport-departure', {
  airport: 'MÜNCHEN',
  airline: 'LH',
  flightno: '441',
  time: '12:30',
  destination: 'FRANKFURT',
  gate: 'A15',
  status: 'PÜNKTLICH'
});

// Add custom template
window.splitflapAPI.addTemplate('custom-welcome', {
  name: 'Custom Welcome',
  description: 'My custom template',
  lines: [
    'WELCOME {{name}}',
    'TO {{location}}',
    '{{message}}',
    '',
    '',
    ''
  ]
});

// Use custom template
window.splitflapAPI.displayTemplate('custom-welcome', {
  name: 'JOHN',
  location: 'MUNICH',
  message: 'ENJOY YOUR STAY'
});

// Built-in templates:
// - 'airport-arrival': Airport arrival board
// - 'airport-departure': Airport departure board
// - 'train-departure': Train departure board
// - 'welcome': Simple welcome message
// - 'clock': Date and time display

// ====== SCHEDULER ======

// Schedule a task to run at specific time (daily)
window.splitflapAPI.schedule({
  name: 'Morning Welcome',
  type: 'time',
  hour: 8,
  minute: 0,
  action: 'template',
  template: 'welcome',
  data: { line1: 'GUTEN MORGEN', line2: 'GOOD MORNING' }
});

// Schedule task for specific days (0=Sunday, 6=Saturday)
window.splitflapAPI.schedule({
  name: 'Weekend Message',
  type: 'time',
  hour: 10,
  minute: 0,
  days: [0, 6], // Only on weekends
  action: 'template',
  template: 'welcome',
  data: { line1: 'SCHÖNES WOCHENENDE' }
});

// Schedule interval-based task (every 30 minutes)
window.splitflapAPI.schedule({
  name: 'Update Clock',
  type: 'interval',
  interval: 30 * 60 * 1000, // 30 minutes in milliseconds
  action: 'datetime'
});

// Schedule custom action
window.splitflapAPI.schedule({
  name: 'Custom Task',
  type: 'interval',
  interval: 60000, // Every minute
  action: 'custom',
  callback: function() {
    console.log('Custom task executed!');
  }
});

// List all scheduled tasks
window.splitflapAPI.listSchedules();

// Remove a scheduled task
window.splitflapAPI.removeSchedule('task_id');

// Enable/disable a scheduled task
window.splitflapAPI.toggleSchedule('task_id', false); // Disable
window.splitflapAPI.toggleSchedule('task_id', true);  // Enable

// Clear all scheduled tasks
window.splitflapAPI.clearSchedules();

// Manually start/stop scheduler
window.splitflapAPI.startScheduler();
window.splitflapAPI.stopScheduler();
```

### URL Parameters
```
# Set display content
http://localhost:8001/?line1=HELLO&line2=WORLD&line3=&line4=&line5=&line6=

# Enable datetime mode
http://localhost:8001/?datetime

# Start demo
http://localhost:8001/?demo

# Clear display
http://localhost:8001/?clear
```

### PostMessage API (for iframe embedding)
```javascript
// Send commands to iframe
iframe.contentWindow.postMessage({
    action: 'setDisplay',
    data: { line1: 'HELLO', line2: 'WORLD', line3: '', line4: '', line5: '', line6: '' }
}, '*');

// Enable datetime mode
iframe.contentWindow.postMessage({
    action: 'datetime',
    data: { enable: true }
}, '*');

// Get current state
iframe.contentWindow.postMessage({ action: 'getState' }, '*');
```

### Manual Controls Access
Manual controls are hidden by default (`display: none`). To show them for debugging:
```javascript
document.querySelector('.controls').style.display = 'block';
```

## Per-Line Color Control

The display supports setting individual colors for each line. Available colors:

- `blau` - Blue (#4A90E2)
- `hellblau` - Light Blue (#87CEEB)
- `rot` - Red (#E74C3C)
- `gruen` - Green (#2ECC71)
- `hellgruen` - Light Green (#A8E6A1)
- `orange` - Orange (#FF8C42)
- `violett` - Violet (#9B59B6)
- `rosa` - Pink (#FFB6C1)
- `gelb` - Yellow (#F1C40F)
- `weiss` - White (#FFFFFF) - Default color

### HTTP API with Colors
```bash
# Individual line format
curl -X POST http://localhost:8001/api/display \
  -H "Content-Type: application/json" \
  -d '{
    "line1": "ANKUNFT",
    "line2": "LH 441",
    "line3": "FRANKFURT",
    "line4": "GATE B7",
    "line5": "12:30",
    "line6": "PÜNKTLICH",
    "color1": "blau",
    "color2": "hellblau",
    "color3": "gruen",
    "color4": "orange",
    "color5": "gelb",
    "color6": "gruen"
  }'

# Array format
curl -X POST http://localhost:8001/api/display \
  -H "Content-Type: application/json" \
  -d '{
    "lines": ["ZEILE 1", "ZEILE 2", "ZEILE 3", "ZEILE 4", "ZEILE 5", "ZEILE 6"],
    "colors": ["rot", "gruen", "blau", "gelb", "orange", "violett"]
  }'
```

### MQTT with Colors
```bash
mosquitto_pub -h mqtt.example.com -t "splitflap/display" \
  -m '{
    "line1": "ANKUNFT",
    "line2": "LH 441",
    "color1": "blau",
    "color2": "gruen"
  }'
```

### JavaScript API with Colors
```javascript
// Set display with colors
window.splitflapAPI.setDisplay(
  'LINE 1', 'LINE 2', 'LINE 3', 'LINE 4', 'LINE 5', 'LINE 6',
  'rot', 'gruen', 'blau', 'gelb', 'orange', 'violett'
);

// Set individual line with color
window.splitflapAPI.setLine(0, 'HELLO WORLD', 'rot');

// Change color of existing line
window.splitflapAPI.setLineColor(1, 'gruen');

// Get available colors
window.splitflapAPI.getColors();
```

**Notes:**
- Colors are optional - lines without colors use the default white
- Color names are case-insensitive but lowercase is recommended
- Invalid color names default to white
- Fully backward compatible with existing API calls
- See `COLOR_EXAMPLES.md` for more examples and use cases

---
> Source: [whaeuser/splitflap](https://github.com/whaeuser/splitflap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
