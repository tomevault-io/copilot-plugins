## papercut-theater

> You are the primary developer for **Jianzhi Theater**, a cross-device application showcasing Chinese Paper-cut art.

# Jianzhi Theater - Project Instructions

You are the primary developer for **Jianzhi Theater**, a cross-device application showcasing Chinese Paper-cut art.

## Project Vision
A digital theater experience where one device acts as the "Stage" (displaying high-quality paper-cut videos) and another acts as the "Director" (remote control console). The aesthetic is "Clean Minimalism" mixed with traditional Chinese artistic elements (Paper-cut red, cream backgrounds).

## Tech Stack
- **Frontend**: React 18, Vite, TypeScript.
- **Backend**: Express (Custom server in `server.ts`).
- **Real-time**: Socket.io for cross-device synchronization.
- **Styling**: Tailwind CSS with custom theme variables in `src/index.css`.
- **Animations**: `motion/react`.
- **Icons**: `lucide-react`.

## Architectural Patterns
- **Role System**: The app starts with a selection screen to choose between `stage` and `director` roles.
- **Room Sync**: Devices join a shared `demo-room-1` (expandable to dynamic rooms) via Socket.io.
- **Command Flow**: The Director emits `video-command` events. The Stage listens for these to control the `<video>` element.

## Styling & Design Guidelines
- **Theme**: "Clean Minimalism". Use white cards with subtle shadows (`shadow-xl shadow-neutral-200/40`), large typography, and intentional white space.
- **Colors**:
  - Primary Red: `#D62828` (Paper-cut red)
  - Background: `#F9F7F2` (Paper-cream)
  - Text: `#2D2D2D` (Ink)
- **Typography**: 
  - Serif (Cormorant Garamond) for artistic accents.
  - Sans-serif (Inter) for UI controls.
- **Visual Cues**: Use `Sparkles`, `Palette`, and traditional patterns (radial gradients in CSS) to reinforce the cultural theme.

## Development Rules
- **Sync Logic**: Always ensure any state change in the Director that affects playback (play, pause, seek, load) is emitted via `socket.emit('video-command', ...)`.
- **Video Handling**: Use `referrerPolicy="no-referrer"` for all external images and video thumbnails to avoid loading issues in the preview.
- **Responsive Controls**: The Director layout should be mobile-friendly but also usable on tablets/desktops. The Stage should always be immersive and center the content.

## Testing
- **Running Tests**: You can run all tests using the command `npm run test`.
- **Test Maintenance**: Whenever you update the source code or add a new module, you MUST also update the corresponding tests (e.g., `src/App.test.tsx`, `src/components/Director.test.tsx`, or `src/components/Stage.test.tsx`) to ensure they reflect the latest logic and functionality.

## Documentation & Skills
- [Cross-Device Sync Architecture](./CROSS_DEVICE_SYNC.md)

### Custom Agent Skills
Always use these specific custom skills when working on the following areas:
- **`assets-management`**: For updating, adding, or managing media assets (Videos, Images) and text.
- **`media-playback-handling`**: For audio/video playback code, catching promises, and mocking HTMLMediaElement in tests.
- **`cross-device-socket-sync`**: For writing Socket.io communication, debouncing events, and room synchronizations.

---
> Source: [zesonzhang/papercut-theater](https://github.com/zesonzhang/papercut-theater) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
