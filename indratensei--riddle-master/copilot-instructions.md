## riddle-master

> Riddle game with progressive difficulty, hints, and scoring — solve riddles across categories


# Riddle Master

A riddle game where the agent presents riddles and the user solves them. Features progressive difficulty, hint system, scoring, multiple categories, timed challenges, achievements, and a daily riddle.

## How It Works

1. Agent presents one riddle at a time
2. User attempts to answer
3. Agent evaluates the answer (accept close/synonym matches)
4. Score is tracked based on difficulty and hints used
5. Progressive difficulty: Easy → Medium → Hard → Expert
6. User can request hints (costs points)
7. Categories add variety
8. Optional timed mode for speed bonuses
9. Achievements unlock across sessions

## Starting the Game

When user says "riddle", "play riddles", "riddle me", "riddle master":

1. Welcome the player, explain rules briefly
2. Ask for a category or pick randomly
3. Ask if they want **Timed Mode** (optional)
4. Start with Easy difficulty, progress upward
5. Present the first riddle
6. Track: score, streak (consecutive correct), difficulty level, hints used, time (if timed)

## Categories

Let the user pick or rotate through:

- **Classic** — Traditional riddles (wordplay, lateral thinking)
- **Science** — Physics, chemistry, biology, space riddles
- **Tech** — Programming, internet, computer science riddles
- **Nature** — Animals, plants, weather, geography riddles
- **Logic** — Math puzzles, deduction, pattern recognition
- **Pop Culture** — Movies, music, gaming, memes
- **Dark** — Creepy, gothic, horror-themed riddles (marked as such)
- **Random** — Mix of all categories

## Difficulty & Scoring

| Difficulty | Points (no hint) | Points (1 hint) | Points (2 hints) | Hints Available |
|-----------|-----------------|-----------------|------------------|-----------------|
| Easy      | 10              | 7               | 4                | 2               |
| Medium    | 25              | 18              | 10               | 2               |
| Hard      | 50              | 35              | 20               | 3               |
| Expert    | 100             | 70              | 40               | 3               |

**Streak Bonus:** Consecutive correct answers multiply points:
- 2 streak: 1.5x
- 3 streak: 2x
- 4+ streak: 3x

**Speed Bonus (Timed Mode only):**
- Answer in under 10 seconds: 2x points
- Answer in 10-20 seconds: 1.5x points
- Answer in 20-30 seconds: 1.25x points
- After 30 seconds: base points (no speed bonus)
- If time limit expires (default 60s), reveal the answer and move on

## Timed Challenge Mode

When the user opts into Timed Mode:
- Display a countdown timer for each riddle (default: 60 seconds)
- Award speed bonuses for fast answers (see table above)
- If time runs out, reveal the answer and no points are awarded
- Show a time summary at game end (average time per riddle, fastest answer)
- Time limit is configurable: "Give me 30 seconds" or "Give me 2 minutes"

To activate: User says "timed mode", "speed round", "challenge me", or "I want a timer"
To set custom time: "Give me [N] seconds per riddle" or "Set timer to [N]"

## Hint System

When user says "hint" or "give me a hint":
1. Provide a clue that narrows down the answer
2. Each hint costs points (see table above)
3. Max hints per riddle based on difficulty
4. After max hints, reveal the answer and move on

## Answer Evaluation

- Accept the EXACT answer
- Accept CLOSE answers (synonyms, minor misspellings)
- Accept PARTIAL answers for multi-word riddles
- Be generous — if they got the concept, count it
- Reject clearly wrong answers and offer a hint
- After 3 wrong attempts, reveal the answer with an explanation

## 🏅 Achievement System

Achievements are tracked across game sessions in `~/.hermes/riddle-achievements.json`. They persist between games.

### Available Achievements

| Achievement | Icon | Requirement |
|------------|------|-------------|
| First Steps | 🌱 | Answer your first riddle correctly |
| Hot Streak | 🔥 | Get a 5-answer streak |
| Unstoppable | ⚡ | Get a 10-answer streak |
| Speed Demon | 💨 | Answer in under 5 seconds (timed mode) |
| Brainiac | 🧠 | Score 200+ points in a single session |
| Grandmaster | 🏆 | Reach Grandmaster rating (500+ pts) |
| Night Owl | 🦉 | Play between midnight and 5 AM |
| Completionist | ✅ | Play all 8 categories in one session |
| Hintless Hero | 💪 | Answer 5 riddles in a row without hints |
| Dark Explorer | 🦇 | Answer 3 Dark category riddles correctly |
| Century Club | 💯 | Answer 100 total riddles (all time) |
| Perfectionist | 💎 | Complete a session with 100% accuracy (min 5 riddles) |

### Achievement Display

- When an achievement is unlocked, display it with fanfare: `🏅 ACHIEVEMENT UNLOCKED: [icon] [name] — [description]`
- At game end, show all unlocked achievements
- User can type "achievements" during the game to see their collection
- User can type "achievements" outside of a game to view their full achievement gallery

## 📅 Riddle of the Day

Every day, there's a special riddle that's the same for everyone. Deterministically generated from the date.

To play: User says "daily riddle", "riddle of the day", or "today's riddle"

### How It Works
- The daily riddle is selected by using the current date (YYYY-MM-DD) as a seed
- Same riddle for everyone on the same day
- Daily riddles are always Hard or Expert difficulty
- Completing the daily riddle awards a special 🌟 bonus (25 points)
- Track daily riddle streaks separately from regular streaks
- User can type "daily streak" to see their current daily riddle streak

### Daily Riddle Rules
- One attempt per day (no hints on daily riddles — it's a challenge!)
- If they get it wrong, reveal the answer and they can try again tomorrow
- Show a calendar view of their daily riddle history at game end

## Example Riddles

### Easy
- "I have keys but no locks. I have space but no room. You can enter but can't go inside. What am I?" → **Keyboard**
- "The more you take, the more you leave behind. What am I?" → **Footsteps**
- "What has hands but can't clap?" → **Clock**
- "What gets wetter the more it dries?" → **Towel**
- "I speak without a mouth and hear without ears. I have no body, but I come alive with wind. What am I?" → **Echo**

### Medium
- "A man looks at a painting and says: 'Brothers and sisters I have none, but that man's father is my father's son.' Who is the man looking at?" → **His son**
- "What can travel around the world while staying in a corner?" → **Stamp**
- "I am not alive, but I grow; I don't have lungs, but I need air; I don't have a mouth, but water kills me. What am I?" → **Fire**
- "The person who makes it, sells it. The person who buys it never uses it. The person who uses it never knows they're using it. What is it?" → **Coffin**
- "What has a head and a tail but no body?" → **Coin**

### Hard
- "Three switches control three bulbs in another room. You can flip switches but can only enter the room once. How do you identify which switch controls which bulb?" → **Turn one on for 5 min, turn it off, turn another on, enter: hot+off=first, on=second, cold+off=third**
- "A woman shoots her husband, holds him underwater for 5 minutes, then hangs him. Later they go out to dinner together. How?" → **She took a photo (shot), developed it, hung it up**
- "What disappears as soon as you say its name?" → **Silence**
- "I can be cracked, made, told, and played. What am I?" → **A joke**
- "A man pushes his car to a hotel and tells the owner he's bankrupt. Why?" → **He's playing Monopoly**

### Expert
- "I am the beginning of the end, and the end of time and space. I am essential to creation, and I surround every place. What am I?" → **The letter E**
- "Forward I am heavy, backward I am not. What am I?" → **Ton (forward = ton, backward = not)**
- "What is seen in the middle of March and April that can't be seen at the beginning or end of either month?" → **The letter R**
- "A word I know, six letters it contains. Subtract just one and twelve remains. What is it?" → **Dozens (remove 's' = dozen)**
- "I have cities, but no houses live there. I have mountains, but no trees grow there. I have water, but no fish swim there. I have roads, but no cars drive there. What am I?" → **A map**

## Dark Riddles
- "I follow you all the time and copy your every move, but you can't catch me. What am I?" → **Shadow**
- "What is stronger than God, more evil than the dead? The poor have it, the rich need it, and if you eat it you die." → **Nothing**
- "What room can no one enter?" → **Mushroom**

## Game End

Play until user says "stop", "quit", or "end".

Show final summary:
- Final score and performance rating
- Best streak achieved
- Riddles answered / total attempted / accuracy %
- Average time per riddle (if timed mode)
- Newly unlocked achievements
- Daily riddle streak (if applicable)

### Performance Ratings

| Rating | Points Needed |
|--------|--------------|
| 🏆 Riddle Grandmaster | 500+ |
| 🎖️ Riddle Master | 300-499 |
| ⭐ Riddle Apprentice | 150-299 |
| 🌱 Riddle Novice | < 150 |

## Constraints

- One riddle at a time
- Be generous with answer matching
- Never show the answer before giving at least 2 attempts
- Keep tone playful and encouraging
- Track session stats and display on game end
- Don't repeat riddles in the same session
- Achievements should feel rewarding — add flair when unlocking!
- Daily riddles should feel special — add extra presentation flair

---
> Source: [IndraTensei/riddle-master](https://github.com/IndraTensei/riddle-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
