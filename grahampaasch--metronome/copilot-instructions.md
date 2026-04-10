## metronome

> This document describes the autonomous components (agents) within the metronome project. Each agent encapsulates a specific concern and exposes configuration parameters, capabilities, and usage examples.

# Agents

This document describes the autonomous components (agents) within the metronome project. Each agent encapsulates a specific concern and exposes configuration parameters, capabilities, and usage examples.

---

## TempoAgent

**Purpose**: Generate beat events at the configured tempo and time signature.

**Key Capabilities**:

* Start, stop, pause metronome.
* Adjust BPM dynamically.
* Support time signature changes and subdivisions.
* Emit `beat` and `subbeat` events.

**Configuration Parameters**:

* `bpm` (Number) – Beats per minute, e.g., `120`.
* `timeSignature` (Array\<Number, Number>) – `[beatsPerBar, noteValue]`, e.g., `[4, 4]`.
* `subdivisions` (Number) – Number of ticks per beat, e.g., `4`.

**Usage Example**:

```js
import { TempoAgent } from './agents/TempoAgent';

const tempoAgent = new TempoAgent({ bpm: 100, timeSignature: [4, 4], subdivisions: 2 });
tempoAgent.on('beat', () => console.log('Beat'));
tempoAgent.on('subbeat', () => console.log('Tick'));
tempoAgent.start();
```

---

## AudioAgent

**Purpose**: Produce audible click sounds on beat and subdivision.

**Key Capabilities**:

* Load and play custom audio files.
* Route sound output to selected audio device.
* Adjust volume level.
* Distinguish primary beats from subdivisions.

**Configuration Parameters**:

* `primaryClickUrl` (String) – URL or path to primary beat sound.
* `subdivisionClickUrl` (String) – URL or path to subdivision click sound.
* `volume` (Number) – Range `0.0` to `1.0`.

**Usage Example**:

```js
import { AudioAgent } from './agents/AudioAgent';

const audioAgent = new AudioAgent({
  primaryClickUrl: '/sounds/primary.wav',
  subdivisionClickUrl: '/sounds/sub.wav',
  volume: 0.8,
});
// Connect to TempoAgent events
tempoAgent.on('beat', () => audioAgent.playPrimary());
tempoAgent.on('subbeat', () => audioAgent.playSubdivision());
```

---

## VisualAgent

**Purpose**: Render visual cues for beats and subdivisions.

**Key Capabilities**:

* Flash or animate UI elements on beat.
* Provide adjustable visual styles (colors, shapes).
* Sync visuals precisely with audio events.

**Configuration Parameters**:

* `elementSelector` (String) – CSS selector for UI container.
* `beatStyle` (Object) – CSS properties for primary beat flash.
* `subbeatStyle` (Object) – CSS properties for subdivision flash.

**Usage Example**:

```js
import { VisualAgent } from './agents/VisualAgent';

const visualAgent = new VisualAgent({
  elementSelector: '#flash-dot',
  beatStyle: { transform: 'scale(1.5)' },
  subbeatStyle: { opacity: 0.5 },
});
tempoAgent.on('beat', () => visualAgent.flashBeat());
tempoAgent.on('subbeat', () => visualAgent.flashSubbeat());
```

---

## InputAgent

**Purpose**: Handle user interactions and configuration changes.

**Key Capabilities**:

* Listen to UI controls (buttons, sliders, form inputs).
* Validate and propagate configuration updates.
* Expose event hooks for custom UI integration.

**Configuration Parameters**:

* `controlSelectors` (Object) – Map of control names to CSS selectors.

**Usage Example**:

```js
import { InputAgent } from './agents/InputAgent';

const inputAgent = new InputAgent({
  controlSelectors: {
    bpmSlider: '#bpm-slider',
    startButton: '#start-btn',
    stopButton: '#stop-btn',
  },
});
inputAgent.on('bpmChange', newBpm => tempoAgent.setBpm(newBpm));
inputAgent.on('start', () => tempoAgent.start());
inputAgent.on('stop', () => tempoAgent.stop());
```

---

## PersistenceAgent

**Purpose**: Save and load user settings.

**Key Capabilities**:

* Store settings in `localStorage` or external API.
* Retrieve and validate persisted configuration.
* Support export/import of presets (JSON).

**Configuration Parameters**:

* `storageKey` (String) – Key under which to save settings.
* `useRemote` (Boolean) – Toggle remote API persistence.

**Usage Example**:

```js
import { PersistenceAgent } from './agents/PersistenceAgent';

const persistenceAgent = new PersistenceAgent({ storageKey: 'metronomeSettings' });
// On app load
document.addEventListener('DOMContentLoaded', () => {
  const settings = persistenceAgent.load();
  tempoAgent.applySettings(settings);
});
// On settings change
inputAgent.on('settingsChange', newSettings => persistenceAgent.save(newSettings));
```

---

## LoggerAgent

**Purpose**: Record events and errors for debugging and metrics.

**Key Capabilities**:

* Log events to console or remote server.
* Categorize logs by level (`info`, `warn`, `error`).
* Optionally batch and transmit metrics.

**Configuration Parameters**:

* `level` (String) – Minimum log level.
* `remoteEndpoint` (String) – URL for remote logging API.

**Usage Example**:

```js
import { LoggerAgent } from './agents/LoggerAgent';

const logger = new LoggerAgent({ level: 'info', remoteEndpoint: '' });
tempoAgent.on('beat', () => logger.info('Beat event'));
logger.error('Failed to load audio file');
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GrahamPaasch)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/GrahamPaasch)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
