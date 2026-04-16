## draw-something

> Describes the project we are trying to build

# Cursor Rules File for Drawing and Guessing Game

## Project Structure
```
/
├── server/         # Server-side code
│   ├── src/        # Server source code
│   ├── data/       # Word lists and game data
│   └── package.json # Server dependencies
├── client/         # Client-side code
│   ├── js/         # JavaScript files
│   ├── css/        # CSS styles
│   ├── assets/     # Images and other assets
│   └── index.html  # Main HTML file
```

## Code Style Rules

### General
- Use camelCase for variable and function names
- Use PascalCase for class names
- Use descriptive variable names
- Maximum line length: 100 characters
- Indent using 2 spaces
- Use semicolons at the end of statements

### JavaScript (Client & Server)
- Prefer `const` over `let` when variables aren't reassigned
- Use template literals for string concatenation
- Use arrow functions for callbacks
- Use async/await for asynchronous operations
- Use destructuring where appropriate
- Add JSDoc comments for functions

### Client-specific
- Organize code into modules with clear responsibilities
- Keep DOM manipulation code separate from game logic
- Use event delegation for dynamic elements
- Prefix private variables/functions with underscore (_)
- Use data attributes for DOM elements related to game state

### Server-specific
- Use meaningful HTTP status codes
- Validate all user input
- Handle errors gracefully
- Log important events
- Use environment variables for configuration

## Implemented Features

### Core Functionality
- Real-time WebSocket communication between client and server
- Room-based multiplayer system with unique room IDs
- Complete game flow with turn management
- Dynamic screen management (home, lobby, game screens)
- Responsive design for various device sizes

### Drawing Tools
- Canvas-based drawing system with different brush sizes
- Multiple color options for drawing
- Fill tool for coloring areas
- Undo functionality for drawing mistakes
- Real-time drawing updates for all players

### Game Mechanics
- Word selection with difficulty levels (Easy, Medium, Hard)
- Random word selection from word pools
- Timer system for different game phases
- Scoring system based on correct guesses and word difficulty
- Close guess detection with visual feedback
- Previous drawings gallery to view past artwork

### User Interface
- Chat system for guessing and communication
- Scroll-to-bottom button for chat navigation
- Visual indicators for game state and turn information
- Game settings management for room creators
- Disabled settings indicators for non-room creators
- Mobile-friendly layout adjustments

### Technical Features
- Offline mode for development and testing
- Message queuing for handling connection issues
- Dynamic server URL configuration based on environment
- Optimized WebSocket message handling
- Improved canvas dimensions and layout

## Game Logic Rules

### Room Management
- Rooms have unique IDs
- Maximum players per room: 8
- Minimum players to start a game: 2
- Room creator can configure game settings
- Game settings can only be modified by the room creator

### Turn Management
- Turns proceed in the order players joined
- Each turn has phases: word selection, drawing, guessing
- Time limits: 
  - Word selection: 15 seconds
  - Drawing: 60/90/120 seconds (Easy/Medium/Hard)
  - Between turns: 5 seconds
- Turn timers are managed server-side for consistency

### Scoring System
- First correct guess: 100 points
- Subsequent correct guesses: 50 points
- Drawer gets 25 points per correct guess
- Bonus points for difficult words
  - Easy: no bonus
  - Medium: 25% bonus
  - Hard: 50% bonus

### Word Selection
- Drawer chooses from 3 options (Easy, Medium, Hard)
- Words are organized by difficulty level in the UI
- Words should not repeat in the same game session
- If drawer doesn't select in time, a random word is chosen
- Used words are reset for new games

### Guessing Rules
- Guesses are not case sensitive
- Close matches are detected and provide feedback to the player
- Typing the exact word is required for a correct match
- Players cannot guess during their drawing turn
- All guesses are visible in the chat

## Technical Implementation Notes

### Communication Protocol
- Use WebSockets for real-time communication
- JSON format for all messages
- Include message type and payload in all communications
- Message queuing for handling offline scenarios

### Drawing Implementation
- Canvas-based drawing system
- Send drawing data as incremental updates
- Support different brush sizes and colors
- Fill tool for coloring areas
- Undo functionality with drawing history
- Optimize network traffic by batching drawing events

### Security Considerations
- Sanitize all user input
- Rate limit API requests
- Implement anti-cheat measures
- Moderate chat content

### Performance Optimization
- Minimize DOM updates
- Use requestAnimationFrame for animations
- Implement debouncing for frequent events
- Optimize WebSocket message size
- Fixed canvas dimensions for consistent drawing experience

## Development Workflow
- Run tests before committing
- Use feature branches
- Document API changes
- Keep the README updated

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Nikamura) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
