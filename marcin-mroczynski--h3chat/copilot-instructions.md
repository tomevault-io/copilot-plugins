## h3chat

> A modern geolocation-based chat application using H3 hexagonal indexing system. Users can chat with people in their geographical neighborhood based on H3 cell proximity.

# H3 Local Chat - Development Guide

## 🏗️ Project Overview
A modern geolocation-based chat application using H3 hexagonal indexing system. Users can chat with people in their geographical neighborhood based on H3 cell proximity.

## 📋 Current Tech Stack
- **Backend**: Node.js + Express + WebSocket + h3-js
- **Frontend**: React 18 + TypeScript + Vite
- **Styling**: Tailwind CSS v4 + Glassmorphism design
- **State**: Zustand store
- **Animations**: Framer Motion
- **Icons**: Heroicons
- **Development**: Nodemon + Vite HMR

## 🚀 Quick Start Commands
```bash
# Development (both servers)
npm run dev        # Backend with nodemon (port 4000)
npm run dev:client # Frontend with Vite HMR (port 5175)

# Production
npm run build      # Build React app
npm run start      # Production server

# Utilities
npm run kill       # Kill processes on common ports
```

## 📁 Project Structure
```
h3chat/
├── server.js                 # WebSocket server with H3 logic
├── index.html                # Legacy HTML (kept for reference)
├── package.json              # Server dependencies & scripts
├── nodemon.json              # Nodemon configuration
├── client/                   # React frontend
│   ├── src/
│   │   ├── App.tsx           # Main app component
│   │   ├── components/       # React components
│   │   │   ├── ChatLayout.tsx
│   │   │   ├── LocationSetup.tsx
│   │   │   ├── ChatSidebar.tsx
│   │   │   ├── ChatMessages.tsx
│   │   │   └── ChatInput.tsx
│   │   ├── hooks/            # Custom hooks
│   │   │   ├── useWebSocket.ts       # Main WebSocket connection
│   │   │   └── useWebSocketActions.ts # WebSocket actions (singleton)
│   │   ├── store/            # State management
│   │   │   └── chatStore.ts  # Zustand store
│   │   ├── types.ts          # TypeScript definitions
│   │   └── index.css         # Tailwind + custom styles
│   ├── package.json          # Frontend dependencies
│   ├── tailwind.config.js    # Tailwind CSS v4 config
│   └── postcss.config.js     # PostCSS config
└── CLAUDE.md                 # This file
```

## 🔧 Architecture Notes

### WebSocket Architecture
- **Single Connection**: WebSocket initialized once in App.tsx
- **Singleton Pattern**: `useWebSocketActions` provides send functions without creating new connections
- **Auto-reconnect**: 3-second delay with connection state checking
- **Rate Limiting**: 5 messages per second server-side

### H3 Geolocation System
- **Resolution 8**: Default hexagon size (~461m diameter)
- **Broadcast Range**: 0-3 levels (current cell to extended area)
- **Presence TTL**: 30 seconds before user cleanup
- **Location Privacy**: Only H3 index shared, not exact GPS coordinates

### State Management
- **Zustand Store**: Centralized app state
- **Real-time Updates**: Immediate UI updates via WebSocket events
- **User Management**: Map-based nearby users tracking

## 🐛 Known Issues & Solutions

### Common Problems:
1. **Port Conflicts**: Use `npm run kill` or change ports in server.js and useWebSocket.ts
2. **Reconnect Loops**: WebSocket should only be initialized in App.tsx
3. **Null Names**: Components handle `name: null` with fallback to "Unknown"
4. **State Delays**: Pass values directly to avoid React async state issues

### Development Workflow:
- **Backend Changes**: Nodemon auto-restarts server
- **Frontend Changes**: Vite HMR auto-reloads browser
- **WebSocket Reconnect**: Automatic on connection loss
- **Type Safety**: Full TypeScript coverage

## 🚧 Current Status
✅ **Completed Features:**
- [x] Modern React + TypeScript setup
- [x] Beautiful glassmorphism UI design
- [x] WebSocket real-time communication
- [x] H3 geolocation-based chat
- [x] User presence management
- [x] Broadcast radius selection (0-3 levels)
- [x] Rate limiting & error handling
- [x] Development workflow (nodemon + HMR)
- [x] Responsive design
- [x] Smooth animations

## 🎯 Potential Future Features
- [ ] **Dark/Light Mode Toggle**
- [ ] **Emoji Picker & Reactions**
- [ ] **Sound Notifications**
- [ ] **Message History Persistence**
- [ ] **User Avatars & Profiles**
- [ ] **Private Direct Messages**
- [ ] **Group Chat Rooms**
- [ ] **Message Encryption**
- [ ] **PWA Support (offline capability)**
- [ ] **Mobile App (React Native)**
- [ ] **Map Visualization of H3 cells**
- [ ] **Message Search & Filtering**
- [ ] **File/Image Sharing**
- [ ] **Multi-language Support**
- [ ] **Admin Panel & Moderation**

## 💡 Development Tips

### Adding New Features:
1. **Backend**: Extend server.js WebSocket message types
2. **Frontend**: Add new components in `components/`
3. **State**: Extend Zustand store in `store/chatStore.ts`
4. **Types**: Update `types.ts` for TypeScript safety

### Debugging:
- **WebSocket**: Check browser console for connection logs
- **Server**: Check terminal for nodemon logs
- **State**: Use React DevTools + Zustand DevTools
- **H3 Cells**: Log H3 indices to verify geolocation logic

### Performance:
- **Bundle Size**: Use `npm run build` and check warnings
- **WebSocket**: Monitor connection stability in Network tab
- **Animations**: Use `will-change` CSS for smooth animations
- **Memory**: Clean up intervals/listeners in useEffect cleanup

## 🔗 Important Links
- **H3 Documentation**: https://h3geo.org/
- **React Docs**: https://react.dev/
- **Tailwind CSS v4**: https://tailwindcss.com/blog/tailwindcss-v4-alpha
- **Zustand**: https://zustand-demo.pmnd.rs/
- **Framer Motion**: https://www.framer.com/motion/

## 📝 Development Notes
- **Port Configuration**: Backend auto-selects available port (currently 4000)
- **WebSocket URL**: Development connects to localhost:4000, production uses same host
- **CSS Framework**: Using Tailwind v4 with @import syntax
- **Build Output**: Client builds to `client/dist/` for production serving

## 🤝 Contribution Guidelines
When continuing development:
1. **Test WebSocket**: Always verify connection stability
2. **Check Types**: Ensure TypeScript compilation passes
3. **Test Responsive**: Check mobile/desktop layouts
4. **Validate H3**: Test geolocation with multiple users
5. **Performance**: Monitor bundle size and runtime performance

---
Last updated: January 2025
Project Status: ✅ Production Ready

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/marcin-mroczynski) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
