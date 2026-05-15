## claude-code-tools

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Commands

### Setup and Configuration
- **Initial setup**: `./setup.sh` - Interactive SMS notification setup wizard
- **Reconfigure**: `./setup.sh --reconfigure` - Modify existing configuration
- **Test configuration**: `./setup.sh --test-config` - Validate current SMS settings

### Feature Management
- **Start all features**: `./start.sh` - Enable SMS notifications and auto-resume, start daemons
- **Stop all features**: `./stop.sh` - Disable features and stop background daemons

### SMS Testing
- **Run comprehensive tests**: `./sms/test-sms.sh` - Full test suite
- **Send test SMS**: `$CC_TOOLS_DIR/sms/sms-sender.sh "Test message" "completion"`
- **Test SMS sender**: `$CC_TOOLS_DIR/sms/sms-sender.sh --test`

### Background Services
- **Start SMS receiver**: `$CC_TOOLS_DIR/sms/sms-receiver.sh --daemon &`
- **Check messages once**: `$CC_TOOLS_DIR/sms/sms-receiver.sh --check-once`
- **Test rate limit simulation**: `$CC_TOOLS_DIR/auto_resume/rate-limit-monitor.sh --simulate-429`

### Debugging and Monitoring
- **Test configuration**: `$CC_TOOLS_DIR/setup.sh --test-config` - Validate all system settings
- **Run specific tests**: 
  - `$CC_TOOLS_DIR/sms/test-sms.sh --basic` - Configuration tests
  - `$CC_TOOLS_DIR/sms/test-sms.sh --messages` - Messages app tests
  - `$CC_TOOLS_DIR/sms/test-sms.sh --hooks` - Hook configuration tests
  - `$CC_TOOLS_DIR/sms/test-sms.sh --integration` - Send test SMS
- **Monitor logs**: `tail -f $CC_TOOLS_DIR/logs/cc-tools.log`
- **Check daemon status**: `ps aux | grep -E "(sms-receiver|rate-limit-monitor)"`

### Environment
- **Required environment variable**: `CC_TOOLS_DIR` must point to the claude-code-tools directory
- **Dependencies**: `jq`, macOS Messages app, Terminal with Full Disk Access

## Code Architecture

### Core Components

#### Hook System
The system integrates with Claude Code through a unified hook in `.claude/settings.json`:
- **Unified CC Hook** (`cc-hook.sh`): Handles both SMS notifications and auto-resume functionality
  - Processes Stop and Notification events
  - Sends SMS for task completions, errors, input requests, and rate limits
  - Detects API rate limits and saves state for automatic resumption
  - Legacy hooks (`sms-hook.sh`, `auto-resume-hook.sh`) available for reference

#### SMS Notification System (`sms/`)
- **sms-sender.sh**: AppleScript-based SMS sending via macOS Messages app
- **sms-receiver.sh**: Monitors Messages database for incoming SMS commands
- **screenshot.sh**: Captures terminal window screenshots for SMS status updates

#### Auto-Resume System (`auto_resume/`)
- **rate-limit-monitor.sh**: Background daemon that monitors for rate limit expiration

#### Utilities (`utils/`)
- **logging.sh**: Centralized logging to `$CC_TOOLS_DIR/logs/cc-tools.log`
- **send-to-claude.sh**: Sends commands to Claude Code terminal sessions

### Configuration Architecture

#### Main Configuration (`config/config.json`)
JSON configuration shared across all Claude Code projects:
```json
{
  "user_phone": "+1234567890",
  "contacts": ["+1234567890"],
  "sms_enabled": true,
  "auto_resume": true,
  "max_message_length": 160,
  "screenshot_enabled": false
}
```

#### Hook Configuration (`.claude/settings.json`)
Defines Claude Code hooks that trigger SMS and auto-resume functionality.

### Integration Points

#### Messages App Integration
- Uses AppleScript to send SMS/iMessage through macOS Messages
- Monitors SQLite database at `~/Library/Messages/chat.db` for incoming replies
- Requires Full Disk Access permission for terminal applications

#### Terminal Integration
- Detects terminal application (Terminal, iTerm, iTerm2) and window/session
- Stores terminal configuration in `config.json` for command routing
- Uses AppleScript to send commands to specific terminal sessions

#### Claude Code Hook Integration
- Unified hook handling both SMS notifications and auto-resume functionality
- Event-driven architecture responding to Stop and Notification events
- Automatic API limit detection and rate limit state management
- Timeout handling for hook execution

### State Management

#### Rate Limit State (`$CC_TOOLS_DIR/logs/rate-limited-state.json`)
Stores rate limit information for auto-resume:
```json
{
  "expires_at": 1234567890
}
```

#### Process Management
- PID files in `$CC_TOOLS_DIR/logs/` for background daemons
- Daemon lifecycle managed by `start.sh` and `stop.sh`
- Automatic cleanup and restart capabilities

## Security Model

- Contact-based authorization: Only configured phone numbers can send commands
- Commands processed only from authorized contacts in `config.json`
- No network services - uses local Messages app and AppleScript
- Logs stored in `$CC_TOOLS_DIR/logs/` for debugging

---
> Source: [cyzhao/claude-code-tools](https://github.com/cyzhao/claude-code-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-08 -->
