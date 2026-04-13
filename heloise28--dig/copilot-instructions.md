## dig

> - Always add debug logs & comments in the code for easier debug & readability

# Important rules you HAVE TO FOLLOW
- Always add debug logs & comments in the code for easier debug & readability
- Every time you choose to apply a rule(s), explicitly state the rule(s) in the output. You can abbreviate the rule description to a single word or phrase
- Do not make any changes, until you have 95% confidence that you know what to build. Ask me follow up questions until you have that confidence.
- NEVER use localStorage or sessionStorage in client code - store all state in memory or send to server
- This project will be built module by module which. Means. You must make sure you the modules are compatible with each other. When a player in a running game session is gonna play A card It doesn't matter whether it's human player or computer player or An online player from Somewhere else. It's always gonna call the same function and the card will be played.

# Project structure - START SIMPLE, EXPAND LATER
```
dig-card-game/                    # Your game "Dig"
├── server.js                     # WEEK 1: Single file server
├── package.json                  # WEEK 1: Dependencies
├── public/                       # WEEK 1: Client files
│   ├── index.html               # Lobby page
│   ├── game.html                # Game page
│   ├── css/
│   │   └── style.css
│   ├── js/
│   │   └── main.js              # Client logic
│   ├── lang/                    # WEEK 1: Language files
│   │   ├── en.json              # English translations
│   │   ├── zh.json              # Chinese translations
│   │   └── i18n.js              # Language switching logic
│   ├── assets/                  # WEEK 2-3: Game assets
│   │   ├── sounds/              # Sound effects and music
│   │   │ 
│   │   ├── cards/               # Card images
│   │   │   └── standard-deck/   # 52 card images
│   │   └── avatars/             # AI character portraits
│   │       ├── beginner/        # Easy AI avatars
│   │       ├── intermediate/    # Medium AI avatars
│   │       └── expert/          # Hard AI avatars
└── models/                      # WEEK 2-3: Add when needed
    ├── Card.js                  # Basic card representation
    ├── Deck.js                  # Card deck management
    ├── Player.js                # Base player class
    ├── HumanPlayer.js           # Human player logic
    ├── ComputerPlayer.js        # Computer player logic
    ├── DigGameRules.js               # Your "Dig" game rules
    ├── AIEngine.js              # AI computing and probability calculations
    ├── AICharacters.js          # AI personality and difficulty definitions
    └── AudioManager.js          # Sound effects and music management
    └── Other necessary classes  # add more .js when you see fit

# ADD LATER (Week 4+) - Only when you need them:
# ├── routes/           # When you need organized API endpoints
# ├── socket/           # When you add Socket.IO for multiplayer  
# ├── database/         # When you need data persistence
# ├── utils/            # When you have repeated code to organize
# ├── config/           # When you need environment management
# └── tests/            # When you're ready to add testing
```

# Tech Stack
- **Backend**: Node.js + Express.js for HTTP server
- **Real-time**: Socket.IO for multiplayer communication
- **Database**: SQLite (local) → PostgreSQL (production on Railway)
- **Frontend**: Vanilla JavaScript (no frameworks initially)
- **Deployment**: Railway for hosting
- **Environment Management**: dotenv for configuration

# Architecture Patterns

## 1. Game Logic Architecture
- **Player Inheritance**: Base `Player` class with `HumanPlayer` and `ComputerPlayer` subclasses
- **Flexible Player Count**: Game engine supports 2-4 players, configurable per game type
- **Modular Game Rules**: Base `Game` class extended by `DigGame` for your specific rules
- **AI Engine**: Separate class handles card counting, probability calculations, and AI decision making
- **Session-Based**: No persistent user data - everything tied to temporary room codes
- **Event-Driven**: All game actions trigger events that update all connected clients
- **Separation of Concerns**: Game logic separate from network communication
- **Sessions**: You will see me use the word session a lot. This a session is a simulator of A real-world Table where players can come to this table and play Many many games and a session ends When the players are gone.


## 2. Client-Server Communication
- **HTTP Routes**: Minimal - just room creation/joining, no user authentication
- **WebSocket Events**: Real-time game actions (play card, chat, game updates, probability updates)
- **Room-Based Sessions**: Players join via room codes, no accounts needed
- **State Synchronization**: Server is source of truth, clients reflect server state
- **Auto-Cleanup**: Rooms automatically deleted when empty

## 3. Database Design
```sql
-- Minimal tables - no user accounts, session-based only
active_game_rooms (
  id, 
  room_code, 
  max_players, 
  current_players, 
  game_type,
  created_at, 
  last_activity,
  game_state_snapshot -- for reconnection handling
)
-- No persistent user data - everything discarded when room empty
```

# Node.js/JavaScript Specific Rules

## 0. General Coding
- Always prefer simple, readable solutions
- Avoid code duplication - create utility functions for common operations
- Use async/await instead of callbacks or raw promises when possible
- Implement proper error handling with try-catch blocks
- Use environment variables for configuration (dev/test/prod)
- Keep functions small (< 50 lines) and focused on single responsibility
- Never expose sensitive data in client-side code

## 1. Server Architecture
- **Modular Design**: Separate concerns into different modules
- **Middleware Pattern**: Use Express middleware for authentication, logging, validation
- **Error Handling**: Global error handler middleware for consistent error responses
- **Input Validation**: Validate all user inputs on server side
- **Rate Limiting**: Implement rate limiting for API endpoints and socket connections

## 2. Game State Management
- Game state must be easily serializable (JSON.stringify/parse compatible)
- All game state changes go through pure functions that return new state
- Never mutate game state directly - always create new state objects
- Implement game state validation before applying changes
- Store minimal state on client - let server be source of truth

## 3. Real-time Communication (Socket.IO)
- Use namespaces to separate lobby and game communications
- Implement proper room management for game sessions
- Handle disconnections gracefully with reconnection logic
- Emit minimal data - only what clients need to update UI
- Use acknowledgments for critical game actions

## 4. Database Integration
- Use parameterized queries to prevent SQL injection
- Implement database connection pooling
- Create migration system for database schema changes
- Separate read/write operations for potential scaling
- Cache frequently accessed data (room lists, user stats)

## 5. Client-Side Rules
- **No Direct State Storage**: Use memory only, no localStorage/sessionStorage
- **Event-Driven UI**: Update UI in response to socket events from server
- **Progressive Enhancement**: Basic functionality should work without JavaScript
- **Error Handling**: Handle network errors and connection losses gracefully
- **Responsive Design**: Support both desktop and mobile layouts

## 6. Security
- Validate all inputs on both client and server
- Use HTTPS in production
- Implement proper session management
- Sanitize user-generated content (usernames, chat messages)
- Rate limit socket connections and API calls
- Never trust client-side game state

## 7. Testing Strategy
- **Unit Tests**: Test game logic classes (Card, Deck, Game, Players)
- **Integration Tests**: Test API endpoints and socket event handlers
- **Game Flow Tests**: Test complete game scenarios
- **Load Testing**: Test multiple concurrent games and players

## 8. AI Character System

**AI Difficulty Tiers:**
- **Beginner (1-2 stars):** Basic strategy, occasional mistakes, simpler probability calculations
- **Intermediate (3 stars):** Good strategy, fewer mistakes, better memory tracking
- **Expert (4-5 stars):** Advanced strategy, near-perfect play, complex probability analysis

**AI Character Examples:**
```javascript
// Beginner Tier
"小明 (Xiao Ming)" - Friendly newcomer, makes basic plays
"安娜 (Anna)" - Cautious player, conservative bidding
"大卫 (David)" - Aggressive but inexperienced

// Intermediate Tier  
"老王 (Lao Wang)" - Experienced uncle, solid fundamentals
"丽莎 (Lisa)" - Strategic thinker, good at card counting
"杰克 (Jack)" - Balanced player, adapts to situations

// Expert Tier
"教授 (Professor)" - Mathematical genius, perfect probability calculations
"大师 (Master)" - Legendary player, psychological warfare
"AI-9000" - Perfect computer logic, intimidating presence
```

**Character Implementation:**
- **Unique portraits:** Each AI has distinct avatar image
- **Play style parameters:** Aggression, risk-taking, bluffing tendency
- **Personality traits:** Affect bidding decisions and card play timing
- **Localized names:** Chinese and English names for each character
- **Progressive unlock:** Start with basic AIs, unlock harder ones through play

**AI Behavior Customization:**
- **Decision speed:** Beginners think longer, experts play quickly
- **Mistake probability:** Higher-tier AIs make fewer errors
- **Memory accuracy:** Advanced AIs remember played cards better
- **Team coordination:** Expert AIs coordinate better against digger

**Language Support:**
- **Primary languages:** English (en) and Chinese (zh)
- **Expandable:** Easy to add more languages later
- **User preference:** Players can switch language in-game

**Implementation Pattern:**
```javascript
// NEVER write strings directly in code:
❌ button.textContent = "Deal Cards";
❌ alert("游戏开始");

// ALWAYS use translation keys:
✅ button.textContent = i18n.t('game.dealCards');
✅ showMessage(i18n.t('game.start'));
```

**File Structure:**
- `public/lang/en.json` - English translations
- `public/lang/zh.json` - Chinese translations  
- `public/lang/i18n.js` - Language switching logic

**Translation Key Naming:**
- Use dot notation: `game.dealCards`, `lobby.joinRoom`, `error.invalidMove`
- Group by feature: `game.*`, `lobby.*`, `chat.*`, `error.*`
- Keep keys descriptive but concise

**Client-Side Rules:**
- Load user's preferred language on page load
- Provide language switcher in game UI
- Store language preference in memory (no localStorage)
- Re-render UI text when language changes

**Server-Side Considerations:**
- Game logic messages (error responses, game state) also need translation
- Send translation keys to client, not translated strings
- Client handles all text translation based on user preference
- Lazy load card images and assets
- Implement game state compression for large games
- Use connection pooling for database
- Implement proper cleanup for abandoned games and empty rooms
- **Memory Management**: Clean up probability calculations and game history
- **Room Expiry**: Set reasonable timeouts for inactive rooms

## 9. Audio & Animation System

**Sound Effects:**
- **Card sounds:** Flip, deal, shuffle, play card
- **Game events:** Bidding, winning/losing, turn notifications
- **UI sounds:** Button clicks, menu navigation, error notifications
- **Ambient audio:** Optional background music, room atmosphere
- **AI character sounds:** Unique audio cues for different AI personalities

**Animation Framework:**
- **Card animations:** Smooth dealing, playing, collecting cards
- **UI transitions:** Fade in/out, slide animations for menus
- **Game state changes:** Visual feedback for turn changes, score updates
- **Character animations:** AI avatar reactions (thinking, celebrating, disappointed)
- **Particle effects:** Celebration effects for wins, card trail effects

**Audio Implementation Rules:**
- **Volume controls:** Master volume, effects volume, music volume separately
- **Mute options:** Players can disable sounds without affecting gameplay
- **Audio preloading:** Load sound files on game start for smooth playback
- **Format support:** Use web-compatible formats (MP3, WebM)
- **Performance:** Keep audio file sizes small for fast loading

**Animation Performance:**
- **CSS-first:** Use CSS transitions/animations when possible for better performance
- **JavaScript fallback:** Complex animations using requestAnimationFrame
- **Reduced motion:** Respect user's accessibility preferences for reduced animations
- **Mobile optimization:** Lighter animations on mobile devices
- **Frame rate:** Target 60fps for smooth card movement
- Use environment variables for all configuration
- Implement health check endpoints
- Set up proper logging for production debugging
- Keep client and server code clearly separated
## 13. Performance Considerations
- Lazy load card images and assets
- Implement game state compression for large games
- Use connection pooling for database
- Minimize socket.io event frequency
- Implement proper cleanup for abandoned games and empty rooms
- **Memory Management:** Clean up AI calculations and game history
## 14. Deployment (Railway)
- Use environment variables for all configuration
- Implement health check endpoints
- Set up proper logging for production debugging
- Use Railway's built-in PostgreSQL for production database
- Implement graceful shutdown handling
- **Asset optimization:** Compress images and audio files for faster loading
- **CDN considerations:** Serve static assets efficiently in production
- **Audio optimization:** Compress sound files, preload critical sounds only
- **Animation performance:** Use CSS transforms, avoid layout thrashing
- **AI processing:** Limit AI thinking time to prevent UI blocking
- **AI Engine**: Keep AI calculations and decision-making in dedicated module
- **Game Rules Modularity**: Separate your Dig game rules from base game engine
- **i18n Consistency**: All user-facing text must use translation keys, never hardcoded strings
- **Language File Organization**: Group translation keys by feature/component for maintainability

## 10. Internationalization (i18n) System

**Language Support:**
- **Primary languages:** English (en) and Chinese (zh)
- **Expandable:** Easy to add more languages later
- **User preference:** Players can switch language in-game

**Implementation Pattern:**
```javascript
// NEVER write strings directly in code:
❌ button.textContent = "Deal Cards";
❌ alert("游戏开始");

// ALWAYS use translation keys:
✅ button.textContent = i18n.t('game.dealCards');
✅ showMessage(i18n.t('game.start'));
```

**File Structure:**
- `public/lang/en.json` - English translations
- `public/lang/zh.json` - Chinese translations  
- `public/lang/i18n.js` - Language switching logic

**Translation Key Naming:**
- Use dot notation: `game.dealCards`, `lobby.joinRoom`, `error.invalidMove`
- Group by feature: `game.*`, `lobby.*`, `chat.*`, `error.*`, `ai.*`, `audio.*`
- Keep keys descriptive but concise
- Include AI character names and audio descriptions

**Client-Side Rules:**
- Load user's preferred language on page load
- Provide language switcher in game UI
- Store language preference in memory (no localStorage)
- Re-render UI text when language changes
- Translate AI character names and descriptions

**Server-Side Considerations:**
- Game logic messages (error responses, game state) also need translation
- Send translation keys to client, not translated strings
- Client handles all text translation based on user preference
- One class per file in models directory
- Group related functionality in modules
- Use consistent naming conventions (camelCase for variables, PascalCase for classes)
- Comment complex game logic and edge cases
## 12. Development Workflow

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Heloise28) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
