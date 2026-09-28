## bodhika

> **Name**: Bodhika Server

# Bodhika Server - Project Context for Claude

## Project Overview
**Name**: Bodhika Server
**Purpose**: WebSocket server for Bodhika AI Science Teacher that connects ESP32 hardware devices with Google's Gemini Live Audio API
**Type**: Node.js WebSocket Bridge Server
**Version**: 1.0.0

## Core Functionality
This server acts as a bridge between ESP32 hardware devices and Google's Gemini AI, enabling:
- Real-time bidirectional audio streaming
- AI-powered science teaching for 6th-grade students
- Vector-based textbook content search
- Session management for multiple connected devices

## Technology Stack
- **Runtime**: Node.js 18+ (ES Modules)
- **WebSocket Library**: `ws` v8.18.0
- **AI Integration**: Google Gemini Live Audio API (`@google/genai` v1.15.0)
- **Audio Format**: PCM16 (16kHz input, 24kHz output)
- **Other Dependencies**:
  - `dotenv` for environment configuration
  - `uuid` for session management
  - `bufferutil` & `utf-8-validate` for WebSocket performance

## Project Structure
```
server/
├── src/
│   ├── index.js              # Main server entry point
│   ├── audio/
│   │   └── bridge.js          # ESP32-Gemini audio bridge
│   ├── gemini/
│   │   ├── client.js          # Gemini Live API client
│   │   └── session.js         # Session management
│   ├── vector/
│   │   └── store.js           # Vector search for textbook content
│   ├── utils/                 # (Currently empty)
│   └── websocket/             # (Currently empty)
├── data/
│   ├── textbook.json          # Indexed textbook content
│   └── system-prompt.md       # AI teacher instructions
├── .env.local                 # Environment configuration
├── package.json               # Project dependencies
└── README.md                  # Basic documentation
```

## Key Components

### 1. Main Server (`src/index.js`)
- **Class**: `BodhikaServer`
- **Port**: 3001 (configurable via `WS_PORT` env)
- **Features**:
  - WebSocket server initialization
  - Device handshake validation
  - Connection management
  - Textbook data loading
  - Graceful shutdown handling

### 2. ESP32-Gemini Bridge (`src/audio/bridge.js`)
- **Class**: `ESP32GeminiBridge`
- **Purpose**: Manages bidirectional communication between ESP32 devices and Gemini
- **Key Methods**:
  - `handleESP32Connection()`: Establishes device connection
  - `handleAudioInput()`: Processes incoming audio from ESP32
  - `sendAudioToESP32()`: Sends Gemini responses back to device

### 3. Gemini Client (`src/gemini/client.js`)
- **Class**: `GeminiLiveClient`
- **Model**: `gemini-2.0-flash-exp` (configurable)
- **Voice**: "Orus" (configurable)
- **Features**:
  - Live audio session creation
  - Tool integration (textbook search)
  - Audio streaming (send/receive)
  - System prompt loading

### 4. Session Manager (`src/gemini/session.js`)
- **Class**: `SessionManager`
- **Purpose**: Manages Gemini sessions for connected devices
- **Features**:
  - Session lifecycle management
  - Tool call handling (textbook search)
  - Audio queue management
  - Automatic cleanup of inactive sessions

### 5. Vector Store (`src/vector/store.js`)
- **Class**: `VectorStore`
- **Purpose**: Enables semantic search through textbook content
- **Features**:
  - JSON data loading
  - Text-based relevance scoring
  - Chapter/section filtering
  - Support for multiple data formats

## Communication Flow
1. **ESP32 → Server**: Device connects via WebSocket with device ID
2. **Handshake**: Device sends capabilities (audio format, sample rate)
3. **Session Creation**: Server creates Gemini session for device
4. **Audio Input**: ESP32 sends PCM16 audio at 16kHz
5. **AI Processing**: Gemini processes audio with textbook context
6. **Audio Output**: Gemini responds with PCM audio at 24kHz
7. **Response Delivery**: Server forwards audio back to ESP32

## Message Protocol

### From ESP32:
```javascript
{
  type: 'handshake' | 'audio_input' | 'control' | 'status',
  data: {
    // Type-specific payload
  }
}
```

### To ESP32:
```javascript
{
  type: 'handshake_ack' | 'audio_output' | 'control' | 'error',
  data: {
    // Type-specific payload
  }
}
```

## Environment Configuration
Required environment variables (in `.env.local`):
- `GEMINI_API_KEY`: Google AI API key
- `WS_PORT`: WebSocket server port (default: 3001)
- `MODEL_NAME`: Gemini model (default: gemini-2.0-flash-exp)
- `VOICE_NAME`: Voice for audio output (default: Orus)
- `SESSION_TIMEOUT`: Session timeout in ms (default: 900000)
- `MAX_SESSIONS`: Maximum concurrent sessions (default: 100)

## AI Teaching Context
The server implements "Bodhika", an AI Science Teacher with:
- Target audience: 6th-grade students at Sacred Heart CMI Public School
- Language: Simple English (ESL-friendly) with Malayalam support
- Teaching style: Warm, patient, encouraging
- Content source: 6th-grade science textbook (indexed in `textbook.json`)
- Interactive features: Student name calls, real-world examples, doubt clearing

## Development Commands
```bash
npm start     # Start production server
npm run dev   # Start with file watching (--watch mode)
```

## Testing & Validation
- No test suite currently implemented
- Manual testing via ESP32 hardware connection
- WebSocket connection test: `ws://localhost:3001?device_id=test`

## Important Notes

### Audio Specifications
- **Input**: PCM16 format, 16kHz sample rate, base64 encoded
- **Output**: PCM format, 24kHz sample rate, base64 encoded
- **Streaming**: Real-time bidirectional audio streaming

### Security Considerations
- API keys stored in environment variables
- No authentication mechanism currently implemented
- WebSocket connections accept any device ID

### Performance
- Automatic session cleanup every 60 seconds
- 30-second WebSocket ping interval for keepalive
- Message queue for audio buffering
- Optimized WebSocket performance with bufferutil

### Data Dependencies
- Requires `textbook.json` in `/data` directory
- System prompt in `/data/system-prompt.md`
- Server runs without textbook data but with limited functionality

## Common Issues & Solutions

1. **Missing textbook.json**:
   - Server runs but without search capability
   - Solution: Copy from indexer project to `/data/textbook.json`

2. **Session creation failure**:
   - Check GEMINI_API_KEY is valid
   - Verify network connectivity to Google APIs

3. **Audio not playing on ESP32**:
   - Verify audio format compatibility
   - Check WebSocket message size limits

## Future Enhancements
- [ ] Add authentication/authorization
- [ ] Implement proper logging system
- [ ] Add metrics and monitoring
- [ ] Create test suite
- [ ] Add WebSocket reconnection logic
- [ ] Implement audio compression
- [ ] Add multi-language support beyond Malayalam
- [ ] Create admin dashboard for session monitoring

## Related Projects
- **Indexer**: Separate application for generating `textbook.json`
- **ESP32 Client**: Hardware device running WebSocket client
- **Web Interface**: Planned admin dashboard (not yet implemented)

## Contact & Support
This is an educational project for Sacred Heart CMI Public School, Thevara, Ernakulam.
The AI teacher "Bodhika" is designed specifically for 6th-grade science education.

---
> Source: [cynerellc/bodhika](https://github.com/cynerellc/bodhika) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
