## pocket-buddy

> Standing brief for Claude Code. Read first every session. This is a PUBLIC,

# CLAUDE.md — Pocket Buddy

Standing brief for Claude Code. Read first every session. This is a PUBLIC,
open-source repo other people will clone, flash, and run with their own API
keys. Write for strangers: clean, commented, safe. When something here proves
wrong, fix the line rather than piling on new ones.

## What this is
A self-contained wearable AI voice companion on an ESP32-S3. You tap (or raise)
it, speak, and it talks back, showing a reactive face on a round screen. It calls
cloud AI APIs DIRECTLY over WiFi. No phone app, no server, no Raspberry Pi. The
device is the whole product.

## Hardware (the fixed target — builders have this exact board)
Waveshare ESP32-S3-Touch-LCD-1.46:
- 412x412 round touch LCD, 16.7M color
- Onboard microphone and speaker
- 6-axis IMU (use for raise-to-talk)
- WiFi + BLE 5, LiPo-capable for wearable use
Firmware: native ESP-IDF (C++), built with idf.py. GitHub owner: minkgx1.

## Architecture — direct-call, self-contained
The device runs the whole loop itself:
  tap/raise -> capture mic audio -> Speech-to-Text API -> LLM API -> Text-to-Speech
  API -> play reply on speaker + show text/face on the round screen.
Three cloud services (STT, LLM, TTS), each behind ONE abstraction function so a
builder swaps provider or voice by editing a single place. LLM supports Claude and
OpenAI at minimum.

## KEY SAFETY — top priority, non-negotiable (this is a public repo)
- NO secrets in source, ever. Not WiFi creds, not API keys. Nothing committed.
- Keys and WiFi creds live ONLY in ESP32 flash (Preferences/NVS), entered by the
  user through a first-boot config portal (device starts an access point + settings
  page). A settings gesture / long-press re-opens the portal to change them.
- Ship `.gitignore` and `config.example.h` with PLACEHOLDER values only. The real
  config file never exists in the repo.
- Before ANY commit-related action, verify no secret could be staged. If asked to
  add a key/SSID to a tracked file, refuse and route it to NVS/the portal instead.
- README must warn: use a budget-capped/rate-limited key, never commit keys, and
  explain these are paid per-use APIs.

## Build order (PHASES — do one at a time, test, then stop)
0. Repo skeleton: ESP-IDF project structure, README outline, LICENSE, .gitignore,
   config.example.h placeholders, provider-abstraction stubs. No secrets anywhere.
1. WiFi + first-boot config portal (AP -> settings page -> save to NVS -> connect).
2. Round 412x412 display: idle face + status line (ready/listening/thinking/
   speaking/error).
3. LLM text round-trip: tap -> hardcoded prompt -> real API reply on screen.
4. TTS playback (BEFORE capture — it's easier and gives capture a known-good
   output to test against): reply text -> speaker, face animates "speaking".
5. Mic capture + STT (THE HARD PART, deliberately last): record -> buffer -> STT ->
   LLM -> TTS. Full loop working.
6. Delight + polish: IMU raise-to-talk, reactive face, battery notes, and a
   genuinely complete README (flashing, portal walkthrough, cost warning,
   provider-swap guide, troubleshooting, demo GIF).

## Hard-won gotchas / rules
- **Audio capture is where these projects fail.** Sample rate, bit depth,
  buffering, and clipping/dropouts on the ESP32 mic are the real work. That's why
  it's Phase 5, after a trusted playback path exists. Do not attempt capture before
  Phase 4 works.
- **Playback before capture.** Always. Getting the speaker path solid first means
  the hard capture phase has something known-good to test against.
- **Replies are read aloud** — the persona/system prompt must produce short,
  natural, spoken-style sentences. No markdown, no lists, no headers in replies.
- **Fail gracefully, never hard-lock.** Unreachable WiFi, bad key, or API error ->
  clear on-screen state + recovery. A stranger's device must not brick on a typo.
- **Provider swap is one line.** LLM/STT/TTS each behind a single function.
- The round screen's reactive face is the soul of the project, not decoration.
  Don't let it get cut.

## Conventions
- Public-repo hygiene: clean, commented, stranger-readable code; a great README is
  half the deliverable.
- No em dashes or en dashes in generated content or docs.
- Verify by running on hardware, not by assuming a compile means it works.

## Don't
- Don't put any secret in a tracked file, ever — route it to NVS/the portal.
- Don't reorder the phases to do audio capture early.
- Don't make replies markdown-formatted; they are spoken.
- Don't add a companion-app or server dependency — this build is self-contained by
  design.

---
> Source: [minkgx1/pocket-buddy](https://github.com/minkgx1/pocket-buddy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-05 -->
