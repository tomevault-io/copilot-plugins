## wordle-battle

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Wordle Battle is a real-time competitive Wordle game built with Phoenix LiveView. Players compete simultaneously to guess the most 5-letter words within a timed session, with scoring based on speed and accuracy.

## Essential Commands

### Development
- `mix setup` - Install dependencies and build assets (first-time setup)
- `mix phx.server` - Start Phoenix server (visit http://localhost:4000)
- `iex -S mix phx.server` - Start Phoenix server in interactive Elixir shell
- `mix precommit` - Run full pre-commit checks (compile with warnings as errors, format, test)

### Testing
- `mix test` - Run all tests
- `mix test test/path/to/test.exs` - Run specific test file
- `mix test --failed` - Run only previously failed tests

### Assets
- `mix assets.build` - Build Tailwind + esbuild assets
- `mix assets.deploy` - Build and minify assets for production

## Architecture

### Game State Management (GenServer + LiveView)

The game uses a **GenServer-based architecture** with Phoenix LiveView for real-time UI:

1. **WordleServer (GenServer)**: Each game session runs as a supervised GenServer process
   - Manages game state, phase transitions (:lobby → :playing → :game_over)
   - Handles word assignment, guess validation, scoring
   - Broadcasts state updates via PubSub
   - Persists state across player disconnections

2. **WordleLive (LiveView)**: Real-time UI for players
   - Subscribes to PubSub topic for session updates
   - Renders game grid, virtual keyboard, leaderboard
   - Handles user interactions (guesses, ready state)
   - Manages player reconnection via localStorage user_id

3. **Dynamic Supervision**: Sessions registered via Registry for efficient lookup
   - `Registry.lookup/2` for finding session processes
   - `DynamicSupervisor` for fault-tolerant session management

4. **PubSub Broadcasting**: Real-time updates to all session participants
   - Timer updates every second
   - Score changes after each guess
   - Player connection status changes

### Word Management

- **Dictionary Loading**: Answer words (~2,300) and valid guesses (~12,000) loaded at app startup in `application.ex`
- **Module**: `WordleBattle.Dictionary` - provides `answer_words/0`, `valid_guesses/0`, `random_word/1`
- **Per-Session Tracking**: `used_words` list prevents word repetition within a session
- **Files**: Located in `priv/dictionary/answer_words.txt` and `priv/dictionary/valid_guesses.txt`

### Player Persistence

- **UUID Storage**: Each player gets persistent UUID in browser localStorage
- **Automatic Reconnection**: Players rejoin session on page refresh via stored user_id
- **Graceful Degradation**: Disconnected players' progress preserved; game continues for others
- **Cleanup Workers**: Automated cleanup of disconnected players (10 min) and inactive sessions (1 hour)

## Key Game Mechanics

### Scoring System
- **Base**: 1 point per correctly guessed word
- **Bonuses**:
  - 1 attempt: +5 (total 6 pts)
  - 2 attempts: +3 (total 4 pts)
  - 3 attempts: +2 (total 3 pts)
  - 4-6 attempts: +1 (total 2 pts)
- **Failed word**: 0 points

### State Machine
```elixir
:lobby -> :playing  # All players ready → start_game
:playing -> :game_over  # Timer expires → calculate winners
:game_over -> :lobby  # start_new_game → reset with preserved players
```

### Guess Validation Algorithm
Two-pass algorithm to handle duplicate letters correctly:
1. First pass: Mark exact matches (green)
2. Second pass: Mark present letters (yellow) from remaining pool
3. Mark remaining as wrong (gray)

## Project-Specific Guidelines

### Phoenix v1.8 Patterns
- Use `<Layouts.app>` wrapper in all LiveView templates (already aliased in `wordle_battle_web.ex`)
- All forms use `to_form/2` in LiveView, access via `@form[:field]` in templates
- Use `<.link navigate={}>` and `<.link patch={}>`, not deprecated `live_redirect`/`live_patch`
- Icons via `<.icon name="hero-x-mark" />` component from core_components.ex

### LiveView Streams for Collections
Use streams for player lists, word history, and leaderboards to prevent memory issues:
```elixir
# In LiveView
stream(socket, :players, [new_player])  # append
stream(socket, :players, players, reset: true)  # replace all

# In template (must set phx-update="stream" on parent)
<div id="players" phx-update="stream">
  <div :for={{id, player} <- @streams.players} id={id}>
    {player.nickname}
  </div>
</div>
```

### Testing with Phoenix.LiveViewTest
- Always add unique DOM IDs to elements (`<.form for={@form} id="guess-form">`)
- Reference these IDs in tests: `assert has_element?(view, "#guess-form")`
- Use `render_submit/2` and `render_change/2` for form testing
- Test outcomes, not implementation details
- Use `LazyHTML` for debugging complex selectors

### Word Validation
- All words normalized to uppercase
- Validate guesses against `Dictionary.valid_guesses/0` MapSet (O(1) lookup)
- Assign only from `Dictionary.answer_words/0` list (~2,300 common words)

### Timer Implementation
Use `:timer.send_interval/2` in GenServer for countdown:
- Broadcast time updates via PubSub every second
- Auto-transition to `:game_over` when time_remaining reaches 0
- Play audio notification (`/sounds/timer_end.mp3`) on expiration

### Player Reconnection Flow
1. Client sends `user_id` from localStorage on mount
2. LiveView calls `WordleServer.restore_session(session_id, user_id)`
3. GenServer marks player as connected, returns current state
4. LiveView renders current game phase with preserved progress

## Important Constraints

### Elixir-Specific
- **No index access on lists**: Use `Enum.at(list, index)` instead of `list[index]`
- **Variable rebinding**: Assign block expression results, don't rebind inside blocks
- **No `else if`**: Use `cond` for multiple conditionals
- **No map access on structs**: Use `struct.field` or `Ecto.Changeset.get_field/2`
- **Atoms from user input**: Never use `String.to_atom/1` on user input (memory leak)

### HEEx Template Syntax
- Interpolation in attributes: `{@value}` not `<%= @value %>`
- Interpolation in body: `{@value}` for values, `<%= for/if/cond %>` for blocks
- Class lists: `class={["base", @flag && "conditional"]}` (must use `[...]`)
- No `else if`: Use `<%= cond do %>` with clauses
- Comments: `<%!-- comment --%>` not `<!-- -->`

### LiveView Streams Limitations
- **Not enumerable**: Cannot use `Enum.filter/2` on streams
- **No empty state check**: Track emptiness in separate assign or use CSS `.hidden.only:block`
- **Reset for filtering**: Re-fetch data and `stream(socket, :items, items, reset: true)`

## Files You'll Work With

### Core Implementation Files (to be created)
- `lib/wordle_battle/dictionary.ex` - Word loading and validation
- `lib/wordle_battle/wordle_server.ex` - Game state GenServer
- `lib/wordle_battle_web/live/wordle_live.ex` - Main game LiveView
- `lib/wordle_battle_web/live/lobby_live.ex` - Session creation/joining
- `priv/dictionary/answer_words.txt` - ~2,300 answer words
- `priv/dictionary/valid_guesses.txt` - ~12,000 valid words

### Existing Phoenix Files
- `lib/wordle_battle/application.ex` - Supervision tree, preload dictionaries here
- `lib/wordle_battle_web/router.ex` - Add LiveView routes (`live "/:session_id", WordleLive`)
- `lib/wordle_battle_web/components/core_components.ex` - Reusable UI components
- `assets/js/app.js` - Client-side hooks for localStorage, keyboard input

## Technical Stack

- **Phoenix 1.8** - Web framework
- **Phoenix LiveView 1.1** - Real-time UI
- **Req** - HTTP client (use instead of HTTPoison/Tesla)
- **Tailwind CSS** - Styling (with DaisyUI recommended per specs)
- **Heroicons** - Icons via `<.icon>` component
- **PubSub** - Real-time broadcasting
- **Registry** - Process discovery
- **GenServer** - Stateful game sessions

## Development Workflow

1. **Make changes** to code
2. **Run `mix precommit`** before committing - this runs:
   - Compilation with warnings as errors
   - Dependency cleanup
   - Code formatting
   - Full test suite
3. **Fix any issues** flagged by precommit checks
4. **Commit** when all checks pass

## Detailed Specifications

See `WORDLE_BATTLE_SPECS.md` for complete feature specifications including:
- Full state machine diagrams
- Data structures
- UI layouts
- Guess validation algorithm
- Scoring calculations
- Cleanup rules

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ElixirMentor) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
