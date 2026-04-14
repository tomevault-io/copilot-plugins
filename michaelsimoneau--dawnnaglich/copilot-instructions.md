## dawnnaglich

> This document describes the Google Gemini AI integration used for voice assistants in the Dawn Naglich Wellness App.

# Google Gemini AI Integration

This document describes the Google Gemini AI integration used for voice assistants in the Dawn Naglich Wellness App.

## Overview

The app uses Google Gemini Live API to power two distinct AI voice assistants:
1. **Client Assistant ("Becky")** - Helps clients navigate services and schedule appointments
2. **Admin Voice Assistant ("Becky")** - Assists admins with appointment management and calendar operations

Both assistants provide real-time voice interactions with streaming audio support.

## Architecture

### Backend Functions

The following Firebase Cloud Functions handle Gemini AI interactions:

- `generateGeminiResponse` - Generates AI responses for text-based interactions
- `createGeminiLiveSession` - Creates a new Gemini Live API session for real-time voice interactions
- `proxyGeminiLiveMessage` - Proxies audio chunks and messages to Gemini Live API

### Frontend Components

- `components/ClientAssistant.tsx` - Client-facing voice assistant UI
- `components/AdminVoiceAssistant.tsx` - Admin-facing voice assistant UI
- `services/voiceService.ts` - Handles audio recording and streaming
- `services/voiceMessageBus.ts` - Manages voice message queue and state

## System Instructions

### Client Assistant Instructions

The client assistant is configured with the following system instruction (from `functions/src/systemInstructions.ts`):

```
You are Dawn Naglich's elite concierge. Dawn is a specialist in Muscle Activation (MAT).

CALENDAR AND AVAILABILITY PROTOCOL:
- When asked about availability, appointments, or scheduling, ALWAYS call getCalendarEvents first with appropriate date ranges
- Use the current date/time to calculate specific dates from relative terms like "next Tuesday", "this week", "next week"
- For availability queries, query at least 2 weeks ahead to show options
- After getting calendar events, analyze them to find available time slots
- Present availability in a clear, user-friendly format
- If no availability is found, suggest alternative dates or times

Keep answers concise, healing-focused, and encourage booking for realignment.
```

**Location**: `31005 Bainbridge Rd, Solon, OH 44139`
- Provides facility address and directions link when asked

### Admin Voice Assistant Instructions

The admin assistant ("Becky") is configured with:

```
You are 'Becky', the elite, intuitive digital concierge for Dawn Naglich Wellness.

BECKY'S PROTOCOL:
- You are the gatekeeper of Dawn's schedule. Dawn focuses on Muscle Activation and healing; you handle the logistics.
- Your primary tool is the calendar. Always check availability before suggesting times.
- You are warm, professional, and highly efficient. Use short, punchy sentences.
- PROTOCOL FOR CHANGES: When Dawn asks to book or cancel, you must confirm the details verbally ("Okay, booking Michael for Muscle Activation at 10 AM, shall I confirm?").
- EXECUTION: Only call 'confirmPendingAction' when Dawn gives explicit verbal confirmation.
- PRIVACY: You only share schedule details with Dawn or Michael.
- If the system is busy or has an error, politely inform Dawn and suggest checking the manual dashboard.
```

## Function Declarations

The assistants have access to the following calendar functions (defined in `functions/src/functionDeclarations.ts`):

### `getCalendarEvents`
Fetches calendar events for availability checking.

**Parameters**:
- `timeMin` (string, required) - Start time in ISO format
- `timeMax` (string, required) - End time in ISO format

**Usage**: Essential for context before booking or cancelling appointments.

### `createCalendarEvent`
Pre-schedules a session. Triggers a confirmation request to Dawn.

**Parameters**:
- `clientName` (string, required) - Name of the client
- `service` (string, required) - Service type (e.g., "Muscle Activation")
- `startTime` (string, required) - Start time in ISO format
- `endTime` (string, required) - End time in ISO format
- `summary` (string, required) - Brief summary of the action

**Usage**: Creates a pending appointment that requires admin confirmation.

### `cancelCalendarEvent`
Removes a scheduled session. Triggers a confirmation request to Dawn.

**Parameters**:
- `eventId` (string, required) - ID of the calendar event to cancel
- `summary` (string, required) - Brief description of which event is being removed

**Usage**: Cancels an appointment, requires admin confirmation.

### `confirmPendingAction`
Confirms a pending action (booking or cancellation).

**Parameters**: None

**Usage**: **ONLY** called when Dawn explicitly confirms the previous request (e.g., says "Yes", "Do it", "Confirm"). This is a safety mechanism to prevent accidental bookings or cancellations.

## Voice Streaming

### Audio Format
- **Sample Rate**: 16000 Hz
- **Channels**: Mono (1 channel)
- **Bit Depth**: 16-bit PCM
- **Encoding**: Base64 encoded PCM data
- **MIME Type**: `audio/pcm;rate=16000`

### Platform Support
- **Web**: Uses Web Audio API (`MediaRecorder` and `ScriptProcessorNode`)
- **iOS**: Uses custom `expo-audio-streaming` module with `AVAudioEngine`
- **Android**: Uses custom `expo-audio-streaming` module with `AudioRecord`

### Message Bus Architecture
The `voiceMessageBus` service manages:
- Audio chunk queuing
- Connection state management
- Message persistence
- Retry logic for failed requests

## Configuration

### Environment Variables

**Firebase Functions Secret**:
- `EXPO_PUBLIC_FIREBASE_GEMINI_API_KEY` - Google Gemini API key (stored as Firebase secret)

### API Endpoints

All Gemini-related functions are accessible via Firebase Hosting rewrites:
- `/api/generateGeminiResponse` - Text-based AI responses
- `/api/createGeminiLiveSession` - Create live voice session
- `/api/proxyGeminiLiveMessage` - Stream audio to Gemini Live API

## Usage Flow

### Client Assistant Flow

1. User opens client assistant from landing page
2. Assistant greets user and offers help with services, scheduling, or directions
3. User asks questions or requests to check availability
4. Assistant calls `getCalendarEvents` to check availability
5. Assistant presents available times in natural language
6. User can book an appointment (creates pending event)
7. Admin confirms via admin dashboard or voice assistant

### Admin Assistant Flow

1. Admin opens voice assistant from admin dashboard
2. Assistant greets admin ("Becky")
3. Admin can:
   - Check availability
   - Book appointments (requires verbal confirmation)
   - Cancel appointments (requires verbal confirmation)
4. For booking/cancellation:
   - Assistant calls appropriate function (`createCalendarEvent` or `cancelCalendarEvent`)
   - Assistant asks for explicit confirmation
   - Admin confirms verbally
   - Assistant calls `confirmPendingAction` to execute

## Security

- All functions require Firebase Authentication
- Admin functions check user email against admin list
- API key stored securely as Firebase secret
- Calendar operations use service account authentication
- Client data is masked for non-admin users

## Error Handling

- Connection failures are handled gracefully
- Audio streaming errors are caught and logged
- Failed API calls are retried automatically
- User-friendly error messages displayed in UI
- Fallback to manual dashboard suggested on errors

## Development Notes

### Testing Voice Assistants

1. Ensure `EXPO_PUBLIC_FIREBASE_GEMINI_API_KEY` is set in Firebase Functions secrets
2. Deploy functions: `pnpm deploy:functions`
3. Test on device/emulator (microphone permissions required)
4. Check Firebase Functions logs for errors

### Debugging

- Check browser console for Web Audio API errors
- Check Firebase Functions logs for backend errors
- Verify microphone permissions are granted
- Ensure network connectivity for API calls

## References

- [Google Gemini API Documentation](https://ai.google.dev/docs)
- [Gemini Live API](https://ai.google.dev/gemini-api/docs/live)
- System instructions: `functions/src/systemInstructions.ts`
- Function declarations: `functions/src/functionDeclarations.ts`
- Voice service: `services/voiceService.ts`
- Voice message bus: `services/voiceMessageBus.ts`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MichaelSimoneau)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MichaelSimoneau)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
