## jarvis-node-mobile

> Expo/React Native mobile app for Pi Zero node provisioning, device discovery, and home automation control.

# jarvis-node-mobile

Expo/React Native mobile app for Pi Zero node provisioning, device discovery, and home automation control.

## Quick Reference

```bash
# Install
npm install

# Development
npm start              # Expo dev server
npm run ios            # iOS simulator
npm run android        # Android emulator

# Tests
npm test
npm run test:coverage
```

## Architecture

```
jarvis-node-mobile/
├── App.tsx                # Root (Auth + Config + Theme + Query providers)
├── src/
│   ├── auth/              # AuthContext, token management
│   ├── screens/
│   │   ├── Auth/          # Login, registration
│   │   ├── Home/          # Main dashboard
│   │   ├── Devices/       # Paired device list
│   │   ├── Provisioning/  # Node setup flow (QR, WiFi config)
│   │   ├── Rooms/         # Room management
│   │   ├── SmartHome/     # Home Assistant integration
│   │   ├── ImportKey/     # Private key import
│   │   └── Settings/      # App settings
│   ├── navigation/        # RootNavigator, tab layout
│   ├── services/
│   │   ├── configDiscoveryService.ts   # Find jarvis-config-service on network
│   │   ├── configPushService.ts        # Push config to nodes
│   │   ├── haApiService.ts             # Home Assistant API
│   │   ├── haDiscoveryService.ts       # HA auto-discovery
│   │   ├── networkDiscoveryService.ts  # Network scanning
│   │   ├── oauthService.ts             # OAuth flows
│   │   ├── qrImportService.ts          # QR code parsing
│   │   └── qrPayloadService.ts         # QR payload generation
│   ├── components/        # Reusable UI components
│   ├── contexts/          # ConfigContext (global config)
│   ├── theme/             # Light/dark mode
│   ├── hooks/             # Custom React hooks
│   ├── config/            # env.ts (API URLs)
│   └── api/               # API clients
├── modules/
│   └── jarvis-crypto/     # Local encryption module
├── __tests__/             # Jest test suite
└── ios/                   # Xcode project
```

## Tech Stack

- **Expo 54** + React Native 0.81 + React 19 + TypeScript
- **React Navigation 7** (bottom tabs + native stacks)
- **React Native Paper** (Material Design UI)
- **React Query 5** (data fetching)
- **expo-camera** (QR scanning)
- **expo-secure-store** (token storage)

## Key Features

- **QR Code Provisioning**: Scan QR to pair Pi Zero nodes
- **WiFi Configuration**: Push WiFi credentials to headless nodes
- **Home Assistant**: Device discovery and control
- **Config Discovery**: Auto-find jarvis-config-service on local network
- **Secure Storage**: Tokens stored in platform keychain

## Environment

API URLs configured in `src/config/env.ts`. Dev mode uses localhost.

## Deployment

- **EAS Build** profiles: development, staging, production
- **CI/CD**: Not yet set up (manual EAS builds)
- iOS and Android supported

## Dependencies

**Service Dependencies:**
- `jarvis-auth` (7701) — login, token management
- `jarvis-config-service` (7700) — service discovery
- `jarvis-command-center` (7703) — node registration

**Used By:**
- End users for node setup and home automation

---
> Source: [alexberardi/jarvis-node-mobile](https://github.com/alexberardi/jarvis-node-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
