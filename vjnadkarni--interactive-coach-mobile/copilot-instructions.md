## interactive-coach-mobile

> This Flutter mobile app provides the Interactive Coach experience on iOS and Android, featuring Hera - an AI health and wellness coach powered by Claude 4.5 Sonnet. The app connects to the same FastAPI backend as the web application and will integrate with Apple Watch (HealthKit) and Android watches (Health Connect) for health vitals tracking.

# CLAUDE.md - AI Assistant Instructions

## Project: Interactive Coach Mobile - Hera Health & Wellness Coach

### Overview
This Flutter mobile app provides the Interactive Coach experience on iOS and Android, featuring Hera - an AI health and wellness coach powered by Claude 4.5 Sonnet. The app connects to the same FastAPI backend as the web application and will integrate with Apple Watch (HealthKit) and Android watches (Health Connect) for health vitals tracking.

---

## 🔴 CRITICAL: Git Workflow - STRICT ENFORCEMENT

### Code Synchronization Between Machines (Mac Mini M4 ↔ MacBook Pro M3)

**NEVER use manual git commands. ALWAYS use the provided scripts.**

#### Leaving Work (Mac Mini M4):
```bash
~/venv/bin/push-wip.sh
```

#### Arriving Home (MacBook Pro M3):
```bash
~/venv/bin/pull-wip.sh
```

#### Leaving Home (MacBook Pro M3):
```bash
~/venv/bin/push-wip.sh
```

#### Arriving at Work (Mac Mini M4):
```bash
~/venv/bin/pull-wip.sh
```

### STRICT RULES (NEVER VIOLATE):
1. ✅ ALL commits MUST use `~/venv/bin/push-wip.sh`
2. ✅ ALL pulls MUST use `~/venv/bin/pull-wip.sh`
3. ❌ NEVER commit manually with `git commit`
4. ❌ NEVER pull manually with `git pull`
5. ⚠️  **Misaligned codebases = DISASTER**

### Why These Scripts?
- Ensure consistent workflow
- Prevent merge conflicts
- Track machine-to-machine transitions
- Maintain clean git history
- Automatic commit message formatting

### Branch Strategy
- **Development**: `wip` branch (ALL ongoing development)
- **Production**: `main` branch (merge only when features work to satisfaction)
- Current branch: `wip`

---

## 🔴 CRITICAL: Hera's Personality Enforcement

### The Problem
Hera must maintain her identity as a health & wellness coach across ALL platforms (web and mobile). She is NOT a business coach, NOT a general assistant, NOT Sofia or Anna or any other persona.

### Hera's Identity (STRICT)
- **Name**: HERA (NOT Sofia, NOT Anna, NOT any business coach)
- **Role**: Health & Wellness Coach ONLY
- **Expertise**: Fitness, nutrition, training, sports, health vitals
- **Personality**: Friendly, bubbly, enthusiastic, warm, engaging
- **Communication**: 2-4 sentences, asks questions, encouraging

### Topics Hera MUST Discuss
✅ Fitness, exercise, training (strength, cardio, HIIT, endurance)
✅ Nutrition, diet, supplements, healthy eating
✅ Sports (all kinds)
✅ Health vitals (heart rate, HRV, SpO2, blood pressure, sleep)
✅ Training plans, progress tracking
✅ Fitness equipment, gyms

### Topics Hera MUST NEVER Discuss
❌ Business strategy, finance, career advice
❌ Technology, software, coding
❌ General assistant tasks
❌ Productivity tools unrelated to fitness

---

## 🔴 CRITICAL: Environment Variables & Security

**NEVER hardcode API keys in source code. NEVER commit .env files.**

### Single .env File
- **Location**: `interactive-coach-mobile/.env` (top-level)
- **Source**: Copied from `~/venv/interactive-avatar2/.env`
- **Status**: ✅ In .gitignore (never committed to git)

### Environment Variables
```bash
# Backend API Configuration
BACKEND_URL=http://192.168.7.30:8000  # Mac Mini IP for WiFi testing

# HeyGen (future use)
HEYGEN_API_KEY=<secret>

# Anthropic (backend handles this)
ANTHROPIC_API_KEY=<secret>

# Deepgram (future use)
DEEPGRAM_API_KEY=<secret>

# Supabase (backend handles this)
SUPABASE_URL=<url>
SUPABASE_ANON_KEY=<secret>
SUPABASE_SERVICE_KEY=<secret>
```

### Usage in Code

**Dart (Flutter) - CORRECT:**
```dart
import 'package:flutter_dotenv/flutter_dotenv.dart';
import '../utils/constants.dart';

final apiUrl = AppConstants.backendUrl;
```

**Dart (Flutter) - FORBIDDEN:**
```dart
final apiUrl = "http://192.168.7.30:8000"; // ❌ NEVER DO THIS
```

---

## 🔴 STRICT: LLM Specification

### Claude 4.5 Sonnet ONLY

**Model ID**: `claude-sonnet-4-5-20250929`

All AI responses come from the FastAPI backend which uses Claude 4.5 Sonnet. The mobile app does NOT make direct calls to Anthropic API - it streams responses from the backend `/chat/stream-test` endpoint (Voice-Only mode with ElevenLabs TTS).

**IMPORTANT: API Endpoint Paths**
The backend endpoints do NOT have an `/api` prefix:
- ✅ CORRECT: `/chat/stream-test` (Voice-Only mode)
- ✅ CORRECT: `/chat/stream` (Video+Voice mode)
- ✅ CORRECT: `/conversation/check`
- ✅ CORRECT: `/conversation/clear`
- ❌ WRONG: `/api/chat/stream` (will return 404)

---

## 🔴 CRITICAL: iOS Permissions Configuration

### Required Info.plist Entries
**Location**: `ios/Runner/Info.plist`

```xml
<!-- Speech Recognition -->
<key>NSSpeechRecognitionUsageDescription</key>
<string>This app uses speech recognition so you can talk to Hera, your health and wellness coach.</string>

<!-- Microphone -->
<key>NSMicrophoneUsageDescription</key>
<string>This app needs microphone access so you can speak with Hera.</string>

<!-- HealthKit (future) -->
<key>NSHealthShareUsageDescription</key>
<string>Interactive Coach needs access to your health data to provide personalized coaching</string>

<key>NSHealthUpdateUsageDescription</key>
<string>Interactive Coach needs to save health data from your workouts</string>
```

### iOS Deployment Target
**Location**: `ios/Podfile`

```ruby
platform :ios, '14.0'  # Required for health package
```

**WHY**: The `health` package v13.2.0 requires iOS 14.0 or later.

---

## 🔴 CRITICAL: Backend Connection

### Development Setup
The mobile app connects to the FastAPI backend running on your Mac.

**IMPORTANT**: The backend must listen on **all interfaces** (0.0.0.0), not just localhost:

```bash
# ❌ WRONG - only accessible from Mac itself
uvicorn main:app --reload

# ✅ CORRECT - accessible from iPhone/Android over WiFi
uvicorn main:app --reload --host 0.0.0.0
```

### Network Configuration
- **Mac Mini IP**: 192.168.7.30 (check with `ifconfig`)
- **Backend URL**: `http://192.168.7.30:8000`
- **Mobile .env**: `BACKEND_URL=http://192.168.7.30:8000`

### Testing Connection
1. Start backend with `--host 0.0.0.0`
2. Ensure iPhone/Android is on same WiFi network as Mac
3. Test from mobile browser: `http://192.168.7.30:8000/docs`
4. If accessible → Mobile app will work

---

## Features

### Current Implementation ✅ WORKING

**Voice-to-Voice Chat Interface** (Tested Nov 8, 2025 on iPhone 12)
- **Voice Input (STT)**: speech_to_text package - iOS native speech recognition
- **Text Input**: Type messages to Hera (alternative to voice)
- **Voice Output (TTS)**: ElevenLabs API with Rachel voice - **FULLY WORKING**
- **Streaming Responses**: Claude responses stream in real-time via SSE with RAG citations
- **In-Memory Audio Playback**: Custom `StreamAudioSource` - zero file I/O latency
- **Multi-turn Conversations**: Conversation history maintained via Supabase
- **Hera Personality**: Correct health & wellness coach identity
- **JWT Authentication**: Supabase session tokens for API authorization
- **Network**: Connects to FastAPI backend over WiFi

**UI Elements**:
- Purple/blue gradient theme
- App bar: "Chat with Hera"
- Mode toggle: Video+Voice (coming soon) / Voice+Text (active)
- Bottom navigation: Health Dashboard ↔ Chat ↔ User Dashboard
- Chat bubbles: Blue (user), Purple (Hera), Red (errors)
- Input area: Microphone button, text field, send button
- Auto-scroll to latest message
- Real-time voice transcription display

### Planned Features 🔲

1. **HeyGen Video Avatar** - Visual Elenora (like web app)
2. **HealthKit Integration** - Apple Watch vitals sync
3. **Health Connect Integration** - Android watch vitals sync
4. **User Authentication** - JWT-based login
5. **Health Dashboard** - Visualize vitals and trends
6. **Background Sync** - 15-minute health data updates
7. **Session Time Limits** - Configurable session duration
8. **Offline Mode** - Cached conversations

---

## 🔊 Text-to-Speech (TTS) Implementation

### ElevenLabs Integration ✅ WORKING

**Voice**: Rachel (same as web app) - Voice ID: `21m00Tcm4TlvDq8ikWAM`

**Architecture**:
1. **Backend Response**: Claude streams text response via SSE
2. **References Stripped**: `_stripReferences()` removes citation section before TTS
3. **ElevenLabs API**: POST request to `/v1/text-to-speech/{voice_id}/stream`
4. **In-Memory Streaming**: Custom `BytesAudioSource` extends `StreamAudioSource`
5. **Zero File I/O**: Audio bytes streamed directly from memory to iOS audio engine
6. **Audio Session**: Configured with `AudioSessionConfiguration.speech()`

**Key Files**:
- [lib/services/tts_service.dart](lib/services/tts_service.dart) - TTS service with custom StreamAudioSource
- [lib/screens/chat_screen.dart](lib/screens/chat_screen.dart#L130) - Automatic playback after AI response

**Custom StreamAudioSource**:
```dart
class BytesAudioSource extends StreamAudioSource {
  final Uint8List _bytes;

  BytesAudioSource(this._bytes);

  @override
  Future<StreamAudioResponse> request([int? start, int? end]) async {
    start = start ?? 0;
    end = end ?? _bytes.length;

    return StreamAudioResponse(
      sourceLength: _bytes.length,
      contentLength: end - start,
      offset: start,
      stream: Stream.value(_bytes.sublist(start, end)),
      contentType: 'audio/mpeg',
    );
  }
}
```

**Why This Works**:
- iOS audio engine can request audio chunks for seeking/buffering
- No filesystem latency - everything stays in RAM
- Proper content type and length metadata
- Supports partial range requests

**Performance**:
- Audio generation: ~1-3 seconds (ElevenLabs API)
- Playback start: Immediate (no file I/O delay)
- Total latency: Minimal, comparable to web app

---

## Technology Stack

### Framework
- **Flutter**: 3.35.3
- **Dart**: 3.9.2
- **Platform**: iOS (iPhone 12, iOS 18.0.1) + Android (future)

### Backend Integration
- **FastAPI**: Shared with web app (localhost:8000 → 0.0.0.0:8000)
- **Streaming**: SSE (Server-Sent Events) via `/chat/stream`
- **Database**: Supabase (shared conversation history)
- **AI**: Claude 4.5 Sonnet via backend

### Key Packages
- **speech_to_text**: ^7.0.0 - iOS native speech recognition (STT)
- **just_audio**: ^0.9.40 - Audio playback for TTS
- **audio_session**: ^0.1.25 - iOS audio session configuration
- **http**: ^1.2.1 - HTTP client for API calls
- **supabase_flutter**: ^2.9.2 - Authentication and database
- **flutter_dotenv**: ^5.1.0 - Environment variables
- **health**: ^13.2.0 - HealthKit + Health Connect (future)
- **video_player**: ^2.9.1 - HeyGen avatar (future)

---

## Project Structure

```
interactive-coach-mobile/
├── .env                        # 🔴 Environment variables (NEVER commit)
├── .gitignore                  # Git ignore rules
├── README.md                   # Project overview
├── CLAUDE.md                   # 🔴 This file - AI instructions
├── pubspec.yaml                # Dart dependencies
│
├── lib/                        # Flutter source code
│   ├── main.dart              # App entry point (loads .env, launches AvatarScreen)
│   ├── screens/
│   │   └── avatar_screen.dart # ✅ Chat interface with Elenora
│   ├── services/
│   │   └── api_service.dart   # ✅ FastAPI backend communication (SSE streaming)
│   └── utils/
│       └── constants.dart     # ✅ Environment config (AppConstants)
│
├── ios/                        # iOS-specific configuration
│   ├── Runner/
│   │   └── Info.plist         # ✅ Permissions (speech, microphone)
│   └── Podfile                # ✅ iOS deployment target (14.0)
│
├── android/                    # Android-specific configuration
│
└── assets/                     # Images, fonts, etc.
```

---

## Development Workflow

### Starting Development Session

1. **Pull latest code** (if switching machines):
```bash
~/venv/bin/pull-wip.sh
```

2. **Start backend** (Terminal 1):
```bash
cd /Users/vijay/venv/interactive-coach/backend
source ../venv/bin/activate
python -m uvicorn main:app --reload --host 0.0.0.0
```

3. **Deploy to iPhone 12** (Terminal 2):
```bash
cd /Users/vijay/venv/interactive-coach-mobile
flutter run -d 00008101-001D44303C08801E
```

**Device Info**:
- **Name**: Vijay's iPhone 12
- **Device ID**: 00008101-001D44303C08801E
- **iOS Version**: 18.0.1 (displayed as 26.0.1 in Xcode)
- **Developer Mode**: Enabled

### Ending Development Session

1. **Stop Flutter** (press 'q' in terminal or Ctrl+C)
2. **Stop backend** (Ctrl+C in backend terminal)
3. **Commit changes**:
```bash
~/venv/bin/push-wip.sh
```

---

## Testing Checklist ✅ VERIFIED (Oct 9, 2025)

### Session Start
- [x] App launches without crash
- [x] Permissions prompt for speech recognition appears
- [x] User grants permission → App shows chat interface
- [x] Hera's greeting message appears: "Hi! I'm Hera, your health and wellness coach..."

### Speech Recognition
- [x] Tap microphone → Red microphone icon
- [x] Speak → Text appears in input field
- [x] Release → Transcript finalized
- [x] Message sent automatically after finalization

### Text Input
- [x] Type message in text field
- [x] Tap send button → Message appears as blue bubble
- [x] Backend receives message → Hera responds

### Conversation
- [x] Hera responds in character (health/fitness topics)
- [x] Streaming responses work (text appears incrementally)
- [x] Purple chat bubble for Hera
- [x] Multi-turn conversation works (tested 5+ exchanges)
- [x] Conversation history maintained
- [x] Hera stays on topic (fitness/health only)

### Network
- [x] Backend accessible at 192.168.7.30:8000
- [x] Mobile app connects successfully
- [x] SSE streaming works over WiFi
- [x] Error handling: Shows "Failed to get response" on connection failure

### UI Layout
- [x] Purple/magenta app bar
- [x] Chat area with scrollable messages
- [x] Input area at bottom (mic, text field, send)
- [x] Auto-scroll to latest message

---

## Troubleshooting

### App Shows Blank White Screen
**Cause**: Missing iOS permissions or initialization error
**Solution**:
1. Check Xcode console for error messages
2. Add required permissions to Info.plist
3. Ensure `WidgetsFlutterBinding.ensureInitialized()` in main.dart

### "Failed to get response" Error
**Cause**: Backend not accessible from mobile device
**Solution**:
1. Verify backend running: `http://192.168.7.30:8000/docs` in browser
2. Ensure backend started with `--host 0.0.0.0`
3. Check iPhone and Mac are on same WiFi network
4. Verify .env has correct IP: `BACKEND_URL=http://192.168.7.30:8000`

### Speech Recognition Not Working
**Cause**: Missing permissions or iOS restrictions
**Solution**:
1. Check Info.plist has NSSpeechRecognitionUsageDescription
2. Grant permission when prompted
3. Settings → Privacy → Speech Recognition → Allow app
4. Restart app after granting permission

### Hot Reload Not Updating .env Changes
**Cause**: Environment variables loaded at app startup
**Solution**:
- Use **Hot Restart** (R) not Hot Reload (r)
- Or kill and redeploy: `flutter run -d <device-id>`

### CocoaPods Version Mismatch
**Cause**: Health package requires iOS 14.0+
**Solution**:
```bash
cd ios
pod deintegrate
# Edit Podfile: platform :ios, '14.0'
pod install
cd ..
```

---

## Important Notes

1. **Single Codebase**: One Flutter project for both iOS and Android (feature parity)
2. **Shared Backend**: Same FastAPI backend as web application
3. **Shared Database**: Supabase stores conversation history for both web and mobile
4. **WiFi Testing**: Development uses local network (Mac IP: 192.168.7.30)
5. **Production Deployment**: Will use HTTPS endpoint (future)
6. **Apple Watch**: Will integrate after basic features are complete
7. **Android Testing**: Will use emulator or physical device (future)

---

## Version History

- **v0.1.0** - Basic chat interface working on iPhone 12 (Oct 9, 2025) ✅
  - Text and speech input
  - Streaming Claude responses
  - Multi-turn conversations
  - Elenora personality correct
  - Status: Production-ready for basic chat

- **v0.2.0** - Apple Watch Health Integration Phase 1-3 (Nov 3, 2025) ✅
  - Phase 1: iOS HealthKit permissions configured
  - Phase 2: HealthService implementation (409 lines, 19 health metrics)
  - Phase 3: Backend API complete (5 endpoints, 800 lines)
  - Backend URL: `http://192.168.6.234:8000`
  - Status: Ready for Phase 4 (mobile sync service)

- **v0.3.0** - Apple Watch Health Integration Phase 4 (Nov 3, 2025) ✅
  - Phase 4: Mobile-to-Backend Sync Service complete
  - File: `lib/services/health_sync_service.dart` (291 lines)
  - Delta sync (only new data since last sync)
  - Retry logic with exponential backoff
  - JWT authentication placeholder (Phase 4.1)
  - Test UI: Updated `health_test_screen.dart` with 3 sync buttons
  - Status: Ready for testing on iPhone 12

- **v0.4.0** - Apple Watch Health Integration Phase 5-6 (Nov 7, 2025) ✅
  - Phase 5: Health Dashboard UI complete
    - File: `lib/screens/health_dashboard_screen.dart` (full UI implementation)
    - Real-time vitals display (HR, HRV, SpO2)
    - Activity metrics (Steps, Calories, Distance)
    - Sleep summary (Total, Deep, REM, Core, Awake)
    - Timestamp freshness indicators ("X m ago")
    - Bottom navigation (Health, Avatar, Dashboard tabs)
  - Phase 6: Polling Implementation complete
    - 5-minute `Timer.periodic` automatic refresh
    - Manual refresh via "Sync All" button
    - Temporary solution until observer entitlements configured
  - Phase 6.5: Observer-Based Notifications (code ready)
    - File: `lib/services/health_observer_service.dart` (209 lines)
    - Package: `health_kit_reporter: ^2.3.1` installed
    - Real-time observer queries for HR, HRV, SpO2
    - Awaiting Xcode entitlement setup (5-minute configuration)
    - Documentation: `HEALTHKIT_OBSERVER_SETUP.md`
  - Status: Fully functional with polling, ready to migrate to observers

- **v0.5.0** - Health Dashboard Enhancements + Chat 404 Fix (Nov 18, 2025) ✅
  - Health Dashboard UI polish (card styling, spacing, icons)
  - Body composition display (weight, body fat %)
  - Fixed Chat 404 error (removed `/api` prefix from endpoints)
  - API endpoint documentation added to CLAUDE.md
  - Status: Production-ready

- **v0.6.0** - Audio Output Settings (Nov 19, 2025) ✅
  - **New Feature**: Audio output routing configuration
    - Dashboard → Settings → Audio Output selection
    - iPhone Speaker mode (uses `.playAndRecord` with `.defaultToSpeaker`)
    - Earpiece/Headphones mode (uses `.playback` category)
    - Settings persist via SharedPreferences
  - **Files Created**:
    - `lib/screens/settings_screen.dart` (audio output UI with radio buttons)
  - **Files Modified**:
    - `ios/Runner/NativeAudioPlayer.swift` (AVAudioSession routing)
    - `lib/services/native_tts_service.dart` (reads user preference)
    - `lib/screens/user_dashboard_screen.dart` (Settings navigation)
  - **Apple TTS Experiment**: Tested AVSpeechSynthesizer as ElevenLabs replacement
    - ❌ Rejected due to robotic voice quality (sounds like "AI voice")
    - ❌ Mid-speech failures with long texts
    - ❌ App crashes after TTS failures
    - ✅ Decision: Keep ElevenLabs TTS for natural-sounding Rachel voice
    - Note: First-word truncation and abbreviation issues are acceptable trade-offs
  - Status: Production-ready

---

## 📱 **Apple Watch Series 9 Integration Status**

**Current Phase**: Phase 6 Complete ✅ (Polling Implementation)
**Next Phase**: Phase 6.5 - Observer-Based Notifications (code ready, needs Xcode entitlements)
**Overall Progress**: 6 of 7 phases complete (86%)

### **✅ Phase 1: iOS Configuration - COMPLETE**
- HealthKit permissions in Info.plist
- iOS deployment target: 14.0+
- Documentation: `HEALTHKIT-SETUP.md`

### **✅ Phase 2: HealthService Implementation - COMPLETE**
- File: `lib/services/health_service.dart` (409 lines)
- 19 health data types supported
- Methods: requestPermissions, getHeartRateData, getLatestHRV, getDailySummary
- Test UI: `lib/screens/health_test_screen.dart`

### **✅ Phase 3: Backend API - COMPLETE**
- 5 REST endpoints implemented in main backend
- Database: 3 new tables (health_vitals, health_activity, health_sleep)
- Authentication: JWT tokens required
- Testing: 6/6 tests passed ✅
- Network: iPhone → Backend connectivity verified
- Backend location: `/Users/vijay/venv/interactive-coach/backend/health/`

**Endpoints Ready**:
- `POST /api/health/vitals` - Store HR, HRV, SpO2
- `POST /api/health/activity` - Store steps, calories
- `POST /api/health/sleep` - Store sleep sessions
- `GET /api/health/vitals/{user_id}` - Retrieve vitals history
- `GET /api/health/summary/{user_id}/{date}` - Daily summary

### **✅ Phase 4: Mobile-to-Backend Sync - COMPLETE**
- File: `lib/services/health_sync_service.dart` (291 lines)
- Features implemented:
  - Delta sync (only new data since last sync)
  - JWT authentication placeholder (ready for Phase 4.1)
  - Retry logic with exponential backoff
  - SharedPreferences for timestamp tracking
  - 3 public sync methods: syncVitalsToBackend(), syncActivityToBackend(), syncAllHealthData()
- Test UI: Updated `health_test_screen.dart` with 3 sync buttons
- Documentation: `PHASE4-SYNC-SERVICE-COMPLETE.md`
- Status: Ready for testing on iPhone 12

### **✅ Phase 5: Health Dashboard UI - COMPLETE**
- File: `lib/screens/health_dashboard_screen.dart` (complete UI implementation)
- Features implemented:
  - Real-time vitals display (Heart Rate, HRV, SpO2)
  - Activity metrics (Steps, Calories, Distance)
  - Sleep summary (Total, Deep, REM, Core, Awake)
  - Timestamp freshness indicators ("X m ago")
  - Sync All button for manual data refresh
  - Error handling with user-friendly messages
- Navigation: Bottom navigation bar with Health, Avatar, Dashboard tabs
- Status: Fully functional and tested on iPhone 12

### **✅ Phase 6: Polling Implementation - COMPLETE**
- Implementation: 5-minute `Timer.periodic` automatic refresh
- Purpose: Temporary solution until observer-based notifications are configured
- Features:
  - Automatic dashboard refresh every 5 minutes
  - Manual refresh via "Sync All" button
  - Clean timer disposal on screen exit
- Status: Working reliably (verified by user)
- Next Step: Migrate to observer-based notifications (Phase 6.5)

### **⏳ Phase 6.5: Observer-Based Notifications - CODE READY**
- Status: Implementation complete, awaiting Xcode entitlement configuration
- File: `lib/services/health_observer_service.dart` (209 lines)
- Package: `health_kit_reporter: ^2.3.1` (installed)
- Features implemented:
  - Real-time observer queries for HR, HRV, SpO2
  - Background delivery enabled for immediate updates
  - VitalReading model with value and timestamp
  - Auto-sync on new health data
- Blocking Issue: Missing `com.apple.developer.healthkit.background-delivery` entitlement
- Documentation: `HEALTHKIT_OBSERVER_SETUP.md` (complete step-by-step guide)
- Estimated Setup Time: 5 minutes (one-time Xcode configuration)
- Expected Latency: 1-3 seconds (vs 0-5 minutes with polling)

### **⏳ Phase 7: Background Sync - FUTURE WORK**
- Background sync at 15-minute intervals
- iOS background task scheduling
- Battery optimization strategies

**Reference Documentation**:
- `MOBILE-AGENT-HANDOFF.md` - Complete phase breakdown
- `APPLE-WATCH-INTEGRATION-STATUS.md` - Current status
- `PHASE4-SYNC-SERVICE-COMPLETE.md` - Phase 4 implementation details
- Backend docs: `interactive-coach/backend/PHASE3-HEALTH-API-COMPLETE.md`

---

## 🎥 **Video+Voice Mode Status**

**Current Status**: Native iOS implementation blocked - WebView approach recommended

### **Problem Summary**
Attempted native iOS implementation of HeyGen Interactive Avatar using LiveKit SDK:
- ✅ LiveKit connection successful
- ✅ Voice-Only mode working perfectly (native Flutter + ElevenLabs)
- ✅ Speech recognition working perfectly (native iOS Speech framework)
- ✅ Backend conversation flow working perfectly
- ❌ HeyGen avatar never joins LiveKit room (participant count stays 0)
- ❌ Missing WebRTC signaling to HeyGen's signaling server
- ❌ No video rendering, no audio playback

### **Root Cause**
HeyGen Streaming Avatar v1 requires:
1. Call `/v1/streaming.new` API ✅
2. Connect to LiveKit room ✅
3. **Connect to WebRTC signaling server** ❌ (missing)
4. Exchange SDP offers/answers ❌ (missing)
5. Avatar joins as remote participant ❌ (never happens)

The web version uses HeyGen's JavaScript SDK (`@heygen/streaming-avatar`) which handles all this automatically. Native implementation requires hundreds of lines of complex WebRTC signaling code.

### **Recommended Solution: WebView Approach**
Instead of rebuilding the entire HeyGen SDK in native Swift:
1. Keep Voice-Only mode as native Flutter (current - works great!)
2. Embed working Next.js web page in WebView for Video+Voice mode
3. Use JavaScript bridge for Flutter ↔ Web communication
4. Reuse battle-tested web implementation

**Advantages**:
- Works immediately (web version already proven)
- Easy to maintain (one codebase for web + mobile)
- No complex WebRTC code to debug
- 1-2 hours implementation vs days of debugging

**Files Modified in Native Attempt**:
- `ios/Runner/HeyGen/` - Complete Swift implementation (blocked)
- `lib/screens/avatar_screen_native.dart` - Native Flutter screen
- LiveKit Swift SDK integrated via SPM

**Documentation**:
- `HEYGEN_NATIVE_STATUS.md` - Complete analysis and WebView implementation plan
- `HEYGEN_NATIVE_IMPLEMENTATION_SUMMARY.md` - Technical details
- `VIDEO_VOICE_DEBUGGING_SESSION.md` - Debugging session notes

**Decision Point**: Proceed with WebView approach or wait for HeyGen native SDK v2

---

- **v0.5.0** - Native HeyGen Investigation (Nov 15, 2025) ⚠️
  - Attempted native iOS HeyGen Interactive Avatar implementation
  - LiveKit Swift SDK 2.10.0 integrated successfully
  - Voice-Only mode remains fully functional (native Flutter + ElevenLabs)
  - Identified blocker: Missing WebRTC signaling to HeyGen's server
  - WebView approach recommended for Video+Voice mode
  - Status: Native blocked, awaiting decision on WebView vs SDK v2

- **v0.6.0** - Health Dashboard Enhancements + API Endpoint Fix (Nov 19, 2025) ✅
  - Enhanced Health Dashboard with "See more..." modal popups
    - Vitals card: Added Resting HR, Walking HR, Respiratory Rate
    - Activity card: Added Exercise Time, Distance
    - Sleep card: Added Total, Deep, REM, Core, Awake sleep metrics
  - Added `health` getter to HealthService for direct Health instance access
  - **Critical Bug Fix**: Fixed Chat 404 error in Release build
    - Root cause: API endpoints had incorrect `/api` prefix
    - Fixed: `/api/chat/stream` → `/chat/stream-test` (Voice-Only mode)
    - Fixed: `/api/conversation/check` → `/conversation/check`
    - Fixed: `/api/conversation/clear` → `/conversation/clear`
  - Updated all Elenora references to Hera throughout documentation
  - **Apple TTS Experiment**: Tested AVSpeechSynthesizer as ElevenLabs replacement
    - ❌ Rejected due to robotic voice quality (sounds like "AI voice")
    - ❌ Mid-speech failures with long texts
    - ❌ App crashes after TTS failures
    - ✅ Decision: Keep ElevenLabs TTS for natural-sounding Rachel voice
    - Note: First-word truncation and abbreviation issues are acceptable trade-offs
  - Status: Release build tested and working on iPhone 12

---

## Contact

For questions about AI implementation or architecture decisions, refer to this document first. The project aims to create a production-ready health coaching platform with emphasis on realistic human-like interactions and personalized health guidance.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vjnadkarni)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/vjnadkarni)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
