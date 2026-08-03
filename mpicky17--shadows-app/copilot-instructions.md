## shadows-app

> A multiplayer PWA adaptation of the physical board game "Shadows Over Camelot" (Bruno Cathala & Serge Laget, 2005).

# Shadows Over Camelot — Digital Board Game

A multiplayer PWA adaptation of the physical board game "Shadows Over Camelot" (Bruno Cathala & Serge Laget, 2005).

## Version Bumps

Before every `git push`, bump both version strings to the next number:

| File | Constant |
|------|----------|
| `index.html` | `APP_VERSION` (e.g. `shadows-v2`) |
| `sw.js` | `CACHE_NAME` (e.g. `shadows-v2`) |

Both strings must match and be incremented together every time.

## Tech Stack

- **Frontend:** Vanilla JS (ES6+), HTML5, CSS3 — all inline in `index.html`
- **Multiplayer:** Firebase Realtime Database (project: separate from my-checklist-app)
- **Auth:** Firebase Anonymous Auth (no Google sign-in; players join via room codes)
- **Hosting:** GitHub Pages
- **PWA:** Service worker with network-first HTML, cache-first assets

## Firebase Config

```javascript
const FIREBASE_CONFIG = {
  apiKey:      'REPLACE_WITH_YOUR_API_KEY',
  authDomain:  'REPLACE.firebaseapp.com',
  databaseURL: 'https://REPLACE.firebaseio.com',
  projectId:   'REPLACE',
  appId:       'REPLACE'
};
```

Setup: Firebase Console > new project > add Web App > copy config > enable Anonymous Auth > enable Realtime Database > paste security rules.

---

## Game Overview

- **Players:** 3–7
- **Type:** Cooperative with possible hidden Traitor
- **Theme:** Knights of the Round Table completing quests while evil forces close in
- **Duration:** 60–90 minutes

### Win Conditions (Loyal Knights)
- 12 or more Swords on the Round Table with a **strict majority** of White Swords

### Loss Conditions (Immediate)
- 12 Siege Engines surrounding Camelot
- 7 or more Black Swords on the Round Table
- All Loyal Knights are dead (Traitor excepted)

### Ties
All ties in the game are resolved in **favor of Evil**: combat ties, sword count ties, siege engine fight ties.

---

## The Knights

Each Knight starts with 4 Life Points (max 6, dead at 0).

### Special Powers

| Knight | Title | Special Power | When Used |
|--------|-------|---------------|-----------|
| King Arthur | Son of Uther Pendragon | Once per turn, exchange 1 White card face-down with another Knight (regardless of location). Recipient must return 1 card. Both exchanges face-down, unknown to other players. | Heroic Action phase (before, after, or between actions) |
| Sir Galahad | Son of Lancelot | Once per turn, play 1 Special White card for free (must still take a different mandatory Heroic Action — cannot play a 2nd Special White as that mandatory action). | Heroic Action phase |
| Sir Gawain | Son of King Lot | When drawing White cards at the Round Table, draw 3 instead of 2. Still subject to 12-card hand limit. | During Round Table draw action |
| Sir Kay | Seneschal of King Arthur | When on a Quest where Combat ends, or when fighting a Siege Engine, may play 1 additional White Fight card after the Black cards/die have been revealed. | At combat quest resolution and siege engine fights |
| Sir Palamedes | Saracen Knight | For each victorious Quest he is on when it ends, gains 1 additional Life point (beyond quest victory spoils). Max 6 LP. | At quest victory |
| Sir Percival | Son of Pellinore | During Progression of Evil, may peek at the top Black card before deciding whether to draw it or select another Evil Action. | Progression of Evil phase |
| Sir Tristan | of Lyonesse | Departing from the Round Table is a free Move (doesn't cost his Heroic Action). Gets another Heroic Action to perform. | Heroic Action phase, when at Round Table |

### First Player
King Arthur goes first. If not in play, the youngest player starts. Play proceeds clockwise.

---

## Relics

Won from specific quests. Placed on the winning Knight's Coat of Arms.

| Relic | Source Quest | Effect | On Knight Death |
|-------|-------------|--------|-----------------|
| Excalibur | Quest for Excalibur | **Passive:** Add +1 to any Combat you participate in. **Active:** Discard Excalibur to counter any 1 Black Card just drawn. Cannot be gifted to another Knight. | Removed from game forever |
| Holy Grail | Quest for Holy Grail | Any dying Knight (reaching 0 LP) who drinks from the Grail gains 4 Life points. Discard after use. Owner can use on self. | N/A (consumed on use) |
| Lancelot's Armor | Quest for Lancelot | Anytime you must draw 1 Black card, draw 2 instead. Play 1, place the other back under the Black Draw pile. | Removed from game (but stays with Traitor if revealed) |

---

## Game Setup

1. Place Master gameboard in center. Place Excalibur, Holy Grail, Lancelot quest boards alongside.
2. Lancelot & Dragon quest starts with Lancelot's side face up.
3. Place relics on their quest boards. Place 12 Siege Engines, 4 Saxons, 4 Picts, 16 Swords in Reserve.
4. Each player gets a random Coat of Arms (Knight). Life die set to 4.
5. Place Knight miniatures at Round Table.
6. Shuffle Black cards into draw pile on board.
7. Give 1 Merlin card to each player. Shuffle remaining White cards. Deal 5 White cards to each player. Place remainder as White draw pile.
8. Shuffle 8 Loyalty cards. Deal 1 to each player (secret). Unused cards go in box unseen.
9. **Card sharing:** Each player places 1 White card face-up on Round Table. Discuss distribution. If no agreement, shuffle and redistribute randomly.

### 3-Player Variant
See "Three Brave Knights" in Advanced Rules — players don't peek at Loyalty cards until 6+ Swords on Round Table.

---

## Turn Structure

Each turn has two phases:

### Phase 1: Progression of Evil
Choose ONE of three actions (all help Evil):

#### a. Draw a Black Card
- Pick top Black card, read aloud, apply effect
- **Sir Percival** may peek first before deciding
- **Lancelot's Armor** holder draws 2, picks 1 to play, puts other back under the Black draw pile
- If Black Knight/Lancelot/Dragon combat card: may play face-down (hide value) and draw 1 free White card
- Standard Black: play on corresponding quest
- Special Black: apply immediately (can be cancelled by 3 Merlin cards collectively — must be immediate, never retroactive)
- If quest for a Black card is no longer in play: add 1 Siege Engine instead, discard the card

#### b. Add a Siege Engine
- Take from Reserve, place on empty Siege Engine spot around Camelot
- If 12th placed: game immediately lost

#### c. Lose 1 Life Point
- Turn Life die down by 1
- If reaches 0: die at end of turn (see Death rules)

**After Phase 1:** Check game-end conditions. If not triggered, proceed to Phase 2.

### Phase 2: Heroic Actions
Perform ONE Heroic Action (must take one — if unable to do quest action, must heal/accuse/play special white/move):

#### a. Move to a New Quest
- Move Knight to any destination (distance irrelevant, costs 1 Heroic Action)
- Exception: Moving between Camelot's Siege Area and Round Table is free
- Solo Quests (Black Knight, Lancelot): only 1 Knight at a time, must be unoccupied
- **Sir Tristan:** departing from Round Table is free (gets another Heroic Action)

#### b. Perform Quest-Specific Action
See Quest Details section below.

#### c. Play a Special White Card
- Read aloud, apply effect, discard
- Max 1 Special White card per turn
- **Sir Galahad:** may play this for free (still needs a different mandatory action)

#### d. Heal Yourself
- Discard 3 identical cards (e.g., 3 Grail cards or 3 Fight cards of same value)
- Gain 1 LP (max 6)

#### e. Accuse a Knight
- **Available only when:** 6+ Siege Engines around Camelot OR 6+ Swords on Round Table
- **Each Knight may accuse only once** in the entire game
- Don't need to be at same location as accused
- Target reveals Loyalty card:
  - If Loyal: flip 1 White Sword to Black (if any White Swords exist)
  - If Traitor: add 1 White Sword to Round Table. Traitor follows revealed Traitor rules.
- Traitor may falsely accuse to sow confusion

### Sacrifice (Extra Heroic Action)
- Once per turn, sacrifice 1 LP for a second **different** Heroic Action
- Cannot perform same action twice in same turn
- Can use Special Power before, after, or between the two actions
- If sacrifice brings you to 0 LP: perform the final action, then die (even if action would grant LP; only Holy Grail saves)

**After Phase 2:** Check game-end conditions. If not triggered, next player's turn.

---

## Quest Details

### Camelot (Two Sections)

#### Round Table (Inside)
- **Heroic Action — Draw White Cards:** Draw 2 White cards (Sir Gawain draws 3). Cannot draw if holding 12+ cards.

#### Siege Area (Outside Walls)
- **Heroic Action — Fight a Siege Engine:**
  1. Play as many Fight cards as desired from hand
  2. Roll the 8-sided die (1–8)
  3. If sum of Fight cards > die value: win. Remove Siege Engine to Reserve, discard cards.
  4. If sum ≤ die value: lose. Lose 1 LP. Siege Engine stays. Discard cards.
  5. Excalibur adds +1 to your sum if you have it.

Moving between Round Table and Siege Area is free (no Heroic Action cost).

### Tournament Against the Black Knight
- **Type:** Perpetual, Solo, Combat Quest
- **White card pattern:** 2 pairs of distinct values (4 cards total on Knights' side, but need to fill them)
- **Progression of Evil:** Black Knight card drawn → placed on quest. 4th Black Knight card ends the quest.
- **Heroic Action:** Play 1 White Fight card (value 1–5) on empty Knights' side spot. 4th White card → quest ends.
- **Resolution:** Compare sum of White Fight cards vs sum of Black cards (shuffle Black cards before revealing). Higher total wins. Ties go to Evil.
- **Victory:** 1 White Sword. Knight gains 1 LP. 3 White cards shared. Knight goes to Round Table.
- **Defeat:** 1 Black Sword. Knight loses 1 LP. Knight goes to Round Table.
- **Perpetual:** Quest resets after completion — a new tournament begins.
- **Sir Kay:** may play 1 extra Fight card after Black cards are revealed.

### Quest for Lancelot
- **Type:** Solo, Combat Quest
- **White card pattern:** Full house (3-of-a-kind + pair) — 5 cards on Knights' side
- **Progression of Evil:** Lancelot card drawn → placed on quest. 5th Lancelot card ends the quest.
- **Heroic Action:** Play 1 White Fight card on empty Knights' side spot. 5th White card → quest ends.
- **Resolution:** Compare sums. Higher wins. Ties go to Evil.
- **Victory:** 1 White Sword. Knight gains 1 LP. Knight gets Lancelot's Armor relic. 4 White cards to winner.
- **Defeat:** 1 Black Sword. Knights present lose 1 LP. Knight goes to Round Table.
- **One-time quest:** Once completed (won or lost), flips to Dragon's Quest.
- **Sir Kay:** may play 1 extra Fight card after Black cards are revealed.

### Dragon's Quest
- **Type:** Group, Combat Quest (replaces Lancelot after Lancelot's Quest ends)
- **White card pattern:** 3 three-of-a-kinds — 9 cards on Knights' side
- **Progression of Evil:** Dragon card drawn → placed on quest. 5th Dragon card ends the quest.
- **Heroic Action:** Play 1 White Fight card on empty Knights' side spot. 9th White card → quest ends.
- **Resolution:** Compare sums. Higher wins. Ties go to Evil.
- **Victory:** 2 White Swords. Knights gain 2 LP. 7 White cards shared. Knight goes to Round Table.
- **Defeat:** 2 Black Swords. Knights present lose 2 LP. Knight goes to Round Table.
- **One-time quest:** Once completed, this quest is removed from game.
- **Note:** Lancelot/Dragon cards use dual values — Lancelot value when Lancelot quest active, Dragon value when Dragon quest active.
- **Sir Kay:** may play 1 extra Fight card after Black cards are revealed.

### Quest for Excalibur
- **Type:** Group Quest (any number of Knights)
- **Board:** River with positions. Excalibur starts in the middle.
- **Progression of Evil:** Excalibur Black card drawn → moves Excalibur 1 space toward Evil side. If reaches last Evil position → quest lost.
- **Heroic Action:** Discard 1 White card face-down → move Excalibur 1 space toward Knight side. If reaches last Knight position → quest won.
- **Victory:** 2 White Swords. Knights gain 1 LP each. 7 White cards divided among knights. Completing Knight gets Excalibur relic. All Knights on quest go to Round Table.
- **Defeat:** 2 Black Swords. Knights lose 1 LP. Excalibur removed from game forever. All Knights on quest go to Round Table.
- **One-time quest:** Once completed, quest is removed.
- **Any White card works** (not just specific types).

### Quest for the Holy Grail
- **Type:** Group Quest (any number of Knights)
- **Board:** 7 card spots leading to the Grail.
- **Progression of Evil:** Despair/Desolation/Dark Forest Black cards placed on the quest, filling spots from the far end.
- **Heroic Action:** Play 1 Grail card on the first empty spot closest to the Grail. If all 7 spots are filled (mix of Grail + Despair cards), instead remove the closest Despair/Desolation card (discard both the Grail card played and the Despair removed).
- **Win Condition:** 7th Grail card placed on the last empty spot → quest won.
- **Loss Condition:** All spots filled with Black cards (Despair/Desolation) with no Grail cards → quest lost.
- **Victory:** 3 White Swords. Knights gain 1 LP. 7 White cards shared. Completing Knight gets Holy Grail relic. All Knights on quest go to Round Table.
- **Defeat:** 3 Black Swords. Knights lose 1 LP. Holy Grail removed from game forever. All Knights on quest go to Round Table.
- **One-time quest.**

### Pict War
- **Type:** Perpetual, Group Quest
- **Progression of Evil:** Pict or Mercenary card drawn → add 1 Pict warrior figure. 4th Pict warrior → quest lost.
- **Heroic Action:** Play 1 Fight card in ascending straight (must start with 1, each subsequent card exactly +1). 5th Fight card (value 5) → quest won.
- **Victory:** 1 White Sword. Knights on quest gain 1 LP each. Draw 4 White cards from top, share among Knights present.
- **Defeat:** 1 Black Sword + 2 Siege Engines. Knights on quest lose 1 LP.
- **Perpetual:** Quest resets after completion.

### Saxon War
- **Type:** Perpetual, Group Quest
- **Identical mechanics to Pict War** but with Saxon warriors and Saxon/Mercenary cards.
- **Victory:** 1 White Sword. Knights on quest gain 1 LP each. Draw 4 White cards from top, share among Knights present.
- **Defeat:** 1 Black Sword + 2 Siege Engines. Knights on quest lose 1 LP.
- **Perpetual:** Quest resets after completion.

### Quest Completion (General Rules)
When any quest ends:
1. All Knights on the quest move back to Round Table (free, no action cost)
2. Victory or Defeat consequences applied immediately
3. All cards on the quest discarded to respective discard piles (face-down)
4. Saxon/Pict warrior figures returned to Reserve
5. Perpetual quests reset for new attempts

---

## Death

- When LP reaches 0 (from any cause): Knight dies at end of their current turn
- Discard all White cards face-down
- Remove miniature from board
- If holding Excalibur or Lancelot's Armor: removed from game forever
- **Holy Grail save:** If Holy Grail was won earlier and its owner offers it (or you are the owner), set LP to 4. Remove Holy Grail from game. Only works once.
- **Sacrifice death:** If sacrifice brings you to 0, complete the final action first, then die. Only Holy Grail can save you.
- **Do NOT reveal Loyalty** even when dead. Revealed only at game end.
- Dead Knights can still win posthumously if their side wins.

---

## The Traitor

### Hidden Traitor (Before Reveal)
- Plays normally like any other Knight
- Should act deceptively — subtle sabotage, not obvious evil moves
- May lie about intent or resources (but never cheat)
- May falsely accuse a Loyal Knight to flip a White Sword to Black
- Can voluntarily reveal at any time

### Revealed Traitor (After Accusation or Voluntary Reveal)
When revealed:
1. Accusing Knight adds 1 White Sword to Round Table (if accused, not voluntary)
2. Remove miniature from board (plays from the Shadows, out of reach)
3. Remove Life die (cannot be killed)
4. Discard all White cards
5. If holding Excalibur or Holy Grail: remove them from game
6. Lancelot's Armor stays with the Traitor

**Revealed Traitor's Turn:**
1. **Taunt the Knights:** Pick 1 White card at random from the hand of a Knight of your choice and discard it.
2. **Help Evil spread (choose 1):**
   - Add 1 Siege Engine to Camelot
   - OR Draw & play the top Black card

**Traitor Victory Conditions:**
- 12 Siege Engines surrounding Camelot
- 7+ Black Swords on the Round Table
- All other Knights dead
- 12+ Swords on Round Table and at least half are Black

### End-Game Traitor Reveal
If the Traitor is still alive and undetected when the 12th Sword is placed: Traitor reveals Loyalty card, and 2 White Swords are flipped to Black. Then count majority.

---

## Game End

The game ends **immediately** when:
1. 12 Siege Engines placed (instant loss for Loyal Knights)
2. 7+ Black Swords on Round Table (instant loss for Loyal Knights)
3. All Loyal Knights dead (instant loss for Loyal Knights)
4. 12th Sword placed on Round Table → hidden Traitor reveals (flips 2 white to black) → count majority:
   - Strictly more White than Black: Loyal Knights win
   - Otherwise: Evil/Traitor wins

Note: If several Swords are laid at once during the final action, the game may end with more than 12 Swords.

---

## Card Manifest

### White Cards (84 total)

#### Standard White — Fight Cards (51 total)
| Value | Count | Used In |
|-------|-------|---------|
| Fight 1 | x14 | Combat Quests, Saxon/Pict Wars, Siege Engine fights |
| Fight 2 | x12 | Combat Quests, Saxon/Pict Wars, Siege Engine fights |
| Fight 3 | x10 | Combat Quests, Saxon/Pict Wars, Siege Engine fights |
| Fight 4 | x8 | Combat Quests, Saxon/Pict Wars, Siege Engine fights |
| Fight 5 | x7 | Combat Quests, Saxon/Pict Wars, Siege Engine fights |

#### Standard White — Grail Cards (18 total)
All identical. Used exclusively in the Quest for the Holy Grail.

#### Special White Cards (15 total)
| Card | Count | Effect |
|------|-------|--------|
| Merlin | x7 | **Cancel:** 3 Merlin cards played collectively (by any players) cancel 1 Special Black card immediately when drawn. Cannot be applied retroactively. **Heroic Play:** Play 1 Merlin to: remove 1 Siege Engine, OR remove last standard black card from any quest, OR remove 1 warrior from Saxon/Pict War. During setup, 1 Merlin given to each player before shuffling. |
| Fate | x1 | **If Loyal:** All Knights draw 1 White card. **If Traitor:** Reveal yourself. All opponents discard 2 White cards. |
| Piety | x1 | Gain 3 Life points (max 6) OR each other Knight gains 1 Life point (max 6). |
| Heroism | x1 | Place on the Quest of your choice. Adds 1 White Sword (if quest won) or 1 Black Sword (if quest lost) to that Quest's outcome. |
| Reinforcements | x1 | Draw 4 White cards OR let each of the other Knights draw 1 White card. |
| Clairvoyance | x1 | Draw the first 5 cards from the top of the Black draw pile, look at them, and reorder them as you wish. |
| Messenger | x1 | Pass 1 to 3 White cards from your hand to the Knight of your choice. |
| Lady of the Lake | x1 | If Excalibur's Quest has not been completed yet, move Excalibur 1 spot closer to victory. Otherwise, gain 2 Life points. |
| Convocation | x1 | All Knights may return to Camelot at once. The gathered Knights draw and share White cards equal to their number. |

### Black Cards (76 total)

#### Standard Black — Black Knight Cards (11 total)
Combat value cards for the Tournament Against the Black Knight:
| Card | Count |
|------|-------|
| Black Knight 1 | x5 |
| Black Knight 3 | x3 |
| Black Knight 5 | x2 |
| Black Knight 7 | x1 |

#### Standard Black — Lancelot/Dragon Cards (11 total)
Dual-value combat cards. Use Lancelot value when Lancelot Quest is active; Dragon value when Dragon Quest is active:
| Card | Count |
|------|-------|
| Lancelot 1 / Dragon 5 | x4 |
| Lancelot 3 / Dragon 7 | x3 |
| Lancelot 5 / Dragon 9 | x3 |
| Lancelot 7 / Dragon 11 | x1 |

#### Standard Black — Quest Cards (42 total)
| Card | Count | Effect |
|------|-------|--------|
| Excalibur | x15 | Move Excalibur 1 space toward Evil side. If quest not in play, add 1 Siege Engine instead. |
| Despair | x15 | Place on Holy Grail quest board. If quest not in play, add 1 Siege Engine instead. |
| Saxon | x4 | Add 1 Saxon warrior to Saxon War. If 4th warrior placed, quest immediately lost. |
| Pict | x4 | Add 1 Pict warrior to Pict War. If 4th warrior placed, quest immediately lost. |
| Mercenaries | x4 | Player's choice: add 1 warrior to either Saxon or Pict War. |

#### Special Black Cards (12 total)
Applied immediately when drawn. Can be cancelled by 3 Merlin cards played collectively (must be immediate, never retroactive).

| Card | Count | Effect |
|------|-------|--------|
| Morgan 1 | x1 | Any knight may volunteer to discard 3 cards. If nobody does, each knight discards 1. |
| Morgan 2 | x1 | Drawing player draws and applies the next 3 Black cards immediately. |
| Morgan 3 | x1 | Add 2 Siege Engines to Camelot. |
| Morgan 4 | x1 | Each Knight loses 1 Life point. |
| Morgan 5 | x1 | Any Knight may volunteer to lose 2 Life points. If nobody volunteers, ALL Knights discard 1 White card each (if they have at least one). |
| Desolation | x2 | Place on Holy Grail quest (same as Despair but Special — cannot be countered by quest action, only by 3 Merlins). |
| Dark Forest | x1 | No Grail cards may be played until a quest is won. If Holy Grail quest inactive, add siege engine. |
| Guinevere | x1 | All knights return to Camelot. Solo quest white cards removed. Drawing player's turn ends immediately. |
| Mist of Avalon | x1 | Permanent. Each quest lost adds 1 extra Black Sword. |
| Mordred | x1 | Player adds 1 warrior to Saxon or Pict War and attaches Mordred. That war needs an extra Fight 5 to win. Removed on win/loss. |
| Vivien | x1 | No Merlin cards may be played until a quest is won. Can be cancelled by 3 Merlins when drawn. |

### Loyalty Cards (8 total)
| Card | Count |
|------|-------|
| Loyal | x7 |
| Traitor | x1 |

---

## Deck Reshuffling

When **either** draw pile (White or Black) runs out:
1. Reshuffle **both** discard piles into fresh draw piles
2. This happens even if the other pile hasn't been depleted yet
3. Both piles are reshuffled simultaneously

---

## Communication Rules

Players may discuss:
- Declarations of intent ("I'll fight the Saxons!")
- General resources ("My cards are strong")
- Capabilities ("That Black Knight looks weak")

Players may **NOT**:
- Reveal exact card values ("I have 3 Grails" or "I have a Fight 5")
- Propose specific trades ("Give me your Fight 3")

Players **MAY** lie about intent or resources (useful for the Traitor). But never cheat.

---

## Advanced and Optional Rules (Lowest Priority)

### Joining Mid-Game
New player draws remaining Coat of Arms + Loyalty card. Sits left of starting player. Life die to 4, draws 5 White cards, places miniature in Camelot.

### Expert Rules

#### The Squire's Challenge
Veteran players start as Squires (no Coat of Arms, no Special Power). Just Life die (4), 5 White cards, 1 Merlin. Earn Knighthood when present on a winning Quest. Then receive Coat of Arms + Special Power.

#### The Traitor Among Us
Take exactly N Loyal cards (N = player count), add Traitor card, shuffle, distribute. Dramatically increases Traitor probability.

#### Three Brave Knights (3 Players)
Don't peek at Loyalty cards until 6+ Swords on Round Table. Traitor should emphasize deception over brute force.

---

## Firebase Data Model

### Shared State — `games/{roomCode}/`
```
meta/
  roomCode, createdAt, hostUid, status (lobby|setup|playing|ended)
  currentTurnKnight, turnPhase, turnNumber, winner, includeTraitor
  sacrificeUsed, heroicActionTaken, specialWhitePlayed
  persistentEffects: { darkForest, mistOfAvalon, mordred: null|'saxonWar'|'pictWar', vivien }
  morgan2Remaining: number|null
board/
  siegeEngines: number (0-12)
  swords: [{color, source}]
quests/
  holyGrail/    { status, grailCards[], blackCards[], knightsPresent[] }
  excalibur/    { status, position, knightsPresent[], ownerKnight }
  blackKnight/  { status, whiteCards[], blackCards[], knightsPresent[] }
  lancelot/     { status, phase, whiteCards[], blackCards[], knightsPresent[] }
  saxonWar/     { status, warriors, fightCards[], knightsPresent[] }
  pictWar/      { status, warriors, fightCards[], knightsPresent[] }
players/{knightId}/
  uid, displayName, knightId, life, location, alive, hasAccused
  relics: { excalibur, holyGrail, lancelotArmor }
  isRevealed: false
decks/
  whiteDrawCount, blackDrawCount
log/
  {pushId}: { timestamp, type, knight, message }
```

### Private State — `private/{roomCode}/`
```
{knightId}/
  uid, loyalty, hand: [{id, type, value/name}]
decks/
  uid, whiteDraw[], blackDraw[], whiteDiscard[], blackDiscard[]
```

---
> Source: [mpicky17/shadows-app](https://github.com/mpicky17/shadows-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
