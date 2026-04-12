## hakatamatrix-app-controller

> This is a GPU application controller designed to manage multiple GPU-dependent applications on a HakataMatrix machine with AMD Radeon Instinct MI50 32GB in ROCm environment. The controller ensures only one GPU application runs at a time and provides Discord-based control interface.

# HakataMatrix App Controller - AI Agent Documentation

## Project Overview

This is a GPU application controller designed to manage multiple GPU-dependent applications on a HakataMatrix machine with AMD Radeon Instinct MI50 32GB in ROCm environment. The controller ensures only one GPU application runs at a time and provides Discord-based control interface.

## Architecture

### Core Components

1. **ProcessManager** (`src/process-manager.ts`)
   - Manages child processes for GPU applications
   - Handles graceful shutdown with SIGINT
   - Emits events for process lifecycle
   - Tracks process states (running, stopping, stopped)

2. **AppController** (`src/controller.ts`)
   - Orchestrates process lifecycle
   - Implements auto-restart logic for default app
   - Listens to process-exit events

3. **DiscordBot** (`src/discord-bot.ts`)
   - Implements Discord slash commands
   - Handles /join, /leave, /status commands
   - Manages application switching

4. **SystemInfo** (`src/system-info.ts`)
   - Fetches GPU metrics via rocm-smi
   - Retrieves memory info from /proc/meminfo
   - Provides formatting utilities

## Managed Applications

### Danbooru ML Classifier Worker
- **Path**: `~/Documents/GitHub/danbooru-ml-classifier/worker`
- **Command**: `./venv/bin/python -u ./vlm_captioner.py`
- **Behavior**: Processes 10 images per run, then exits
- **Auto-restart**: Enabled (default app)

### YouTube Comment Generator
- **Path**: `~/Documents/GitHub/youtube-comment-generator`
- **Command**: `poetry run python -m src.main`
- **Behavior**: Long-running Discord bot
- **Auto-restart**: Disabled (manual control via Discord)

## Key Implementation Details

### Process Management
- Uses Node.js `child_process.spawn()` for process creation
- Sends SIGINT for graceful shutdown (30s timeout)
- Falls back to SIGKILL if timeout exceeded
- Waits for complete termination before starting new app

### Auto-restart Logic
- Only applies to default app (danbooru-ml-classifier)
- 5-second delay between restart attempts
- Disabled during application shutdown

### Discord Commands
- **Authentication**: Uses Discord.js v14 with slash commands
- **Response Pattern**: Deferred replies for long-running operations
- **Error Handling**: Catches and reports errors to user

### System Monitoring
- GPU metrics from `rocm-smi --showuse --showmeminfo vram`
- Memory info from `/proc/meminfo`
- Formatted output in Discord messages

## Development Guidelines

### TypeScript Configuration
- Strict mode enabled with comprehensive checks
- ESNext module system
- No implicit any types allowed

### Code Style
- Biome for linting and formatting
- Tab indentation
- Single quotes for strings
- No default exports except where necessary

### Testing
- Vitest for unit tests
- Focus on utility functions and pure logic
- External dependencies mocked where appropriate

## Environment Variables

Required in `.env`:
- `DISCORD_TOKEN`: Discord bot authentication token
- `DISCORD_CLIENT_ID`: Discord application client ID

## Deployment

### Systemd Service
- Run as user `hakatashi`
- Auto-restart on failure
- Journal logging for stdout/stderr
- Graceful shutdown on SIGTERM/SIGINT

### Build Process
1. `npm install` - Install dependencies
2. `npm run build` - Compile TypeScript
3. `npm start` - Run production build

## Error Handling

### Common Scenarios
1. **Application fails to start**: Logs error, doesn't retry automatically
2. **Application crashes**: Auto-restart if default app
3. **Discord disconnection**: Bot handles reconnection internally
4. **GPU command failure**: Returns zero values, logs error

### Graceful Shutdown
1. Disable auto-restart
2. Stop Discord bot
3. Send SIGINT to all running processes
4. Wait for clean exit
5. Force kill after timeout

## Monitoring and Debugging

### Logging
- Console output for all operations
- Prefixed logs with app name: `[app-name] message`
- Process stdout/stderr forwarded to main process

### Status Information
- Process states tracked in memory
- Last start/stop times recorded
- Restart counter maintained
- PID tracking for running processes

## Extension Points

### Adding New Applications
1. Add app config to `APP_CONFIGS` in `config.ts`
2. Define AppName type in `types.ts`
3. Update Discord commands if needed
4. Consider auto-restart behavior

### Custom Commands
1. Add slash command definition in `registerCommands()`
2. Implement handler in `handleCommand()`
3. Update command routing switch statement

## Security Considerations

- `.env` file excluded from Git
- No credentials in source code
- Discord token validation on startup
- Process isolation via child_process

## Performance Notes

- Minimal overhead: Event-driven architecture
- Efficient polling: System info fetched on-demand
- Non-blocking I/O for all operations
- Clean resource management

## Known Limitations

1. Single GPU only - no multi-GPU support
2. Linux-specific (ROCm, /proc/meminfo)
3. No persistent state across restarts
4. No application queue or scheduling

## Future Considerations

- Persistent state storage (SQLite/JSON)
- Web dashboard for monitoring
- Application scheduling system
- Multi-GPU support
- Metrics collection and graphing
- Health checks for managed apps

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hakatashi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hakatashi)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
