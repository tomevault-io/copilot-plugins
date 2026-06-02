## voice-chat-room-web

> - **Frontend**: React 18 + TypeScript + Vite + TailwindCSS + Zustand + mediasoup-client + Socket.io-client

# Project: voice-chat-room

## Stack
- **Frontend**: React 18 + TypeScript + Vite + TailwindCSS + Zustand + mediasoup-client + Socket.io-client
- **Backend**: Node.js + Express + Socket.io + Mediasoup 3.x + Mongoose (+ mongodb-memory-server for dev)
- **Deploy**: Docker + Docker Compose (mongo + backend + nginx)

## Quick Dev
```bash
# Terminal 1 - backend (nodemon auto-restart)
cd backend && npm run dev

# Terminal 2 - frontend (Vite HMR)
cd frontend && npm run dev
```
- Backend: `http://localhost:3001` (health: `/health`)
- Frontend: `https://localhost:5173` or `https://<LAN-IP>:5173` (LAN)

## Production URLs
- Frontend: `https://<your-frontend-domain>`
- Backend API/WS: `https://<your-backend-domain>`
- Server IP: `<your-server-ip>`
- RTP ports: UDP 40000-49999

## Deploy
```bash
git pull && bash deploy.sh
```
- Backend: `node:22-slim` (NOT alpine — mediasoup incompatible with musl)
- Backend binds `127.0.0.1:3001`, nginx binds `127.0.0.1:8080`
- User's own reverse proxy in front of both

## Key Design Decisions
- **No registration**: deviceId via FingerprintJS (fallback to localStorage random ID)
- **Voice-only room grid**: users appear ONLY after clicking "加入语音" (creating a producer)
- **Speaker indicator**: remote speakers use server-side fixed threshold (-55dB via AudioLevelObserver); self uses local noise gate threshold (user-adjustable, default -45dB)
- **Remote audio playback**: native `<audio>` elements (NOT Web Audio API) — more reliable across browsers
- **Voice disconnect**: emits `producer:close` via Socket.io to reliably trigger `USER_LEFT` broadcast
- **DB**: MongoDB for persistent config (channels, bans, settings); Node.js Map for volatile WebRTC state
- **Dev MongoDB**: mongodb-memory-server auto-starts when no MONGODB_URI set

## Architecture Flow
```
login → room:join (no USER_JOINED yet)
  → click "加入语音" → create Producer
    → broadcast USER_JOINED + NEW_PRODUCER
    → other clients consume and play via <audio>
  → click "断开语音" → emit producer:close
    → broadcast USER_LEFT (if last producer)
    → user removed from grid
```

## Common Issues Fixed
- dotenv path: `__dirname` is in `src/config/` → need `../..` to reach backend root
- `store.producer` was storing Transport, not Producer → fixed to capture `transport.produce()` return value
- Stop order: close transport BEFORE producer, otherwise `transportclose` doesn't fire
- AudioContext resume must be awaited
- Windows npm: must use `cmd /c "npm ..."` in PowerShell due to execution policy
- `npx tsc --noEmit` for TS check, `npx vite build` for production build

## Rules for Admin Panel Settings
- **All settings must show current values as defaults** — never show empty/placeholder values when editing
- **Every save action must give user feedback** — use `showToast('xxx已更新', 'success')` after backend ACK
- Settings must be re-fetched from server after save to ensure consistency (`admin:config-getall`)
- Use `useEffect` to sync UI state with current values when the panel opens

---

# Design Specification

## 1. UI Conventions

### 1.1 Colors
| Usage | Class |
|-------|-------|
| Primary accent | `primary-500` (#6366f1), `primary-600` (#4f46e5) |
| Success/green | `green-400`, `green-500` |
| Danger/red | `red-400`, `red-500` |
| Warning/yellow | `yellow-400`, `yellow-500` |
| Panel background | `glass-panel` (semi-transparent + backdrop-blur) |
| Card background | `glass-card` |
| Input background | `bg-gray-800/60` |
| Input border | `border border-gray-600/50` |
| Text primary | `text-white` |
| Text secondary | `text-gray-400` |
| Text muted | `text-gray-500` |
| Placeholder | `placeholder-gray-600` |

### 1.1.1 Theme Design Philosophy
- The appearance system is split into two independent dimensions: **appearance style** and **primary color**.
- Appearance styles currently include `暗夜`, `日光`, `纯粹`, and `深夜`. `暗夜` and `深夜` are dark-mode appearances; `日光` and `纯粹` are light-mode appearances. The default is `暗夜 + 紫色`.
- Primary colors currently include common presets such as `红`, `黄`, `蓝`, `绿`, `青`, `紫`, `粉`, and `橙`, plus `自定义`. Custom colors are stored separately for dark and light modes, because the same hex value may not read well in both modes.
- `暗夜` uses the dark glassmorphism foundation: dark body background, translucent panels/cards, white primary text, light-gray secondary/muted text, and accent-colored glow.
- `日光` keeps the glassmorphism structure but uses warm white or very pale tinted backgrounds with soft accent glows. Primary text must be black or near-black, and secondary/muted text must be dark gray; do not use overly pale gray text in light appearances.
- `纯粹` is a flatter light special case: pure white background, minimal gradients, and doodle-like line borders. It still follows the selected primary color, but the treatment should feel hand-drawn rather than glossy.
- `深夜` is a flatter dark special case: pure black background, minimal gradients, and primary-color glowing borders.
- Inputs, selects, checkboxes, secondary buttons, and small utility controls must remain readable in both theme modes. In the light theme, form controls should be white background with dark text and visible borders. SVG icons should become dark unless they are inside a semantic colored button.
- Primary theme buttons follow the active primary color. In light appearances, gradients should stay bright and warm with a subtle border so buttons do not disappear into the bright background. In `纯粹` and `深夜`, reduce gradients and emphasize the appearance-specific border treatment.
- Semantic action colors override the current theme:
  - Joining a **voice session inside a room** stays green.
  - Disconnecting voice, leaving a room, and logging out stay red, preferably as a softer red/rose gradient rather than a harsh flat red.
  - Joining a **channel card** follows the current theme accent, not the semantic green voice color.
- Status states should be theme-aware: muted microphone uses red tones, muted speaker uses warm amber tones, warnings/notification bars are yellow-family but must be softer and darker-text in the light theme.
- User avatars must carry both dark and light color palettes and switch immediately with `data-theme-mode`; dark appearances use saturated colors, while light appearances use brighter/pastel tones. Speaking glow should also switch with the avatar palette.
- Prefer CSS variables and theme-aware utility classes in `frontend/src/index.css` for cross-cutting theme behavior. Avoid scattering one-off light-theme overrides throughout components unless the behavior is truly component-specific.
- When adding new UI, first decide whether its color is **primary color**, **semantic action/status**, or **neutral surface/text**. Primary color follows the selected color; semantic colors keep their meaning across all appearances; neutral surfaces/text follow dark/light mode readability rules.

### 1.2 Typography
| Element | Class |
|---------|-------|
| Page title | `text-xl font-bold text-white` |
| Section title | `text-lg font-semibold text-white` |
| Subsection heading | `text-sm font-medium text-gray-300` |
| Body text | `text-sm text-white` |
| Label (forms, popovers) | `text-xs text-gray-400` |
| Caption / hint | `text-xs text-gray-500` |
| Slider value | `text-xs text-gray-400 text-right` |
| Error text | `text-red-400 text-xs` |
| Toast text | `text-sm` |

### 1.3 Inputs & Controls
```css
/* Text input */
<input className="w-full bg-gray-800/60 border border-gray-600/50 rounded-lg px-3 py-2 text-sm text-white placeholder-gray-600 focus:outline-none focus:border-primary-500/50 h-8" />

/* Select dropdown */
<select className="w-full bg-gray-800/60 border border-gray-600/50 rounded-lg px-2 h-7 text-xs text-white focus:outline-none focus:border-primary-500/50" />

/* Range slider */
<input type="range" className="flex-1 h-1.5 accent-primary-500 cursor-pointer" />

/* Toggle switch */
<button className="relative w-9 h-5 rounded-full transition-colors {enabled ? 'bg-primary-500' : 'bg-gray-600'}">
  <div className="absolute top-0.5 w-4 h-4 bg-white rounded-full shadow transition-transform {enabled ? 'translate-x-4' : 'translate-x-0.5'}" />
</button>

/* Primary button */
<button className="bg-primary-600 hover:bg-primary-500 disabled:bg-gray-700 disabled:text-gray-500 text-white text-sm py-2.5 rounded-xl" />

/* Secondary button */
<button className="bg-gray-700 hover:bg-gray-600 text-white text-sm py-2.5 rounded-xl" />
```

### 1.4 Spacing & Layout
- Form fields gap: `space-y-3` or `gap-2` in flex rows
- Section gap: `mb-2` between groups, `mb-4` between major sections
- Popover width: `min-w-[240px]` for mic, `min-w-[200px]` for speaker
- Row with label + slider + value: `flex items-center gap-2`
- Label fixed width: `w-10 shrink-0` (Chinese labels), `w-8` for short labels
- Value fixed width: `w-8` for short (st, dB), `w-10` for longer
- Divider line: `border-t border-gray-700/50` with `pt-3` above content
- Modal width: `max-w-sm` for simple, `max-w-lg` for complex forms

### 1.5 StepperInput (special control)
```tsx
<div className="flex items-center rounded-lg border border-gray-600/50 bg-gray-800/60 overflow-hidden h-8">
  <button className="w-7 h-7 shrink-0 ...">-</button>
  <span className="w-px h-4 bg-gray-600/50 shrink-0" />
  <input className="flex-1 min-w-0 h-7 text-center bg-transparent text-sm text-white outline-none" />
  <span className="w-px h-4 bg-gray-600/50 shrink-0" />
  <button className="w-7 h-7 shrink-0 ...">+</button>
</div>
```
Never change StepperInput internal dimensions. Always wrap in container with appropriate width.

### 1.6 Mobile Responsive
- Use `sm:` breakpoint (640px) for desktop/mobile split
- Mobile: controls at bottom (`fixed bottom-0`), popovers slide up (`slide-in-from-bottom-1`)
- Desktop: controls inline, popovers drop down (`slide-in-from-top-1`)
- Grid layout: `grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4` pattern
- Text truncate: `truncate max-w-[80px] sm:max-w-[100px]`

### 1.7 File & Directory Naming
| Type | Convention | Example |
|------|-----------|---------|
| Component | PascalCase, `.tsx` | `UserCard.tsx`, `AudioControls.tsx` |
| Service | camelCase, `.ts` | `audioService.ts`, `socketService.ts` |
| Store | camelCase, `Store` suffix, `.ts` | `roomStore.ts`, `voiceChangerStore.ts` |
| Hook | `use` prefix, camelCase, `.ts` | `useAudioGraph.ts`, `useDevices.ts` |
| Utility | camelCase, `.ts` | `constants.ts`, `voicePresets.ts` |
| Backend model | PascalCase, `.js` | `Channel.js`, `SiteSettings.js` |
| Backend handler | camelCase, `.js` | `adminHandler.js`, `roomHandler.js` |

### 1.8 Import Order
```tsx
// 1. React
import React, { useState, useEffect, useCallback, useRef } from 'react';

// 2. Third-party
import { create } from 'zustand';
import * as Tone from 'tone';

// 3. Stores
import { useRoomStore } from '../../stores/roomStore';

// 4. Services
import { getSocket } from '../../services/socketService';

// 5. Hooks
import { useAudioGraph } from '../../hooks/useAudioGraph';

// 6. Components
import { UserCard } from './UserCard';

// 7. Utils / constants
import { EVENTS } from '../../utils/constants';
```

### 1.9 Custom CSS Classes
These are defined in `frontend/src/index.css`:

```css
@layer components {
  .glass-panel {
    @apply bg-gray-900/60 backdrop-blur-xl border border-gray-700/50 rounded-2xl;
  }
  .glass-card {
    @apply bg-gray-800/40 backdrop-blur-md border border-gray-700/30 rounded-xl;
  }
}
```

**Usage rule**: `glass-panel` for outer containers and modals; `glass-card` for inner cards and setting groups.

Custom animations in `tailwind.config.js`:
| Class | Purpose |
|-------|---------|
| `animate-pulse-ring` | Speaking indicator ring pulse (scale 0.8→2.0, fade 1→0, 1.5s loop) |
| `animate-audio-bar` | Audio bar height oscillation (4px↔20px, 0.5s alternate) |

### 1.10 localStorage Key Convention
All app keys use `vc_` prefix to avoid collisions:
| Key | Type | Purpose |
|-----|------|---------|
| `vc_gain` | string | Mic gain value (float) |
| `vc_muted` | string | Mic muted state (`'true'`/`'false'`) |
| `vc_threshold` | string | Noise gate threshold (int, -60 ~ -30) |
| `vc_denoise_enabled` | string | RNNoise toggle state |
| `vc_random_device_id` | string | Random device ID override |
| `vc_master_volume` | string | Master output volume (float) |
| `vc_voice_changer` | JSON | Voice changer params: `{ presetId, pitch, distortion, filterFreq, filterQ, reverbWet }` |
| `vc_device_id` | string | Fingerprint fallback device ID |
| `vc_selected_input` | string | Selected microphone device ID |
| `vc_selected_output` | string | Selected speaker device ID |
| `vc_sound_settings` | JSON | Sound effect toggles/volumes |
| `vc_remote_volumes` | JSON | Per-device remote output volumes |
| `vc_dismissed_announcements` | JSON | Locally dismissed announcement IDs |
| `vc_admin_pass` | string | Remembered admin password |

**Rules**:
- Prefer reading in store initializer, hook initializer, or service init rather than scattered component reads
- Prefer writing in store action, hook callback, or service setter rather than scattered component writes
- Parse safely: wrap `JSON.parse` and `parseInt` in `try-catch` or provide defaults

---

## 2. Toast / Notification System

### 2.1 Toast (`showToast(message, type)`)
- **types**: `'success'` | `'error'` | `'warning'` | `'info'`
- **When to use**: server ACK received or a user-initiated local action completed
- **Message format**: `'{what}已{action}'` (e.g. '封禁时长已更新', '频道已创建')
- Always Chinese, concise, present-tense

### 2.2 Inline Error (inside modal/form)
- **When to use**: validation errors, server errors during form submission in modals
- Display as `<p className="text-red-400 text-xs">{error}</p>` inside the form
- Modal MUST stay open on error, MUST close on success
- Clear error when reopening modal

### 2.3 Notification Bar (RoomPanel)
- **When to use**: room-level events (kicked, server-muted, announcements)
- Display as fixed yellow bar: `bg-yellow-500/10 border border-yellow-500/30 text-yellow-300`
- Auto-dismiss after 10 seconds
- Use `useRoomStore.setNotification(message)` to set

### 2.4 Golden Rules
1. **Wait for server ACK before toast** — `socket.once(ackEvent, () => showToast(...))`; never show toast before the server responds
2. **Modal errors stay in-modal** — don't use toast for modal form errors; modal stays open on error
3. **One notification per event** — each user-visible notification must have exactly one source; never emit the same toast from two code paths (e.g., both in an ACK handler AND in a broadcast handler)
4. **Use `socket.once()` for ACK listeners** — never `socket.on()` without cleanup; prevents stale listeners
5. **Specific messages** — '封禁时长已更新' not '设置已保存'

---

## 3. Feature Toggle Pattern (Adding a new config flag)

When adding a new toggle-able feature (e.g., `voiceChangerEnabled`):

### 3.1 Backend (4 files)
```
1. models/Channel.js            → add field with default
2. services/configService.js    → add default in DEFAULTS (global flag)
3. handlers/adminHandler.js     → include in channel CRUD + 'user:channel-config' response
4. handlers/userChannelHandler.js → include in user channel create/update
```

### 3.2 Frontend (6 files)
```
1. utils/constants.ts           → add field to Channel type
2. stores/adminStore.ts         → add to AdminConfig interface + default value
3. services/socketService.ts    → add to 'admin:config-list' handler + ROOM_INFO_UPDATED handler
4. admin/AdminPanel.tsx         → add toggle in create/edit forms + global switch in panel
5. lobby modals (if user-created) → add toggle in create/edit modals
6. Consumer component           → read config + conditionally render/hide controls
```

### 3.3 Channel-level field (in Channel model)
- Default should be `true` (opt-out, not opt-in)
- All channel CRUD broadcasts must include the field in `ROOM_INFO_UPDATED`
- `socketService.ts` must include it in ALL branches of `ROOM_INFO_UPDATED` handler

### 3.4 Auto-disable when toggled off
When a channel-level feature is turned off while a user has it active:
```tsx
useEffect(() => {
  if (!featureAllowed && featureEnabled) {
    handleToggle(false);  // turn off + disconnect from audio graph
  }
}, [featureAllowed, featureEnabled, handleToggle]);
```

---

## 4. Audio Pipeline

### 4.1 Signal Chain
```
localAudioSource → micGainNode → [voiceChanger?] → analyserNode
                                                   → [RNNoise?] → gateGainNode → processedDestination
```

### 4.2 All audio routing goes through `reconnectAudioGraph()`
- Called after any toggle: noise suppressor, voice changer, device switch
- Handles all 4 combinations (VC on/off × RNNoise on/off)
- Pattern: `disconnect all → reconnect in correct order`
- Never directly `connect()`/`disconnect()` nodes outside this function

### 4.3 Adding a new audio effect
1. Create `services/effectService.ts` with: init, connect, disconnect, update params, destroy
2. Export a `isReady()` check and a `getOutput()` for graph connection
3. Add branch in `reconnectAudioGraph()`
4. Follow RNNoise/voiceChanger pattern: lazy-init on first use, keep alive during toggles
5. When OFF: nodes fully disconnected from graph (zero CPU)
6. When ON: nodes reconnected without recreating

### 4.4 Tone.js / External Audio Libraries
- Must share the existing `AudioContext` (never create a second one)
- Pattern: `new Tone.Context({ context: getAudioContext() })` then `Tone.setContext(ctx)`
- Never call `Tone.start()` — it creates a conflicting default context
- Init asynchronously with `init().then(() => reconnectAudioGraph())`
- Wrap in try-catch, log warnings on failure

---

## 5. Socket Event Patterns

### 5.1 Request-ACK (admin operations)
```ts
// Backend: socket.emit('admin:channel-created', { roomId, name })  — ACK to sender
// Backend: io.emit('room:info-updated', data)                       — broadcast to all

// Frontend: wait for ACK (see Toast golden rules 1 & 3)
socket.once('admin:channel-created', () => showToast('频道已创建', 'success'));
socket.once(EVENTS.SERVER.ERROR, (data) => {
  if (data.event === EVENTS.CLIENT.ADMIN_CHANNEL_CREATE) showToast(data.message, 'error');
});
socket.emit(EVENTS.CLIENT.ADMIN_CHANNEL_CREATE, payload);
```

### 5.2 Settings Update ACK
```ts
// Backend sends: socket.emit('admin:settings-updated', { key, value })
// Frontend uses: socket.once('admin:settings-updated', handler)
// Then: re-fetch config via socket.emit('admin:config-getall')
```

### 5.3 Broadcast Events
- `ROOM_INFO_UPDATED` — channel create/update/delete → update roomStore
- `ANNOUNCEMENTS_UPDATED` — announcement list change → update roomStore
- `ROOM_LIST` — full channel list → set roomStore.channels
- Broadcast handlers must NOT call `showToast` — only update stores or room-level notification state

### 5.4 Event Naming
- Client→Server: `admin:channel-create`, `user:channel-create`
- Server→Client ACK: `admin:channel-created`, `user:channel-created`
- Server→All broadcast: `room:info-updated`, `announcements:updated`

---

## 6. State Management (Zustand)

### 6.1 Store Files
| Store | Purpose | Key Fields |
|-------|---------|------------|
| `userStore` | User identity, current room | userId, deviceId, nickname, currentRoom |
| `roomStore` | Room/channel data, users, speakers | channels, users, activeSpeakers, announcements |
| `mediaStore` | WebRTC state, audio controls, voice state | producer/consumer, muted state, noise gate threshold |
| `adminStore` | Admin config, ban/kick lists | isAdmin, config, bans, kickedList |
| `voiceChangerStore` | Voice changer params + state | enabled, presetId, pitch, distortion, filter, reverb |

### 6.2 Store Conventions
- Use `create<StateType>((set, get) => ({...}))` pattern
- Computed/derived values as methods: `getParams: () => ({ pitch: get().pitch, ... })`
- Reset function for cleanup: `reset: () => set({ ...initialState })`
- Action names: `setXxx` for setters, verbNoun for actions (`applyPreset`, `toggleMute`)
- localStorage persistence: separate `loadParams()`/`saveParams()` functions outside the store

### 6.3 Using Stores in Components
```tsx
// Selective subscription (prevent unnecessary re-renders)
const enabled = useVoiceChangerStore((s) => s.enabled);

// Get state without subscribing (for callbacks)
useVoiceChangerStore.getState().setEnabled(true);
```

### 6.4 Hooks Pattern
```tsx
// hooks/useExample.ts
import { useRef, useCallback, useState, useEffect } from 'react';
import { someService } from '../services/someService';

export function useExample() {
  // 1. State from localStorage on init
  const [value, setValue] = useState(() => {
    const saved = localStorage.getItem('vc_example');
    return saved ? parseInt(saved) : 0;
  });

  // 2. Refs for animation/timer handles
  const timerRef = useRef<number>(0);

  // 3. Stable callbacks (useCallback with correct deps)
  const update = useCallback((v: number) => {
    setValue(v);
    someService(v);
    localStorage.setItem('vc_example', String(v));
  }, []);

  // 4. rAF / interval loops with cleanup
  useEffect(() => {
    const loop = () => {
      // read from service, update state
      timerRef.current = requestAnimationFrame(loop);
    };
    timerRef.current = requestAnimationFrame(loop);
    return () => cancelAnimationFrame(timerRef.current);
  }, []);

  // 5. Cleanup function
  const cleanup = useCallback(() => {
    cancelAnimationFrame(timerRef.current);
    someService.destroy();
  }, []);

  return { value, update, cleanup };
}
```

**Rules**:
- Hooks are the ONLY place where React state and service layer meet
- Components consume hooks; hooks consume services/stores
- Never call service functions directly from JSX event handlers — route through a hook callback
- `useCallback` on every function returned to components; list correct deps
- `useRef` for non-render-critical mutable values (timers, animation frames)

---

## 7. Service Layer

### 7.1 File Structure
```
frontend/src/services/
├── audioService.ts       # Audio graph, noise gate, remote audio
├── rnnoiseService.ts     # RNNoise WASM loader and node management
├── voiceChangerService.ts # tone.js effect chain
├── socketService.ts      # ALL socket.io event handlers + connection
├── soundService.ts       # Sound effect playback
└── mediasoupService.ts   # mediasoup device management
```

### 7.2 Service Conventions
- Module-level variables for state (NOT React state): `let audioContext = null;`
- Export functions, not classes
- Functions should not depend on React — call stores via `useXxxStore.getState()`
- Init functions should be idempotent (check `if (chainReady) return;`)
- Destroy functions nullify all module-level variables

---

## 8. Component Patterns

### 8.1 Modal
```tsx
<div className="fixed inset-0 z-50 flex items-center justify-center bg-black/60 backdrop-blur-sm">
  <div className="glass-panel p-5 w-full max-w-sm mx-4 animate-in zoom-in-95 fade-in duration-200">
    <h3 className="text-lg font-semibold text-white mb-4">标题</h3>
    <div className="space-y-3">
      {/* form fields */}
      {error && <p className="text-red-400 text-xs">{error}</p>}
      <div className="flex gap-2 pt-2">
        <button onClick={onClose} disabled={saving} className="flex-1 bg-gray-700 ...">取消</button>
        <button onClick={handleSave} disabled={...} className="flex-1 bg-primary-600 ...">
          {saving ? '保存中...' : '保存'}
        </button>
      </div>
    </div>
  </div>
</div>
```

### 8.2 Popover (AudioControls pattern)
- Triggered by button hover (not click) with 200ms close delay on mouse leave
- Uses `createPortal` to `document.body`
- Positioned relative to anchor button via `getBoundingClientRect()`
- `onMouseEnter` on popover div keeps it open
- Handle: `enterPopup()` → clear close timer, `leavePopup()` → start 200ms close timer

### 8.3 Admin Panel
- Fixed overlay: `fixed inset-0 z-40 pt-16 bg-black/40`
- Glass panel: `glass-panel max-w-2xl max-h-[80vh] flex flex-col`
- Sticky header with close button
- Tab bar: `('channels'|'announcement'|'bans'|'settings'|'userchannels')`
- Scrollable content area: `flex-1 overflow-y-auto min-h-0 p-5`
- Settings cards: `glass-card p-4 space-y-3`
- Save button always right-aligned with input

---

## 9. Backend Conventions

### 9.1 Handler Pattern
```js
socket.on(EVENT_NAME, async (data) => {
  // 1. Auth check
  if (!isAdmin(socket.id)) return;
  
  // 2. Validate
  if (!data.field) return;
  
  // 3. Execute
  try {
    const result = await someService(data);
    socket.emit('ack:event-name', result);      // ACK to sender
    io.emit('broadcast:event', enrichedData);    // Broadcast to all
  } catch (e) {
    socket.emit(EVENTS.SERVER.ERROR, { event: EVENT_NAME, message: '可读的错误信息' });
  }
});
```

### 9.2 Service Functions
- Export as named functions via `module.exports = { fn1, fn2, ... }`
- Lean results: `.lean()` on Mongoose queries
- Default values: `data.field ?? defaultValue`
- Boolean defaults: `data.field !== false` (so `undefined` → `true`)

### 9.3 Channel Operations
- All channel CRUD must broadcast `ROOM_INFO_UPDATED` to `io`
- Include ALL relevant fields (new fields too, don't forget)
- Send dedicated ACK to admin socket: `socket.emit('admin:channel-created', ...)`

---

## 10. Error Handling Tiers

| Level | Where | When | Example |
|-------|-------|------|---------|
| `console.warn` | Service init, non-critical | Feature unavailable but app works without it | RNNoise failed to load, tone.js init failed |
| `console.error` | Unexpected failures | Something that should never fail | `try-catch` around `JSON.parse` for our own stored data |
| `throw` | Service/backend functions | Let caller decide how to handle | Business logic violation, unprocessable input |
| Toast `'error'` | User-facing operation | User needs to know and retry | "创建失败", "无法加入语音" |
| Inline error | Modal forms | Validation or server rejection during form submit | "频道名已存在", "密码错误" |
| Notification bar | Room-level events | Broadcast to all room members | "你已被踢出", server mute |

**Rules**:
- Services NEVER call `showToast` — they either return an error or log a warning
- Stores NEVER call `showToast` — notification is a UI concern
- Only Component handlers and Hooks call `showToast`
- Backend handlers: `console.error` + emit `EVENTS.SERVER.ERROR` with user-readable `message`

---

## 11. File Checklist for New Features

Before committing, verify:
- [ ] `npx tsc --noEmit` passes (frontend)
- [ ] `node --check` passes for ALL modified backend files
- [ ] No unused imports
- [ ] No `any` types where avoidable
- [ ] Toast messages specific and in Chinese
- [ ] localStorage keys prefixed with `vc_`
- [ ] Feature toggle defaults to `true` (not `false`)
- [ ] Both admin and regular user paths tested
- [ ] Dev defaults relaxed (via `isDev` check in config service)

---
> Source: [barryu9/Voice_Chat_Room_Web](https://github.com/barryu9/Voice_Chat_Room_Web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
