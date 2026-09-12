## a-thing-that-lets-you-play-a-text-adventure-with-your-friends

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a multi-user text adventure / RPG chat application built with Sinatra and Ruby. It allows users to create rooms, chat in real-time via WebSockets, and roll dice using RPG-style commands. The application features role-based access (Game Masters vs Players) and supports AI users.

## Development Commands

### Running the Application
```bash
bundle install           # Install dependencies
ruby app.rb             # Run the Sinatra application
```

The application runs on the default Sinatra port (4567). Access via http://localhost:4567

### CSS/Styling
```bash
# Tailwind CSS is used via input.css in src/
# Output is in public/css/style.css
```

## Architecture

### Database Schema (SQLite via Sequel)

The application uses SQLite with Sequel ORM. Database initialization happens in `initializers/db.rb`. Key tables:

- **admins**: Tracks whether initial admin setup is complete
- **magic_links**: Token-based invite system for new users
- **users**: User accounts with bcrypt password hashing. Fields include `is_admin` and `is_ai` flags
- **rooms**: Chat rooms with `game_master_id` and `created_by` references
- **user_rooms**: Many-to-many relationship between users and rooms
- **chat_messages**: Messages with optional JSON `data` field for rich content (dice rolls)

### WebSocket Architecture

- WebSocket connections are stored per-room in `settings.sockets` hash (keyed by room_id)
- Thread-safe access via `settings.sockets_mutex`
- Background ping thread keeps connections alive every 5 seconds
- When a message is received, it's saved to DB then broadcast to all room participants

### Authentication & Authorization

- Session-based authentication using Sinatra sessions
- First user setup creates admin account via `/setup` route
- New users join via magic links (created by admins) which redirect to password setup
- Helper methods: `current_user`, `require_auth`, `authenticate_user`
- Room access controlled via `user_rooms` join table

### Message Rendering System

The `render_message` helper dynamically chooses message templates:
- `_dice_roll_message.erb`: For dice roll results (when message data has `type: 'dice_roll'`)
- `_gm_message.erb`: For messages from Game Master
- `_player_message.erb`: For regular player messages

Messages can include structured data in the `data` JSON field. The client sends dice commands like `/roll 2d20+5` which are parsed server-side.

### Dice Rolling System

Implemented in `initializers/dice_roll.rb`:
- Supports standard RPG dice: d4, d6, d8, d10, d12, d20, d100
- Command format: `/roll XdY+Z` (e.g., `/roll 2d20+5`)
- Parses multiple dice and modifiers from a single command
- Returns individual dice results, modifiers, and total
- Results stored in message `data` field as JSON

ASCII dice visualizations are in `views/ascii/_d*.html.erb` partials.

### Room Management

- Users can create rooms and assign a Game Master (can be an AI user)
- Game Masters can manage players in their rooms
- Room creators can manage players if GM is AI
- Players must be explicitly added to rooms via `user_rooms` table

### View Structure

- Main layout: `views/layout.erb` (terminal/retro theme with Tailwind CSS)
- Partials pattern:
  - `views/messages/_*.erb`: Message type templates
  - `views/buttons/_*.erb`: Reusable button components
  - `views/ascii/_*.erb`: Dice visualization SVGs
- Helper methods: `link_button`, `submit_button`, `ascii_dice`

## Key Implementation Details

### WebSocket Message Flow

1. Client connects to `/ws?room_id=X` with session authentication
2. WebSocket added to room-specific array in `settings.sockets[room_id]`
3. On message receive:
   - Parse for `/roll` commands and execute dice rolls
   - Save message to `chat_messages` table
   - Render HTML using `render_message` helper
   - Broadcast rendered HTML to all room participants
4. Client receives pre-rendered HTML and appends to chat

### Infinite Scroll

Endpoint `/rooms/:id/messages?offset=N` returns 15 older messages at a time as rendered HTML. Initial page load shows last 15 messages.

### AI Game Master System

Implemented in `lib/ai_game_master.rb`, this autonomous agent runs text-based RPG adventures:

**Initialization:**
- Created when a room is created with the AI user as Game Master
- Instances tracked in `settings.ai_game_masters` hash (keyed by room_id)
- Initializes with room name/description to generate adventure context
- Uses OpenAI's GPT-4 with function calling for game control

**AI Capabilities (via function calling):**
- `send_message`: Send narration and NPC dialogue to the room
- `mute_player`/`unmute_player`: Control turn order and game flow
- `ask_for_dice_roll`: Request specific dice rolls from players with context
- `get_raised_hands`: See which muted players want to speak
- `check_existence`: Determine if items/features exist (60% probability) - adds realism
- `end_game`: End adventure with final message, "THE END", and mute all players

**Event Triggers:**
- Player messages (including dice roll results) → `handle_player_message`
- Player joins room → `handle_player_joined`
- Player leaves room → `handle_player_left`
- Hand raised → `handle_hand_raised`

**Conversation History:**
- Maintains full game context in `@conversation_history`
- Includes system prompts, player actions, dice results, and AI responses
- Function call results fed back to AI for continued decision-making

**Threading:**
- AI processing runs in background threads to avoid blocking WebSocket
- Messages broadcast via existing WebSocket infrastructure

**AI GM Guidelines:**
- **Combat Flow**: AI mutes player → describes attack → rolls dice → explains outcome → unmutes player
- **Narrative Control**: AI controls story progression; players describe actions but don't dictate outcomes
- **No Emotional Assumptions**: AI never tells players how they feel, only what they perceive
- **Selective Suggestions**: AI doesn't suggest actions unless explicitly asked
- **Realistic World**: Uses `check_existence` to determine if requested items/features exist (60% probability)

### Mute and Hand Raise System

**Muting:**
- GMs can mute players to control turn order (`user_rooms.is_muted`)
- Muted players cannot send messages (enforced client and server-side)
- AI GM can programmatically mute/unmute via function calls
- GMs cannot mute themselves

**Hand Raising:**
- Muted players can raise hands to signal they want to speak (`user_rooms.hand_raised`)
- Hand status shown with ✋ emoji in GM's mute controls
- AI GM can query raised hands and respond appropriately
- Routes: `/rooms/:id/raise-hand` and `/rooms/:id/lower-hand`

**WebSocket Events:**
- `mute_status`: Broadcasts when player is muted/unmuted
- `hand_raised`: Broadcasts when player raises/lowers hand
- Real-time UI updates for all participants

### Environment Variables

- `SESSION_SECRET`: Used for session encryption (auto-generated if not provided)
- `OPENAI_API_KEY`: API key for OpenAI integration (required for AI Game Master)

---
> Source: [MakeThingsWorkDotDev/a-thing-that-lets-you-play-a-text-adventure-with-your-friends](https://github.com/MakeThingsWorkDotDev/a-thing-that-lets-you-play-a-text-adventure-with-your-friends) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-12 -->
