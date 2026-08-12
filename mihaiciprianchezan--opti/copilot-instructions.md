## opti

> - KSCM—Keep It Simple, Smart, Clean, and Maintainable

# Coding Philosophy

- KSCM—Keep It Simple, Smart, Clean, and Maintainable
- Senior-level code—Don't dumb it down
- Simple error handling—Just log with traceback
- Intelligent complexity—Use it when needed, not for its own sake
- Simple, Direct, No unnecessary complexity, Respects the reader's intelligence

## Core Principles

**Keep It Simple, Smart, Clean, and Maintainable — KSCM**

### 1. **Smart-Simple, Not Over-Engineered**
- **Goal**: Maximum functionality with minimum complexity
- **Approach**: Write intelligently to achieve more with less code
- **Avoid**: Enterprise-level abstractions, unnecessary layers, premature optimization

### 2. **Single Developer + AI Team**
- **Team Size**: One human developer and AI assistants
- **Implication**: No need for enterprise patterns, complex abstractions, or team coordination overhead
- **Focus**: Direct, readable, maintainable code that one person can understand and control

### 3. **Maintainability Over Scalability**
- **Priority**: Code that's straightforward to understand, modify, and debug
- **Avoid**: Complex dependency injection, service meshes, microservices
- **Prefer**: Simple functions, clear data flow, minimal abstractions

## Code Quality Standards

**Readability**
Senior Software Engineer with 10+ year Python programming experience, no need to make the code stupid just to make it understandable to less experienced persons. But the code and the flows should be logical and understandable.
**Error Handling**
Unified, standardized, all exception are logged/printed with "The error message: {e}, {traceback.format_exec()}"

## Architecture

```
src/
├── main.py              # Entry point — Qt main thread + shell thread
├── adapter/             # AgentAdapter ABC + backends (local_llm, claude, openai)
├── core/                # Config, EventBus, AgentShell orchestrator
├── ui/                  # UIBridge (event bus → Qt signals → Energy Star)
├── voice/               # VoiceIO (ASR + TTS with interruptible pygame playback)
├── energy/              # Energy Star ball UI (Qt widget + video player + color overlay)
├── speech/              # ASR via Vosk (SpeechRecognizer + ASRx worker)
├── llm/                 # LlamaCppServer + Inference manager
├── utils/               # AppLogger, Folders, TextCleaner, WorkerThread
└── constants/           # Precompiled regex patterns
```

## Key Design Decisions
- **Engine-per-utterance TTS**: Fresh pyttsx3 engine per text to avoid SAPI5 hang bug
- **save_to_file + pygame**: Interruptible TTS playback (user can speak over the agent)
- **SynchronousEventBus**: Pure threaded pub/sub, callbacks execute in publisher's thread
- **Qt aboutToQuit**: Background threads stop before Qt destroys widgets
- **TextCleaner pipeline**: markdown → HTML → text → emojis → whitespace (mistune + bs4)
- **_NON_SPEAKABLE_RE**: Strips mojibake/garbled chars before TTS

## Running
```
cd C:\development\opti
python src/main.py
```

---
> Source: [MihaiCiprianChezan/opti](https://github.com/MihaiCiprianChezan/opti) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
