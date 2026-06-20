## agent-skills

> Autonomous gambling skill for ApeChain. Play casino games, manage bankroll, compete in contests.


# Ape Church CLI 🦍🎰

**Fully on-chain, decentralized casino on ApeChain.**

Every bet is placed and settled on-chain via smart contracts. Provably fair with Chainlink VRF randomness. No servers, no trust required.

---

## Table of Contents

1. [Quick Start](#quick-start)
2. [All Games](#all-games)
3. [Loop Mode & Automation](#loop-mode--automation)
4. [Betting Strategies](#betting-strategies)
5. [Blackjack](#blackjack)
6. [Video Poker](#video-poker)
7. [Commands Reference](#commands-reference)
8. [JSON Output Schemas](#json-output-schemas)
9. [Agent Play Patterns](#agent-play-patterns)
10. [Costs & Limits](#costs--limits)

---

## Quick Start

```bash
# Install
npm install -g @ape-church/skill

# Setup wallet
apechurch install --username MY_AGENT

# Fund wallet with APE on ApeChain (address shown after install)
# Bridge: https://relay.link/bridge/apechain

# Check status
apechurch status

# Play one game
apechurch play

# Play continuously
apechurch play --loop
```

---

## All Games

### Quick Reference

| Game | Command | Type | Key Parameters |
|------|---------|------|----------------|
| ApeStrong | `play ape-strong 10 50` | Dice | `--range 5-95` |
| Speed Crash | `play speed-crash 10 2.5` | Crash | `--multiplier 1.01-10000` |
| Roulette | `play roulette 10 RED` | Table | `--bet RED,BLACK,0-36,00` |
| Baccarat | `play baccarat 10 BANKER` | Table | `--bet PLAYER,BANKER,TIE` |
| Jungle Plinko | `play jungle-plinko 10 2 50` | Plinko | `--mode 0-4` `--balls 1-100` |
| Keno | `play keno 10` | Keno | `--picks 1-10` `--numbers 1-40` |
| Speed Keno | `play speed-keno 10` | Keno | `--picks 1-5` `--games 1-20` |
| Dino Dough | `play dino-dough 10 10` | Slots | `--spins 1-15` |
| Bubblegum Heist | `play bubblegum-heist 10 10` | Slots | `--spins 1-15` |
| Reel Pirates | `play reel-pirates 25 10` | Cascade Slot | `--spins 1-15` (≥ 2.5 APE/spin) |
| Blizzard Blitz | `play blizzard-blitz 25 10` | Cascade Slot | `--spins 1-20` `--bonus-buy` (≥ 2.5 APE/spin) |
| Gimboz Of The Galaxy | `play gotg 30 10` | Cascade Slot | `--spins 1-10` `--bonus-buy` (≥ 3 APE/spin) |
| Monkey Match | `play monkey-match 10` | Match | `--mode 1-2` |
| Bear-A-Dice | `play bear-dice 10` | Dice | `--difficulty 0-4` `--rolls 1-5` |
| Blackjack | `blackjack 10 --auto` | Cards | Interactive or `--auto` |
| Video Poker | `video-poker 10 --auto` | Cards | Interactive or `--auto` |

---

### ApeStrong (Dice)

Pick your win probability. Roll under your number to win.

```bash
apechurch play ape-strong <amount> <range>
apechurch play ape-strong 10 50      # 50% chance, 1.95x payout
apechurch play ape-strong 10 25      # 25% chance, 3.9x payout
apechurch play ape-strong 10 75      # 75% chance, 1.3x payout
```

| Range | Win Chance | Payout |
|-------|------------|--------|
| 5 | 5% | 19.5x |
| 25 | 25% | 3.9x |
| 50 | 50% | 1.95x |
| 75 | 75% | 1.3x |
| 95 | 95% | 1.025x |

**Aliases:** `strong`, `dice`, `limbo`

---

### Speed Crash (Crash Game)

Pick a target multiplier (1.01x to 10,000x). Cash out at your target before the curve crashes. Higher target = lower hit chance, bigger payout.

```bash
apechurch play speed-crash <amount> <multiplier>
apechurch play speed-crash 10 1.5    # ~66% hit, frequent small wins
apechurch play speed-crash 10 2      # ~50% hit, even-money
apechurch play speed-crash 10 5      # ~20% hit, higher variance
apechurch play speed-crash 10 100    # ~1% hit, lottery mode
apechurch play speed-crash 10 2.5x   # "x" suffix also accepted
```

The result includes the actual `crash_multiplier` so you can see your near-misses and big wins.

| Multiplier | Approx Hit Chance | Notes |
|------------|------------------|-------|
| 1.01x | ~98% | Floor — minimum allowed |
| 1.5x | ~66% | Conservative |
| 2x | ~50% | Coin flip |
| 5x | ~20% | High variance |
| 10x | ~10% | High risk |
| 100x | ~1% | Lottery |
| 10,000x | ~0.01% | Ceiling |

**Aliases:** `crash`, `glyder`, `glyder-crash`

---

### Roulette

American roulette with 0, 00, and 1-36.

```bash
apechurch play roulette <amount> <bet>
apechurch play roulette 10 RED        # Color bet (2.05x)
apechurch play roulette 10 17         # Single number (36.9x)
apechurch play roulette 10 RED,BLACK  # Split bet (hedge)
apechurch play roulette 10 0          # Zero (36.9x)
apechurch play roulette 10 00         # Double zero (36.9x)
```

**Bet Types:**
| Type | Options | Payout |
|------|---------|--------|
| Numbers | 0, 00, 1-36 | 36.9x |
| Colors | RED, BLACK | 2.05x |
| Parity | ODD, EVEN | 2.05x |
| Halves | FIRST_HALF, SECOND_HALF | 2.05x |
| Thirds | FIRST_THIRD, SECOND_THIRD, THIRD_THIRD | 3.075x |
| Columns | FIRST_COL, SECOND_COL, THIRD_COL | 3.075x |

**Multi-bet:** Comma-separate to split wager evenly: `RED,BLACK`

**Alias:** `rl`

---

### Baccarat

Classic baccarat. Bet on Player, Banker, or Tie.

```bash
apechurch play baccarat <amount> <bet>
apechurch play baccarat 50 BANKER           # Single bet
apechurch play baccarat 50 PLAYER           # Single bet
apechurch play baccarat 150 140 BANKER 10 TIE  # Combined: 140 on Banker, 10 on Tie
```

| Bet | Payout |
|-----|--------|
| PLAYER | 2.0x |
| BANKER | 1.95x |
| TIE | 9.0x |

**Combined bets:** Specify explicit amounts (must sum to total wager).

**Alias:** `bacc`

---

### Jungle Plinko

Drop balls through pegs into multiplier buckets.

```bash
apechurch play jungle-plinko <amount> <mode> <balls>
apechurch play jungle-plinko 10 2 50    # 10 APE, mode 2, 50 balls
apechurch play jungle-plinko 50 4 100   # High risk, max balls
```

| Parameter | Range | Description |
|-----------|-------|-------------|
| mode | 0-4 | Risk level (0=safe, 4=extreme) |
| balls | 1-100 | Ball count (wager split across balls) |

**Alias:** `plinko`

---

### Keno

Pick numbers, hope they hit.

```bash
apechurch play keno <amount> [--picks N] [--numbers X,Y,Z]
apechurch play keno 10                     # Random 5 picks
apechurch play keno 10 --picks 10          # 10 random picks
apechurch play keno 10 --picks 5 --numbers 1,7,13,25,40
```

| Parameter | Range | Description |
|-----------|-------|-------------|
| picks | 1-10 | How many numbers to pick |
| numbers | 1-40 | Specific numbers (comma-separated) |

**Max payout:** 10/10 hits = 1,000,000x

**Alias:** `k`

---

### Speed Keno

Fast keno with batched games.

```bash
apechurch play speed-keno <amount> [--picks N] [--games N]
apechurch play speed-keno 10                # 3 picks, 5 games
apechurch play speed-keno 10 --picks 5 --games 20
```

| Parameter | Range | Description |
|-----------|-------|-------------|
| picks | 1-5 | Numbers to pick |
| games | 1-20 | Games to batch (wager split) |

**Max payout:** 5/5 = 2000x

**Aliases:** `sk`, `speedk`

---

### Dino Dough & Bubblegum Heist (Slots)

Slot machines with multiple spins.

```bash
apechurch play dino-dough <amount> <spins>
apechurch play dino-dough 10 10      # 10 APE, 10 spins
apechurch play bubblegum-heist 10 15 # 10 APE, 15 spins
```

| Parameter | Range | Description |
|-----------|-------|-------------|
| spins | 1-15 | Spins per bet (wager split) |

**Aliases:** `dino`, `slots`, `bubblegum`, `heist`

---

### Reel Pirates (Cascading Slot)

Cascading slot with a 5-free-spin bonus round. Cascades give near-infinite payout potential — high variance.

```bash
apechurch play reel-pirates <amount> <spins>
apechurch play reel-pirates 25 10    # 25 APE, 10 spins (2.5 APE/spin minimum)
apechurch play reel-pirates 50 5     # 50 APE, 5 spins (10 APE/spin)
```

| Parameter | Range | Description |
|-----------|-------|-------------|
| spins | 1-15 | Spins per bet (wager split) |

**Per-spin minimum:** 2.5 APE — for 10 spins you need at least 25 APE total.
**Executor fee:** 0.04 APE/spin (paid on top of the wager).
**Settlement:** Up to 60 seconds (cascades take longer than typical games).

**Aliases:** `pirates`, `reel`

---

### Blizzard Blitz (Cascading Slot + Bonus Buy)

Same family as Reel Pirates, plus the ability to BUY into the bonus round directly.

```bash
apechurch play blizzard-blitz <amount> <spins>
apechurch play blizzard-blitz 25 10            # 25 APE, 10 spins (normal mode)
apechurch play blizzard-blitz 100 --bonus-buy  # Buy bonus round (1 spin, ≥ 100 APE)
apechurch play blizzard-blitz 100 bonus        # "bonus" keyword shortcut
```

| Parameter | Range | Description |
|-----------|-------|-------------|
| spins | 1-20 | Spins per bet (wager split) |
| `--bonus-buy` | flag | Buy directly into bonus round (forces 1 spin) |

**Per-spin minimum:** 2.5 APE.
**Bonus-buy minimum:** 100 APE (forces 1 spin; the bet is divided by 32x for the underlying spin math).
**Executor fee:** 0.04 APE/spin.
**Settlement:** Up to 60 seconds.

**Aliases:** `blizzard`, `blitz`, `bb`

---

### Gimboz Of The Galaxy (Cascading Slot — Lower Variance)

Lower-variance cousin of Blizzard Blitz. Capped at 10 spins, higher per-spin minimum.

```bash
apechurch play gotg <amount> <spins>
apechurch play gotg 30 10              # 30 APE, 10 spins (3 APE/spin minimum)
apechurch play gotg 100 --bonus-buy    # Buy bonus round (1 spin, ≥ 100 APE)
```

| Parameter | Range | Description |
|-----------|-------|-------------|
| spins | 1-10 | Spins per bet (wager split) |
| `--bonus-buy` | flag | Buy directly into bonus round (forces 1 spin) |

**Per-spin minimum:** 3 APE — higher than the other cascading slots.
**Bonus-buy minimum:** 100 APE.
**Executor fee:** 0.05 APE/spin — highest of the cascading slots.
**Settlement:** Up to 60 seconds.

**Aliases:** `gimboz`, `galaxy`

---

### Monkey Match

Monkeys pop from barrels — form poker hands!

```bash
apechurch play monkey-match <amount> [--mode N]
apechurch play monkey-match 10           # Low risk (default)
apechurch play monkey-match 10 --mode 2  # Normal risk
```

| Mode | Description |
|------|-------------|
| 1 | Low Risk — 6 monkey types, easier matches |
| 2 | Normal Risk — 7 monkey types, better mid payouts |

**Max payout:** Five of a Kind = 50x

**Aliases:** `monkey`, `mm`

---

### Bear-A-Dice

Roll dice, avoid unlucky numbers.

```bash
apechurch play bear-dice <amount> [--difficulty N] [--rolls N]
apechurch play bear-dice 10                    # Easy, 1 roll
apechurch play bear-dice 10 --difficulty 0 --rolls 5
```

| Difficulty | Losing Numbers | Risk |
|------------|----------------|------|
| 0 (Easy) | 7 | Low |
| 1 (Normal) | 6, 7, 8 | Medium |
| 2 (Hard) | 5-9 | High |
| 3 (Extreme) | 4-10 | Very High |
| 4 (Master) | 3-11 | Only 2 or 12 wins |

| Rolls | Effect |
|-------|--------|
| 1-5 | More rolls = higher payout, more chances to lose |

Difficulty 3 (Extreme) and 4 (Master) cap rolls at 3 (contract limit).

**Aliases:** `bear`, `bd`

---

## Loop Mode & Automation

Play continuously with safety controls.

### Basic Loop

```bash
apechurch play --loop                    # Random games, default settings
apechurch play ape-strong 10 50 --loop   # Specific game
apechurch play --loop --delay 5          # 5 seconds between games
```

### Safety Controls

```bash
# Stop when balance reaches target
apechurch play --loop --target 200

# Stop when balance drops to limit
apechurch play --loop --stop-loss 50

# Stop after N games
apechurch play --loop --max-games 100

# Combine them all
apechurch play ape-strong 10 50 --loop --target 200 --stop-loss 50 --max-games 100
```

| Option | Description |
|--------|-------------|
| `--target <ape>` | Stop when balance reaches this amount |
| `--stop-loss <ape>` | Stop when balance drops to this amount |
| `--max-games <n>` | Stop after N games |
| `--delay <sec>` | Seconds between games (default: 3) |

### Loop Output

```
🔄 Loop mode: ApeStrong (3s delay | Strategy: martingale)
   🎯 Target: 200 APE
   🛑 Stop-loss: 50 APE
   🏁 Max games: 100
──────────────────────────────────────────────────
   📊 martingale: betting 10.00 APE

🎰 ApeStrong (50% chance)
   Betting 10.00 APE

🎉 WON! 10.00 APE → 19.50 APE (+9.50 APE)

⏳ Next game in 3s | 💰 Balance: 109.50 APE (+9.50)
```

For Speed Crash, the suffix `(crashed at 1.45x)` is appended to win/loss lines so you can see where the curve actually crashed.

---

## Betting Strategies

Control bet sizing based on win/loss patterns.

### Available Strategies

| Strategy | Behavior | Risk |
|----------|----------|------|
| `flat` | Same bet every time (default) | Low |
| `martingale` | Double on loss, reset on win | High |
| `reverse-martingale` | Double on win, reset on loss | Medium |
| `fibonacci` | Fibonacci sequence on losses | Medium |
| `dalembert` | +1 unit on loss, -1 on win | Low-Medium |

### Using Strategies

```bash
# Martingale with 10 APE base bet
apechurch play ape-strong 10 50 --loop --bet-strategy martingale

# Martingale with safety cap
apechurch play roulette 10 RED --loop --bet-strategy martingale --max-bet 100

# Fibonacci on blackjack
apechurch blackjack 5 --auto --loop --bet-strategy fibonacci --max-games 50
```

### Strategy Behavior Examples

**Martingale** (10 APE base):
```
Win → bet 10 → Win → bet 10 → Lose → bet 20 → Lose → bet 40 → Win → bet 10
```

**Fibonacci** (10 APE base):
```
Lose → bet 10 → Lose → bet 10 → Lose → bet 20 → Lose → bet 30 → Win → bet 10
```

### Safety Options

```bash
--max-bet <ape>    # Cap maximum bet (prevents runaway martingale)
--stop-loss <ape>  # Stop if balance drops too low
```

**Recommended:** Always use `--max-bet` with progressive strategies.

---

## Blackjack

Interactive or auto-play blackjack with optimal strategy.

### Quick Play (Auto)

```bash
apechurch blackjack 10 --auto              # Single game, optimal play
apechurch blackjack 10 --auto --loop       # Continuous auto-play
apechurch blackjack 10 --auto --loop --max-games 20 --bet-strategy martingale
```

### Interactive Play

```bash
apechurch blackjack 10   # Prompts for each decision
```

### Actions

| Action | When Available |
|--------|----------------|
| Hit | Always (unless bust/stand) |
| Stand | Always |
| Double | First two cards only |
| Split | Pair of same value |
| Insurance | Dealer shows Ace |
| Surrender | First action only |

### Auto Strategy

The `--auto` flag uses mathematically optimal basic strategy:
- Considers your hand value (hard/soft)
- Considers dealer's upcard
- Makes statistically best decision

### Managing Games

```bash
apechurch blackjack resume     # Resume unfinished game
apechurch blackjack status     # Check active games
apechurch blackjack clear      # Clear stuck games
```

---

## Video Poker

Jacks or Better video poker with optimal hold strategy.

### Quick Play (Auto)

```bash
apechurch video-poker 10 --auto              # Single game
apechurch video-poker 10 --auto --loop       # Continuous
apechurch vp 10 --auto --loop --max-games 50 # Using alias
```

### Bet Amounts

Video poker uses fixed denominations: **1, 5, 10, 25, 50, 100 APE**

### Hand Rankings

| Hand | Payout |
|------|--------|
| Royal Flush | Jackpot (max bet) or 800x |
| Straight Flush | 50x |
| Four of a Kind | 25x |
| Full House | 9x |
| Flush | 6x |
| Straight | 4x |
| Three of a Kind | 3x |
| Two Pair | 2x |
| Jacks or Better | 1x |

### Managing Games

```bash
apechurch video-poker resume   # Resume unfinished game
apechurch video-poker status   # Check active games
apechurch video-poker clear    # Clear stuck games
apechurch video-poker payouts  # Show payout table
```

**Aliases:** `vp`, `gimboz-poker`

---

## Commands Reference

### Core Commands

| Command | Description |
|---------|-------------|
| `apechurch install` | Setup wallet and register |
| `apechurch status` | Check balance, address, settings |
| `apechurch play [game] [amount]` | Play a game |
| `apechurch blackjack <amount>` | Play blackjack |
| `apechurch video-poker <amount>` | Play video poker |
| `apechurch games` | List all games |
| `apechurch game <name>` | Detailed game info |
| `apechurch history` | Recent game history |
| `apechurch pause` | Stop autonomous play |
| `apechurch resume` | Resume play |

### Wallet Commands

| Command | Description |
|---------|-------------|
| `apechurch wallet export` | Show private key |
| `apechurch wallet encrypt` | Password protect wallet |
| `apechurch wallet decrypt` | Remove password |
| `apechurch wallet unlock` | Start session (if encrypted) |
| `apechurch send APE <amount> <address>` | Send APE |
| `apechurch send GP <amount> <address>` | Send Gimbo Points |

### Profile Commands

| Command | Description |
|---------|-------------|
| `apechurch profile show` | View settings |
| `apechurch profile set --persona <type>` | Change play style |
| `apechurch register --username <name>` | Change username |

### Global Flags

| Flag | Description |
|------|-------------|
| `--json` | Machine-readable JSON output |
| `--loop` | Continuous play mode |
| `--delay <sec>` | Delay between games |
| `--target <ape>` | Stop at target balance |
| `--stop-loss <ape>` | Stop at loss limit |
| `--max-games <n>` | Stop after N games |
| `--bet-strategy <name>` | Betting strategy |
| `--max-bet <ape>` | Maximum bet cap |
| `--bonus-buy` | Buy bonus round (Blizzard Blitz, GOTG) |
| `--multiplier <m>` | Speed Crash target multiplier |

---

## JSON Output Schemas

All commands support `--json` for machine-readable output.

### Status Response

```json
{
  "address": "0x1234...abcd",
  "balance": "52.4500",
  "available_ape": "51.4500",
  "gas_reserve_ape": "1.0000",
  "gp_balance": "150",
  "house_balance": "0",
  "paused": false,
  "persona": "balanced",
  "username": "MY_AGENT",
  "can_play": true
}
```

`gp_balance` reads from the v2 GP token contract (`0x0382...0b93`). Anything held on the deprecated v1 contract (`0x8046...78CD`) is not surfaced.

### Play Response (standard game)

```json
{
  "status": "complete",
  "game": "ape-strong",
  "tx": "0xabc123...",
  "game_url": "https://www.ape.church/games/ape-strong?id=...",
  "wager_ape": "10.000000",
  "config": {
    "range": 50,
    "winChance": "50%",
    "approxPayout": "1.95x"
  },
  "result": {
    "buy_in_wei": "10000000000000000000",
    "buy_in_ape": "10",
    "payout_wei": "19500000000000000000",
    "payout_ape": "19.5",
    "won": true,
    "pnl_ape": "9.500000"
  }
}
```

### Play Response (Speed Crash — adds crash detail)

```json
{
  "status": "complete",
  "game": "speed-crash",
  "tx": "0x...",
  "game_url": "...",
  "wager_ape": "10.000000",
  "config": {
    "multiplier": 2.5,
    "target": "2.5x",
    "approxHitChance": "39.6%"
  },
  "result": {
    "buy_in_ape": "10",
    "payout_ape": "0",
    "target_multiplier": "2.5x",
    "crash_multiplier": "1.42x",
    "hit": false,
    "won": false,
    "pnl_ape": "-10.000000"
  }
}
```

### Play Response (cascading slots — includes executor fee)

```json
{
  "status": "complete",
  "game": "blizzard-blitz",
  "tx": "0x...",
  "wager_ape": "25.000000",
  "config": {
    "spins": 10,
    "bonusBuy": false
  },
  "result": {
    "buy_in_ape": "25",
    "payout_ape": "37.5",
    "vrf_fee_ape": "0.045",
    "executor_fee_ape": "0.4",
    "total_value_ape": "25.445",
    "won": true,
    "pnl_ape": "12.500000"
  }
}
```

### Error Response

```json
{
  "error": "Reel Pirates requires ≥ 2.5 APE per spin. For 10 spins you need at least 25.00 APE total. Either increase the wager or reduce --spins."
}
```

Cascading-slot games and Speed Crash validate inputs **before** any RPC call — invalid bets surface clear errors with no gas spent.

---

## Agent Play Patterns

### Pattern 1: Simple Loop with Safety

Best for hands-off autonomous play.

```bash
apechurch play --loop --target 200 --stop-loss 50 --max-games 500
```

The agent will:
- Play random games with balanced strategy
- Stop if balance reaches 200 APE (profit target)
- Stop if balance drops to 50 APE (loss limit)
- Stop after 500 games maximum

### Pattern 2: Specific Game Grinding

Target a specific game with fixed parameters.

```bash
apechurch play ape-strong 10 50 --loop --target 150 --stop-loss 80
```

### Pattern 3: Martingale Recovery

Progressive betting to recover losses.

```bash
apechurch play roulette 5 RED --loop --bet-strategy martingale --max-bet 50 --stop-loss 20
```

**Important:** Always set `--max-bet` with martingale to prevent exponential loss.

### Pattern 4: Crash Hunting

Hunt big multipliers on Speed Crash.

```bash
apechurch play speed-crash 5 10 --loop --max-games 50 --stop-loss 200
```

The result includes `crash_multiplier` so the agent can adapt strategy based on observed crash distribution.

### Pattern 5: Cascading-Slot Bonus Buy

Buy directly into the Blizzard Blitz bonus round.

```bash
apechurch play blizzard-blitz 100 --bonus-buy --json
```

Requires ≥ 100 APE wager. Returns same result schema as a normal play.

### Pattern 6: Session-Based Play

Play a fixed number of games per session.

```bash
apechurch play --loop --max-games 20
```

### Pattern 7: Blackjack Grinding

Auto-play blackjack with optimal strategy.

```bash
apechurch blackjack 10 --auto --loop --max-games 50 --target 200
```

### Pattern 8: Check Before Play

Always verify state before starting:

```bash
# Check if can play
apechurch status --json | jq '.can_play'

# Check balance
apechurch status --json | jq '.available_ape'

# Then play
apechurch play --loop --max-games 10
```

### Pattern 9: Handle Pause/Resume

```bash
# Human says "stop gambling"
apechurch pause

# Human says "you can play again"
apechurch resume

# Check state
apechurch status --json | jq '.paused'
```

---

## Costs & Limits

### Transaction Costs

| Cost | Amount | Notes |
|------|--------|-------|
| Gas per game | ~0.02-0.2 APE | Varies by game complexity |
| VRF fee | ~0.01-0.1 APE | Randomness oracle cost |
| Executor fee | 0.04-0.05 APE/spin | Cascading slots only (Reel Pirates, Blizzard Blitz, GOTG) |
| Gas reserve | 1 APE | Always kept in wallet |

For cascading slots, total tx value = wager + VRF fee + (spins × executor fee). All three are sent in one transaction — the CLI handles this transparently.

### Minimum Bets

| Game | Minimum |
|------|---------|
| Most games | 1 APE |
| Reel Pirates | 2.5 APE/spin (e.g. 10 spins = 25 APE) |
| Blizzard Blitz | 2.5 APE/spin (or 100 APE for `--bonus-buy`) |
| Gimboz Of The Galaxy | 3 APE/spin (or 100 APE for `--bonus-buy`) |
| Speed Crash | 1 APE (multiplier must be ≥ 1.01x) |
| Video Poker | 1 APE (fixed denominations) |

### Recommended Starting Balance

- **Minimum:** 30 APE (covers single bets on cascading slots)
- **Recommended:** 100+ APE (for variety + bonus buys)
- **For martingale:** 200+ APE
- **For Blizzard Blitz / GOTG bonus buys:** 100+ APE per buy

---

## Security

Your private key is stored at `~/.apechurch/wallet.json`.

⚠️ **CRITICAL:**
- Never share your private key
- Never paste it into prompts or third-party tools
- The CLI handles all signing locally
- Consider using `wallet encrypt` for password protection

---

## Updates

```bash
npm update -g @ape-church/skill
apechurch --version
```

---

## Links

- **Website:** https://ape.church
- **Games:** https://ape.church/games
- **GitHub:** https://github.com/ape-church/agent-skills
- **npm:** https://www.npmjs.com/package/@ape-church/skill

---

*Built for apes, agents, and degens.* 🦍

---
> Source: [ape-church/agent-skills](https://github.com/ape-church/agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
