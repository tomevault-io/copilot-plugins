## legends-of-lore

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 🎮 Project: Legends of Lore - Gothic Retro Side-Scrolling Fantasy Game

A 2D side-scrolling action-adventure game inspired by Castlevania and Altered Beast, featuring retro pixel art graphics with a dark gothic aesthetic. The game follows Gabriel Thorne, a monster hunter with beast transformation abilities, on his quest to defeat Count Vladok.

## 🚨 IMPORTANT: Use CoralCollective Agents for Development

This project uses the CoralCollective framework for AI-driven development. ALWAYS use the specialized agents through the Task tool for different aspects of game development.

### Required Development Workflow:
1. **Start with Project Architect**: Design game architecture and systems
2. **Technical Writer Phase 1**: Document game requirements and specifications  
3. **Specialized Development**: Use appropriate agents for each component
4. **QA & Testing**: Comprehensive testing of game mechanics
5. **Technical Writer Phase 2**: Final documentation and guides

## Common Commands

### Game Development Setup
```bash
# Create virtual environment (ALWAYS DO THIS FIRST)
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies (when package.json/requirements.txt exist)
pip install -r requirements.txt  # Python dependencies
npm install                       # Node dependencies for web build

# Game engine setup (to be determined - Phaser.js recommended for web/mobile)
npm install phaser                # If using Phaser.js
```

### Running the Game
```bash
# Development server (once implemented)
npm run dev                       # Start development server
npm run build                     # Build for production
npm run test                      # Run tests

# Mobile builds (future)
npm run build:ios                 # Build for iOS
npm run build:android             # Build for Android
```

### Using CoralCollective Agents
```bash
# Access agent system
python .coral/agent_runner.py list              # List available agents
python .coral/agent_runner.py run               # Run specific agent
python .coral/agent_runner.py workflow           # Run workflow

# Use subagent invocation in Claude
@backend_developer "Create game server API"
@frontend_developer "Build game UI"
@ai_ml_specialist "Implement enemy AI"
```

## Architecture

### Game Architecture (To Be Implemented)

#### Core Systems
1. **Game Engine**: Phaser.js (recommended) or PixiJS for web/mobile compatibility
2. **Backend**: Node.js/Express API for:
   - User authentication & profiles
   - Save game management
   - Leaderboards
   - In-app purchases
   - Ad serving integration
3. **Database**: PostgreSQL or MongoDB for user data
4. **State Management**: Redux or Zustand for game state
5. **Mobile**: Capacitor or React Native for mobile builds

#### Game Components Structure
```
src/
├── game/                   # Core game code
│   ├── scenes/            # Game scenes/levels
│   │   ├── MainMenu.ts
│   │   ├── Level1_HauntedVillage.ts
│   │   ├── Level2_ForbiddenForest.ts
│   │   ├── Level3_Catacombs.ts
│   │   ├── Level4_ClockTower.ts
│   │   └── Level5_CastleKeep.ts
│   ├── entities/          # Game entities
│   │   ├── player/        # Player character
│   │   │   ├── Gabriel.ts
│   │   │   └── transformations/
│   │   ├── enemies/       # Enemy types
│   │   ├── bosses/        # Boss enemies
│   │   └── npcs/          # Non-player characters
│   ├── systems/           # Game systems
│   │   ├── combat/        # Combat mechanics
│   │   ├── transformation/# Beast transformation system
│   │   ├── progression/   # RPG elements
│   │   └── inventory/     # Items and weapons
│   ├── ui/                # Game UI components
│   │   ├── hud/          # Heads-up display
│   │   ├── menus/        # Game menus
│   │   └── dialogs/      # Dialog system
│   └── assets/            # Game assets
│       ├── sprites/       # Pixel art sprites
│       ├── audio/         # Music and SFX
│       └── levels/        # Level data
├── server/                 # Backend API
│   ├── routes/           # API endpoints
│   ├── models/           # Data models
│   └── services/         # Business logic
└── mobile/                 # Mobile-specific code
```

### Key Game Features to Implement

#### Phase 1: Core Gameplay
- Side-scrolling movement and combat
- Basic enemy AI
- Level progression system
- Save/load functionality

#### Phase 2: RPG Elements
- Experience and leveling system
- Skill tree implementation
- Weapon upgrades
- Item inventory

#### Phase 3: Transformation System
- Beast transformation mechanics
- Werewolf, dragon, specter forms
- Rage meter implementation
- Form-specific abilities

#### Phase 4: Monetization
- Free-to-play implementation
- $4.99/month subscription system
- Ad integration (AdMob/Unity Ads)
- In-app purchases
- Cosmetic skins and items

## Development Guidelines

### Game-Specific Patterns

#### Pixel Art Style
- Use 16-bit era inspired graphics
- Limited color palette for gothic aesthetic
- Consistent sprite sizes (32x32 or 64x64 base)
- Retro visual effects

#### Level Design Principles
- Gothic horror themes
- Platform challenges
- Hidden secrets and collectibles
- Boss encounters at level end
- Environmental storytelling

#### Combat System
- Responsive controls
- Attack patterns for enemies
- Combo system
- Sub-weapons (daggers, holy water)
- Transformation abilities

### Performance Optimization
- Sprite atlases for efficient rendering
- Object pooling for enemies/projectiles
- Efficient collision detection
- Mobile-optimized assets
- Progressive loading of levels

## CoralCollective Agent Usage for Game Development

### Recommended Agent Sequence for Game Features:

#### 1. Core Game Setup
```
@project_architect "Design architecture for 2D side-scrolling game with Phaser.js, supporting web and mobile platforms with free-to-play monetization"
@technical_writer_phase1 "Document game requirements from concept PDF including gameplay, levels, monetization, and technical specifications"
```

#### 2. Backend Development
```
@backend_developer "Create Node.js API for user authentication, save games, leaderboards, and subscription management"
@database_specialist "Design database schema for user profiles, game progress, purchases, and analytics"
```

#### 3. Game Development
```
@frontend_developer "Implement core game loop, player controls, and level structure using Phaser.js"
@ai_ml_specialist "Create enemy AI behaviors, pathfinding, and boss battle patterns"
```

#### 4. Game Systems
```
@full_stack_engineer "Implement RPG progression system with experience, skills, and inventory"
@backend_developer "Create transformation system with rage meter and beast forms"
```

#### 5. Monetization
```
@backend_developer "Implement subscription system, IAP handling, and ad serving integration"
@security_specialist "Secure payment processing and user data protection"
```

#### 6. Polish & Deploy
```
@performance_engineer "Optimize game performance for mobile devices"
@qa_testing "Test gameplay, progression, monetization flows"
@devops_deployment "Deploy to web, iOS App Store, and Google Play"
@technical_writer_phase2 "Create player guides and documentation"
```

### Agent-Specific Context

When invoking agents, provide this game context:
- **Game Type**: 2D side-scrolling action-adventure
- **Art Style**: Retro pixel art, gothic horror theme
- **Platforms**: Web (primary), iOS, Android
- **Monetization**: F2P with $4.99/month subscription + ads
- **Target Audience**: 13+ rating, nostalgia-driven 30-50 year olds
- **Core Inspiration**: Castlevania + Altered Beast mechanics

## Testing Requirements

### Game Testing Areas
```bash
# Unit Tests
- Game mechanics (combat, movement)
- Transformation system
- Progression calculations
- Inventory management

# Integration Tests
- Level transitions
- Save/load functionality
- API integrations
- Payment processing

# E2E Tests
- Complete gameplay flow
- Tutorial completion
- Boss battles
- Monetization flows

# Performance Tests
- Frame rate on target devices
- Load times
- Memory usage
- Battery consumption (mobile)
```

## Deployment Checklist

### Web Deployment
- [ ] Optimize assets for web
- [ ] Configure CDN for assets
- [ ] Set up SSL certificates
- [ ] Configure analytics

### Mobile Deployment
- [ ] App store assets (icons, screenshots)
- [ ] App store descriptions
- [ ] Age rating submissions
- [ ] IAP configuration
- [ ] Ad network integration

### Backend Deployment
- [ ] Database migrations
- [ ] API security hardening
- [ ] Payment gateway setup
- [ ] Monitoring and logging

## Important Files

### Game Configuration
- `game.config.js` - Phaser game configuration
- `levels.json` - Level definitions and progression
- `enemies.json` - Enemy types and behaviors
- `items.json` - Weapons and items database

### Monetization Config
- `subscription.config.js` - Subscription tiers
- `iap.config.js` - In-app purchase items
- `ads.config.js` - Ad placement configuration

## Development Tips

### Quick Start for New Features
1. Always start with the appropriate CoralCollective agent
2. Reference the concept PDF for game design decisions
3. Test on both web and mobile platforms
4. Ensure monetization doesn't affect core gameplay
5. Maintain the gothic aesthetic in all additions

### Performance Guidelines
- Target 60 FPS on modern mobile devices
- Keep sprite sheets under 2048x2048
- Minimize draw calls through batching
- Use object pooling for frequently spawned entities
- Compress audio files appropriately

### Monetization Best Practices
- Free players must be able to complete the game
- Subscription benefits should enhance, not gate content
- Ad placements should not interrupt gameplay
- Clear value proposition for subscription
- Transparent pricing and benefits

## Resources

### Game Development
- Phaser.js documentation: https://phaser.io/docs
- Pixel art tools: Aseprite, Piskel
- Audio tools: Audacity, sfxr

### Monetization
- AdMob integration guide
- Apple/Google IAP documentation
- Subscription management best practices

### CoralCollective
- Agent documentation: `.coral/agents/`
- Integration guide: `.coral/INTEGRATION.md`
- Agent workflow examples: `.coral/agent_runner.py`

---

Remember: This is a passion project bringing classic gaming nostalgia to modern platforms. Focus on fun gameplay, respectful monetization, and that gothic atmosphere that made games like Castlevania legendary.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/natesmalley) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
