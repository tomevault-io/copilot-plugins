## trade-wars

> Trade Wars is a **learning/educational blockchain game** built on Sui, demonstrating full-stack dApp development with Move smart contracts and a React frontend. This is a **work in progress** project by a **solo developer** intended to be **open sourced**.

# CLAUDE.md - Trade Wars Project Guide

## Project Overview

Trade Wars is a **learning/educational blockchain game** built on Sui, demonstrating full-stack dApp development with Move smart contracts and a React frontend. This is a **work in progress** project by a **solo developer** intended to be **open sourced**.

### Game Mechanics
- **Resource Mining**: Three elements (Erbium, Lanthanum, Thorium) are mined from planets over time
- **Mine Upgrades**: Players spend resources to upgrade mines for faster production
- **Universe Expansion**: Players create an Overseer, join universes, and control multiple planets
- **Future Features**: Combat and trading mechanics are planned but not yet implemented

## Project Structure

```
trade-wars/
├── move/                    # Sui Move smart contracts
│   └── sources/
│       ├── overseer.move    # Player character management
│       ├── universe.move    # Game world management
│       ├── planet.move      # Planet and mining logic
│       ├── element_mine.move # Mine production calculations
│       └── *.move           # Token types and utilities
│
├── web/                     # React frontend
│   └── src/
│       ├── components/      # UI components (Tailwind)
│       │   ├── ui/          # Primitives (Card, Button, Badge, Spinner)
│       │   ├── layout/      # Header, Footer, PageContainer
│       │   ├── overseer/    # OverseerList, OverseerEmpire
│       │   └── planet/      # PlanetGrid, PlanetDetailsView
│       ├── contexts/        # React Context (NavigationContext)
│       ├── hooks/           # Custom hooks (useOverseer, usePlanet, etc.)
│       ├── services/sui/    # Transaction builders
│       ├── utils/           # Parsing, formatting, validation
│       ├── types/           # TypeScript type definitions
│       ├── constants/       # Environment, colors, blockchain constants
│       └── pages/           # Page components
│
├── cli/                     # CLI tools and deployment scripts
│   └── .env.testnet         # Testnet deployment IDs
│
├── CLAUDE.md                # This file - project guide for Claude
└── DEPLOYMENTS.md           # Deployment history and object IDs
```

## Development Commands

### Frontend (web/)
```bash
npm run dev      # Start development server (http://localhost:5173)
npm run build    # Build for production (runs tsc + vite build)
npm run lint     # Run ESLint
npm run preview  # Preview production build
```

### Smart Contracts (move/)
```bash
sui move build   # Compile Move contracts
sui move test    # Run Move unit tests
sui client publish --gas-budget 100000000  # Deploy to network
```

## Code Style & Conventions

### TypeScript/React
- **Functional components only** - No class components
- **Strict TypeScript** - Avoid `any` types, use proper typing
- **Tailwind only** - No inline styles or CSS files, only Tailwind classes
- **Minimal comments** - Code should be self-documenting

### File Naming
- Components: PascalCase (`PlanetGrid.tsx`)
- Hooks: camelCase with `use` prefix (`usePlanet.ts`)
- Utils/Services: camelCase (`data-access.ts`)
- Types: camelCase (`game.ts`)

### Import Order
1. React/external libraries
2. Internal components
3. Hooks
4. Utils/services
5. Types/constants

## Sui SDK Best Practices

### Data Access Strategy
| Data Type | Method | Reason |
|-----------|--------|--------|
| Time-dependent (reserves) | `devInspect` + `bcs.U64.parse()` | Computes production based on time |
| Static fields (mine levels) | `getObject` → access fields | No computation needed |
| Vector returns (planet IDs) | `devInspect` + `bcs.vector(bcs.Address).parse()` | Returns computed list |

### Correct SDK Usage
```typescript
// ✅ Use BCS module for parsing
import { bcs } from '@mysten/sui/bcs';
const value = bcs.U64.parse(new Uint8Array(returnValue));
const ids = bcs.vector(bcs.Address).parse(new Uint8Array(returnValue));

// ❌ Don't manually parse bytes
// for (let i = 0; i < 8; i++) { result += bytes[i] * Math.pow(256, i); }
```

### Transaction Building
- Use transaction builders in `services/sui/transactions/`
- Batch multiple calls in one PTB when possible
- Always consider gas costs

## Smart Contract Entry Functions

### Overseer Module
- `vest_overseer()` - Create new player (Overseer)
- `join_universe(overseer, universe, sources, random, clock)` - Join a universe
- `upgrade_erbium_planet_mine(...)` - Upgrade erbium mine
- `upgrade_lanthanum_planet_mine(...)` - Upgrade lanthanum mine
- `upgrade_thorium_planet_mine(...)` - Upgrade thorium mine
- `get_universe_planets(overseer, universe_id)` - Get player's planets in universe

### Planet Module (View Functions)
- `get_*_reserves(planet, clock)` - Get current reserves (time-dependent)
- `get_*_mine_level(planet)` - Get mine level (static)
- `get_*_mine_*_upgrade_cost(planet)` - Get upgrade costs (static)

## Environment Variables

Located in `web/.env`:
```
VITE_TRADE_WARS_PKG_DEV=<package_id>      # Deployed package ID
VITE_TRADE_WARS_INFO_DEV=<info_object_id> # TradeWarsInfo shared object
```

See `DEPLOYMENTS.md` for current testnet IDs.

**IMPORTANT**: Never commit actual values to version control.

## Testing Requirements

**Full E2E testing required** before committing changes:
1. `npm run build` - Must pass without errors
2. `npm run dev` - Start dev server
3. Test all user flows:
   - Wallet connection/disconnection
   - Overseer creation
   - Universe joining
   - Planet viewing
   - Mine upgrades (all 3 types)
   - Navigation (forward/back)

## Working with Claude

### Approach
- **Ask before suggesting** - Identify potential improvements but ask before implementing
- **Clean code first** - Prioritize maintainability and code quality over speed
- **Full stack capable** - Can modify both frontend and smart contracts

### Sensitive Areas (Extra Caution)
- **Smart contract calls** - Verify transaction building is correct
- **Environment variables** - Never expose or commit sensitive values
- **Wallet interactions** - Be careful with user wallet code
- **Gas optimization** - Consider transaction costs when building PTBs

### Documentation
Good documentation is important for this educational project. When making significant changes:
- Update this CLAUDE.md if architecture changes
- Update DEPLOYMENTS.md after new deployments
- Add JSDoc comments for complex functions

## Current Network

**Testnet** - Smart contracts are deployed to Sui testnet.
- Explorer: https://testnet.suivision.xyz/
- RPC: https://fullnode.testnet.sui.io
- See `DEPLOYMENTS.md` for all object IDs and explorer links

## Upcoming Work: Smart Contract Development

When working on Move smart contracts:

### Move Development Workflow
1. **Read existing contracts first** - Understand the current architecture before changes
2. **Run `sui move build`** - Verify compilation after changes
3. **Run `sui move test`** - All tests must pass
4. **Consider upgrade implications** - Sui packages are immutable once published; plan for upgradability patterns if needed
5. **Update frontend** - After contract changes, update corresponding:
   - Transaction builders in `services/sui/transactions/`
   - Types in `types/game.ts` or `types/sui.ts`
   - Hooks if query patterns change
6. **Update DEPLOYMENTS.md** - Record new package ID and object IDs

### Move Best Practices for This Project
- Use capability pattern for access control (already implemented)
- Shared objects for game state (Universe, Planet)
- Owned objects for player assets (Overseer, PlanetCap)
- Events for important state changes (useful for indexing later)
- Consider gas costs for complex operations

### Shared Object Considerations
After Phase 1, the `ElementSource<T>` objects will be **highly contended shared objects** accessed by all players across all universes. Keep in mind:
- Sui handles shared object contention via consensus ordering
- High contention = potential throughput bottleneck (this is intentional as a stress-test)
- Consider batching operations where possible
- Monitor transaction latency in testing
- This architecture choice prioritizes simplicity over maximum throughput

### Adding New Entry Functions
When adding new Move entry functions:
1. Implement in Move with proper access control
2. Add transaction builder in `web/src/services/sui/transactions/`
3. Create/update hook in `web/src/hooks/`
4. Update UI components to use the new functionality
5. Test full flow on testnet

## Development Roadmap

### Phase 1: Element Source Simplification (Architecture Change)
**Goal**: Remove `universe_element_source.move` and consolidate to a single `element_source.move`

**Current State**:
- Each universe has its own `UniverseElementSource<T>` objects (one per element type)
- Mines reference universe-specific sources

**Target State**:
- Single shared `ElementSource<T>` objects for all universes
- All mines across all planets/universes use the same source objects
- Simplifies code and serves as a **stress-test for shared object access patterns**

**Impact**:
- Modify `planet.move` - update mine creation and operations
- Modify `universe.move` - remove universe-specific source creation
- Modify `overseer.move` - update entry functions that pass sources
- Delete `universe_element_source.move`
- Update frontend transaction builders
- **New deployment required** - update DEPLOYMENTS.md

### Phase 2: Smart Planet Allocation
**Goal**: Improve planet assignment when overseers join a universe for the first time

**Current State**: Fully random planet allocation

**Target State**: Partially random allocation with clustering logic:
- New players assigned to random planet → random system → random galaxy
- BUT players are clustered close together for future interaction features
- Systems are NOT fully filled - leave room for colonization feature

**Design Considerations** (to be refined during implementation):
- Define threshold: how many players per system before moving to next?
- Define threshold: how many systems per galaxy before moving to next?
- Balance between clustering (for interaction) and spacing (for colonization)
- Consider "frontier" concept - active allocation zones vs reserved zones

### Phase 3: Trading System (PoC)
**Goal**: Enable resource trading to reach Proof of Concept state

**Planned Features**:
- Player-to-player trading
- Player-to-computer trading (NPC market / exchange)

**Specifications**: To be defined at implementation time

### Future Features (Post-PoC)
- Planet colonization (claim additional planets in a universe)
- Fleet/ship system for combat
- Alliance/guild system
- Leaderboards

## Known Limitations / TODOs

- Combat/trading features not yet implemented
- No unit tests for frontend (E2E manual testing only)
- Bundle size could be optimized with code-splitting
- Some placeholder UI elements
- Package upgrade strategy not yet defined

## Quick Reference

### Key Files
- `web/src/App.tsx` - Main app with routing
- `web/src/contexts/NavigationContext.tsx` - State-based routing
- `web/src/hooks/usePlanet.ts` - Planet data queries
- `web/src/services/sui/transactions/` - All transaction builders
- `move/sources/overseer.move` - Main entry points
- `cli/.env.testnet` - Deployment IDs

### Color Palette
- Gold (primary): `#d4af37`
- Erbium: `#ff6347`
- Lanthanum: `#00ff7f`
- Thorium: `#8a2be2`
- Success: `#4baf4b`
- Warning: `#ffa500`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alilloig) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
