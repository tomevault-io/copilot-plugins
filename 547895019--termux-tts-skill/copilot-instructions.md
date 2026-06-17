## termux-tts-skill

> Use termux-tts-speak to play text using Android system TTS engine. Use when the user wants to convert text to speech using their device's native TTS engine in Termux. Triggers on phrases like "speak this", "read aloud", "TTS", "text to speech", "朗读", "播放语音" when in Termux/Android environment.


# Termux TTS Skill

This skill provides text-to-speech functionality using Android's native TTS engine through Termux.

## Prerequisites

- Termux app installed
- `termux-api` package installed (`pkg install termux-api`)
- Termux:API app installed from F-Droid or Play Store

## Usage

### Basic speech

```bash
termux-tts-speak "Hello, this is a test"
```

### With options

```bash
# Speed: 0.5 (slow) to 2.0 (fast), default 1.0
termux-tts-speak -s 1.5 "Faster speech"

# Pitch: 0.5 (low) to 2.0 (high), default 1.0  
termux-tts-speak -p 0.8 "Lower pitch"

# Language (if supported by TTS engine)
termux-tts-speak -l zh-CN "中文朗读"
```

### Using the helper script

```bash
# Make executable first time
chmod +x ~/.openclaw/skills/termux-tts/scripts/tts.sh

# Use the script
~/.openclaw/skills/termux-tts/scripts/tts.sh "Text to speak"
```

## Helper Script

See `scripts/tts.sh` for a convenient wrapper with default settings optimized for Chinese speech.

## Common Issues

- **No sound**: Check volume and ensure Termux:API app is installed
- **Permission denied**: Run `termux-api` command first to establish connection
- **Wrong language**: Set system TTS language in Android Settings > Accessibility > Text-to-speech output

---
> Source: [547895019/termux-tts-skill](https://github.com/547895019/termux-tts-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
