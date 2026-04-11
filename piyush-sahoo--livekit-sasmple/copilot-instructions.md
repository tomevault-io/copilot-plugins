## livekit-sasmple

> Get your LiveKit voice agent running with Google Gemini in under 5 minutes!

# Quick Start Guide - Google Gemini Setup

Get your LiveKit voice agent running with Google Gemini in under 5 minutes!

## Prerequisites

- Python 3.9+ installed
- Google Gemini API key ([Get one free](https://aistudio.google.com/))
- LiveKit Cloud account ([Sign up free](https://cloud.livekit.io/))

## Setup Steps

### 1. Environment Setup

```bash
# Navigate to project
cd livekit-agent

# Create virtual environment
python -m venv .venv

# Activate it
.venv\Scripts\activate  # Windows
# source .venv/bin/activate  # macOS/Linux

# Install dependencies
pip install -r requirements-dev.txt
```

### 2. Configure API Keys

Copy the example environment file:

```bash
cp .env.example .env.local
```

Edit `.env.local` and add your credentials:

```bash
LIVEKIT_URL=wss://your-project.livekit.cloud
LIVEKIT_API_KEY=your-livekit-api-key
LIVEKIT_API_SECRET=your-livekit-api-secret

GOOGLE_API_KEY=your-gemini-api-key
```

**Get your keys:**
- LiveKit: https://cloud.livekit.io/projects
- Gemini: https://aistudio.google.com/app/apikey

### 3. Download Required Models

```bash
python src/agent.py download-files
```

This downloads VAD and turn detection models (one-time setup).

### 4. Run the Agent

**Option A: Console Mode (Talk in Terminal)**

```bash
python src/agent.py console
```

Speak directly with your AI agent in the terminal!

**Option B: Dev Mode (For Frontend Integration)**

```bash
python src/agent.py dev
```

Then connect from:
- Web: https://agents-playground.livekit.io/
- Mobile: Use LiveKit example apps
- Telephony: Configure SIP integration

**Option C: Production Mode**

```bash
python src/agent.py start
```

## What's Configured

Your agent uses the full Google Gemini stack:

- **LLM**: Gemini 2.0 Flash (ultra-fast for real-time)
- **STT**: Google Speech-to-Text (streaming enabled)
- **TTS**: Google Text-to-Speech (female neural voice)
- **VAD**: Silero (optimized for 100ms detection)
- **Turn Detection**: Multilingual model

## Test Your Agent

### Quick Voice Test

1. Run: `python src/agent.py console`
2. Say: "Hello, what can you help me with?"
3. Agent responds as "Riya" from Amazon Customer Care

### Test Function Calling

1. Say: "What's the weather in Tokyo?"
2. Agent calls the `lookup_weather` function
3. Returns result in natural language

## Customization

### Change Voice Speed

In [src/agent.py:390](src/agent.py#L390):

```python
speaking_rate=1.1,  # Change to 0.8 (slower) or 1.5 (faster)
```

### Change Model for Quality vs Speed

In [src/agent.py:369](src/agent.py#L369):

```python
model="gemini-2.0-flash-001",  # Fast (current)
# model="gemini-2.5-pro-preview-05-06",  # Slower but more capable
# model="gemini-2.0-flash-lite-preview-02-05",  # Ultra-fast
```

### Add More Languages

In [src/agent.py:377](src/agent.py#L377):

```python
languages="en-US",  # English only (current)
# languages=["en-US", "hi-IN", "es-ES"],  # Multiple languages
# detect_language=True,  # Enable auto-detection
```

## Troubleshooting

### API Key Issues

**Error**: `google.api_core.exceptions.Unauthenticated`

**Fix**: Check your `GOOGLE_API_KEY` in `.env.local`

```bash
# Verify it's set
python -c "import os; from dotenv import load_dotenv; load_dotenv('.env.local'); print(os.getenv('GOOGLE_API_KEY'))"
```

### Import Errors

**Error**: `ModuleNotFoundError: No module named 'livekit'`

**Fix**: Reinstall dependencies

```bash
pip install -r requirements-dev.txt
```

### Models Not Found

**Error**: "Models not found" or VAD errors

**Fix**: Download models again

```bash
python src/agent.py download-files
```

### LiveKit Connection Issues

**Error**: Connection timeout or "Failed to connect"

**Fix**: Verify LiveKit credentials

```bash
# Test connection
python -c "import os; from dotenv import load_dotenv; load_dotenv('.env.local'); print('URL:', os.getenv('LIVEKIT_URL')); print('Key:', os.getenv('LIVEKIT_API_KEY')[:10] + '...')"
```

## Next Steps

1. **Customize the Agent**: Edit the instructions in [src/agent.py:32-328](src/agent.py#L32-L328)
2. **Add Functions**: Create new `@function_tool` decorated methods
3. **Optimize Latency**: See [LATENCY_OPTIMIZATIONS.md](LATENCY_OPTIMIZATIONS.md)
4. **Advanced Config**: See [GEMINI_CONFIGURATION.md](GEMINI_CONFIGURATION.md)
5. **Build Frontend**: Use [example React app](https://github.com/livekit-examples/agent-starter-react)

## Performance Expectations

With the current Gemini configuration:

- **First response**: ~1-2 seconds
- **Subsequent responses**: ~0.8-1.5 seconds
- **STT latency**: ~150-300ms
- **LLM latency**: ~400-800ms
- **TTS latency**: ~200-400ms

For ultra-low latency (< 1 second total), use:
- `model="gemini-2.0-flash-lite-preview-02-05"`
- `max_output_tokens=100`
- `speaking_rate=1.3`

## Resources

- [Gemini API Docs](https://ai.google.dev/docs)
- [LiveKit Agents Docs](https://docs.livekit.io/agents/)
- [Full Configuration Guide](GEMINI_CONFIGURATION.md)
- [Testing Guide](tests/test_agent.py)

## Support

- **LiveKit Discord**: https://livekit.io/discord
- **GitHub Issues**: Report problems in your fork
- **Documentation**: https://docs.livekit.io/

---

**Ready to go!** Run `python src/agent.py console` and start talking to your AI agent! 🎙️

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Piyush-sahoo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Piyush-sahoo)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
