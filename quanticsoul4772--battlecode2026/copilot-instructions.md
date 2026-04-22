## battlecode2026

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Context

**Battlecode 2026** - MIT competitive programming competition where AI bots control rat armies to gather cheese, damage cats, and eliminate enemy kings. Games run under strict bytecode constraints (17,500/turn for baby rats, 20,000/turn for kings).

**Current Status**: Active development, Tournament runs Jan 12 - Jan 31, 2026. Multiple bot iterations exist (ratbot through ratbot6), with ratbot5 being the current champion (~3,500 lines, sophisticated kiting and combat).

## Build Commands

All commands run from `scaffold/` directory:

```bash
cd scaffold

# Development
./gradlew build                    # Compile all bots
./gradlew test                     # Run JUnit tests
./gradlew run                      # Run match (uses gradle.properties settings)

# Custom matches
./gradlew run -PteamA=ratbot5 -PteamB=examplefuncsplayer -Pmaps=DefaultMedium

# Code quality
./gradlew spotlessApply            # Auto-format with Google Java Format
./gradlew checkstyleMain           # Lint main code (currently disabled in build.gradle)

# Submission
./gradlew zipForSubmit             # Create submission.zip for tournament upload

# Utilities
./gradlew listPlayers              # Show all bot implementations
./gradlew listMaps                 # Show available maps
./gradlew update                   # Update to latest Battlecode engine
```

**Match Configuration**: Edit `scaffold/gradle.properties` to set teamA, teamB, maps, and debug options.

**Viewing Replays**: Run client from `scaffold/client/Battlecode Client.exe`, load `.bc26` files from `scaffold/matches/`.

## Critical Architecture Patterns

### Bot Iterations & Evolution

**Bot progression** (oldest to newest):
- `ratbot` - Original modular architecture with separated algorithms/ and logging/ packages
- `ratbot2` - Combat/Economy role separation experiment
- `ratbot3` - Simplified single-file approach
- `ratbot4` - Population-based strategies (12 rats, 6 attackers / 6 collectors)
- **`ratbot5`** - Current champion: sophisticated kiting state machine, enemy tracking, overkill prevention, adaptive strategies (~3,500 lines)
- `ratbot6` - In development: Counter-strategy to beat ratbot5 using value functions and mobile king

**Current active bot**: `ratbot5` (configured in gradle.properties)

### Code Organization Philosophy

**Two approaches exist in this codebase:**

1. **Modular (ratbot)**: Separated packages for algorithms, logging, with RobotPlayer delegating to BabyRat/RatKing classes
   - Easier to test and reuse
   - More files to navigate

2. **Monolithic (ratbot3/4/5)**: Single RobotPlayer.java with all logic
   - Faster bytecode (no method overhead)
   - Easier to optimize holistically
   - Current competitive approach

**Use monolithic style for new competitive bots** - bytecode optimization is critical in competition.

### Bytecode Optimization is Critical

**Battlecode is bytecode-limited**, not time-limited. Every operation costs bytecode:
- Sense operations: ~100-500 bytecode
- Movement/actions: ~200-400 bytecode
- Math.abs(), Math.sqrt(): ~50 bytecode each
- Division: ~30 bytecode
- Object allocation: EXPENSIVE (avoid in main loop)

**Optimization patterns used in ratbot5:**
```java
// GOOD: Inline abs instead of Math.abs()
int dx = rx - currentLoc.x;
if (dx < 0) dx = -dx;

// GOOD: Bitshift instead of division
int rallyX = (me.x + cachedMapCenter.x) >> 1;  // Not: / 2

// GOOD: Pre-allocated arrays (avoid allocation in hot path)
private static final int[] KING_TILE_DX = {-1, 0, 1, -1, 1, -1, 0, 1};

// GOOD: Cache computed values across turn
private static boolean cachedIsAssassin = false;
private static int lastIsAssassinID = -1;

// GOOD: Deferred sensing (only sense when needed)
RobotInfo[] allies = null;  // Sense later only if overkill check needed

// BAD: Repeated sensing
for (int i = 0; i < enemies.length; i++) {
    RobotInfo[] nearby = rc.senseNearbyRobots(10, team);  // WASTEFUL
}
```

**Profile regularly**: Use `Clock.getBytecodeNum()` before/after expensive operations to track usage.

### State Machine Combat Pattern (ratbot5)

ratbot5 uses **kiting state machine** for combat:
```
APPROACH → ATTACK → RETREAT → (repeat)
```

**Key states**:
- `KITE_STATE_APPROACH (0)` - Move toward enemy until in attack range
- `KITE_STATE_ATTACK (1)` - Execute attack if action ready
- `KITE_STATE_RETREAT (2)` - Move away 1-3 tiles based on HP (wounded rats retreat farther)

**Critical features**:
- **Overkill prevention**: Don't attack enemies <20 HP if allies nearby (avoid wasted damage)
- **Focus fire**: King broadcasts priority target via shared array
- **Retreat coordination**: Broadcast when army HP ratio <40% of enemy
- **Dynamic kite distance**: Healthy (1 tile) → Wounded (2 tiles) → Critical (3 tiles)

### Shared Array Communication

**64 slots × 10 bits** (values 0-1023) for team coordination. DO NOT use squeaking (attracts cats).

**Common slot layout** (example from ratbot5):
```java
// Slot 0-1: Our king position (x, y)
// Slot 2-3: Enemy king position (x, y)
// Slot 4: Enemy king HP
// Slot 5: Economy mode (0=normal, 1=low, 2=critical)
// Slot 6: Focus fire target ID (robot ID % 1024)
// Slot 7: Retreat signal (1=retreat, 0=attack)
// Slot 8-11: Enemy ring buffer (last 4 enemy positions)
```

**Always validate array reads**: Slots can be stale, check round timestamps.

### Vision Cone Management

**Baby rats have 90° vision cone** - they can only see in the direction they're facing!

**Common pitfall**: Sensing enemies that exist but aren't visible
```java
// WRONG: Will miss enemies outside vision cone
RobotInfo[] enemies = rc.senseNearbyRobots(20, opponent);

// RIGHT: Check if enemy is actually visible
MapLocation enemyLoc = /* enemy location */;
Direction facing = rc.getDirection();
// Must verify enemy is in 90° cone from facing direction
```

**King has 360° vision** (3x3 size sees all directions).

### Movement Costs & Optimization

**Critical for pathfinding**:
- Forward movement: 10 cooldown
- Strafe (sideways): 18 cooldown (AVOID - 80% slower)
- Turn: 10 cooldown

**Best practice**: Always face target, then move forward (20 total) rather than strafe (18).

**Turn optimization**:
```java
// GOOD: Turn to face, then move forward next turn
if (!isFacing(target)) {
    rc.turn(directionTo(target));
} else if (rc.canMove(rc.getDirection())) {
    rc.move(rc.getDirection());
}

// BAD: Strafe sideways (costs 18 vs 10)
rc.move(Direction.EAST);  // If facing NORTH
```

## Success Criteria (CRITICAL)

**This is a month-long competition** - never say "final submission" or "final test". Always "current", "latest".

**Success = King survives via cheese economy**, NOT round count or "beating opponent"

### What Matters
✅ King receives regular cheese deliveries (look for TRANSFER messages in logs)
✅ Positive cheese income (deliveries > 3 cheese/round consumption)
✅ King cheese level stable or increasing
✅ Death from combat, NOT starvation

### What Doesn't Matter
❌ Total rounds survived
❌ Whether we "won" the match
❌ Beating examplefuncsplayer

### How to Diagnose Failure
1. Run match and check logs
2. Look at FINAL 50 rounds before king death
3. Count TRANSFER messages (cheese deliveries)
4. Check king cheese trend (increasing or decreasing?)
5. Determine cause of death: starvation vs combat

**If king starves = FAILURE** even if we survive 1000 rounds.

## Map-Specific Strategy Tuning

**ratbot5 uses map size detection** for adaptive strategies:

```java
int area = rc.getMapWidth() * rc.getMapHeight();
if (area < SMALL_MAP_AREA) {
    // Small maps (40×40): Aggressive all-attack early game
    // Higher spawn rate, lower cheese reserve, 80% attackers
} else if (area < MEDIUM_MAP_AREA) {
    // Medium maps (50×50): Balanced aggression
    // Moderate spawn rate, extended all-attack phase
} else {
    // Large maps (60×60+): Economic focus
    // Conservative spawning, 50% collectors
}
```

**Key insight**: Medium maps should be AGGRESSIVE (like small), not balanced - "balanced" creates worst-of-both-worlds.

## Role Assignment Patterns

**Population-based** (ratbot4 approach):
```java
int role = rc.getID() % 2;
// role 0 = Attacker (50%)
// role 1 = Collector (50%)
```

**Percentage-based with dynamic ratios** (ratbot5 approach):
```java
int attackerRatio = getAttackerRatio();  // Returns 4-8 based on phase/economy
boolean isAttacker = (rc.getID() % 10) < attackerRatio;
// Allows 40%-80% attacker ratios dynamically
```

**Specialized roles** (ratbot5):
```java
// Assassins: Rush enemy king directly (5-10% of population)
boolean isAssassin = (rc.getID() % ASSASSIN_DIVISOR) == 0;

// Interceptors: Patrol midfield to slow enemy rushes (20%)
boolean isInterceptor = (rc.getID() % 5) == 0;
```

## Common Pitfalls & Solutions

### King Starvation
**Problem**: King consumes 3 cheese/round, dies if cheese reaches 0
**Solution**: Track `rc.getTeamCheese()` every turn, trigger emergency collection mode at <200 cheese
**Formula**: `roundsUntilStarvation = teamCheese / (kingCount * 3)`

### Bytecode Limit Exceeded
**Problem**: Bot skips turns when exceeds 17,500 bytecode
**Solution**: Profile with `Clock.getBytecodeNum()`, cache sensing results, avoid nested loops
**Check**: `rc.senseNearbyRobots()` is expensive (~500 bytecode), sense once per turn

### Traffic Jams (Collectors Can't Deliver)
**Problem**: Rats carrying cheese can't reach king (blocked by allies)
**Solution**: Anti-clumping logic, spread when >3 allies within 3 tiles, prefer movement away from allies
**Diagnosis**: High cheese but no TRANSFER messages = traffic jam = starvation

### Vision Cone Blindness
**Problem**: Enemies exist but aren't attacked (outside 90° cone)
**Solution**: Turn toward targets before attacking, scan by rotating periodically
**Remember**: Baby rats must face target to see/attack it

### Premature All-In Attacks
**Problem**: Rush enemy king too early, lose army, can't recover
**Solution**: Wait until enemy king <150 HP OR round >80 AND have >5 attackers
**Pattern**: Coordinate via shared array all-in signal with duration

## Testing Strategy

### Quick Validation
```bash
cd scaffold
./gradlew build && ./gradlew run -PoutputVerbose=true | grep "TRANSFER"
```
Look for regular TRANSFER messages indicating successful cheese deliveries.

### Integration Testing
```bash
# Test vs reference bot
./gradlew run -PteamA=ratbot5 -PteamB=examplefuncsplayer -Pmaps=DefaultSmall

# Test map-specific tuning
./gradlew run -PteamA=ratbot5 -PteamB=ratbot5 -Pmaps=DefaultMedium

# Test as both Team A and Team B (spawn positions differ)
./gradlew run -PteamA=ratbot5 -PteamB=ratbot4 -Pmaps=DefaultLarge
```

### Metrics to Track
- King survival time (less important than HOW they died)
- Cheese deliveries per round (should exceed 3)
- Bytecode usage (stay below 15,000 avg for baby rats)
- Combat win rate (do we kill more than we lose?)
- Economy stability (cheese never drops to emergency levels)

## Key Game Mechanics Reference

**Spawn cost formula**: `10 + 10 × floor(baby_rats / 4)` cheese
- 0-3 rats: 10 cheese each
- 4-7 rats: 20 cheese each
- 8-11 rats: 30 cheese each

**Damage formula**:
- Base bite: 10 damage
- Enhanced bite: 10 + ceil(log2(cheeseSpent)) damage
- Cost-effective: Spend 16 cheese for +4 damage when enemy >50 HP

**Vision ranges** (radius²):
- Baby rat: 20 (√20 ≈ 4.47 tiles), 90° cone
- Rat king: 25 (√25 = 5 tiles), 360° view
- Cat: 30 (√30 ≈ 5.48 tiles), 180° cone

**Movement**:
- Adjacent = distanceSquared ≤ 2 (includes diagonals)
- Delivery range = distanceSquared ≤ 9 (3 tiles to king center)

## Documentation Resources

**Essential files**:
- `README.md` - Project overview and quick start
- `AGENTS.md` - Comprehensive developer guide (start here)
- `SUCCESS_CRITERIA.md` - How to measure bot performance (READ THIS)
- `claudedocs/complete_spec.md` - Full game rules (17KB)
- `claudedocs/quick_reference.md` - Stats and costs lookup
- `claudedocs/INDEX.md` - Documentation navigation

**Planning docs** (in claudedocs/):
- `RATBOT4_PLAN.md` - Population-based strategy approach
- `RATBOT5_FROM_SCRATCH_PLAN.md` - Kiting state machine design
- `RATBOT6_DESIGN.md` - Value function architecture (in development)

**API Reference**: https://releases.battlecode.org/javadoc/battlecode26/1.0.1/

## Development Workflow

### Adding New Features to Existing Bot
1. Read current bot code to understand architecture (ratbot5 is monolithic)
2. Add constants at top of RobotPlayer.java (keep tunable params grouped)
3. Implement feature in appropriate section (king behavior, baby rat behavior, etc.)
4. Add bytecode profiling around expensive operations
5. Test via `./gradlew run` and check logs
6. Profile bytecode usage (should stay <15K avg)
7. Iterate based on match results (watch replays in client)

### Creating New Bot Iteration
1. Copy previous bot: `cp -r scaffold/src/ratbot5 scaffold/src/ratbot7`
2. Update package name in all .java files
3. Configure in gradle.properties: `teamA=ratbot7`
4. Implement changes
5. Test head-to-head against previous version
6. Document strategy in claudedocs/RATBOT7_PLAN.md

### Before Committing
- Run `./gradlew build` to ensure compilation
- Check `./gradlew test` passes (if tests exist for modified code)
- Test at least one match vs previous bot version
- Verify king doesn't starve (check for TRANSFER messages)
- Document significant strategy changes in planning docs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/quanticsoul4772) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
