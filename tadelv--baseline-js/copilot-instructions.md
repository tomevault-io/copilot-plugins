## baseline-js

> Baseline.js is a **reference implementation** and **showcase project** for building custom user interfaces (skins) for [Streamline-Bridge](https://github.com/tadelv/reaprime). This project demonstrates best practices for integrating with the Bridge API and creating an intuitive, visually appealing espresso machine interface.

# Claude Development Guide for Baseline.js

## Project Overview

Baseline.js is a **reference implementation** and **showcase project** for building custom user interfaces (skins) for [Streamline-Bridge](https://github.com/tadelv/reaprime). This project demonstrates best practices for integrating with the Bridge API and creating an intuitive, visually appealing espresso machine interface.

**Primary Purpose**: To serve as a learning resource and starting point for developers who want to build their own custom skins for Streamline-Bridge.

## Core Architecture

### Technology Stack

- **Framework**: Next.js 16 with App Router
- **UI Library**: React 19
- **Language**: TypeScript
- **Styling**: Tailwind CSS 4
- **Components**: Radix UI for accessible primitives
- **Visualization**: HTML5 Canvas API

### Application Structure

The application is built as a single-page application (SPA) with the main logic embedded in `app/page.tsx`. This unconventional approach uses `dangerouslySetInnerHTML` to render a self-contained HTML/CSS/JavaScript application within a React component.

**Why this approach?**
- Single-file deployment for easy distribution
- Minimal dependencies at runtime
- Can be easily extracted and used standalone
- Demonstrates that Streamline-Bridge skins don't require complex tooling

### State Management

The application uses vanilla JavaScript state management (`STATE` object) rather than React hooks, since the core application logic is embedded HTML. State includes:

- **screen** - Current UI screen (main, sleep, carousel, brewing, done, settings)
- **machineState** - Current machine status from Bridge API
- **machineReady** - Whether machine is ready to brew
- **carouselStep** - Current step in brewing workflow
- **profiles** - Available brewing profiles from machine
- **WebSocket connections** - For real-time data (machine snapshots, scale readings)

## Streamline-Bridge Integration

### API Communication

All communication with Streamline-Bridge happens through:

1. **REST API** (`CONFIG.apiUrl` - default: `http://localhost:8080`)
   - `/api/v1/machine/state` - Get machine status
   - `/api/v1/profiles` - List available profiles
   - `/api/v1/workflow` - Set active profile
   - `/api/v1/workflow/start` - Start brewing
   - `/api/v1/devices` - List connected devices (scale, etc.)
   - `/api/v1/devices/scan?connect=true` - Scan for and connect devices

2. **WebSocket Endpoints**
   - `/ws/v1/machine/snapshot` - Real-time machine data (pressure, flow)
   - `/ws/v1/scale/snapshot` - Real-time scale weight readings

### Key Integration Points

#### Machine State Monitoring
```javascript
// Polls every 1 second
setInterval(async () => {
  await getMachineState();
  // Auto-transitions based on state changes
}, 1000);
```

#### Profile Management
When setting a profile, the full profile object must be sent, not just the ID:
```javascript
const profile = STATE.profiles.find(p => p.id === profileId);
await apiCall('/api/v1/workflow', {
  method: 'PUT',
  body: JSON.stringify({ profile: profile.profile })
});
```

#### Scale Integration
The app automatically scans for and connects to scales when entering the brewing workflow. Real-time weight is displayed during the "Weigh Coffee" step.

## Development Guidelines

### When Making Changes

1. **Preserve the Reference Nature**: This is meant to be a simple, educational example. Avoid over-engineering.

2. **API Compatibility**: Always check the [Streamline-Bridge API documentation](https://github.com/tadelv/reaprime) when modifying API calls.

3. **Visual Consistency**: The design uses a dark theme with blue accents. Maintain this aesthetic or document intentional departures.

4. **Performance**: Canvas animations should be smooth. Use requestAnimationFrame and be mindful of particle counts.

5. **Browser Compatibility**: The app should work in modern browsers and the Streamline-Bridge embedded view.

### Common Tasks

#### Adding a New Screen
1. Create a render function (e.g., `renderNewScreen()`)
2. Add the screen name to the STATE object
3. Update the main `render()` function to handle the new screen
4. Add navigation logic to transition to/from the screen

#### Modifying the Workflow
The carousel steps are defined in the `steps` array within `renderCarousel()`. Each step has:
- `title` - Step name
- `description` - Instructions for the user
- `hasInput` - Whether the step requires user input
- `showWeight` - Whether to display scale readings

#### Adding New Settings
1. Update the `CONFIG` object with default value
2. Add localStorage save/load in `saveSettings()`
3. Add input field in `renderSettingsScreen()`
4. Update `saveSettingsFromDialog()` to read and save the new setting

### Canvas Animations

Three distinct animations are used:

1. **Ambient Animation** (`renderAmbientAnimation`) - Subtle particle effects on main screens
2. **Brewing Visualization** (`renderBrewingVisualization`) - Data-driven particles responding to pressure/flow
3. **Sleep Animation** (`renderSleepAnimation`) - Gentle wave patterns for sleep mode

All use low-pass filtering for smooth transitions and respond to real-time data from the machine.

## Testing Considerations

### Without Physical Hardware

When developing without an espresso machine:
- Use mock API responses
- Test all state transitions manually
- Verify WebSocket reconnection logic
- Test with various screen sizes

### With Streamline-Bridge

When connected to a real Bridge instance:
- Test profile switching
- Verify state transitions during actual brewing
- Check scale connection and weight display
- Validate WebSocket data accuracy

## File Organization

```
app/
├── layout.tsx           # Root layout with metadata
└── page.tsx            # Main app (embedded HTML/CSS/JS)

components/
└── ui/                 # Unused Radix UI components (from v0)
                        # Can be removed or used for future features

screens/                # Marketing/documentation screenshots
├── idle.png           # Machine ready state
├── scale.png          # Scale integration
└── sleeping.png       # Sleep mode

public/                # Static assets (if needed)
```

## Important Reminders

### This is a Reference Project

The primary goal of Baseline.js is to **demonstrate** how to build interfaces for Streamline-Bridge. When working on this project:

- Prioritize clarity over cleverness
- Comment non-obvious integration points
- Keep dependencies minimal
- Document API usage patterns
- Make it easy for others to learn from this code

### Streamline-Bridge Documentation

Always refer to the [Streamline-Bridge repository](https://github.com/tadelv/reaprime) for:
- API endpoint specifications
- WebSocket message formats
- Available machine states
- Profile structure requirements
- Device management details

## Common Issues & Solutions

### WebSocket Connection Drops
- Implement automatic reconnection logic
- Handle connection errors gracefully
- Display connection status to user

### State Synchronization
- Poll `/api/v1/machine/state` regularly
- Don't assume state transitions
- Handle unexpected states gracefully

### Scale Connection
- Scales may take time to connect
- Show loading/scanning feedback
- Handle connection failures with retry logic

### Profile Selection
- Always fetch fresh profile list on settings open
- Send complete profile object, not just ID
- Verify profile was set successfully

## Future Enhancement Ideas

- Multi-language support
- Custom color themes
- Brew history tracking
- Shot timer presets
- Temperature monitoring
- Maintenance reminders
- Multiple machine support

## Resources

- **Streamline-Bridge**: https://github.com/tadelv/reaprime
- **Next.js Documentation**: https://nextjs.org/docs
- **Radix UI**: https://www.radix-ui.com
- **Tailwind CSS**: https://tailwindcss.com

---

Remember: This is a showcase and learning tool. Keep it simple, well-documented, and true to its educational purpose.

## Design Context

### Users
Home espresso enthusiasts using this interface on a screen near their machine (typically Streamline-Bridge's embedded webview or a tablet). They interact while their hands may be busy with coffee prep — the UI must be glanceable and operable with minimal attention. Think appliance interface, not power-user dashboard.

### Brand Personality
**Simple, inviting, effortless.** The interface should feel like a friendly appliance that anyone could operate — so intuitive a toddler could use it. No learning curve, no cognitive overhead. Quiet confidence over technical complexity.

### Aesthetic Direction
- **Reference**: Apple Home — sparse, generous whitespace, understated controls
- **Anti-reference**: Dense data dashboards, cluttered settings panels, anything that looks like a developer tool
- **Theme**: Dark mode (deep navy `#0a0e27`) with blue accent gradients. Subtle glassmorphism and canvas particle animations add atmosphere without distraction
- **Typography**: System font stack (-apple-system), uppercase status labels with letter-spacing, large readable text
- **Color semantics**: Green = ready, yellow = warming/caution, red = error/disconnected, blue = primary actions

### Accessibility
- WCAG 2.1 AA minimum across the interface
- Respect `prefers-reduced-motion` — disable or simplify all canvas animations and CSS transitions
- Touch targets minimum 44x44px for the kiosk/tablet context
- Maintain sufficient contrast ratios on the dark background

### Design Principles
1. **One glance, one action.** Every screen should communicate its state and primary action instantly. No hunting, no reading paragraphs.
2. **Appliance-grade simplicity.** Fewer choices, bigger targets, clearer labels. Remove anything that doesn't serve the immediate task.
3. **Ambient, not anxious.** Animations and status indicators should feel calming and informative, never urgent or flashy. The UI recedes when things are going well.
4. **Progressive disclosure.** Settings and advanced options exist but stay hidden until sought. The happy path is front and center.
5. **Touch-first, always.** Design for fingers on a screen near a coffee machine — generous spacing, forgiving tap areas, swipe-friendly navigation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tadelv) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
