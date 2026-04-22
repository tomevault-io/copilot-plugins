## agentos-cli

> 1. Always aim for small wins rather than big shots.

## Most Important Rules

1. Always aim for small wins rather than big shots.
2. Make sure you use use cases for testing, not traditional unit tests.

## General Rules

1. we use `uv` for package management, always use `uv add`to install packages, and `uv run` to run scripts. other commands in `uv` are also preferred.
2. do not code unless the user asks to.
3. always spin up subagents for multiple tasks that can be done in parallel, like reading 10 urls or read 5 code scripts, etc.
4. always write pseudo codes (classes and function with only names, parameters, and descriptions but no real implementations ) first and explain your pseudo codes to the user and get approval before writing real implementation codes.
5. always create temperary test codes to make sure each function and class work as described.

## AgentOS EventChain Architecture Progress

### Completed Components:

1. **modules/eventbus/** - Core event infrastructure organized in dedicated folder
   - `event_bus.py` - Basic pub/sub system with validation (existing)
   - `event_registry.py` - Schema registration with Pydantic (existing)
   - `event_chain.py` - Event chain execution engine (new)
   - `parameter_interpolator.py` - Parameter interpolation engine (new)
   - `thread_manager.py` - Thread persistence and management (new)

### Architecture Overview:

- **EventChain**: Everything flows through event chains - sequential or parallel
- **Threads**: Persistent event chains with full context and history
- **Three-Model Strategy**: Heavy (think), Fast (chain), Ultra-light (decide)
- **Parameter Flow**: Results flow between events via interpolation ({event.result})

## Task System Implementation Notes

### Completed:
1. **JSON-based task system** - No SQLite dependencies, pure JSON storage
2. **Scheduled tasks** - Using APScheduler for time-based execution
3. **Hook system** - Pattern-based event matching with before/after positions
4. **Event chain support** - Tasks can define sequences of events to execute

### Known Issues:
1. **Event Loop Threading Problem**: 
   - APScheduler runs tasks in separate threads without async event loops
   - EventBus components (locks, storage) are tied to the main event loop
   - Current workaround: Event chains print what they would do instead of actually executing
   - Proper fix would require: `asyncio.run_coroutine_threadsafe()` or async-native scheduler

2. **Hook Implementation**:
   - Uses method interception on `eventbus.publish()` 
   - Wildcard patterns work via `fnmatch`
   - Must use `source="hook"` when publishing from hooks to prevent infinite loops

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zhengyishen0) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
