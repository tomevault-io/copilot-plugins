## gptsovits-codex-voice-skill

> Use when the user asks Codex to speak, read aloud, synthesize, dub, voice, vocalize, or convert Codex's replies into audio with GPT-SoVITS. Chinese triggers include 朗读, 说出来, 转成声音, 配音, 合成语音, 用 GPT-SoVITS 读, 把你刚才说的话变成音频. This skill invokes the local GPT-SoVITS API at 127.0.0.1:9880 through PowerShell scripts and saves wav files locally.


# GPT-SoVITS Codex Voice

Use this skill when the user wants Codex text, especially Codex's own replies, converted into speech through the local GPT-SoVITS server.

This skill also supports a persistent voice mode. If the user asks to enable this skill, start voice mode, keep speaking, always play your replies, or similar, run `scripts\enable.ps1`. This starts a hidden session watcher that reads new Codex assistant messages from the current session jsonl and sends them to `scripts\speak.ps1`; do not manually duplicate every reply into a tool call while the watcher is active. When the user turns it off, run `scripts\disable.ps1` and stop auto-speaking future replies.

The implementation is a local PowerShell script. Do not look for a UI tool or remote service. The user is expected to have GPT-SoVITS running at `http://127.0.0.1:9880`; if it is not running, ask them to start `api_v2.py` or start it from the GPT-SoVITS package before retrying.

## Defaults

- GPT-SoVITS API: `http://127.0.0.1:9880/tts`
- Reference audio: `C:\Users\Administrator\Downloads\.lian\qu1\vocal_这本书的国际标准书号是九七八七零二零零二四七五九，全书一共分为三十二个章节。.wav.reformatted.wav_10.wav`
- Prompt text: `这本书的国际标准书号是九七八七零二零零二四七五九，全书一共分为三十二个章节。`
- Reference language: `zh`
- Target language: `zh`
- Output directory: `C:\Users\Administrator\Downloads\codex-gptsovits-voice`

## Speak Text

```powershell
C:\Users\Administrator\.codex\skills\gptsovits-codex-voice\scripts\speak.ps1 `
  -Text "你好，我是 Codex。这段话会用 GPT-SoVITS 合成为声音。"
```

The script saves the wav, prints the saved wav path and a JSON sidecar path, then automatically plays the generated audio through PowerShell/.NET's built-in WAV player. It must not open the system default audio app such as NetEase Cloud Music.

## Voice Mode

Enable persistent voice mode:

```powershell
C:\Users\Administrator\.codex\skills\gptsovits-codex-voice\scripts\enable.ps1
```

Disable persistent voice mode:

```powershell
C:\Users\Administrator\.codex\skills\gptsovits-codex-voice\scripts\disable.ps1
```

Check whether voice mode is active:

```powershell
C:\Users\Administrator\.codex\skills\gptsovits-codex-voice\scripts\status.ps1
```

When voice mode is active, the watcher should automatically speak new assistant messages from the Codex session log. Manual speaking is only needed if the watcher is not running or if the user asks to synthesize a specific text. For manual fallback, run:

```powershell
C:\Users\Administrator\.codex\skills\gptsovits-codex-voice\scripts\speak.ps1 `
  -Text "这里放即将显示给用户的完整回复文本。"
```

Then send that same text to the user. If the text is too long for convenient command-line quoting, write it to a UTF-8 text file and pass `-TextFile`.

## Speak Codex's Reply

When the user asks to turn "what you just said" into audio, pass the exact assistant text to `-Text`:

```powershell
C:\Users\Administrator\.codex\skills\gptsovits-codex-voice\scripts\speak.ps1 `
  -Text "这里放 Codex 刚刚说过、需要朗读的完整文本。"
```

If the text is long, write it to a UTF-8 text file and pass `-TextFile`:

```powershell
C:\Users\Administrator\.codex\skills\gptsovits-codex-voice\scripts\speak.ps1 `
  -TextFile "C:\path\codex-reply.txt"
```

## Useful Options

```powershell
# Choose an exact output path
.\speak.ps1 -Text "..." -OutputPath "C:\Users\Administrator\Downloads\reply.wav"

# Save into another directory
.\speak.ps1 -Text "..." -OutputDir ".\output\voice"

# Save the generated wav without opening the audio player
.\speak.ps1 -Text "..." -NoPlay

# Adjust speed
.\speak.ps1 -Text "..." -SpeedFactor 1.08

# Use a different local GPT-SoVITS API endpoint
.\speak.ps1 -Text "..." -ApiUrl "http://127.0.0.1:9880/tts"
```

## Notes

- The script sends UTF-8 JSON to GPT-SoVITS so Chinese text stays intact.
- It validates the reference audio path before calling the API.
- It saves a small `.json` sidecar containing the request metadata, not the generated audio bytes.
- It automatically plays the generated wav through `System.Media.SoundPlayer` unless `-NoPlay` is passed. Do not use `Start-Process` for playback because that opens the Windows default audio app.
- Persistent voice mode is tracked by `C:\Users\Administrator\.codex\skills\gptsovits-codex-voice\state.json`.
- The session watcher is `scripts\watch-session.ps1`; it listens for `agent_message` entries with `commentary` or `final_answer` phases.
- If the API returns an error JSON instead of audio, the script prints the response preview and fails.

---
> Source: [dianfangsihuo/gptsovits-codex-voice-skill](https://github.com/dianfangsihuo/gptsovits-codex-voice-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
