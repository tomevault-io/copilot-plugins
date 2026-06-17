## sonos-ytmusic

> >


# Sonos YTMusic 🎵

Play literally any song from YouTube directly on your Sonos speakers using direct SOAP commands over your local network.

## Agent Onboarding Checklist

1. **Check Requirements**: Verify `yt-dlp` and `curl` are installed.
2. **Find Speaker**: Ask the user for their Sonos speaker name (e.g., "Living Room").
3. **Set Environment**: Suggest the user adds `export SONOS_SPEAKER="Living Room"` to their shell profile.
4. **Network Access**: Ensure the speaker is on the same local network as the agent.
5. **Initial Test**: Run `python3 sonos-ytmusic.py "play Lofi Girl"` to verify both playback and network connectivity.

## Core Commands

### 1. Playback

```bash
# Search and play
python3 sonos-ytmusic.py "play [song name or URL]"

# Play artist
python3 sonos-ytmusic.py "play [artist name]"
```

### 2. Controls

```bash
# Pause / Resume
python3 sonos-ytmusic.py "pause"
python3 sonos-ytmusic.py "resume"

# Skip
python3 sonos-ytmusic.py "next"
python3 sonos-ytmusic.py "previous"

# Volume (0-100)
python3 sonos-ytmusic.py "volume 50"

# Stop (clear queue and stop)
python3 sonos-ytmusic.py "stop"
```

## Setup & Configuration

For the best experience, the agent should set the `SONOS_SPEAKER` environment variable for the user. If the speaker name is not provided, the script will attempt to discover one, but specifying it is much more reliable.

```bash
# Check if a speaker is reachable
python3 sonos-ytmusic.py "status"
```

---
*Created because playing music shouldn't require three subscriptions and a broken API.*

---
> Source: [0xCosmosly/Sonos-YTMusic](https://github.com/0xCosmosly/Sonos-YTMusic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
