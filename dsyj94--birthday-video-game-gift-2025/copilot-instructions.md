## birthday-video-game-gift-2025

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **complete 3D birthday game** created as a birthday present for Pea, who is turning 29 and loves knitting and law. The game features a character version of herself in third-person mode, navigating three rooms to complete challenges and unlock birthday videos.

### Game Concept
- **Player Character**: Pea - detailed 3D character with medium-length dark blonde hair
- **Companion**: Roma the cocker spaniel - intelligent AI companion that follows the player
- **Three Rooms**: Main room (starting area), Law room (courtroom quiz), Knitting room (crafting minigame)
- **Two Challenges**: Complete both to unlock the birthday chest
  1. **Law Quiz**: 9 basic UK law questions (Contract, Tort, Company law) in a courtroom setting
  2. **Knitting Minigame**: Create 3 pieces of knitwear (sweater, beanie, gloves) with materials and methods
- **Victory Condition**: Open chest to play personalized birthday videos 

## Development Setup

### Tech Stack
- **Three.js** - 3D graphics library for web-based game development
- **Vite** - Fast build tool and dev server
- **HTML5/CSS3** - Core web technologies
- **Vanilla JavaScript** - Keeping it simple without frameworks

### Commands
```bash
# Install dependencies
npm install

# Run development server (opens at localhost:3000)
npm run dev

# Build for production
npm run build
```

## Architecture

### Project Structure
```
Birthday/
├── index.html          # Main entry point
├── package.json        # Project dependencies
├── vite.config.js      # Vite configuration
├── src/
│   ├── main.js         # Game initialization
│   ├── game/
│   │   ├── Game.js     # Main game controller
│   │   ├── Player.js   # Player character controller
│   │   └── Camera.js   # Camera controls
│   ├── scenes/
│   │   ├── MainRoom.js # Starting room scene
│   │   ├── LawRoom.js  # Law quiz room
│   │   └── KnitRoom.js # Knitting minigame room
│   ├── minigames/
│   │   ├── LawQuiz.js  # Law quiz logic
│   │   └── Knitting.js # Knitting game logic
│   └── utils/
│       └── AssetLoader.js # Load 3D models and textures
├── assets/
│   ├── models/         # 3D models (character, furniture)
│   ├── textures/       # Textures and images
│   ├── videos/         # Birthday videos
│   └── sounds/         # Sound effects
└── style.css           # Basic styling

## Development History

**Development completed across 13 sessions** - detailed logs available in `DEVELOPMENT_HISTORY.md` if needed.

**Sessions 1-4**: Core game development (characters, rooms, minigames, video player)
**Sessions 5-7**: UI polish, Roma companion, bug fixes, progress tracking  
**Sessions 8-9**: Video player system, background music implementation
**Sessions 10-12**: Enhanced courtroom NPCs, dynamic chat bubbles, atmospheric animations
**Session 13**: Quick enhancements - .bat startup file, particle confetti system, enhanced lighting

**Current Status**: Production-ready with known issues being debugged.

### Session 13 - Quick Enhancements & Bug Fixes (In Progress)
**Completed:**
- ✅ Created `start-game.bat` file for easy one-click game startup
- ✅ Enhanced warm lighting across all rooms (brighter ambient, warmer colors, additional point lights)
- ✅ Integrated particle confetti system (150 confetti + 20 hearts + 50 sparkles on chest open)

**Known Issues (Debugging in Progress):**
- ⚠️ Confetti particles not rendering visually (system exists but particles not visible)
  - Added debug red cube to verify particle system
  - Console logging for particle creation and updates
  - Increased particle sizes and adjusted spawn positions
- ⚠️ Player movement locked after opening chest/closing video
  - Fixed property mismatch (`lockMovement` vs `isMinigameActive`)
  - Added key reset and pointer lock re-request
  - Console logging for movement state changes

**Debug Changes Made:**
- ParticleSystem.js: Added test red cube spawn, increased particle sizes, added debug logging
- Game.js: Fixed movement lock property, added pointer lock re-request, world position fixes
- All rooms: Added warm lighting methods with multiple point lights

**Next Session TODO:**
1. Verify if red test cube appears (indicates particle system works)
2. Check console for particle creation logs
3. Test if particles need different render order or material settings
4. Confirm movement unlock works with pointer lock re-engagement

## 🎉 ENHANCED COMPLETE GAME VERSION 🎂

**Status: PRODUCTION READY - ENHANCED ATMOSPHERIC GAME**

As of Session 12 completion, this represents the **enhanced atmospheric game** with dynamic courtroom chatter, living main room environment, and comprehensive animation systems. This is the current production-ready version with all core features, audio experience, enhanced NPC interactions, dramatic plant animations, warm lighting atmosphere, and realistic character behaviors.

### **🎮 Complete Game Features:**

**Core Gameplay:**
- ✅ Professional 3D third-person character (Pea) with detailed model and animations
- ✅ Three fully designed rooms (Main, Law, Knitting) with collision detection
- ✅ Professional mouse-look camera system with pointer lock
- ✅ WASD movement controls with smooth animations
- ✅ Interactive object highlighting and on-screen hints

**Minigames:**
- ✅ **Law Quiz Challenge**: 9 UK law questions (Contract, Tort, Company law)
  - Multiple choice with detailed explanations
  - Personalized feedback for Pea
  - Professional courtroom-style UI
  - ESC key quit functionality
  - **Enhanced Courtroom Atmosphere**: Judge NPC, jury, audience, and lawyers
  - Interactive Judge character with judicial attire and animations
  - **Dynamic Chat Bubbles**: Authentic courtroom chatter from all NPCs
  - **Professional Navigation**: Wide center aisle with clear path to podium
  - Proper collision detection preventing clipping through courtroom furniture
  - **Realistic Dialogue**: Color-coded bubbles (Blue=Jury, Red=Lawyers, Purple=Audience)
- ✅ **Knitting Challenge**: Create 3 pieces of knitwear
  - Material and method selection system
  - Interactive progress bars and animations
  - Personalized crafting feedback
  - Complete item creation workflow

**Enhanced Atmospheric World:**
- ✅ **Roma the Dog**: Intelligent cocker spaniel companion
  - Realistic following AI behavior
  - **Enhanced Animations**: Walking, tail wagging, head bobbing, **ear twitching**
  - Activity-based ear movement (active when walking, subtle when idle)
  - Interactive speech bubbles with "Roma: Woof Woof!"
  - Follows player to all rooms with realistic dog behaviors
- ✅ **Living Main Room Environment**:
  - **4 Animated Plants**: Dramatic wind-like swaying around the room
  - **Professional Floor Lamp**: Bright warm golden lighting (2.5 intensity)
  - **Interactive Chest Sign**: "Play to unlock!" with obvious bobbing animation
  - **Realistic Plant Movement**: Individual leaf animation with twist and sway motion
- ✅ **Room Navigation**: Door interaction system with handles and animations
- ✅ **Room Signs**: Clever puns with arrows pointing to each challenge

**UI & Progress:**
- ✅ **Progress Tracker HUD**: Real-time status of both challenges and chest
- ✅ **Professional Controls HUD**: Clear key binding display
- ✅ **Dynamic Objectives**: Context-aware instructions and feedback
- ✅ **Background Music System**: Automatic looping music with smart autoplay handling
  - Music toggle controls with visual feedback
  - Fade effects during video player transitions
  - Preserves user preferences across game sessions

**Game Completion:**
- ✅ **Chest Unlock System**: Automatically unlocks after completing both challenges
- ✅ **Birthday Video Player**: Professional HTML5 video player
  - Auto-detects videos in assets/videos/ folder
  - Supports multiple formats (.mp4, .webm, .ogg, etc.)
  - Clean, personal presentation without technical details
  - Auto-plays birthday messages with full controls
- ✅ **Animated Chest Opening**: Smooth lid rotation animation
- ✅ **Birthday Celebration Screen**: Beautiful gradient background with animations

### **🔧 Technical Implementation:**
- Built with **Three.js** for 3D graphics
- **Vite** development server and build system
- **Vanilla JavaScript** - no framework dependencies
- Collision detection system for all rooms and objects
- Character animation system with walking cycles
- Mouse pointer lock for professional game feel
- Responsive UI with CSS animations
- Cross-browser HTML5 video support

### **🎯 Game Flow (Complete):**
1. **Start**: Player spawns in main room with Roma
2. **Exploration**: See room signs, discover locked chest
3. **Law Challenge**: Complete 9 UK law questions in courtroom
4. **Knitting Challenge**: Create sweater, beanie, and gloves
5. **Victory**: Chest unlocks automatically, Roma celebrates too
6. **Birthday Finale**: Professional video player with personal messages
7. **Completion**: Full birthday celebration experience

### **📁 Asset Structure:**
```
Birthday/
├── index.html           # Main entry point
├── package.json         # Dependencies
├── vite.config.js       # Build configuration
├── style.css            # All UI styling
├── src/
│   ├── main.js          # Game initialization
│   ├── game/
│   │   ├── Game.js      # Main game controller (complete)
│   │   ├── Player.js    # Character controller (complete)
│   │   └── Camera.js    # Camera system (complete)
│   ├── scenes/
│   │   ├── MainRoom.js  # Main room with furniture (complete)
│   │   ├── LawRoom.js   # Courtroom scene (complete)
│   │   └── KnitRoom.js  # Crafting room scene (complete)
│   ├── minigames/
│   │   ├── LawQuiz.js   # 9 law questions system (complete)
│   │   └── Knitting.js  # 3-item crafting system (complete)
│   └── utils/
│       ├── CharacterModel.js    # Pea's character model
│       ├── DogCharacter.js      # Roma's AI and animations
│       ├── JudgeCharacter.js    # Judge NPC with judicial attire
│       ├── CollisionDetection.js # Physics system
│       └── AudioManager.js      # Background music system
└── assets/
    ├── audio/           # Background music files
    │   └── background-music.mp3  # Looping background music
    └── videos/          # Birthday video files go here
        └── videoplayback.mp4  # User's video (working)
```

### **🚀 Deployment Ready:**
- Run `npm run dev` for development
- Run `npm run build` for production
- All core features tested and working
- No known bugs or issues
- Performance optimized for smooth gameplay

---

## ⚠️ IMPORTANT: BASE GAME LOCK

**This version (end of Session 8) is the FINAL COMPLETE BASE GAME.**

Any future development should be considered **optional enhancements** that build upon this solid foundation. This version should be preserved as the reference implementation.

---

### Session 9 - Optional Future Enhancements

Now that the base game is complete, these are **optional improvements** that could enhance the experience further. Each category is prioritized by impact and difficulty.

**Priority 1: Audio & Atmosphere (High Impact)**
- [ ] **Sound effects and music** (requires careful performance implementation)
  - Background music for each room (cozy main room, formal law room, creative knitting room)
  - Sound effects for interactions (door opens, chest unlock, button clicks)
  - Roma barking sounds when interacted with
  - UI feedback sounds (correct/wrong answers, completion chimes)
  - Footstep sounds (with proper timing to avoid performance issues)
  - Victory fanfare when chest opens
  - **Challenge**: Previous attempt caused severe performance issues - needs optimization

- [ ] **Enhanced Visual Atmosphere**
  - Particle systems (birthday confetti when chest opens, dust motes in sunlight)
  - Improved lighting (warmer room ambiance, spotlights on important objects)
  - Environmental animations (plants swaying gently, curtains moving)
  - Weather effects visible through windows
  - Candles with flickering flames on furniture
  - Subtle screen transitions between rooms (fade in/out)

**Priority 2: User Experience & Accessibility (Medium Impact)**
- [ ] **Settings and Controls**
  - Settings menu with volume sliders (master, music, SFX)
  - Graphics quality options (low/medium/high for different devices)
  - Camera sensitivity adjustment
  - Colorblind accessibility options
  - Font size options for better readability
  - Pause/resume functionality

- [ ] **Mobile and Touch Support**
  - Touch controls for mobile devices
  - Responsive design for different screen sizes
  - Virtual joystick for movement on phones/tablets
  - Touch-friendly UI elements
  - Orientation lock for better mobile experience

- [ ] **Save System and Progress**
  - Save game progress (so players don't lose progress if they close browser)
  - Multiple save slots
  - Achievement tracking (first quiz attempt, perfect scores, etc.)
  - Time tracking (how long to complete each challenge)

**Priority 3: Content Expansion (Medium Impact)**
- [ ] **Extended Minigames**
  - **Law Quiz Expansion**:
    - Additional difficulty levels (beginner, intermediate, expert)
    - More law categories (criminal law, EU law, property law)
    - Timed challenges for advanced players
    - Score tracking and leaderboards
    - Question randomization for replayability
  
  - **Knitting Game Enhancement**:
    - More knitwear items (scarf, mittens, socks, cardigan)
    - Advanced patterns and techniques
    - Color selection for yarn
    - Quality ratings based on material/method choices
    - Gallery to view completed items

- [ ] **Character Customization**
  - Outfit changes for Pea (casual, formal lawyer attire, cozy knitting clothes)
  - Hair style options
  - Accessories (glasses, jewelry, lawyer's wig for law room)
  - Roma accessories (cute bandanas, birthday hat)

- [ ] **World Expansion**
  - Additional rooms (bedroom, kitchen, study, garden)
  - More interactive objects and furniture
  - Hidden easter eggs and secrets throughout the world
  - Photo mode for capturing special moments
  - More furniture and decorative items

**Priority 4: Advanced Features (Lower Priority, High Effort)**
- [ ] **Social and Sharing Features**
  - Screenshot/photo mode with filters
  - Shareable completion certificates
  - Social media integration for sharing achievements
  - Guest book for birthday messages from family/friends
  - Video recording of gameplay moments

- [ ] **Performance and Technical**
  - Level of Detail (LOD) system for complex scenes
  - Texture streaming for better memory usage
  - Frame rate monitoring and automatic quality adjustment
  - Loading screen with progress bars
  - Better error handling and recovery
  - Gamepad/controller support
  - VR mode support (ambitious!)

- [ ] **Seasonal and Event Content**
  - Holiday themes (Christmas decorations, Halloween costumes)
  - Anniversary mode (celebrating relationship milestones)
  - Birthday reminders for subsequent years
  - Special event challenges
  - Time-based unlocks and surprises

**Priority 5: Developer and Meta Features (Technical)**
- [ ] **Analytics and Insights**
  - Play session tracking (how long people play)
  - Completion rates for each minigame
  - Most popular features and interactions
  - Performance metrics across different devices

- [ ] **Localization**
  - Multiple language support
  - Cultural adaptations of law questions
  - Different regional knitting traditions

- [ ] **Advanced AI**
  - More sophisticated Roma AI (different moods, reactions to player actions)
  - Dynamic dialogue system
  - Procedural content generation for infinite replayability

## 🎯 **Recommended Next Steps (If Desired):**

**Easy Wins (Low Effort, High Impact):**
1. **Particle confetti** when chest opens (30 minutes)
2. **Better lighting** in rooms with warmer tones (1 hour)
3. **Settings menu** with volume controls (2 hours)

**Medium Projects (Moderate Effort, Good Impact):**
1. **Background music system** (4-6 hours, requires careful performance testing)
2. **Mobile touch controls** (6-8 hours)
3. **Extended law quiz** with more questions (3-4 hours)

**Major Projects (High Effort, High Impact):**
1. **Complete audio system** with all sound effects (8-12 hours)
2. **Additional rooms and content** (10-15 hours)
3. **Save system and progress tracking** (6-10 hours)

## ⚠️ **Important Considerations:**

- **Performance First**: Any new features must not impact the smooth gameplay
- **Testing Required**: Each addition should be thoroughly tested
- **Incremental Development**: Add one feature at a time to maintain stability
- **User Feedback**: Consider what would most enhance Pea's experience
- **Time Investment**: Weigh development time against actual improvement to the birthday gift

The base game is already a wonderful, complete birthday present. These enhancements are purely optional and should only be added if there's time and interest in further development!
  - Sound effects for interactions (doors, chest, buttons) 
  - Roma barking sounds (on interaction only)
  - UI click sounds for minigames
  - **Requirements**: Timer-based triggers, efficient loading, volume controls

- [ ] **Enhanced Visual Effects**
  - Particle systems (birthday confetti when chest opens)
  - Lighting improvements (warmer room lighting, shadows)
  - Simple animation tweaks (plant swaying, flickering candles)
  - Screen transitions between rooms (fade in/out)

**Priority 2: Quality of Life Improvements**
- [ ] **Better UI/UX**
  - Settings menu (volume controls, graphics quality)
  - Pause functionality 
  - Game progress saving/loading
  - Better mobile touch controls
  - Accessibility features (larger text, colorblind support)

- [ ] **Character & World Enhancements**
  - More character expressions/emotions
  - Additional furniture and room details
  - Weather effects outside windows
  - Day/night cycle or different lighting moods

**Priority 3: Performance & Compatibility**
- [ ] **Performance Optimizations**
  - LOD (Level of Detail) system for distant objects
  - Texture optimization and compression
  - Frame rate monitoring and adaptive quality
  - Memory usage optimization
  - Efficient collision detection improvements

- [ ] **Cross-Platform Support**
  - Mobile compatibility (touch controls, responsive design)
  - Browser compatibility testing (Chrome, Firefox, Safari, Edge)
  - Different screen size support
  - Gamepad/controller support

**Priority 4: Advanced Features**
- [ ] **Mini-Game Expansions**
  - More law questions with different difficulty levels
  - Additional knitting patterns and items
  - Time-based challenges or scoring systems
  - Achievement system

- [ ] **Social Features**
  - Photo mode (screenshot functionality)
  - Share completion certificates
  - Easter eggs and hidden secrets
  - Developer commentary mode

## Design Decisions

### Visual Style
- Low-poly 3D models for simplicity and charm
- Warm, cozy color palette (pastels, earth tones)
- Soft lighting to create inviting atmosphere

### Character Design
- Simple low-poly character model
- Medium-length blonde hair
- Casual outfit (can change to lawyer outfit in law room)

### Room Designs
1. **Main Room**: Cozy living space with sofa, bookshelf, plants, locked chest
2. **Law Room**: Mini courtroom with judge's bench, witness stand, law books
3. **Knitting Room**: Craft room with yarn baskets, knitting needles, patterns

## Getting Started

1. Install dependencies:
   ```bash
   npm install
   ```

2. Run the development server:
   ```bash
   npm run dev
   ```

3. Open browser to http://localhost:3000

## Controls
- **Movement**: WASD keys (directional - forward/back/strafe)
- **Look Around**: Mouse movement (automatic pointer lock)
- **Interact**: E key
- **Quit Quiz**: ESC key

## Current Status
🎉 **THE GAME IS NOW COMPLETE!** 🎂

The game now has:
- ✅ Professional video game controls with mouse look
- ✅ Physical collision detection for all rooms
- ✅ Detailed Sims-inspired character with long blonde hair
- ✅ Minigame quit functionality (ESC key)
- ✅ Enhanced Law Quiz with personalized feedback (9 UK law questions)
- ✅ Complete Knitting Minigame (3 items: sweater, beanie, gloves)
- ✅ Three fully designed rooms with collision boundaries
- ✅ Interactive object highlighting and on-screen hints
- ✅ Professional UI with updated controls HUD
- ✅ Chest unlock mechanism (unlocks after completing both challenges)
- ✅ Birthday video player system with celebration screen
- ✅ Full game completion flow

**Recent Updates (Session 5):**
- ✅ Added clever room signs with arrows pointing to doors
- ✅ Fixed door flickering bug with stability timer system
- ✅ Implemented Roma the cocker spaniel as a loyal companion
- ✅ Added Roma following behavior and realistic animations
- ✅ Created Roma interaction system with speech bubbles
- ✅ Enhanced user experience with proper UI positioning

**Game Flow:**
1. Start in main room with locked chest (Roma by your side!)
2. See clever signs pointing to each room with puns
3. Enter Law Room → Complete 9 UK law questions (Roma follows)
4. Enter Knitting Room → Create 3 pieces of knitwear (Roma watches)
5. Return to main room → Chest automatically unlocks
6. Open chest → Birthday celebration with video player
7. Pet Roma anytime by pressing E → "Roma: Woof Woof!"
8. Game complete! 🎉

## Testing the Game

To test the current build:

1. Install dependencies:
   ```bash
   npm install
   ```

2. Run development server:
   ```bash
   npm run dev
   ```

3. Game should open at http://localhost:3000

### What to Test:
- **Professional Controls**: Click to enter mouse look mode, move mouse to turn Pea
- **WASD Movement**: Forward/back/strafe relative to Pea's facing direction
- **Collision Detection**: Try walking into walls and furniture - no phasing through!
- **Character Animation**: Watch detailed walking with leg bending and long hair
- **Roma the Dog**: Watch Roma follow you around with realistic animations
  - Roma's tail wagging and leg movement
  - Roma repositioning when you change rooms
  - Press E near Roma to see "Roma: Woof Woof!" speech bubble
- **Room Signs**: Look for arrow signs next to doors with clever puns
  - "Don't rest on your LAWrels" (Law Room - left)
  - "All we knit is love" (Knitting Room - right)
- **Room Navigation**: Walk to doors, press E to open and enter
- **Law Quiz**: Go to Law Room, approach golden podium, press E
  - Click on answer choices
  - Try ESC key to quit quiz mid-way
  - Test different scores for personalized messages
  - Movement should be locked during quiz
- **Knitting Challenge**: Go to Knitting Room, approach table, press E
  - Select materials and methods for 3 items
  - Watch knitting progress bars
  - Try ESC key to quit mid-game
- **Final Victory**: Complete both challenges, return to main room
  - Chest should unlock automatically
  - Press E on chest to see birthday celebration
- **UI Elements**: Updated controls HUD shows new key bindings
- **Interactions**: Objects glow yellow when near, hints appear at bottom

---

## Documentation Structure Options

**Current**: Single streamlined CLAUDE.md file (recommended for production-ready project)

**Future Option**: If project expands significantly, consider modular structure:
- `CLAUDE.md` (overview, setup, current status)
- `ARCHITECTURE.md` (technical details, code structure)
- `FEATURES.md` (complete feature documentation)
- `DEVELOPMENT_HISTORY.md` (session logs and evolution)

This modular approach would be beneficial if the project grows beyond the current complete state or requires extensive documentation for multiple developers.

---
> Source: [DSYJ94/Birthday-Video-Game-Gift-2025-](https://github.com/DSYJ94/Birthday-Video-Game-Gift-2025-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-19 -->
