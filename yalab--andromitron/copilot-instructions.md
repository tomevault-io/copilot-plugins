## andromitron

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Andromitron is an Android application for implementing a local HTTPS proxy with domain-based filtering capabilities. The project aims to create a VPN-based proxy service that can filter network traffic at the system level.

## Development Commands

### Build and Development
```bash
# Build the project
./gradlew build

# Clean build
./gradlew clean

# Build debug APK
./gradlew assembleDebug

# Build release APK
./gradlew assembleRelease

# Install debug APK on connected device
./gradlew installDebug
```

### Testing
```bash
# Run unit tests
./gradlew test

# Run unit tests for debug build
./gradlew testDebugUnitTest

# Run instrumented tests (requires connected device/emulator)
./gradlew connectedAndroidTest

# Run specific test class
./gradlew test --tests "net.yalab.andromitron.ExampleUnitTest"
```

### Code Quality
```bash
# Run lint checks
./gradlew lint

# Generate lint report
./gradlew lintDebug
```

## Project Architecture

### Core Components (Implemented)
Based on the specification in `android_proxy_spec.md`:

- **VPN Service**: ✅ System-level traffic capture using Android's VPNService
- **Filter Engine**: ✅ Domain-based filtering logic (blocklist/whitelist)
- **TUN Interface**: ✅ Network packet processing
- **Database Layer**: ✅ Room database for storing blocklists and logs
- **UI Layer**: ✅ Jetpack Compose interface with Material3
- **Network Layer**: ⚠️ Basic structure (needs full integration)

### Current State (Fully Implemented)
- ✅ Complete Android project structure with comprehensive architecture
- ✅ Package: `net.yalab.andromitron`
- ✅ Target SDK: 36, Min SDK: 35
- ✅ Kotlin-based development with coroutines
- ✅ Comprehensive test coverage for all components
- ✅ VPN service with packet processing integration
- ✅ Domain filtering with exact and wildcard matching
- ✅ Network packet analysis (IPv4, TCP, UDP, DNS, HTTP, HTTPS/TLS)
- ✅ Room database with DAOs and entities
- ✅ Complete Jetpack Compose UI with Material3 design

### Key Technical Approaches
1. **VPNService + TUN Interface**: Primary method for system-wide traffic interception
2. **DNS-level blocking**: For HTTPS traffic filtering
3. **SNI parsing**: Extract domain names from TLS handshakes
4. **Hybrid filtering**: Combination of DNS monitoring and SNI analysis

### Required Permissions (Implemented)
- ✅ `BIND_VPN_SERVICE` - VPN service binding
- ✅ `INTERNET` - Network access
- ✅ `FOREGROUND_SERVICE` - Background service operation
- ✅ `FOREGROUND_SERVICE_SPECIAL_USE` - VPN-specific foreground service

## File Structure

### Application Structure
- `app/`: Main application module
- `app/src/main/`: Application source code
  - `java/net/yalab/andromitron/`: Main package
    - `MainActivity.kt`: Main activity with Compose integration
    - `service/ProxyVpnService.kt`: VPN service implementation
    - `filter/`: Domain filtering system
      - `DomainFilter.kt`: Core filtering logic
      - `FilterManager.kt`: Thread-safe filter management
      - `FilterRule.kt`: Filter rule data structures
    - `packet/`: Network packet processing
      - `IpPacket.kt`: IPv4 packet parsing
      - `PacketProcessor.kt`: Main packet analysis engine
      - `TlsPacketAnalyzer.kt`: TLS/SSL traffic analysis
    - `database/`: Room database layer
      - `AppDatabase.kt`: Main database configuration
      - `entities/`: Database entities
      - `dao/`: Data Access Objects
    - `ui/`: Jetpack Compose UI components
      - `AndromitronApp.kt`: Main app navigation
      - `components/`: Reusable UI components
      - `theme/`: Material3 theme system
- `app/src/test/`: Comprehensive unit tests
- `app/src/androidTest/`: Instrumented tests
- `android_proxy_spec.md`: Detailed technical specification
- `gradle/libs.versions.toml`: Dependency version management
- `app/lint-baseline.xml`: Lint baseline for VPN-specific warnings

## Implementation Status

### ✅ Completed Components
1. **VPN Service Infrastructure** - Complete implementation with lifecycle management
2. **Domain Filtering System** - Exact and wildcard domain matching with caching
3. **Network Packet Processing** - IPv4, TCP, UDP, DNS, HTTP, and HTTPS/TLS analysis
4. **Database Layer** - Room database with entities, DAOs, and default data seeding
5. **User Interface** - Complete Jetpack Compose UI with Material3 design system
   - Bottom navigation (Home, Rules, Logs, Settings)
   - Connection status card with VPN controls
   - Statistics card with traffic metrics
   - Filter rules management interface
   - Light/dark theme support

### 🔧 Technical Implementation Details
- **Architecture**: MVVM pattern with Repository pattern for data layer
- **Concurrency**: Kotlin coroutines for async operations
- **Threading**: Thread-safe components with proper synchronization
- **Testing**: Comprehensive unit tests for all core components
- **Build System**: Gradle with version catalog and Compose support
- **Code Quality**: Lint checks with baseline for VPN-specific permissions

### 🚀 Ready for Next Steps
- Device testing and VPN permission flow integration
- Real network traffic interception and forwarding
- Integration with actual blocklist/whitelist data sources
- Performance optimization and battery usage monitoring
- Security hardening and encrypted local storage

### 📝 Development Notes
- All core functionality implemented and tested
- UI provides complete user experience flow
- VPN service ready for actual traffic processing
- Database supports persistent rule and log storage
- Modern Android development practices followed throughout

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yalab)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yalab)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
