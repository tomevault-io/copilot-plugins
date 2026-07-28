## crazycar

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**CrazyCar** is a networked multiplayer racing game with three main modules:
- **CrazyCarClient** - Unity 2021.3+ game client
- **CrazyCarServer** - Spring Boot 2.7.1 game server (Java 17)
- **CrazyCarBackground** - Vue 2.6 admin dashboard

## Build Commands

### Server (Maven)
```bash
cd CrazyCarServer
mvn clean package -DskipTests
java -jar target/crazy_car.jar                    # Development
java -jar target/crazy_car.jar --spring.profiles.active=prod  # Production
```

### Admin Dashboard (npm)
```bash
cd CrazyCarBackground
npm run dev              # Development server
npm run build:prod       # Production build
npm run build:stage      # Staging build
npm run lint             # ESLint
npm run test:unit        # Unit tests
npm run test:ci          # Lint + unit tests
```

### Client (Unity)
- Build targets: Android (Gradle/IL2CPP), iOS (Xcode), WebGL
- Use Unity Editor Build System
- Key Unity packages: Addressables 1.19.19, HybridCLR, URP 12.1.7

## Architecture

### Client: QFramework 4-Layer Architecture

The client uses a modified QFramework with strict layer rules:

```
ViewController (表现层)
  - Can get System, Model
  - Can send Command, listen to Event
     ↓
System (系统层)
  - Can get System, Model, Utility
  - Can send/listen to Event
     ↓
Model (数据层)
  - Can get Utility
  - Can send Event
     ↓
Utility (工具层)
  - Third-party library wrappers
```

**Command (跨层级):** Can get System/Model/Utility, send Event/Command

All Systems and Models are registered in `CrazyCarClient/Assets/Scripts/CrazyCar.cs`.

**Registered Systems:**
- `IPlayerManagerSystem` - Player management
- `IWebSocketSystem` / `IKCPSystem` - Dual-protocol network (KCP/WebSocket)
- `INetworkSystem` - Network orchestration
- `IScreenEffectsSystem`, `ISoundSystem`, `IVibrationSystem`
- `ICheckpointSystem`, `IAddressableSystem`, `IGuidanceSystem`
- `IMatchRoomSystem` - Match room management

**Registered Models:**
- `IGameModel`, `IUserModel`, `IAvatarModel`, `ITimeTrialModel`
- `IMatchModel`, `IEquipModel`, `ISettingsModel`, `IRoomMsgModel`
- `IMapControllerModel`, `IInputModel`

### Server: Spring Boot MVC

Standard MVC pattern: `Controller → Service → Mapper → Model`

**Key Controllers:**
- REST APIs: `LoginController`, `UserInfoController`, `AvatarController`, `EquipController`
- Game Controllers: `TimeTrialController`, `MatchController`, `GameController`
- WebSocket: `TimeTrialWebSocket`, `MatchRoomWebSocket`, `MatchWebSocket`
- Background Admin: `BackgroundDashboardController`, `BackgroundUser`, etc.

**WebSocket Message Types:**
```java
CreatePlayer = 0, PlayerState = 1, DelPlayer = 2,
MatchRoomCreate = 3, MatchRoomJoin = 4, MatchRoomStatus = 5,
MatchRoomExit = 6, MatchRoomStart = 7
```

### Admin Dashboard: Vue 2.6 + Element UI

Standard Vue CLI project with:
- `src/api/` - API wrappers
- `src/components/` - Reusable components
- `src/layout/` - Layout components
- `src/router/` - Vue Router config
- `src/store/` - Vuex state management
- `src/views/` - Page views

## Key Technical Details

### Network Synchronization
- **Dual Protocol:** KCP (low-latency UDP) + WebSocket (TCP fallback)
- **Physics Sync:** Server-authoritative physics, client prediction
- **Message Queue:** High-concurrency message buffering

### Hot Update System
- **Resources:** Addressables for runtime asset downloads
- **Code:** HybridCLR for C# hot fixes (AOT + interpreter)
- Asset bundles built via `Build/AB/Remote` menu

### Authentication
- JWT tokens stored in `Authorization` header
- Token payload contains user ID (`jti` claim)
- Server utility: `CrazyCarServer/src/main/java/com/tastsong/crazycar/utils/Util.java`

### Database Schema (Key Tables)
- `all_user` - User accounts with `uid`, `user_name`, `star`, `aid` (avatar_id)
- `avatar_name`, `avatar_uid` - Avatar ownership
- `all_equip`, `equip_uid` - Car/equipment ownership
- `time_trial_class`, `time_trial_record`, `time_trial_user_map` - Time trial mode
- `match_class`, `match_record` - Multiplayer match mode
- `ab_resource` - Addressable resource metadata (hash, crc, url, size)
- `forced_updating` - Version control and forced updates

## Important File Paths

### Client Core
- `CrazyCarClient/Assets/Scripts/CrazyCar.cs` - Architecture entry point
- `CrazyCarClient/Assets/Scripts/System/NetworkSystem.cs` - Network orchestration
- `CrazyCarClient/Assets/Scripts/System/KCPSystem.cs` - KCP protocol
- `CrazyCarClient/Assets/Scripts/Game/MPlayer.cs` - Player controller
- `CrazyCarClient/Assets/Scripts/Game/AIController.cs` - AI bot logic
- `CrazyCarClient/Packages/manifest.json` - Unity package dependencies

### Server Core
- `CrazyCarServer/src/main/java/com/tastsong/crazycar/CrazyCarApplication.java` - Spring Boot main
- `CrazyCarServer/src/main/java/com/tastsong/crazycar/controller/` - All REST/WebSocket controllers
- `CrazyCarServer/src/main/resources/application.properties` - Server configuration
- `CrazyCarServer/src/main/resources/data.sql` - Database init script
- `CrazyCarServer/pom.xml` - Maven dependencies

## Development Notes

- Unity 2021.3+ required with URP (Universal Render Pipeline)
- Java 17 required for server
- Node.js >= 8.9, npm >= 3.0.0 for admin dashboard
- MySQL 8.0.32 for database
- QFramework uses IoC container with singleton pattern for layer access
- Login page supports single-player mode toggle for offline testing

---
> Source: [TastSong/CrazyCar](https://github.com/TastSong/CrazyCar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
