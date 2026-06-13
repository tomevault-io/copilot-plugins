## callcenter

> A voice-based call center application that enables real-time conversations between users and an AI agent powered by Claude. The system handles the complete pipeline: call reception → speech-to-text → LLM processing → text-to-speech.

# Voice Call Center AI - Development Guide

## Project Overview

A voice-based call center application that enables real-time conversations between users and an AI agent powered by Claude. The system handles the complete pipeline: call reception → speech-to-text → LLM processing → text-to-speech.

## Tech Stack

- **FastAPI**: Web framework for handling Twilio webhooks
- **Twilio**: VoIP and telephony
- **Deepgram**: Speech-to-text transcription
- **Claude API**: Large language model for intelligent responses
- **ElevenLabs**: Text-to-speech voice synthesis

## Project Structure

```
voice-call-center/
├── main.py                 # FastAPI application entry point
├── requirements.txt        # Python dependencies
├── .env.example           # Environment variables template
├── config/
│   ├── __init__.py
│   └── settings.py        # Configuration management
├── services/
│   ├── __init__.py
│   ├── speech_to_text.py  # Deepgram integration
│   ├── llm.py             # Claude API integration
│   └── text_to_speech.py  # ElevenLabs integration
└── utils/
    ├── __init__.py
    └── helpers.py         # Utility functions
```

## Setup Instructions

1. **Install dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

2. **Create .env file** from template:
   ```bash
   cp .env.example .env
   # Fill in your API credentials
   ```

3. **Run the application**:
   ```bash
   python main.py
   ```

4. **Expose to internet** (for Twilio webhooks):
   ```bash
   ngrok http 8000
   ```

5. **Configure Twilio webhook**:
   - Set your webhook URL in Twilio dashboard to: `https://your-ngrok-url/incoming-call`

## Development Notes

### Architecture Overview

```
Twilio Webhook
    ↓
FastAPI Endpoint (/incoming-call)
    ↓
CallManager
    ├→ LLMService (Claude)
    ├→ SpeechToTextService (Deepgram)
    ├→ TextToSpeechService (ElevenLabs)
    ├→ MonitoringService
    └→ CallDataStorage
    ↓
TwiML Response (XML)
    ↓
Twilio → User Audio
```

### Key Components

1. **CallManager** (`services/call_manager.py`)
   - Manages call state and conversation history
   - Orchestrates service calls
   - Tracks active calls

2. **LLMService** (`services/llm.py`)
   - Maintains conversation context
   - Generates intelligent responses
   - Keeps responses concise for voice

3. **MonitoringService** (`services/monitoring.py`)
   - Tracks call metrics
   - Records errors and quality issues
   - Provides health status

4. **CallDataStorage** (`services/storage.py`)
   - Persists call data to JSON
   - Enables analytics and auditing
   - Supports CSV export

### System Prompts

Different call center scenarios available:
- `customer_service` - General support
- `technical_support` - Technical assistance
- `appointment_scheduling` - Schedule appointments
- `billing` - Billing inquiries
- `sales` - Product sales support

Switch via `CALL_CENTER_TYPE` environment variable.

### Latency Optimization

Current implementation:
- Sequential processing: Transcribe → Process → Respond
- Average response time: ~2-3 seconds
- Suitable for customer service interactions

For further optimization:
- Stream text-to-speech response as LLM generates
- Implement VAD (Voice Activity Detection)
- Cache common responses
- Use prompt caching for frequently used system prompts

### Error Handling

The application handles:
- Low confidence transcriptions (< 30%)
- Missing or malformed requests
- API failures (fallback responses)
- Network timeouts

All errors are logged to console with call_sid for debugging.

### Testing

Manual testing:
```bash
# Health check
curl http://localhost:8000/health

# View active calls
curl http://localhost:8000/calls

# Get call details
curl http://localhost:8000/calls/{call_sid}
```

Interactive testing:
- Use Twilio Console to simulate calls
- Check API docs at `/docs`

### Performance Considerations

- Max tokens per response: 300 (keeps responses natural)
- Timeout per turn: 5 seconds for user input
- Max call duration: 1 hour
- Concurrent call limit: Based on uvicorn workers

### Future Enhancements

- [ ] Real-time conversation streaming with partial responses
- [ ] Voice Activity Detection for natural turn-taking
- [ ] Call transfer to human agents
- [ ] Sentiment analysis
- [ ] Multi-language support
- [ ] Integration with CRM systems
- [ ] Advanced analytics dashboard

## API Endpoints

- `GET /health` - Health check
- `POST /incoming-call` - Initial call handler
- `POST /process-speech` - Process user input and generate response

## Testing

Use curl or Postman to test endpoints locally before configuring Twilio:
```bash
curl http://localhost:8000/health
```

## Monitoring & Logging

- Logs are printed to stdout
- Call data should be logged for analytics and debugging
- Monitor API rate limits for Deepgram and ElevenLabs

---
> Source: [bhobho/callcenter](https://github.com/bhobho/callcenter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-13 -->
