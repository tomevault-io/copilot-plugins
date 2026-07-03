## sunseeker-app

> Build a PebbleOS watchapp called **SunSeeker** that helps users locate where the sun is rising and setting. The app uses the compass sensor to show real-time directional guidance - point your wrist toward sunrise or sunset. It combines GPS location data (from the paired phone), the watch's magnetometer/compass, time of day, and a solar position algorithm to give you a live compass view with sunrise/sunset bearings.

# SunSeeker - PebbleOS Sun Tracker App

## Project Overview

Build a PebbleOS watchapp called **SunSeeker** that helps users locate where the sun is rising and setting. The app uses the compass sensor to show real-time directional guidance - point your wrist toward sunrise or sunset. It combines GPS location data (from the paired phone), the watch's magnetometer/compass, time of day, and a solar position algorithm to give you a live compass view with sunrise/sunset bearings.

## Target SDK

Use the **Alloy** JavaScript SDK (the new Pebble JS framework powered by Moddable, released Feb 2026). Do NOT use the old C SDK or Rocky.js. Alloy lets you write the entire app in modern JavaScript (ES2025) that runs natively on the watch via the XS engine.

Key Alloy references:
- Official examples: https://github.com/Moddable-OpenSource/pebble-examples
- Alloy docs: https://developer.repebble.com/guides/alloy/
- Sensor APIs follow the ECMA-419 Sensor Class Pattern
- UI uses the **Piu** framework (declarative, component-based) or **Poco** (low-level graphics)
- CloudPebble (browser IDE): https://cloudpebble.repebble.com/

## Target Platforms

- **emery** (Pebble Time 2) - 200×228 px, color, compass sensor
- **gabbro** (Pebble Round 2) - 260×260 px, color, round display

The app should work on both. Use `screen` from `pebble` to detect dimensions.

## Architecture

### Module Structure

```
sunseeker/
├── package.json                    # Pebble project config
├── src/
│   ├── embeddedjs/
│   │   ├── manifest.json           # Alloy build manifest
│   │   ├── main.js                 # App entry point, UI, sensor wiring
│   │   └── sun.js                  # Solar position calculator (pure math, no Pebble deps)
│   └── pkjs/
│       └── index.js                # PebbleKit JS (phone-side, for location proxy)
├── tests/
│   └── sun.test.js                 # Unit tests for solar calculator (runs in Node.js)
└── CLAUDE.md                       # This file
```

### 1. Solar Position Calculator (`sun.js`)

This module must be **pure math with zero Pebble dependencies** so it can be unit-tested in Node.js.

Export these functions:

#### `sunTimes(lat, lon, date) → { sunrise: Date, sunset: Date, polarDay: bool, polarNight: bool }`
Calculate sunrise and sunset times for a given location and date.
- Use the standard solar position equations (Julian day, solar declination, hour angle)
- Standard zenith of 90.833° (accounts for atmospheric refraction + solar disc radius)
- Handle polar edge cases (midnight sun / polar night) gracefully
- Return local times adjusted for timezone

#### `sunAzimuth(lat, lon, date) → { sunrise: number, sunset: number, current: number, elevation: number }`
Calculate the compass bearing (0-360°, 0=North, 90=East) for:
- Sunrise azimuth (the bearing where the sun rose/will rise)
- Sunset azimuth (the bearing where the sun sets/will set)
- Current sun azimuth (where the sun is right now)
- Current sun elevation (degrees above/below horizon, negative = below)

#### `solarNoon(lat, lon, date) → Date`
Return the time of solar noon for the location.

#### `dayLength(lat, lon, date) → number`
Return day length in minutes.

Algorithm notes:
- Julian Day from calendar date
- Solar mean anomaly and ecliptic longitude
- Solar declination from obliquity of the ecliptic
- Hour angle for sunrise/sunset from latitude + declination
- Azimuth from hour angle + declination + latitude
- Equation of time for solar noon correction

### 2. Main App (`main.js`)

#### Sensors (ECMA-419 pattern)
All Pebble sensors use the ECMA-419 Sensor Class Pattern. Reference the examples at:
- `hellolocation/main.js` for GPS location
- `piu/apps/compass/main.js` for compass heading

```js
// Compass - heading in degrees from magnetic north
import Compass from "pebble/sensor/Compass";
const compass = new Compass();
compass.onreading = function() {
    const heading = compass.heading; // degrees, 0-360
};
compass.start();

// Location - GPS from paired phone
import Location from "pebble/sensor/Location";
const location = new Location();
location.onreading = function() {
    const lat = location.latitude;
    const lon = location.longitude;
};
location.start();
```

**Important**: These import paths are based on the Alloy examples. Before writing the code, check the actual examples at `piu/apps/compass` and `hellolocation` for the correct import syntax. The sensor pattern uses `.onreading` callbacks and `.start()` / `.stop()` methods per ECMA-419.

#### UI Design (Piu framework)

The app should show a **compass rose view** with:

1. **Compass ring** - rotating ring showing N/S/E/W that moves with the watch's compass heading, so North always points to actual north
2. **Sun position indicators**:
   - A sunrise marker (☀ or a wedge/arc in warm yellow/orange) drawn at the sunrise azimuth bearing
   - A sunset marker (similar, in deeper orange/red) drawn at the sunset azimuth bearing
   - These markers rotate with the compass ring so they always point to the correct real-world direction
3. **Fixed pointer** - a triangle or line at the 12 o'clock position (top of screen) indicating "the direction you're facing"
4. **Info panel** at the bottom showing:
   - Sunrise time (e.g., "↑ 05:23")
   - Sunset time (e.g., "↓ 21:14")
   - Current sun elevation (e.g., "38° up" or "below horizon")
   - Optional: time until next sunrise or sunset

Color scheme:
- Background: dark (black or very dark blue)
- Compass ring/text: white or light gray
- Sunrise marker: warm yellow (#FFD700 or similar)
- Sunset marker: deep orange/red (#FF6347 or similar)
- Current sun position: bright yellow dot if above horizon
- Use Pebble's built-in Gothic or Leco fonts for text

#### Button Interactions
- **Up button**: Toggle between compass view and detail view (shows lat/lon, day length, solar noon, next event countdown)
- **Select button**: Force-refresh location from phone GPS
- **Down button**: Toggle between today's data and tomorrow's preview
- **Back button**: Exit app (default Pebble behavior)

#### State Management
- Store last-known location in `localStorage` so the app works briefly without phone connection
- Update location every 10 minutes (GPS is battery-intensive on the phone)
- Update compass heading continuously (use the sensor's default rate)
- Recalculate sun positions whenever location updates or date changes

### 3. PebbleKit JS (`src/pkjs/index.js`)

The phone-side JavaScript handles the location proxy for Alloy's networking. If the Alloy `Location` sensor handles this natively (check the `hellolocation` example), this file may just need the standard proxy setup:

```js
import "@moddable/pebbleproxy";
```

Or if location needs manual handling via PebbleKit JS geolocation:

```js
Pebble.addEventListener("ready", function() {
    console.log("SunSeeker PKJS ready");
    // Send initial location
    getLocation();
});

function getLocation() {
    navigator.geolocation.getCurrentPosition(
        function(pos) {
            Pebble.sendAppMessage({
                lat: Math.round(pos.coords.latitude * 10000),
                lon: Math.round(pos.coords.longitude * 10000)
            });
        },
        function(err) {
            console.log("Location error: " + err.message);
        },
        { enableHighAccuracy: true, maximumAge: 300000, timeout: 15000 }
    );
}
```

Check the `hellolocation` example to see which approach Alloy uses. The ECMA-419 Location sensor likely handles this automatically via the proxy.

## Testing Strategy

### Unit Tests (`tests/sun.test.js`)

The solar calculator is pure math and can be tested in plain Node.js without any Pebble SDK. Write thorough tests:

```
node tests/sun.test.js
```

Test cases to cover:

1. **Known sunrise/sunset times** - Verify against published data:
   - Copenhagen (55.68°N, 12.57°E) summer solstice 2026: sunrise ~04:25, sunset ~21:57
   - Copenhagen winter solstice: sunrise ~08:37, sunset ~15:38
   - New York (40.71°N, -74.01°W) equinox: sunrise ~06:00, sunset ~18:00
   - Sydney (-33.87°S, 151.21°E) - southern hemisphere verification
   - Allow ±3 minute tolerance for simplified algorithm

2. **Solar azimuth known values**:
   - At equinox, sunrise azimuth ≈ 90° (East) and sunset ≈ 270° (West) at all latitudes
   - Summer solstice at 55°N: sunrise azimuth should be well north of east (~40-50°)
   - Solar noon azimuth should be ~180° (due South) in northern hemisphere

3. **Polar edge cases**:
   - Tromsø (69.65°N) in June: should return polarDay = true
   - Tromsø in December: should return polarNight = true
   - Just south of Arctic Circle in June: should return very long but finite day

4. **Day length**:
   - Equinox at equator: ~12 hours
   - Summer solstice at 55°N: ~17.5 hours

5. **Current sun position**:
   - Solar noon: azimuth ≈ 180° (south), elevation = (90 - lat + declination)
   - Midnight: elevation should be negative

6. **Edge cases**:
   - Date boundary (midnight)
   - Longitude ±180° (date line area)
   - Equator (lat = 0)

Use a simple assert-based test runner (no external deps needed):

```js
function assertApprox(actual, expected, tolerance, label) {
    const diff = Math.abs(actual - expected);
    if (diff > tolerance) {
        console.error(`FAIL: ${label} - expected ${expected} ±${tolerance}, got ${actual}`);
        process.exitCode = 1;
    } else {
        console.log(`PASS: ${label}`);
    }
}
```

### Emulator Testing

Use the Pebble emulator for visual/integration testing:

```bash
pebble build
pebble install --emulator emery
```

Feed compass data to the emulator:
```bash
pebble emu-compass --heading 45 --calibrated
pebble emu-compass --heading 180 --calibrated
```

Test the compass ring rotation by sweeping headings:
```bash
for h in 0 45 90 135 180 225 270 315; do
    pebble emu-compass --heading $h --calibrated
    sleep 2
done
```

### On-Device Testing

Once you have a physical Pebble Time 2 or Round 2:
```bash
pebble install --phone <IP_ADDRESS>
```

Verify:
- Compass ring tracks correctly when rotating wrist
- Sunrise/sunset markers point to the correct real-world directions
- Location updates work via phone GPS
- App handles phone disconnect gracefully (uses cached location)
- Battery impact is acceptable

## Development Setup

### Option A: Local SDK

```bash
# Install Pebble tool
uv tool install pebble-tool --python 3.13

# Install latest SDK
pebble sdk install latest

# Create project (or use existing files)
pebble new-project --alloy sunseeker

# Build and run
pebble build
pebble install --emulator emery
```

### Option B: CloudPebble

Go to https://cloudpebble.repebble.com/, create a new Alloy project, and paste the source files in. CloudPebble has a built-in emulator.

### Option C: Docker (for CI / headless)

See https://github.com/FBarrca/pebble-devcontainer for a Docker-based dev environment.

## Implementation Order

1. **Start with `sun.js`** - Write the solar calculator as a pure ES module. Test it in Node.js until all test cases pass. This is the algorithmic core and must be solid.

2. **Write tests** - Create `tests/sun.test.js` and validate against known astronomical data before touching any Pebble code.

3. **Scaffold the Alloy app** - Get a minimal `main.js` running in the emulator that just shows "Hello" with the Piu framework. Confirm the build pipeline works.

4. **Add compass** - Wire up the compass sensor and draw a basic rotating compass ring. Test with `pebble emu-compass`.

5. **Add location** - Wire up the Location sensor (or PebbleKit JS geolocation). Display lat/lon on screen to confirm it works.

6. **Integrate sun calculations** - Connect location + time to `sun.js`, draw sunrise/sunset markers on the compass ring.

7. **Polish UI** - Add the info panel, button interactions, detail view, smooth animations, localStorage caching.

8. **Test on Round 2** - Verify the layout works on the round 260×260 gabbro display.

## Key Pebble/Alloy Gotchas

- **No CommonJS** - Use ES module `import`/`export` syntax only. No `require()`.
- **No `eval` or `Function`** - JS is precompiled to bytecode at build time.
- **Memory is tight** - Minimize number of modules. Keep data structures lean.
- **Compass orientation** - Best readings when the watch face is parallel to the ground (arm extended flat), not tilted toward the user.
- **Compass calibration** - Handle the calibration state. Show a message if the compass needs calibrating (user needs to rotate their wrist in a figure-8).
- **Location battery impact** - Don't poll GPS too frequently. Cache aggressively.
- **Hardened JavaScript** - All primordials are immutable. No monkey-patching.
- **Fonts** - Use Pebble built-in fonts (Gothic, Leco, Bitham, etc.). Custom fonts are possible via bmfont but add binary size.
- **Colors** - Pebble Time 2 has a 64-color display. Use `rgb()` or `hsl()` Piu globals. Check the Pebble color guide for available values.

## Reference: Example Alloy Sensor Code Pattern

From the Moddable pebble-examples repo, sensors follow this pattern:

```js
// Generic ECMA-419 sensor pattern
const sensor = new SensorClass({
    // optional config
});

sensor.onreading = function() {
    // access sensor.propertyName for readings
};

sensor.onerror = function(err) {
    // handle errors
};

sensor.start();
// later: sensor.stop();
```

Study these specific examples before implementing:
- `piu/apps/compass/` - Full compass visualization with Piu
- `hellolocation/` - Location sensor usage
- `piu/watchfaces/zurich/` - Watchface with rotating SVG hands (good reference for rotating elements)
- `hellopiu-port/` - Custom drawing with Piu Port (for the compass ring)
- `hellolocalstorage/` - Persisting data between launches

---
> Source: [djeppers/sunseeker_app](https://github.com/djeppers/sunseeker_app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->
