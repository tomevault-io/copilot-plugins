## frontend-guidline

> description: Frontend Guideline

---
description: Frontend Guideline
globs: 
---
# DevHangout: Frontend Development Guidelines

## 1. Architecture Overview

DevHangout's frontend is a real-time, interactive application focused on delivering a seamless spatial experience for developers. The architecture follows a component-based approach with specialized systems for rendering, interaction, and communication.

### 1.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  Application Shell                       │
├─────────────┬─────────────────────────┬─────────────────┤
│ Auth System │      Routing System     │   UI Library    │
├─────────────┴─────────────────────────┴─────────────────┤
│                      Core Systems                        │
├───────────┬──────────┬───────────┬──────────┬───────────┤
│ Rendering │ Movement │ Collision │  Input   │ Animation │
├───────────┴──────────┼───────────┼──────────┴───────────┤
│ Audio/Voice System   │ Network   │ Integration System   │
├────────────────────┬─┴───────────┴─┬────────────────────┤
│  Component Library │  State System  │    Asset System   │
└────────────────────┴────────────────┴────────────────────┘
```

### 1.2 Core Frontend Systems

1. **Rendering System** - Canvas/WebGL rendering for the virtual space
2. **Movement System** - Character movement and physics
3. **Input System** - Keyboard, mouse, and touch input handling
4. **Audio System** - WebRTC integration and spatial audio
5. **Network System** - WebSocket communication and state synchronization
6. **UI System** - Component library and overlay UI
7. **Asset System** - Loading and managing graphical assets

## 2. Technology Stack

### 2.1 Primary Technologies

| Component | Technology |
|-----------|------------|
| Framework | React |
| State Management | Redux Toolkit |
| Rendering | PixiJS (for 2D retro rendering) |
| Styling | CSS Modules with custom retro theming |
| UI Components | Custom 8-bit inspired component library |
| Networking | Socket.io client |
| Audio/Video | WebRTC with simple-peer |
| Build System | Vite |
| Testing | Vitest, React Testing Library |
| Code Quality | ESLint, Prettier |

### 2.2 Justification

- **React**: Provides component-based architecture and efficient rendering
- **Redux Toolkit**: For global state management with built-in immutability
- **PixiJS**: High-performance 2D WebGL renderer with good TypeScript support
- **Tailwind CSS**: Utility-first CSS for rapid UI development
- **Socket.io**: Reliable WebSocket implementation with fallbacks
- **Vite**: Fast development and optimized production builds

## 3. Project Structure

### 3.1 Directory Structure

```
src/
├── assets/            # Static assets (images, fonts, etc.)
├── components/        # Reusable React components
│   ├── ui/            # Basic UI components
│   ├── avatar/        # Avatar-related components
│   ├── space/         # Virtual space components
│   └── integration/   # Integration-related components
├── systems/           # Core functionality modules
│   ├── rendering/     # Canvas/WebGL rendering logic
│   ├── movement/      # Character movement and physics
│   ├── audio/         # Voice chat and spatial audio
│   ├── network/       # WebSocket and API communication
│   └── interaction/   # Object interaction logic
├── state/             # Redux state management
│   ├── slices/        # Redux Toolkit slices
│   ├── selectors/     # Memoized selectors
│   └── middleware/    # Custom Redux middleware
├── hooks/             # Custom React hooks
├── utils/             # Utility functions
├── pages/             # Full page components
├── routes/            # Routing configuration
├── api/               # API client functions
├── types/             # TypeScript type definitions
├── config/            # Configuration files
├── styles/            # Global styles and Tailwind config
└── App.tsx            # Application entry point
```

### 3.2 Component Structure

Components should follow a consistent directory structure:

```
ComponentName/
├── index.ts           # Exports the component
├── ComponentName.tsx  # Main component file
├── ComponentName.test.tsx  # Tests
├── ComponentName.module.css  # Component-specific styles (if needed)
└── hooks/             # Component-specific hooks
```

## 4. Rendering Architecture

### 4.1 Canvas Rendering

DevHangout uses a layered canvas approach:

```
┌─────────────────────────────────────────────┐
│               Overlay UI Layer               │ React DOM
└─────────────────────────────────────────────┘
┌─────────────────────────────────────────────┐
│               Interactive UI Layer           │ PixiJS
└─────────────────────────────────────────────┘
┌─────────────────────────────────────────────┐
│               Characters Layer               │ PixiJS
└─────────────────────────────────────────────┘
┌─────────────────────────────────────────────┐
│            Interactive Objects Layer         │ PixiJS
└─────────────────────────────────────────────┘
┌─────────────────────────────────────────────┐
│               Background Layer               │ PixiJS
└─────────────────────────────────────────────┘
```

### 4.2 Rendering Loop

The rendering system should implement:

1. A fixed timestep for physics and movement (60 FPS)
2. Interpolation for smooth movement between network updates
3. Dynamic optimization based on device capability
4. Camera system with panning, zooming, and following

```typescript
// Pseudocode for the render loop
class Renderer {
  private lastTime: number = 0;
  private accumulator: number = 0;
  private readonly FIXED_TIMESTEP: number = 1000 / 60;

  public start(): void {
    this.lastTime = performance.now();
    this.loop();
  }

  private loop(): void {
    const currentTime = performance.now();
    const deltaTime = currentTime - this.lastTime;
    this.lastTime = currentTime;
    
    this.accumulator += deltaTime;
    
    // Update physics at fixed timestep
    while (this.accumulator >= this.FIXED_TIMESTEP) {
      this.update(this.FIXED_TIMESTEP);
      this.accumulator -= this.FIXED_TIMESTEP;
    }
    
    // Render with interpolation
    const alpha = this.accumulator / this.FIXED_TIMESTEP;
    this.render(alpha);
    
    requestAnimationFrame(() => this.loop());
  }
  
  private update(timestep: number): void {
    // Update game state, physics, etc.
  }
  
  private render(interpolation: number): void {
    // Render with interpolation between states
  }
}
```

### 4.3 Asset Management

- Implement progressive loading for map assets
- Use texture atlases for character sprites
- Preload common assets during initial loading
- Implement asset streaming for large spaces
- Cache assets in IndexedDB for faster reloading

## 5. User Interface Guidelines

### 5.1 Retro Game Design System

DevHangout should implement a consistent 1980s retro game design system with:

- Pixel-perfect typography using "Press Start 2P" and "VCR OSD Mono" fonts
- Neon color palette (CRT Blue, Magenta, Neon Green, 8-bit Yellow, Digital Red)
- 8px grid-based spacing and sizing system
- 8-bit style component variants with pixel art borders
- Accessibility considerations with toggle options for effects
- Dark mode as default with "arcade cabinet" and "terminal green" themes

### 5.2 UI Component Structure

Split UI into three arcade-inspired categories:

1. **Arcade Controls** - Basic elements (pixel buttons, terminal inputs, game cards)
2. **Game Screens** - Complex, reusable components (dialog boxes, 8-bit menus, inventory screens)
3. **Power-Ups** - Specific feature interfaces (character profile cards, retro code editors, map screens)

### 5.3 UI Layer Integration

The UI should be integrated with the pixel game world through:

1. Position mapping between world and screen coordinates using 8px grid alignment
2. 8-bit speech bubble and pixel art nametags for characters
3. Screen wipe and pixelation transitions between spatial and dialog box UI
4. Context-sensitive game controls that appear based on proximity and interactions

```typescript
// Example of world-to-screen coordinate mapping
function worldToScreen(
  worldX: number, 
  worldY: number,
  camera: Camera
): { x: number, y: number } {
  return {
    x: (worldX - camera.x) * camera.zoom + viewport.width / 2,
    y: (worldY - camera.y) * camera.zoom + viewport.height / 2
  };
}
```

## 6. Avatar System

### 6.1 8-Bit Character Representation

- Use limited frame sprite-based characters (4-8 frames per animation)
- Implement 4-directional pixel art character rendering
- Support customization with limited palette color swaps and pixel accessories
- Implement chunky movement with subtle pixel-perfect interpolation
- Add simple looping idle animations and interaction states (2-4 frames)

### 6.2 Arcade Movement Mechanics

- Implement D-pad style keyboard (WASD/Arrow) movement as primary control
- Add pixel-perfect collision detection with environmental objects
- Implement grid-based pathfinding for click-to-move with visible "target" indicator
- Add 8-bit style movement with minimal acceleration/deceleration (2-3 frames)
- Support "Press X to interact" style proximity interactions with visual prompts

```typescript
// Example of basic character movement logic
function moveCharacter(character, input, deltaTime, collisionMap) {
  // Calculate desired velocity from input
  const desiredVelocityX = input.horizontal * character.maxSpeed;
  const desiredVelocityY = input.vertical * character.maxSpeed;
  
  // Apply acceleration
  character.velocityX = lerp(
    character.velocityX, 
    desiredVelocityX, 
    character.acceleration * deltaTime
  );
  
  character.velocityY = lerp(
    character.velocityY, 
    desiredVelocityY, 
    character.acceleration * deltaTime
  );
  
  // Calculate new position
  const newX = character.x + character.velocityX * deltaTime;
  const newY = character.y + character.velocityY * deltaTime;
  
  // Check collision and update position
  if (!checkCollision(newX, character.y, collisionMap)) {
    character.x = newX;
  } else {
    character.velocityX = 0;
  }
  
  if (!checkCollision(character.x, newY, collisionMap)) {
    character.y = newY;
  } else {
    character.velocityY = 0;
  }
  
  // Update character direction
  if (character.velocityX !== 0 || character.velocityY !== 0) {
    character.direction = getDirection(character.velocityX, character.velocityY);
    character.state = 'walking';
  } else {
    character.state = 'idle';
  }
}
```

## 7. State Management

### 7.1 State Organization

Organize Redux state by domain:

```typescript
interface RootState {
  auth: AuthState;
  user: UserState;
  space: SpaceState;
  characters: CharactersState;
  communication: CommunicationState;
  ui: UIState;
  objects: ObjectsState;
  settings: SettingsState;
}
```

### 7.2 State Synchronization

- Implement optimistic updates for user actions
- Apply server authority for conflict resolution
- Use timestamped events for ordering
- Batch state updates for efficiency
- Implement state reconciliation for disconnections

### 7.3 Performance Optimization

- Use selectors for derived state
- Implement memoization for expensive calculations
- Apply immutable data patterns
- Use normalized state for entities
- Implement entity filtering based on proximity

```typescript
// Example of memoized selectors
import { createSelector } from '@reduxjs/toolkit';

const selectCharactersState = (state: RootState) => state.characters;
const selectUserPosition = (state: RootState) => state.user.position;

export const selectNearbyCharacters = createSelector(
  [selectCharactersState, selectUserPosition],
  (characters, userPosition) => {
    return Object.values(characters.entities).filter(character => {
      if (!character) return false;
      
      const distance = calculateDistance(
        userPosition.x,
        userPosition.y,
        character.position.x,
        character.position.y
      );
      
      return distance <= NEARBY_THRESHOLD;
    });
  }
);
```

## 8. Network Communication

### 8.1 WebSocket Implementation

- Implement connection management with reconnection logic
- Add heartbeat for connection status monitoring
- Use binary protocols for position updates (MessagePack or Protocol Buffers)
- Implement event batching for efficiency

### 8.2 State Synchronization

- Implement client-side prediction for movement
- Apply server reconciliation for corrections
- Use delta compression for state updates
- Implement interest management (proximity-based updates)
- Add jitter buffering for smooth playback

### 8.3 API Communication

- Implement REST API client with authorization
- Add request caching for common resources
- Implement retry logic with exponential backoff
- Use optimistic UI updates where appropriate

```typescript
// Example of a WebSocket connection manager
class WebSocketManager {
  private socket: Socket | null = null;
  private reconnectAttempts = 0;
  private reconnectTimeout: any = null;
  
  constructor(private store: Store, private authManager: AuthManager) {}
  
  public connect(): void {
    const token = this.authManager.getToken();
    
    this.socket = io(API_URL, {
      auth: { token },
      transports: ['websocket', 'polling'],
      reconnectionAttempts: 10,
      reconnectionDelay: 1000,
      reconnectionDelayMax: 30000
    });
    
    this.setupEventHandlers();
  }
  
  private setupEventHandlers(): void {
    if (!this.socket) return;
    
    this.socket.on('connect', this.handleConnect.bind(this));
    this.socket.on('disconnect', this.handleDisconnect.bind(this));
    this.socket.on('reconnect_attempt', this.handleReconnectAttempt.bind(this));
    this.socket.on('error', this.handleError.bind(this));
    
    // Add event handlers for game events
    this.socket.on('space.update', this.handleSpaceUpdate.bind(this));
    this.socket.on('user.joined', this.handleUserJoined.bind(this));
    this.socket.on('user.left', this.handleUserLeft.bind(this));
    // ... more event handlers
  }
  
  // ... implementation of handler methods and event dispatchers
}
```

## 9. Audio System

### 9.1 WebRTC Implementation with Retro Effects

- Use simple-peer for WebRTC connections with optional 8-bit audio filter effect
- Implement signaling through WebSocket with arcade-style connection animations
- Add connection management for peer connections with visual "signal strength" indicators
- Implement fallback to server relay with "radio static" feedback when connection quality degrades

### 9.2 Arcade-Style Spatial Audio

- Implement chunky volume steps based on distance (8-10 distinct levels, not continuous)
- Add simplified left/right stereo panning with visible audio wave indicators
- Implement audio range with visible "hearing radius" circle around characters
- Add themed audio zones with different 8-bit ambient background tracks and sound effects

### 9.3 Retro Voice Processing

- Implement echo cancellation with optional "echo chamber" effect for large spaces
- Add noise suppression with toggleable "arcade ambience" background option
- Implement voice activity detection with pixel-art speech bubble animations
- Add audio level visualization using chunky vertical bar meters or expanding pixel rings

```typescript
// Example of a proximity-based audio manager
class SpatialAudioManager {
  private peerConnections: Map<string, SimplePeer.Instance> = new Map();
  private audioElements: Map<string, HTMLAudioElement> = new Map();
  private audioContext: AudioContext | null = null;
  private audioNodes: Map<string, AudioNode[]> = new Map();
  
  constructor(private signaler: SignalingService) {
    this.audioContext = new (window.AudioContext || window.webkitAudioContext)();
    this.setupSignaling();
  }
  
  private setupSignaling(): void {
    this.signaler.on('user-joined-voice', this.initiateConnection.bind(this));
    this.signaler.on('voice-signal', this.handleSignal.bind(this));
    this.signaler.on('user-left-voice', this.removeConnection.bind(this));
  }
  
  public updatePositions(userPositions: Record<string, Position>): void {
    const myPosition = userPositions[this.signaler.getUserId()];
    
    for (const [userId, position] of Object.entries(userPositions)) {
      if (userId === this.signaler.getUserId()) continue;
      
      const audioNodes = this.audioNodes.get(userId);
      if (!audioNodes) continue;
      
      const [panner] = audioNodes.filter(node => node instanceof PannerNode) as PannerNode[];
      
      if (panner) {
        // Calculate relative position for 3D audio
        const relativeX = position.x - myPosition.x;
        const relativeY = position.y - myPosition.y;
        
        // Update panner position
        panner.positionX.value = relativeX;
        panner.positionY.value = relativeY;
        panner.positionZ.value = 0;
        
        // Calculate distance for volume adjustment
        const distance = Math.sqrt(relativeX * relativeX + relativeY * relativeY);
        
        // Update gain based on distance
        const [gainNode] = audioNodes.filter(node => node instanceof GainNode) as GainNode[];
        if (gainNode) {
          // Linear falloff with distance clamped to reasonable range
          const volume = Math.max(0, Math.min(1, 1 - distance / MAX_AUDIO_DISTANCE));
          gainNode.gain.value = volume;
        }
      }
    }
  }
  
  // ... implementation of WebRTC connection methods
}
```

## 10. Performance Optimization

### 10.1 Retro-Inspired Rendering Optimization

- Implement object pooling for frequently created entities (like in original arcade games)
- Use sprite batching for grouped rendering with deliberately limited entity counts
- Apply visibility culling based on screen boundaries (like original NES games)
- Implement chunk-based loading for large spaces with visible "loading zones"
- Use WebWorkers for offloading heavy calculations with themed "co-processor" loading indicators

### 10.2 Network Optimization

- Prioritize different types of updates
- Implement variable update rates based on importance
- Use compression for large payloads
- Cache static resources aggressively
- Implement progressive loading for large spaces

### 10.3 Memory Management

- Monitor and optimize memory usage
- Implement asset unloading for unused resources
- Add reference counting for shared resources
- Implement texture compression for sprites
- Profile and fix memory leaks

## 11. Accessibility

### 11.1 Core Requirements with Retro Compatibility

- Implement keyboard navigation using traditional gaming keys (WASD, Space, E)
- Add screen reader support for UI elements with arcade-themed announcements
- Support high contrast mode with classic video game color schemes (e.g., "Game Boy" palette)
- Implement resizable pixel-art text that maintains the retro aesthetic
- Add 8-bit style captions for audio content with typewriter animation effect

### 11.2 Spatial Experience

- Provide alternative navigation methods for movement
- Implement audio cues for spatial awareness
- Add visual indicators for audio sources
- Provide text alternatives for voice chat
- Implement zoom controls for visibility

## 12. Internationalization

- Implement i18next for translation management
- Support right-to-left languages
- Use ICU message format for pluralization and formatting
- Implement language detection and selection
- Add locale-specific formatting for dates and numbers

## 13. Testing Strategy

### 13.1 Unit Testing (Game Cartridge Testing)

- Test individual components and utilities like "game cartridge modules"
- Implement pixel-perfect snapshot testing for retro UI components
- Test state management logic with arcade-style state diagrams
- Use mock services for external dependencies as "simulation mode"
- Aim for high test coverage of business logic with "debug mode" visualization

### 13.2 Integration Testing

- Test communication between components
- Test WebSocket interaction
- Verify WebRTC functionality
- Test authentication flows
- Test responsive layouts

### 13.3 Performance Testing

- Profile rendering performance
- Measure memory usage
- Test network efficiency
- Verify smooth animation
- Benchmark on various devices

## 14. Build and Deployment

### 14.1 Build Process

- Use Vite for fast builds
- Implement code splitting for efficient loading
- Optimize assets during build
- Generate source maps for debugging
- Run automated tests as part of the build

### 14.2 Deployment Strategy

- Implement continuous integration pipeline
- Use Docker for consistent environments
- Deploy to CDN for static assets
- Implement canary deployments
- Add monitoring and error tracking

## 15. Developer Experience

### 15.1 Development Environment

- Provide comprehensive documentation
- Implement hot module reloading
- Add development-only debug tools
- Provide mock data generators
- Implement development modes for features

### 15.2 Debugging Tools

- Add custom Redux DevTools instrumentation
- Implement canvas inspection tools
- Add network request visualization
- Implement performance monitoring
- Add collision visualization

---

## Appendix A: 8-Bit Component Library Structure

### Arcade UI Components

- **Arcade Button**: Command, Action, Boss-Key variants with pixel borders
- **Terminal Input**: Text, number, password variants with blinking cursor
- **Dialog Box**: Quest, Alert, Game-Over variants with ASCII borders
- **Game Card**: Item, Character, Achievement variants with pixelated frames
- **Speech Bubble**: Information, Warning, Error variants with chunky tails
- **Game Menu**: Select Screen, Pause Menu, Inventory variants
- **Level Tabs**: World Map, Stage Select, Character Select variants
- **Status Message**: Power-Up, Damage, Level-Up variants with animation

### 8-Bit Avatar Components

- **Pixel Character**: 16x16 or 32x32 sprite-based character with limited frame animations
- **Floating NameTag**: Username display with status indicator in pixel font
- **Sprite Emotes**: Limited frame sprite animations for expressions (4-8 frames max)
- **Character Select**: Arcade-style avatar customization interface with color palette selection

### Game World Components

- **Tile Map**: Grid-based pixel art environment with 8-bit textures
- **Game Object**: Interactive sprite-based objects with limited animation states
- **Level Zone**: Special area with unique visual themes and background music
- **Quest Marker**: Blinking pixel arrows or exclamation points for navigation

## Appendix B: 8-Bit Animation System

### Sprite Animation States

| State | Description | Frames | Triggers |
|-------|-------------|--------|----------|
| `idle` | Standing still | 2-4 frames | No movement input |
| `walk` | Walking animation | 2-4 frames per direction | Movement at normal speed |
| `action` | Action animation | 3-6 frames | When interacting with object |
| `emote` | Emotional expression | 4-8 frames | When triggering an emote |
| `teleport` | Zone transition | 6-8 frames | When changing areas |

### Animation Implementation

```typescript
interface AnimationState {
  name: string;
  frames: number[];
  frameRate: number;
  loop: boolean;
}

interface AnimationController {
  currentAnimation: string;
  currentFrame: number;
  frameTimer: number;
  sprite: PIXI.Sprite;
  animations: Record<string, AnimationState>;
  
  update(deltaTime: number): void;
  play(animationName: string, reset?: boolean): void;
  stop(): void;
}
```

## Appendix C: Optimization Checklist

### Initial Load Performance

- [ ] Implement code splitting for routes
- [ ] Preload critical assets
- [ ] Optimize and compress images
- [ ] Minimize CSS and JavaScript
- [ ] Use modern image formats (WebP)

### Runtime Performance

- [ ] Optimize render loop
- [ ] Implement object pooling
- [ ] Use requestAnimationFrame correctly
- [ ] Apply texture batching
- [ ] Implement spatial partitioning
- [ ] Use WebWorkers for heavy calculations
- [ ] Profile and fix memory leaks
- [ ] Implement efficient event handling

### Network Efficiency

- [ ] Batch network requests
- [ ] Compress payloads
- [ ] Implement caching strategies
- [ ] Use binary protocols where appropriate
- [ ] Prioritize critical updates
- [ ] Implement reconnection handling

## Appendix D: WebRTC Implementation

### Connection Establishment Flow

1. User A enables voice chat
2. Server notifies nearby users
3. User B receives notification
4. User A and B exchange SDP offers through server
5. WebRTC connection established
6. Audio stream connected to spatial audio system

### ICE Configuration

```typescript
const ICE_CONFIG = {
  iceServers: [
    { urls: 'stun:stun.l.google.com:19302' },
    { urls: 'stun:stun1.l.google.com:19302' },
    { 
      urls: 'turn:turn.devhangout.io:3478',
      username: 'devhangout',
      credential: 'dynamicCredential'
    }
  ],
  iceTransportPolicy: 'all',
  rtcpMuxPolicy: 'require'
};
```

## Appendix E: Retro CSS Configuration

```javascript
// retro-theme.js
module.exports = {
  colors: {
    // Main palette based on CRT colors
    crtBlue: '#00FFFF',
    magenta: '#FF00FF',
    neonGreen: '#00FF00',
    bitYellow: '#FFFF00',
    digitalRed: '#FF0000',
    
    // Background colors
    spaceBlack: '#0A0A1F',
    terminalGreen: '#001100',
    darkNavy: '#000033',
    
    // UI element colors
    pixelGray: '#AAAAAA',
    screenGlow: '#00BBBB',
    errorRed: '#FF0000',
    powerUpGreen: '#00FF66'
  },
  
  fonts: {
    heading: '"Press Start 2P", monospace',
    subheading: '"VCR OSD Mono", monospace',
    body: '"Share Tech Mono", monospace',
    code: '"DOS", monospace'
  },
  
  // Grid based on 8px (pixel art friendly)
  grid: {
    base: 8,
    scale: [0, 1, 2, 3, 4, 5, 6, 8, 10, 12, 16, 20, 24, 32, 40, 48, 64]
  },
  
  // Border styles with pixel art aesthetics
  borders: {
    pixel: '2px solid',
    pixelDouble: '4px double',
    pixelDashed: '2px dashed',
    pixelInset: '2px inset'
  },
  
  // Animation durations based on frame counts
  animations: {
    cursor: 'blink 1s step-end infinite',
    walk: 'walkCycle 0.5s steps(4) infinite',
    idle: 'idleAnimation 1s steps(2) infinite',
    flash: 'screenFlash 0.2s step-end',
    textType: 'textTyping 3s steps(40, end)'
  }
}
```

---
> Source: [virtualdev-xyz/dev-hangout-client](https://github.com/virtualdev-xyz/dev-hangout-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
