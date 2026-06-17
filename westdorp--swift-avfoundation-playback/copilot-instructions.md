## swift-avfoundation-playback

> Fundamentals-first Swift AVFoundation playback guidance for AVAsset/AVPlayerItem/AVPlayer, seeking timelines, observation, queueing, AirPlay, audio session, PiP, HLS, DRM, ads, and diagnostics across Apple platforms. Use when implementing or reviewing playback behavior.


# Swift AVFoundation Playback Skill

Use this skill to build playback behavior from a stable fundamentals core, then load only the topic files you need.

## Read Order

1. `references/player-item-lifecycle.md`
2. `references/seeking-and-timelines.md`
3. `references/observable-setup.md`
4. `references/time-observation.md`
5. `references/intent-tracking.md`

## Topic Map

- Queue and transitions: `references/queue-playback.md`, `references/gapless-playback.md`, `references/coordinated-playback.md`
- System integrations: `references/airplay-routing.md`, `references/audio-session.md`, `references/now-playing.md`, `references/picture-in-picture.md`, `references/platform-patterns.md`
- Streaming and reliability: `references/hls-streaming.md`, `references/stall-recovery.md`, `references/error-handling.md`
- Protected and monetized playback: `references/fairplay-drm.md`, `references/ad-insertion.md`

## Policy Rules

- Treat AVFoundation transport properties as signals into your app state machine.
- Keep UI mutation on the main actor.
- If Apple documentation does not guarantee a behavior, express it as app policy.

Sources:
- https://developer.apple.com/documentation/avfoundation/avplayer — Defines the core transport API used throughout this skill.
- https://developer.apple.com/library/archive/documentation/AudioVideo/Conceptual/AVFoundationPG/Articles/02_Playback.html — Defines playback object relationships and readiness concepts.
- https://developer.apple.com/library/archive/documentation/AudioVideo/Conceptual/MediaPlaybackGuide/Contents/Resources/en.lproj/Introduction/Introduction.html — Provides Apple’s conceptual structure for media playback fundamentals.

---
> Source: [westdorp/swift-avfoundation-playback](https://github.com/westdorp/swift-avfoundation-playback) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
