## bonzipony

> AI-powered Windows desktop pet: voice interaction, autonomous behavior, screen monitoring, desktop control, multi-pony system. Built on PyQt5, Whisper STT, ElevenLabs/PVT TTS, and multiple LLM backends.

# CLAUDE.md — bonziPONY Codebase Guide

AI-powered Windows desktop pet: voice interaction, autonomous behavior, screen monitoring, desktop control, multi-pony system. Built on PyQt5, Whisper STT, ElevenLabs/PVT TTS, and multiple LLM backends.

## Architecture Overview

```
main.py (bootstrap + wiring)
  │
  ├─ Activation Thread ─── wake word / PTT / double-click detection
  │       │
  │       └─ Pipeline Thread ─── IDLE → ACK → LISTEN → THINK → SPEAK → convo loop
  │               │
  │               ├─ stt/transcriber.py      Whisper STT
  │               ├─ llm/ providers          LLM call (chat/generate_once)
  │               ├─ llm/response_parser.py  extract tags from LLM output
  │               └─ core/tts_queue.py       enqueue speech (priority-ordered)
  │
  ├─ Agent Loop Thread ─── 1-3s ticks, autonomous behavior
  │       │
  │       ├─ core/screen_monitor.py    free window title polling (no LLM)
  │       ├─ directives.json           persistent tasks with urgency 1-10
  │       ├─ core/routines.py          scheduled recurring actions
  │       └─ core/event_timeline.py    shared event log (thread-safe)
  │
  ├─ TTSQueue Consumer Thread ─── serialized audio playback
  │       │
  │       └─ tts/ engines              ElevenLabs or OpenAI-compatible
  │
  └─ Main Thread (Qt) ─── GUI event loop
          │
          ├─ desktop_pet/pet_window.py     sprite animation (~60fps)
          ├─ desktop_pet/speech_bubble.py  comic-style response display
          ├─ desktop_pet/heard_text.py     STT transcription overlay
          └─ desktop_pet/context_menu.py   right-click settings UI
```

## Thread Safety Rules

| Component | Thread(s) | Sync mechanism |
|-----------|-----------|----------------|
| Pipeline | Activation thread spawns it | Isolated; one conversation at a time |
| Agent Loop | Own daemon thread | `_conversation_active` flag silences it during user interaction |
| TTSQueue | Any thread enqueues; consumer thread plays | `PriorityQueue` + `_seq_lock` |
| EventTimeline | Pipeline + Agent both write | `threading.Lock` on all reads/writes |
| TTS engine | Pipeline + TTSQueue both call speak() | **`_tts_lock` in main.py** wraps speak() |
| Qt GUI updates | Must happen on main thread | `PetController` uses Qt signals with `QueuedConnection` |

**Critical**: Never call Qt widget methods from background threads. Always go through PetController signals.

## File Ownership Map

### core/ — Brain and coordination
| File | Owns | Key class |
|------|------|-----------|
| `pipeline.py` | Conversation state machine (wake→listen→think→speak) | `Pipeline` |
| `agent_loop.py` | Autonomous behavior, directives, enforcement, AFK mischief | `AgentLoop` |
| `tts_queue.py` | Priority-ordered multi-pony audio serialization | `TTSQueue` |
| `pony_manager.py` | Multi-pony lifecycle, voice routing, group chat scheduling | `PonyManager` |
| `pony_instance.py` | Per-pony state bundle (GUI + LLM + sprites + config) | `PonyInstance` |
| `group_conversation.py` | Inter-pony turn-taking conversations | `GroupConversation` |
| `routines.py` | Persistent scheduled actions (wake/sleep/daily/weekly/interval) | `RoutineManager` |
| `event_timeline.py` | Shared event log bridging Pipeline and AgentLoop | `EventTimeline` |
| `screen_monitor.py` | Win32 window title polling (free, no API calls) | `ScreenMonitor` |
| `config_loader.py` | YAML config → typed dataclasses | `AppConfig` and sub-configs |
| `character_registry.py` | Scans Ponies/ dirs, maps slugs ↔ display names | `scan_ponies()` |
| `memory.py` | Session summaries persisted across restarts | `save_summary()`, `load_recent()` |
| `user_profile.py` | Extracted user facts (name, interests, events) | `load_profile()`, `update_from_conversation()` |
| `diary.py` | Per-character in-character journal | `write_entry()`, `read_recent()` |
| `monitor_utils.py` | Win32 multi-monitor bounds via ctypes | `get_monitor_rect_for_point()` |
| `audio_utils.py` | Audio device enumeration helpers | `list_pyaudio_devices()` |
| `updater.py` | Git-based self-update from GitHub | `check_for_updates()` |

### llm/ — LLM abstraction
| File | Owns |
|------|------|
| `base.py` | Abstract `LLMProvider` interface: `chat()`, `generate_once()`, `describe_image()` |
| `factory.py` | Provider routing: Anthropic, OpenAI, OpenRouter, DeepSeek, Groq, Ollama, local servers |
| `anthropic_provider.py` | Claude SDK with retry logic and vision support |
| `openai_provider.py` | OpenAI-compatible provider (handles 12+ backends) |
| `ollama_provider.py` | Local Ollama wrapper |
| `vision_provider.py` | Dedicated vision LLM with API key cycling (rate limit distribution) |
| `prompt.py` | System prompt generation from presets + relationship + user profile + desktop commands |
| `response_parser.py` | Tag extraction (`[ACTION]`, `[DESKTOP]`, `[DIRECTIVE]`, etc.) + TTS text sanitization |

### desktop_pet/ — GUI
| File | Owns |
|------|------|
| `pet_window.py` | Main transparent frameless always-on-top window, sprite rendering, roaming, drag |
| `pet_controller.py` | Thread-safe Qt signal bridge: pipeline thread → main thread |
| `sprite_manager.py` | GIF frame extraction, caching, scaling |
| `behavior_manager.py` | Parses `pony.ini` behavior definitions (CSV format from Desktop Ponies) |
| `effect_renderer.py` | Overlay visual effects (Sonic Rainboom, etc.) |
| `speech_bubble.py` | Comic-style bubble with typing animation, auto-hide, position tracking |
| `heard_text.py` | Translucent STT transcription overlay below pony |
| `context_menu.py` | Right-click menu: full in-app settings, character switching, directive viewer |
| `countdown_timer.py` | On-screen timer widget for enforcement tasks |

### stt/ — Speech-to-text
| File | Owns |
|------|------|
| `transcriber.py` | Whisper STT with energy-based VAD. Two modes: `listen()` (auto-silence) and `listen_ptt()` (push-to-talk) |
| `mic_lock.py` | Global threading.Lock preventing PyAudio heap corruption from concurrent init/exit |

### tts/ — Text-to-speech
| File | Owns |
|------|------|
| `elevenlabs_tts.py` | ElevenLabs cloud TTS via SDK. PCM playback via sounddevice |
| `openai_compatible_tts.py` | OpenAI-compatible `/v1/audio/speech` endpoint (ponyvoicetool, AllTalk, etc.). Built-in voice map for 25+ MLP characters |

### Other directories
| Directory | Purpose |
|-----------|---------|
| `wake_word/detector.py` | Whisper-based offline keyword spotting for per-character wake phrases |
| `robot/desktop_controller.py` | Windows desktop automation (pyautogui, pywin32). Security: blocked hotkeys, allowlisted apps |
| `robot/actions.py` | `RobotAction` enum (walk, sit, wave, volume, window ops) |
| `vision/screen.py` | Screenshot capture via mss |
| `vision/camera.py` | Webcam capture via OpenCV |
| `vision/watch_mode.py` | CLIP + OCR continuous screen understanding (zero API cost) |
| `acknowledgement/player.py` | Plays per-character beep/chime on wake word detection |
| `presets/` | Character personality .txt files (system prompts). `_template.txt` for auto-generation |
| `Ponies/` | 311+ Desktop Ponies sprite packs (pony.ini + GIFs) |
| `memory/` | `user_profile.txt`, `user_events.txt`, `sessions.txt` |
| `diary/` | Per-character journal files |
| `scripts/` | `list_audio_devices.py`, `test_pipeline.py` |

## Key Data Flow

### User speaks → pony responds
```
Wake word detected (wake_word/detector.py)
  → Pipeline.run_conversation()
    → AcknowledgementPlayer.play()           # immediate audio feedback
    → Transcriber.listen()                   # record + Whisper STT
    → LLMProvider.chat(user_text)            # LLM call with history
    → parse_response(raw)                    # extract tags + clean text
    → TTSQueue.enqueue(text, blocking=True)  # blocks until audio done
    → DesktopController.execute(commands)    # run [DESKTOP:...] tags
    → [conversation mode: wait for follow-up speech for timeout_s]
```

### Autonomous speech (agent loop)
```
AgentLoop.tick()
  → check directives, screen changes, idle time
  → LLMProvider.generate_once(context_prompt)
  → parse_response(raw)
  → AgentLoop._speak(text)                   # enqueue with PRIORITY_AUTONOMOUS, blocking=True
  → _listen_for_reply()                      # wait for user response
```

### Multi-pony group chat
```
PonyManager.trigger_inter_pony_chat()
  → GroupConversation.start(initiator)
    → For each turn: pony.llm.generate_once(turn_prompt)
    → TTSQueue.enqueue(text, priority=PRIORITY_SPONTANEOUS_CHAT)
    → Stop on [PASS] or max_depth reached
```

## LLM Response Tag System

The LLM embeds structured tags in its natural language response. `response_parser.py` extracts them and strips them from TTS text.

| Tag | Purpose | Example |
|-----|---------|---------|
| `[ACTION:name]` | Trigger sprite animation | `[ACTION:WALK_FORWARD]` |
| `[DESKTOP:cmd:args]` | Desktop automation | `[DESKTOP:BROWSE:youtube.com]`, `[DESKTOP:HOTKEY:ctrl:w]` |
| `[DIRECTIVE:goal:urgency]` | Create persistent task | `[DIRECTIVE:go to the gym:7]` |
| `[DIRECTIVE:goal:urgency:delay]` | Delayed directive | `[DIRECTIVE:take meds:8:30]` (30 min delay) |
| `[TIMER:HH:MM:action]` | One-shot scheduled action | `[TIMER:21:00:remind user to sleep]` |
| `[ROUTINE:schedule:goal:urgency]` | Recurring schedule | `[ROUTINE:daily:09:00:check calendar:5]` |
| `[ENFORCE:minutes]` | Monitor task completion | `[ENFORCE:15]` |
| `[DELAY:minutes:keyword]` | Postpone a directive | `[DELAY:30:gym]` |
| `[DONE:keyword]` | Mark directive complete | `[DONE:gym]` |
| `[CONVO:END\|CONTINUE]` | Conversation flow signal | `[CONVO:END]` |
| `[PERSIST:seconds]` | Hold animation N seconds | `[PERSIST:600]` |
| `[MOVETO:region]` | Move pony to screen area | `[MOVETO:top_left]` |
| `[RULE:description]` | Create standing behavioral rule | `[RULE:quit porn]` |

## Directive System

Directives are persistent goals stored in `directives.json`. Created by LLM via `[DIRECTIVE:goal:urgency]` tag.

- **Urgency 1-6**: Verbal nagging at timed intervals
- **Urgency 7-9**: High priority — shorter intervals, window shaking, closing distracting apps
- **Urgency 10**: Burst mode — nags every 15-45 seconds (for demos/presentations)
- Directives survive restarts. Agent loop checks and fires them each tick.
- Standing rules (`[RULE:...]`) are separate — regex patterns auto-matched against window titles every tick (no LLM call for detection).

## Configuration

- **`config.yaml`** — Main config. Sections: `llm`, `tts`, `stt`, `wake_word`, `conversation`, `vision`, `vision_llm`, `agent`, `desktop_control`, `audio`, `logging`
- **`directives.json`** — Active task directives + standing rules (managed by AgentLoop)
- **`routines.json`** — Recurring scheduled actions (managed by RoutineManager)
- **`wake_state.json`** — Tracks wake/sleep state across restarts
- **`presets/*.txt`** — Per-character system prompts. `_template.txt` for auto-generation
- **`memory/`** — User profile, events, session summaries (injected into system prompt at runtime)
- **`.env`** — Optional environment variable overrides for API keys

## Conventions and Patterns

### LLM provider interface
All providers implement `LLMProvider` (llm/base.py):
- `chat(user_message) → str` — Multi-turn with history (conversation mode)
- `generate_once(prompt, max_tokens, system_prompt) → str` — One-shot, no history impact (utility tasks)
- `describe_image(jpeg_bytes) → str | None` — Vision call
- `inject_history(user_msg, assistant_msg)` — Add exchange without API call
- `reset_history()` — Clear conversation state

### TTS blocking semantics
- `blocking=True` → caller blocks until audio finishes (used for user-response path so mic doesn't reopen during speech)
- `blocking=False` (default) → fire-and-forget enqueue
- Pipeline user-response: always blocking
- Agent loop `_speak()`: blocking (to prevent IDLE state race)
- Group conversation: non-blocking (turns managed by GroupConversation)

### Echo detection
Pipeline tracks `_recently_spoken` list. When Whisper transcribes the pony's own TTS output back through the mic, it's filtered by substring match + word overlap (>60% threshold).

### Window title sanitization (prompt injection defense)
Agent loop strips control characters, truncates to 120 chars, and removes bracket expressions from window titles before passing to LLM. This prevents malicious window titles from injecting tags.

### Qt thread marshaling
All GUI updates go through `PetController` Qt signals with `QueuedConnection`. The one exception is `speech_text` which uses `BlockingQueuedConnection` so the pipeline knows when the bubble is shown.

## Gotchas and Warnings

1. **PyAudio heap corruption**: Never open two `sr.Microphone()` contexts simultaneously. `stt/mic_lock.py` exists specifically for this — always use it.

2. **TTS lock**: `main.py` wraps the TTS engine's `speak()` in a `threading.Lock`. If you add a new speech path, make sure it goes through TTSQueue or acquires this lock.

3. **`_current_item` timing in tts_queue.py**: `_current_item` is set BEFORE the breathing pause, not after. This was a bug fix — don't move it back or `is_speaking` will report False during the gap.

4. **Desktop Ponies pony.ini format**: CSV-based, not INI. The `behavior_manager.py` parser is fragile with edge cases. Don't assume standard INI parsing.

5. **Preset files are large**: Character presets (presets/*.txt) are 16-28KB system prompts. They contain the full personality, available commands, relationship framing, and behavioral rules. Changes here affect everything.

6. **`generate_once` vs `chat`**: `generate_once` is for utility tasks (summarization, profile extraction, AFK decisions). It does NOT affect conversation history. `chat` is for actual conversation turns and maintains history.

7. **Standing rules use regex**: When a standing rule is created, the LLM generates regex patterns at creation time. Detection is pure regex matching on window titles — no LLM calls per tick.

8. **Vision key cycling**: `vision_provider.py` distributes requests across multiple API keys to stay under per-key rate limits. The key index rotates on each call.

9. **Wake state detection**: `routines.py` distinguishes program restart (same day or <4 hours gap) from actual wake-up (>4 hours gap). Don't lower the gap threshold or wake routines will fire on every restart.

10. **Agent loop is silenced during conversation**: When `_conversation_active` is True, the agent loop skips all speech. Pipeline sets this flag. If you add a new speech path in agent_loop, check this flag.

## Testing

No test suite exists. Validate changes with:
```bash
python -m py_compile <file.py>
```
For integration testing, use `scripts/test_pipeline.py` which tests STT → LLM → TTS stages individually.

## Persistent State Files

| File | Written by | Survives restart | Format |
|------|-----------|-----------------|--------|
| `directives.json` | AgentLoop | Yes | `{"directives": [...], "enforcement": null, "standing_rules": [...]}` |
| `routines.json` | RoutineManager | Yes | `[{"id": ..., "schedule": ..., "goal": ..., ...}]` |
| `wake_state.json` | RoutineManager | Yes | `{"wake_time": ISO, "last_active": ISO}` |
| `memory/sessions.txt` | Pipeline (summarize_session) | Yes | Plain text, last 3 sessions |
| `memory/user_profile.txt` | user_profile.py | Yes | Structured text |
| `memory/user_events.txt` | user_profile.py | Yes | Structured text |
| `diary/*.txt` | diary.py | Yes | Timestamped journal entries |
| `config.yaml` | context_menu.py (settings UI) | Yes | YAML with comments |

---
> Source: [maresmaremares/bonziPONY](https://github.com/maresmaremares/bonziPONY) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
