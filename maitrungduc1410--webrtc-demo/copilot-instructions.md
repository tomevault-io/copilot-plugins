## webrtc-demo

> **WebRTC-Demo** is a comprehensive, cross-platform WebRTC demonstration project that showcases real-time peer-to-peer communication capabilities across Web, Android, and iOS platforms. The project implements modern WebRTC APIs (currently using M125) to enable video calls, audio communication, screen sharing, and data channel messaging between multiple clients.

# AGENTS.md

## Project Overview

**WebRTC-Demo** is a comprehensive, cross-platform WebRTC demonstration project that showcases real-time peer-to-peer communication capabilities across Web, Android, and iOS platforms. The project implements modern WebRTC APIs (currently using M125) to enable video calls, audio communication, screen sharing, and data channel messaging between multiple clients.

## Architecture

### High-Level Architecture

The project follows a client-server architecture with the following components:

```
┌─────────────────────────────────────────────────────────────┐
│                       Clients                                │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐              │
│  │   Web    │    │  Android │    │   iOS    │              │
│  │ (Vue.js) │    │  (Native)│    │ (Native) │              │
│  └─────┬────┘    └─────┬────┘    └─────┬────┘              │
│        │               │               │                     │
│        └───────────────┴───────────────┘                     │
│                        │                                     │
│                        ▼                                     │
│              ┌─────────────────┐                            │
│              │  Signaling      │                            │
│              │  Server         │                            │
│              │  (Node.js +     │                            │
│              │   Socket.io)    │                            │
│              └─────────────────┘                            │
│                                                              │
│        P2P Connection (WebRTC - STUN/TURN)                  │
│        ════════════════════════════════════                  │
│        Client A ◄──────────────────► Client B               │
└─────────────────────────────────────────────────────────────┘
```

### Components

#### 1. Signaling Server (`signaling-server/`)

**Technology Stack:**
- Node.js
- Express.js
- Socket.io

**Purpose:**
The signaling server facilitates the initial peer discovery and exchange of connection information (SDP offers/answers and ICE candidates) between WebRTC clients. It does not handle the actual media streaming.

**Key Features:**
- Room-based architecture (max 2 participants per room)
- WebSocket-based real-time communication
- SDP offer/answer exchange
- ICE candidate relay
- Room management and participant tracking

**Protocol Flow:**
1. **Join Room**: Client connects and joins a room by ID
2. **Offer Exchange**: First client creates offer when second client joins
3. **Answer Exchange**: Second client responds with answer
4. **ICE Candidate Exchange**: Both peers exchange ICE candidates
5. **P2P Connection**: Direct peer-to-peer connection established

**Server Events:**
- `join room` - Client joins a specific room
- `offer` - Send WebRTC offer to peer
- `answer` - Send WebRTC answer to peer
- `new ice candidate` - Exchange ICE candidates
- `receive encryption key` - Exchange E2EE keys (web only)
- `send data channel message` - Relay data channel messages
- `leave room` - Leave the current room

#### 2. Web Client (`web/`)

**Technology Stack:**
- Vue.js 3 (Composition API)
- TypeScript
- Vite (build tool)
- Socket.io-client
- Native WebRTC API

**Architecture:**
```
App.vue
├── WebRTC PeerConnection Management
├── Socket.io Connection
├── Media Stream Management
│   ├── Local Camera/Microphone
│   ├── Screen Sharing
│   └── Remote Stream
├── Data Channel
└── E2EE (End-to-End Encryption)
    ├── Main Thread Implementation
    └── Web Worker Implementation
```

**Key Features:**

1. **Audio/Video Control**
   - Toggle microphone on/off
   - Toggle camera on/off
   - Switch between front/back camera (mobile)
   - Switch between microphone and speaker output

2. **Screen Sharing**
   - Desktop/tab/window sharing via `getDisplayMedia()`
   - Automatic track replacement in peer connection
   - Switch back to camera functionality

3. **Data Channel Messaging**
   - Real-time text messaging
   - Low-latency data transfer
   - Chat interface for peer-to-peer communication

4. **End-to-End Encryption (E2EE)**
   - **Current Status**: Only supported for Web-Web communication
   - **Algorithm**: AES-GCM with 256-bit keys
   - **Implementation Options**:
     - Main thread encryption/decryption
     - Web Worker-based encryption (offload processing)
   
   **E2EE Architecture**:
   ```
   Sender                           Receiver
   ──────                           ────────
   Media Frame
      │
      ▼
   Generate IV (12 bytes)
      │
      ▼
   AES-GCM Encrypt
      │
      ▼
   Append IV to encrypted data
      │
      ▼
   Send via WebRTC ──────────►  Extract IV
                                    │
                                    ▼
                                AES-GCM Decrypt
                                    │
                                    ▼
                                Render Frame
   ```

   **E2EE Process**:
   - Initiator generates AES-GCM encryption key
   - Key is exported and sent to remote peer via signaling server
   - Both peers use Insertable Streams API to:
     - Encrypt outgoing media frames
     - Decrypt incoming media frames
   - Each frame gets unique IV (Initialization Vector)
   - **Debug Mode**: `shouldSendEncryptionKey=false` shows raw encrypted video

   **Files**:
   - `e2ee.ts` - Encryption/decryption stream transformers
   - `encryptionWorker.ts` - Web Worker for offloading crypto operations

5. **Media Streams API Integration**
   - `getUserMedia()` for camera/microphone access
   - `getDisplayMedia()` for screen sharing
   - Track replacement for switching sources

#### 3. iOS Client (`ios/`)

**Technology Stack:**
- Swift
- WebRTC framework (native)
- UIKit
- ReplayKit (for screen broadcasting)

**Project Structure:**
```
WebRTCDemo (Main App)
├── AppDelegate.swift
├── SceneDelegate.swift
├── ViewController.swift (Main UI)
├── CallViewController.swift (Call management)
├── PeerConnectionClient.swift (WebRTC logic)
├── CameraSession.swift (Camera management)
└── RTCCustomFrameCapturer.swift (Custom video capture)

WebRTCDemoScreenBroadcast (Broadcast Extension)
└── SampleHandler.swift (Screen capture handler)

WebRTCDemoScreenBroadcastSetupUI (Broadcast Setup)
└── BroadcastSetupViewController.swift
```

**Key Components:**

1. **PeerConnectionClient.swift**
   - Manages RTCPeerConnection lifecycle
   - Handles ICE candidate generation and exchange
   - Video/audio track management
   - Data channel creation and messaging
   - Camera switching (front/back)
   - Local and remote video rendering

2. **CallViewController.swift**
   - Main call UI controller
   - WebSocket connection management
   - Room joining/leaving logic
   - UI controls for audio/video/data channel

3. **Screen Broadcasting**
   - Uses ReplayKit framework
   - Broadcast Extension for system-level screen capture
   - Separate process for privacy and security

**Key Features:**
- Native WebRTC SDK integration
- Camera switching
- Audio routing (speaker/earpiece)
- Data channel messaging
- **Full-screen broadcasting (via Broadcast Extension)**
- Unix domain socket communication between extension and main app
- Automatic track switching for screen sharing

**Screen Broadcasting Architecture:**
- Uses `RPBroadcastSampleHandler` for system-wide screen capture
- Broadcast Extension captures and encodes frames to JPEG
- Unix domain socket (via App Group) transfers frames to main app
- `FlutterBroadcastScreenCapturer` feeds frames into WebRTC
- Darwin notifications coordinate lifecycle between processes
- Automatic camera ↔ screen track switching

**Configuration:**
- Server URL configured in `CallViewController.swift`
- Minimum deployment target considerations for dependencies

#### 4. Android Client (`android/`)

**Technology Stack:**
- Kotlin/Java (native Android)
- WebRTC framework for Android
- Gradle build system

**Project Structure:**
```
android/
├── app/
│   ├── build.gradle.kts
│   ├── proguard-rules.pro
│   └── src/
│       └── main/
│           ├── AndroidManifest.xml
│           ├── java/
│           └── res/
├── build.gradle.kts
├── settings.gradle.kts
└── gradle/
```

**Key Features:**
- Native WebRTC integration for Android
- Camera and microphone access
- Screen sharing (device screen capture)
- Data channel messaging
- Audio/video controls

**Configuration:**
- Server address configured in `res/values/strings.xml`
- Permissions for camera, microphone, and internet in AndroidManifest.xml

## WebRTC Core Concepts

### Peer Connection Lifecycle

```
1. Initialize RTCPeerConnection
         │
         ▼
2. Add Local Media Tracks (getUserMedia)
         │
         ▼
3. Create Offer (Caller) / Wait for Offer (Callee)
         │
         ▼
4. Set Local Description (SDP)
         │
         ▼
5. Send SDP to Remote Peer (via signaling)
         │
         ▼
6. Receive Remote SDP & Set Remote Description
         │
         ▼
7. Exchange ICE Candidates
         │
         ▼
8. ICE Connection Established
         │
         ▼
9. Media Flows Directly Between Peers
```

### ICE (Interactive Connectivity Establishment)

The project uses Google's public STUN server:
- STUN Server: `stun:stun.l.google.com:19302`
- Purpose: NAT traversal, discover public IP addresses
- No TURN server configured (may fail behind restrictive NATs)

### Data Channel

- **Use Case**: Text messaging between peers
- **Reliability**: Configurable (reliable/unreliable)
- **Ordering**: Can be ordered or unordered
- **Low Latency**: Direct P2P, bypasses signaling server

## Setup and Installation

### Prerequisites
- Node.js and npm/yarn
- iOS: Xcode, CocoaPods
- Android: Android Studio, Android SDK
- Modern web browser with WebRTC support

### Quick Start

1. **Start Signaling Server**
   ```bash
   cd signaling-server
   npm install
   npm run dev
   ```
   Server runs on port 4000 and displays local IP address

2. **Start Web Client**
   ```bash
   cd web
   npm install
   npm run dev
   ```
   Available at `http://localhost:5173`

3. **iOS Setup**
   ```bash
   cd ios
   pod install
   ```
   - Open `WebRTCDemo.xcworkspace` in Xcode
   - Update `SERVER_URL` in `CallViewController.swift`
   - Build and run

4. **Android Setup**
   - Update `serverAddress` in `android/app/src/main/res/values/strings.xml`
   - Open project in Android Studio
   - Build and run

### Usage Flow

1. Start signaling server and note the IP address
2. Launch any two clients (Web, iOS, or Android)
3. Enter the same **Room ID** on both clients
4. First client waits in the room
5. Second client joins and connection is established
6. Start communicating with audio, video, and messages

## Technical Deep Dive

### End-to-End Encryption Implementation (Web)

The E2EE implementation uses the **Insertable Streams API** (also known as WebRTC Encoded Transform):

```typescript
// For sender (encoding)
const senderStreams = sender.createEncodedStreams();
encryptStream(encryptionKey, 
              senderStreams.readable, 
              senderStreams.writable);

// For receiver (decoding)
const receiverStreams = receiver.createEncodedStreams();
decryptStream(encryptionKey, 
              receiverStreams.readable, 
              receiverStreams.writable);
```

**Transform Pipeline**:
- Intercept encoded video/audio frames
- Apply AES-GCM encryption/decryption
- IV (Initialization Vector) is prepended to each frame
- Frames remain opaque to intermediaries

**Worker Architecture** (optional):
```
Main Thread                    Encryption Worker
───────────                    ─────────────────
Generate Key ──────────────► Store Key
                              
Encoded Frame ────────────► Encrypt Frame
                              │
                              ▼
Encrypted Frame ◄───────── Return Encrypted
```

### Room Management

Rooms are temporary and in-memory:
- **Maximum Capacity**: 2 participants
- **Lifecycle**: Created when first user joins, implicitly destroyed when empty
- **Identification**: String-based room IDs
- **Validation**: Prevents duplicate joins, full room notifications

### Media Stream Handling

**Web Client Flow**:
1. Request permissions: `navigator.mediaDevices.getUserMedia()`
2. Create local stream with constraints (resolution, frame rate)
3. Add tracks to peer connection: `peerConnection.addTrack()`
4. Display local preview: `videoElement.srcObject = stream`
5. Receive remote stream via `track` event
6. Render remote video

**Native Clients**:
- Use platform-specific WebRTC SDK APIs
- Similar flow with native API equivalents
- Hardware acceleration for encoding/decoding

## Features Comparison

| Feature                    | Web | Android | iOS |
|---------------------------|-----|---------|-----|
| Audio Control             | ✅  | ✅      | ✅  |
| Video Control             | ✅  | ✅      | ✅  |
| Camera Switching          | ✅  | ✅      | ✅  |
| Speaker/Microphone Toggle | ✅  | ✅      | ✅  |
| Screen Sharing            | ✅  | ✅      | ✅  |
| Data Channel Messaging    | ✅  | ✅      | ✅  |
| End-to-End Encryption     | ✅  | 🚧      | 🚧  |
| Stream Video File         | 🚧  | 🚧      | 🚧  |

✅ = Supported | ❌ = Not Supported | 🚧 = In Progress

## Known Issues & Troubleshooting

### iOS Issues

**Issue 1: Minimum Deployment Target**
- **Error**: "Compiling for iOS 11.0, but module..."
- **Solution**: Update pod's Minimum Deployment to latest iOS version in Xcode project settings

**Issue 2: Sandbox rsync Error**
- **Error**: `Sandbox: rsync.samba(13105)...`
- **Solution**: Set `ENABLE_USER_SCRIPT_SANDBOXING` to `No` in Xcode build options

### General Issues

- **Connection Fails**: Check firewall settings, ensure signaling server is accessible
- **No Video/Audio**: Verify permissions granted for camera/microphone
- **NAT Traversal**: May need TURN server for restrictive networks
- **E2EE Performance**: Use Web Worker mode to prevent UI freezing

## Future Enhancements

1. **E2EE for Native Platforms**
   - Android: Implement frame encryption using native WebRTC frame cryptor
   - iOS: Similar native implementation

2. **Stream Video Files**
   - Play video files to remote peer
   - Use `RTCVideoSource` with custom capturer

3. **Multi-Party Conferencing**
   - Mesh, SFU, or MCU architecture
   - More than 2 participants per room

4. **Recording**
   - Record calls locally
   - Server-side recording option

5. **TURN Server Integration**
   - Better connectivity in restrictive networks
   - Fallback for failed STUN connections

6. **Advanced Features**
   - Simulcast for adaptive bitrate
   - SVC (Scalable Video Coding)
   - Background blur/virtual backgrounds
   - Noise suppression

## Security Considerations

### Current Implementation

- **E2EE**: Only web-to-web, prevents MITM attacks on media
- **Signaling**: Unencrypted WebSocket (not production-ready)
- **Authentication**: No user authentication implemented
- **Room Access**: Anyone with room ID can join

### Production Recommendations

1. **Use WSS (WebSocket Secure)** for signaling
2. **Implement authentication** (JWT, OAuth)
3. **Room access control** with passwords or invitations
4. **HTTPS** for web client
5. **Complete E2EE** across all platforms
6. **Validate and sanitize** all inputs
7. **Rate limiting** on signaling server

## Development Guidelines

### Code Organization

- **Separation of Concerns**: WebRTC logic separated from UI
- **Reusability**: Platform-specific wrappers around WebRTC
- **Error Handling**: Comprehensive error handling for network issues
- **Logging**: Debug logs for troubleshooting

### Testing Strategies

1. **Local Testing**: Multiple browser tabs/windows
2. **Network Testing**: Different networks (WiFi, cellular)
3. **Cross-Platform**: Test web-to-native, native-to-native
4. **Load Testing**: Signaling server under multiple rooms
5. **Edge Cases**: Disconnections, reconnections, poor network

## Contributing

When contributing to this project:

1. Follow the existing code style
2. Test across all platforms when possible
3. Update this documentation for significant changes
4. Handle errors gracefully
5. Add logging for debugging purposes

## License

Refer to the LICENSE file in the repository root.

## Disclaimer

This project is intended for **educational and demonstration purposes** to showcase WebRTC capabilities across multiple platforms. It may contain bugs and is not production-ready. Use with caution in production environments.

---

**Last Updated**: January 2026  
**WebRTC Version**: M125  
**Maintainer**: Project repository maintainers

---
> Source: [maitrungduc1410/WebRTC-Demo](https://github.com/maitrungduc1410/WebRTC-Demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
