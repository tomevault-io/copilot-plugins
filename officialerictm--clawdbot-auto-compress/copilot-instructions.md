## clawdbot-auto-compress

> Automatically compresses conversation context when approaching token limits, similar to Claude.ai's "Hold on, compressing our conversation" feature.

# Auto-Compress Context Skill

Automatically compresses conversation context when approaching token limits, similar to Claude.ai's "Hold on, compressing our conversation" feature.

## Overview
Monitors context usage and automatically triggers compression at 80% capacity (160k of 200k tokens), allowing conversations to continue indefinitely without manual resets.

## Requirements
- Clawdbot with session access
- Python 3.8+
- Node.js (for hook integration)

## How It Works
1. Monitors context usage in real-time
2. At 80% threshold, automatically compresses conversation
3. Preserves critical information while removing verbose content
4. Seamlessly resumes conversation with compressed history

## Installation

1. Install the monitoring hook:
```bash
cd /home/ubuntu/clawd/skills/auto-compress
chmod +x install.sh
./install.sh
```

2. Add to your Clawdbot config:
```yaml
agent:
  hooks:
    afterResponse:
      - /home/ubuntu/clawd/skills/auto-compress/context-monitor.js
```

## Usage
Once installed, it works automatically. You'll see:
- 📊 Context usage in status messages
- ⚠️ Warning at 70% capacity
- 🗜️ Automatic compression at 80%
- ✅ Confirmation when compression completes

## Configuration
Edit `config.json` to customize:
```json
{
  "threshold": 0.8,        // Trigger at 80% capacity
  "warningThreshold": 0.7, // Warn at 70% capacity
  "preserveRecent": 20,    // Keep last 20 messages
  "compressTools": true,   // Compress tool outputs
  "notifyUser": true       // Show compression message
}
```

## Manual Commands
- `compress-now` - Force compression immediately
- `compress-status` - Show compression statistics
- `compress-disable` - Temporarily disable auto-compression

## Compression Strategy
1. **Preserve**: Recent messages, user requests, important results
2. **Compress**: Tool outputs, repetitive content, old context
3. **Summarize**: Long conversations into key points
4. **Archive**: Full history to session transcript

## Example
When compression triggers, you'll see:
```
🗜️ Hold on, I need to compress our conversation to continue...
[Compressing 160k → 40k tokens]
✅ Compression complete! We can continue our conversation.
```

The conversation then continues seamlessly with all important context preserved.

---
> Source: [officialerictm/clawdbot-auto-compress](https://github.com/officialerictm/clawdbot-auto-compress) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
