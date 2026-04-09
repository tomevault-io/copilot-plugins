## game-4

> **IMPORTANT: When adding to this file, ALWAYS write in compact form - remove general knowledge, keep only MUST/SHOULD/ALWAYS/NEVER rules and specific constraints.**

# CLAUDE.md

**IMPORTANT: When adding to this file, ALWAYS write in compact form - remove general knowledge, keep only MUST/SHOULD/ALWAYS/NEVER rules and specific constraints.**

Circuit Quest: educational sandbox game - build computers from household items, 2D "inventor's sketchbook" aesthetic.

**Tech:** React + Vite + Vitest

## Development Rules

**ALWAYS use TDD when implementing features or fixing bugs:**
1. Write test first (RED) - test must fail initially
2. Implement minimal code to pass (GREEN) - don't over-engineer
3. Refactor while keeping tests green
4. Commit when all tests pass - include test + implementation together
5. ALWAYS commit after successful change - don't accumulate multiple unrelated changes

**MUST refactor when file exceeds line limits:**
- Implementation files: 500 lines
- Test files: 1500 lines
- Split into logical modules/classes
- Extract related functions into separate files
- Keep test files close to implementation
- Maintain single responsibility principle

**ALWAYS verify before removing files:**
- Run all tests to ensure split modules work correctly
- Verify no functionality lost in refactoring
- Check git status to confirm all new files added
- Only remove original file after verification passes

**Engine/Logic Layer MUST have tests:**
- Circuit simulation, component behavior, wire connectivity, voltage/current calculations, graph traversal, state management

**UI Layer tests optional but encouraged**

**Challenge Tests MUST include:**
- Positive test: Working solution that passes validation (in ChallengeSolutions.test.js)
- Negative tests: Verify challenge cannot be cheesed with easier solutions (in ChallengeNegative.test.js)
- ALWAYS create both when adding/modifying challenges

**Test Quality:** < 1ms per test, descriptive names, Arrange-Act-Assert structure, one assertion focus per test

## Architecture Constraints

**Circuit Simulation Engine:**
- 95% analytical formulas, 5% numerical (RK4 ODE lazy-loaded)
- Performance target: 60 FPS with 100+ components, <100ms simulation step

**Components:** Visual states NOT facial expressions
- MUST show state through: charge bars, glow intensity, mechanical positions, particle flows, voltage gradients
- NEVER use faces or anthropomorphism

**Visual Design:**
- 2D sketch aesthetic: wobbly lines, hatching, hand-drawn components
- MUST feel like inventor's notebook

**Color Palette:**
```
Paper White: #F5F5DC, Pencil Gray: #4A4A4A, Pen Blue: #1E3A8A
Copper: #B87333, Warning Red: #DC2626, Success Green: #16A34A
Power Yellow: #FBBF24, Highlight Orange: #F97316
```

**Code Organization:**
```
src/
├── components/    # React UI
├── engine/        # Circuit simulation
├── rendering/     # Visual rendering
├── challenges/    # Challenge system
├── progression/   # Crafting & tech tree
└── utils/
```

## Game Structure
- Act 1: Maker Era (1800s-1920s) - potato batteries, LEDs, crystal radios
- Act 2: Machine Era (1930s-1970s) - relays, tubes, logic gates
- Act 3: Silicon Era (1980s-2000s) - transistors, ICs, CPUs

## Challenge Design Rules

**MUST follow puzzle game best practices:**
- One new concept per level (component OR technique)
- Gradual complexity increase (no difficulty spikes)
- Combination challenges ONLY after individual concepts mastered
- Clear progression: learn → practice → combine
- Each challenge unlocks ONLY when previous solved (strict progression)

**Timed challenge constraints:**
- MUST NOT exceed 60 seconds (1 minute max)
- MUST run for minimum 10 seconds before validation
- Validation checks every frame but success only after minimum time
- Prevents instant-win exploits while keeping challenges fast-paced

**Hint system guidelines:**
- Multi-level hints (concept → guidance → solution)
- Track circuit state between hint requests
- If unchanged state + new hint request = player stuck, give more help
- Progressive disclosure: don't reveal full solution immediately

**Star rating criteria (for future):**
- ⭐ 1 star: Challenge completed (any solution)
- ⭐⭐ 2 stars: Efficient (fewer components)
- ⭐⭐⭐ 3 stars: Optimal (minimal components + fast time)

## Important Rules
- ALL circuits MUST use real electrical engineering principles
- Challenges MUST be auto-validated AND have solution tests proving solvability
- NO lectures or forced tutorials - learning by doing only
- Performance: <50MB download, runs on 5-year-old hardware

**Circuit Editing Rules:**
- NEVER allow circuit changes during simulation (no adding/removing components or wires while running)
- MUST disable all edit controls when simulation is running
- Build circuit → Start → Observe → Stop → Edit → Repeat
- This forces intentional design instead of live-tweaking

## Responsive UX Rules

**NEVER assume device type = capabilities:**
- WRONG: "mobile vs desktop" or "touch vs mouse"
- RIGHT: Detect orthogonal capabilities that can combine in any way
- Capabilities: pointer precision (coarse/fine), viewport size (small/medium/large), multi-touch, hover, keyboard
- Example: Touchscreen laptop = fine pointer + large viewport + multi-touch + hover + keyboard

**Capability-based adaptation MUST be used:**
- Touch target sizing: 60px for coarse pointers, 32px for fine pointers
- Component scale: 1.5x for coarse pointers, 1.0x for fine pointers
- Toolbar position: bottom for small viewports, top for medium/large

**NEVER use long-press for common actions:**
- Long-press = high friction, slow workflow
- Use explicit mode toggles with visual feedback instead
- Mode-based interactions: Component placement mode, wire mode, selection mode
- Clear exit paths: Escape key, click mode button again, click empty space to finish

**Click-sequence > drag for complex operations:**
- Wiring: Click A → B → C creates chain of wires (can zoom/pan between clicks)
- More precise than dragging
- Works equally well for touch and mouse
- Preview with dotted lines + component highlighting

**Multi-level undo MUST be available:**
- 20-action history minimum
- Toast notifications with inline [UNDO] button (3s auto-dismiss)
- Keyboard shortcut (Ctrl+Z)
- Track: add/delete components, create/delete wires, move components
- Clear undo history when simulation starts (prevents inconsistent state)

**Progressive enhancement approach:**
- Base functionality MUST work on all devices
- Enhanced features when capabilities available
- NEVER break base experience when adding enhancements

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/groupsky)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/groupsky)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
