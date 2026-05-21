## game

> https://pokeapi.co/docs/v2#info

# API

https://pokeapi.co/docs/v2#info

# Pokémon Tamagotchi Game

## Game Concept
- Players start with one Pokémon from the first generation (1-151)
- Players must take care of their Pokémon like a virtual pet (Tamagotchi style)
- Experience is gained through care activities
- Pokémon can evolve when reaching level 10

## Core Features

### Starting the Game
- New players select or are randomly assigned a starter Pokémon from Gen 1
- Each Pokémon starts at level 1 with basic stats
- Player creates a trainer profile

### Pokémon Care Mechanics
- **Hunger**: Feed your Pokémon regularly to maintain health
- **Happiness**: Play with your Pokémon to increase happiness
- **Cleanliness**: Groom your Pokémon to keep it clean
- **Energy**: Let your Pokémon rest to restore energy
- **Health**: Monitor overall health status
- All stats decline over time if neglected

### Experience & Leveling
- Gain experience points (XP) through various care activities
- Daily bonuses for consecutive logins
- Special events provide bonus XP
- Level progression: 1-10 for first evolution stage
- XP requirements increase with each level

### Evolution
- Pokémon can evolve when reaching level 10
- Evolution requires a special evolution item or completing a challenge
- Evolved Pokémon have better stats and new abilities
- Some Pokémon have multiple evolution paths

### Mini-games
- **Training**: Mini-games to boost stats
- **Battles**: Simple battle system against AI opponents
- **Exploration**: Find items in the environment

## Technical Implementation

### Pokémon Data
- Use PokeAPI to fetch Gen 1 Pokémon data
- Store the following data for each Pokémon:
  - Basic info (name, ID, types, sprites)
  - Stats (HP, attack, defense, etc.)
  - Evolution chain
  - Moves and abilities

```typescript
interface PokemonBase {
  id: number;
  name: string;
  types: string[];
  sprite: string;
  evolvesAt: number;
  evolvesTo: number | null;
}

interface PlayerPokemon extends PokemonBase {
  level: number;
  experience: number;
  hunger: number; // 0-100
  happiness: number; // 0-100
  cleanliness: number; // 0-100
  energy: number; // 0-100
  health: number; // 0-100
  lastCaredFor: Date;
}
```

### Game State Management
- Use local storage for game progress
- Implement auto-save functionality
- Track time even when the app is closed
- Create state hooks for managing game data

```typescript
// Example state management with hooks
const usePokemonState = () => {
  const [playerPokemon, setPlayerPokemon] = useState<PlayerPokemon | null>(null);
  
  // Load saved Pokémon from storage
  useEffect(() => {
    const savedPokemon = localStorage.getItem('playerPokemon');
    if (savedPokemon) {
      setPlayerPokemon(JSON.parse(savedPokemon));
    }
  }, []);
  
  // Save Pokémon state to storage
  useEffect(() => {
    if (playerPokemon) {
      localStorage.setItem('playerPokemon', JSON.stringify(playerPokemon));
    }
  }, [playerPokemon]);
  
  return { playerPokemon, setPlayerPokemon };
};
```

### Time Management
- Implement real-time stat decay
- Schedule notifications for feeding and care
- Day/night cycle affecting certain activities

### UI/UX Guidelines
- Main screen shows Pokémon with status indicators
- Interactive elements for feeding, playing, etc.
- Animations for status changes and level ups
- Celebratory animations for evolution

## Screens and Components

### Main Screens
1. **Welcome/Login**: Player profile creation/login
2. **Starter Selection**: Choose starter Pokémon
3. **Main Care Screen**: Primary interaction with Pokémon
4. **Stats Screen**: Detailed Pokémon stats
5. **Activities Screen**: Mini-games and training
6. **Evolution Screen**: Evolution process and celebration
7. **Settings Screen**: Game settings and options

### Key Components
- **PokemonDisplay**: Shows Pokémon sprite with animations
- **StatBar**: Visual indicator for stats (hunger, happiness, etc.)
- **ActionButton**: Buttons for care actions
- **EvolutionModal**: Handles evolution process
- **MiniGameContainer**: Wrapper for mini-games
- **NotificationSystem**: Alerts for Pokémon needs

## Future Features
- Multiple Pokémon collection
- Pokémon breeding
- Trading with other players
- Seasonal events
- Special location-based activities


## Setup Constants File
- File used to setup experience needed for evolving;
- HP for each evolution
- Basic stats
- Time for things decreasing

---
> Source: [gabrielumatch/pokemon](https://github.com/gabrielumatch/pokemon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
