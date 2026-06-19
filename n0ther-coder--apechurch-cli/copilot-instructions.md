## apechurch-cli

> Autonomous gambling skill for ApeChain. Play casino games, manage bankroll, compete in contests.


# Ape Church CLI 🦍🎰

> Summary: Agent-facing manual for the Ape Church CLI skill bundle. Documents commands, game syntax, automation patterns, JSON schemas, bankroll constraints, and recommended operating flows.

**Fully on-chain, decentralized casino on ApeChain.**

Every bet is placed and settled on-chain via smart contracts. Provably fair with Chainlink VRF randomness. No servers, no trust required.

---

## Table of Contents

1. [Quick Start](#quick-start)
2. [History Download & Reporting](#history-download--reporting)
3. [All Games](#all-games)
4. [Loop Mode & Automation](#loop-mode--automation)
5. [Betting Strategies](#betting-strategies)
6. [Blackjack](#blackjack-)
7. [Video Poker](#video-poker)
8. [Commands Reference](#commands-reference)
9. [JSON Output Schemas](#json-output-schemas)
10. [Agent Play Patterns](#agent-play-patterns)
11. [Costs & Limits](#costs--limits)

---

## Quick Start

```bash
# Install
npm install -g @n0ther/apechurch-cli

# Optional for non-interactive local signing
export APECHURCH_CLI_PASS=your-local-password

# Optional to override the username/profile API
export APECHURCH_CLI_PROFILE_URL=https://www.ape.church/api/profile

# Fresh install/import prompts securely for the private key
apechurch-cli install --username MY_AGENT

# Fund wallet with APE on ApeChain (address shown after install)
# Bridge: https://relay.link/bridge/apechain

# Check status
apechurch-cli status

# Download wallet history from chain data
apechurch-cli wallet download

# Read cached history stats
apechurch-cli history --stats

# Play one random game/config automatically
apechurch-cli play --auto

# Play continuously
apechurch-cli play --loop
```

On a fresh install/reinstall, `apechurch-cli install` prompts securely for the private key with hidden input unless `~/.apechurch-cli/wallets/current.json` already points to an encrypted `wallets/<address>.json` entry. `APECHURCH_CLI_PK` remains an optional non-interactive fallback, `APECHURCH_CLI_PASS` is required for non-interactive install/signing, and `APECHURCH_CLI_PROFILE_URL` overrides the default username/profile API.

---

## History Download & Reporting

Use `wallet download` to reconstruct supported gaming history directly from ApeChain into a local per-wallet cache, then read it with `history`.

If `[address]` is omitted, both commands use the local wallet address.

```bash
# Download the local wallet history
apechurch-cli wallet download

# Download a specific address
apechurch-cli wallet download 0x1234...abcd

# Scan only a recent block range
apechurch-cli wallet download 0x1234...abcd --from-block 35000000 --to-block 35300000

# Rebuild the local history file from genesis
apechurch-cli wallet download 0x1234...abcd --from-block 0

# JSON output for automation
apechurch-cli wallet download 0x1234...abcd --json

# Read saved history and stats
apechurch-cli history 0x1234...abcd

# Show more than the default 10 cached games
apechurch-cli history 0x1234...abcd --limit 25

# Show every cached game
apechurch-cli history 0x1234...abcd --all

# Stats only
apechurch-cli history 0x1234...abcd --stats

# Stats split by game
apechurch-cli history 0x1234...abcd --breakdown

# Weekly wAPE wagered totals with play breakdowns
apechurch-cli history 0x1234...abcd --leaderboard

# Refresh before reading
apechurch-cli history 0x1234...abcd --refresh

# Merge a full-range refresh before reading
apechurch-cli history 0x1234...abcd --refresh --from-block 0
```

Sync and cache behavior:

- `wallet download` is incremental by default. Without `--from-block`, it resumes from `last_synced_block + 1`.
- Use `wallet download --from-block 0` to rebuild the local history file from scratch, or pass an explicit historical range to fill older blocks.
- Explicit backfills and `history --refresh` are merged into the local file and deduplicated by `contract + game_id`.
- `history --refresh` runs the same sync path before reading the local file, but it does not clear cached records first.
- `history` shows `👀 Recent Games` plus `📜 History Stats` by default. `--stats` suppresses the game list, while `--breakdown` appends the same stats split by game.
- Standard `history` output also includes a compact `🎮 Game Status` section with per-game `played`, `net`, `win rate`, `RTP`, and local `unfinished` counts when available.

### Report Fields

| Field | Meaning |
|------|---------|
| `🎰 Games` | Economically synced games included in totals |
| `💸 Contract fees paid` | Contract-side fees effectively paid by the wallet |
| `⛽️ Gas paid` | Network gas effectively paid by the wallet |
| `Net result` | `payout - wager - contract fees - gas` |
| `✌️ Win rate` | Wins divided by economically synced games |
| `🎲 RTP` | `total payout / total wagered` |
| `🎟️  APE Wagered (wAPE)` | Current on-chain balance / total APE wagered by synced games |
| `🧮 Gimbo Points (GP)` | Current on-chain balance / total received from synced games; every `10,000 GP` equals `1 Level` |

### Wallet Download Options

| Option | Description |
|------|---------|
| `wallet download --from-block <n>` | Start block for the sync; `--from-block 0` rebuilds the history file |
| `wallet download --to-block <n>` | End block for the sync (default: latest block) |
| `wallet download --chunk-size <n>` | Block span per log query (default: `50000`) |
| `wallet download --json` | Machine-readable download report |

### History Options

| Option | Description |
|------|---------|
| `history --limit <n>` | Number of recent cached games to show (default: `10`) |
| `history --all` | Show all cached games instead of the recent slice |
| `history --stats` | Show only history stats |
| `history --breakdown` | Show history stats split by game |
| `history --leaderboard` | Show global and weekly wAPE wagered totals grouped from Sunday 00:00 UTC |
| `history --refresh` | Merge an on-chain sync before rendering |
| `history --from-block <n>` | Start block for `--refresh` |
| `history --to-block <n>` | End block for `--refresh` |
| `history --chunk-size <n>` | Block span per log query for `--refresh` |
| `history --json` | Full machine-readable local report |

### Coverage Limits

- Downloaded histories live under `~/.apechurch-cli/history/<wallet>_history.json`.
- Economic totals only include games whose wager, payout, fees, gas, and GP can be reconstructed exactly from on-chain data. Total wAPE wagered uses the canonical `wager_wei`.
- Enumerates the supported single-transaction games in the local registry via indexed `GameEnded(user, ...)` logs.
- `Blackjack` and `Video Poker` cannot yet be generically enumerated from raw RPC, so locally-known entries remain minimal until a reliable fetch path is implemented.
- Sponsored transactions contribute `0` contract fees and `0` gas for the analyzed wallet.

---

## All Games

### Quick Reference

| Game | Command | Type | Key Parameters |
|------|---------|------|----------------|
| ApeStrong ✔︎ | `play ape-strong 10 --cover 50` | Dice | `--cover 5-95` |
| Roulette ✔︎ | `play roulette 10 RED` | Table | `--bet RED,BLACK,0-36,00` |
| Baccarat ✔︎ | `play baccarat 10 BANKER` | Table | `--bet PLAYER,BANKER,TIE` |
| Jungle Plinko ✔︎ | `play jungle 10 2 50` | Plinko | `--mode 0-4` `--balls 1-100` |
| Cosmic Plinko ✔︎ | `play cosmic 10 1 10` | Plinko | `--mode 0-2` `--balls 1-30` |
| Keno ✔︎ | `play keno 10` | Keno | `--picks 1-10` `--numbers 1-40` |
| Speed Keno ✔︎ | `play speed-keno 10` | Keno | `--picks 1-5` `--games 1-20` |
| Dino Dough ✔︎ | `play dino-dough 10 10` | Slots | `--spins 1-15` |
| Bubblegum Heist ✔︎ | `play bubblegum-heist 10 10` | Slots | `--spins 1-15` |
| Geez Diggerz ✔︎ | `play geez-diggerz 10 10` | Slots | `--spins 1-15` |
| Sushi Showdown ✔︎ | `play sushi-showdown 10 10` | Slots | `--spins 1-15` |
| Reel Pirates | `play reel-pirates 20` | Cascade Slots | `--spins 1-15` |
| Monkey Match ✔︎ | `play monkey-match 10` | Match | `--mode 1-2` |
| Bear-A-Dice ✔︎ | `play bear-dice 10` | Dice | `--difficulty 0-4` `--rolls 1-5` |
| Primes ✔︎ | `play primes 10 0 20` | Number | `--difficulty 0-3` `--runs 1-20` |
| Blackjack ✔︎ | `blackjack 10 --auto` | Cards | Interactive or `--auto` |
| Video Poker ✔︎ / Gimboz Poker | `video-poker 10 --auto` | Cards | Interactive or `--auto` |

---

### ApeStrong ✔︎ (Dice)

Pick your win probability. Roll under your number to win.

```bash
apechurch-cli play ape-strong <amount> --cover <cover>
apechurch-cli play ape-strong 10 --cover 50      # 50% chance, 1.95x payout
apechurch-cli play ape-strong 10 --cover 25      # 25% chance, 3.9x payout
apechurch-cli play ape-strong 10 --cover 75      # 75% chance, 1.2999x payout
```

| Range | Win Chance | Payout |
|-------|------------|--------|
| 5 | 5% | 19.5x |
| 25 | 25% | 3.9x |
| 50 | 50% | 1.95x |
| 75 | 75% | 1.2999x |
| 95 | 95% | 1.025x |

**Aliases:** `strong`, `dice`, `limbo`
**ABI verified:** `true`

---

### Roulette

American roulette with 0, 00, and 1-36.

```bash
apechurch-cli play roulette <amount> <bet>
apechurch-cli play roulette 10 RED        # Color bet (2.05x)
apechurch-cli play roulette 10 17         # Single number (36.9x)
apechurch-cli play roulette 10 RED,BLACK  # Split bet (hedge)
apechurch-cli play roulette 10 0          # Zero (36.9x)
apechurch-cli play roulette 10 00         # Double zero (36.9x)
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

### Baccarat ✔︎

Classic baccarat. Bet on Player, Banker, or Tie.

```bash
apechurch-cli play baccarat <amount> <bet>
apechurch-cli play baccarat 50 BANKER           # Single bet
apechurch-cli play baccarat 50 PLAYER           # Single bet
apechurch-cli play baccarat 150 140 BANKER 10 TIE  # Combined: 140 on Banker, 10 on Tie
```

| Bet | Payout |
|-----|--------|
| PLAYER | 2.0x |
| BANKER | 1.95x |
| TIE | 9.0x |

**Combined bets:** Specify explicit amounts (must sum to total wager).

**Alias:** `bacc`
**ABI verified:** `true`

---

### Jungle Plinko ✔︎

Drop balls through pegs into multiplier buckets.

```bash
apechurch-cli play jungle <amount> <mode> <balls>
apechurch-cli play jungle 10 2 50           # 10 APE, mode 2, 50 balls
apechurch-cli play jungle 50 4 100          # High risk, max balls
```

| Parameter | Range | Description |
|-----------|-------|-------------|
| mode | 0-4 | Risk level (0=safe, 4=extreme) |
| balls | 1-100 | Ball count (wager split across balls) |

**Alias:** `jungle`
**ABI verified:** `true`

---

### Cosmic Plinko ✔︎

Drop biased balls through pegs into asymmetric multiplier buckets.

```bash
apechurch-cli play cosmic <amount> <mode> <balls>
apechurch-cli play cosmic 10 1 10           # 10 APE, mode 1, 10 balls
apechurch-cli play cosmic 25 2 30           # High risk, max balls
```

| Parameter | Range | Description |
|-----------|-------|-------------|
| mode | 0-2 | Risk level (0=low, 2=high) |
| balls | 1-30 | Ball count (wager split across balls) |

**Alias:** `cosmic`
**ABI verified:** `true`

---

### Keno ✔︎

Pick numbers, hope they hit.

**ABI verified:** `true`

```bash
apechurch-cli play keno <amount> [--picks N] [--numbers X,Y,Z]
apechurch-cli play keno 10                     # Random 5 picks
apechurch-cli play keno 10 --picks 10          # 10 random picks
apechurch-cli play keno 10 --picks 5 --numbers 1,7,13,25,40
```

| Parameter | Range | Description |
|-----------|-------|-------------|
| picks | 1-10 | How many numbers to pick |
| numbers | 1-40 | Specific numbers (comma-separated) |

**Max payout:** 10/10 hits = 1,000,000x

**Alias:** `k`

---

### Speed Keno ✔︎

Fast keno with batched games.

**ABI verified:** `true`

```bash
apechurch-cli play speed-keno <amount> [--picks N] [--games N]
apechurch-cli play speed-keno 10                # 3 picks, 5 games
apechurch-cli play speed-keno 10 --picks 5 --games 20
```

| Parameter | Range | Description |
|-----------|-------|-------------|
| picks | 1-5 | Numbers to pick |
| games | 1-20 | Games to batch (wager split) |

**Max payout:** 5/5 = 2000x

**Aliases:** `speedkeno`, `skeno`, `speed`

---

### Dino Dough ✔︎, Bubblegum Heist ✔︎, Geez Diggerz ✔︎ & Sushi Showdown ✔︎ (Slots)

Verified multi-spin slot contracts with ordered left-to-right symbol triples.

```bash
apechurch-cli play dino-dough <amount> <spins>
apechurch-cli play dino-dough 10 10      # 10 APE, 10 spins
apechurch-cli play bubblegum-heist 10 15 # 10 APE, 15 spins
apechurch-cli play geez-diggerz 10 10    # 10 APE, 10 spins
apechurch-cli play sushi-showdown 10 10  # 10 APE, 10 spins
```

| Parameter | Range | Description |
|-----------|-------|-------------|
| spins | 1-15 | Spins per bet (wager split) |

**ABI verified:** `true`
**Verified runtime:** all four games decode `(gameId, numSpins, ref, userRandomWord)`, use a live `getVRFFee()` read, request `numSpins * 3` VRF words, and resolve ordered symbol triples through cumulative reel-stop tables.
**Slot-family note:** Dino Dough and Geez Diggerz are currently exact-match verified on ApeScan; Bubblegum Heist and Sushi Showdown are currently linked `Similar Match` deployments in the same verified `Slots` family.
**Exact RTP:** `97.90%` for Dino Dough, `97.80%` for Bubblegum Heist, `97.69%` for Geez Diggerz, and `97.87%` for Sushi Showdown from live reel/payout snapshots recorded in the repo.
**Reel Pirates note:** playable but not ABI verified; the observed gameplay tx uses `(numSpins, gameId, ref, userRandomWord)` and the exact cascade odds are not public in this repo.
**Current max payout:** `333x` for Dino Dough, `100x` for Bubblegum Heist, `50x` for Geez Diggerz, and `500x` for Sushi Showdown.
**Aliases:** `dinodough`, `dino`, `bubblegumheist`, `bubblegum`, `heist`, `geezdiggerz`, `geez`, `diggerz`, `sushishowdown`, `sushi`


---

### Monkey Match ✔︎

Monkeys pop from barrels — form poker hands!

```bash
apechurch-cli play monkey-match <amount> [--mode N]
apechurch-cli play monkey-match 10           # Low risk (default)
apechurch-cli play monkey-match 10 --mode 2  # Normal risk / best EV
```

| Mode | Description |
|------|-------------|
| 1 | Low Risk — 6 monkey types, easier matches |
| 2 | Normal Risk — 7 monkey types, better mid payouts |

**Max payout:** Five of a Kind = 50x

**Exact RTP:** `97.99%` in Low Risk, `98.29%` in Normal Risk

**Best-play note:** there is no redraw or post-deal decision tree; the only strategic choice is mode selection, and mode `2` has the higher exact RTP

**Aliases:** `monkey`, `mm`

---

### Bear-A-Dice ✔︎

Roll dice, avoid unlucky numbers.

**ABI verified:** `true`

```bash
apechurch-cli play bear-dice <amount> [--difficulty N] [--rolls N]
apechurch-cli play bear-dice 10                    # Easy, 1 roll
apechurch-cli play bear-dice 10 --difficulty 0 --rolls 5
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

**Verified runtime:** all five difficulties support `1-5` rolls on-chain; the local CLI no longer applies a fake `3`-roll cap to Extreme or Master.
**Settlement detail:** the contract is all-or-nothing. It stops on the first losing sum, zeroes the full payout, and leaves any remaining `dice1Results` / `dice2Results` slots at `0`, so the CLI filters those tails as "not executed".
**Exact RTP:** ranges from `97.25%` (`Normal / 5 rolls`) to `97.94%` (`Normal / 1 roll`) across the verified on-chain table.
**Exact win rate:** ranges from `83.33%` (`Easy / 1 roll`) down to `0.000053%` (`Master / 5 rolls`).
**Max payout:** up to `1,847,949.193x` at `Master / 5 rolls`
**Aliases:** `bear`, `bd`

---

## Loop Mode & Automation

Play continuously with safety controls.

### Basic Loop

```bash
apechurch-cli play --loop                    # Random games, default settings
apechurch-cli play ape-strong 10 --cover 50 --loop   # Specific game
apechurch-cli play --loop --delay 5          # 5 seconds between games
```

### Safety Controls

```bash
# Stop when balance reaches target
apechurch-cli play --loop --target 200

# Stop when balance drops to limit
apechurch-cli play --loop --stop-loss 50

# Stop after N games
apechurch-cli play --loop --max-games 100

# Combine them all
apechurch-cli play ape-strong 10 --cover 50 --loop --target 200 --stop-loss 50 --max-games 100
```

| Option | Description |
|--------|-------------|
| `--target <ape>` | Stop when balance reaches this amount |
| `--stop-loss <ape>` | Stop when balance drops to this amount; derives bankroll when no `--bankroll`/`--max-loss` is set |
| `--bankroll <ape>`, `--max-loss <ape>` | Stop when session P&L reaches this loss; derives wallet stop-loss when no `--stop-loss` is set |
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
| `bankroll-fraction=<n>` | Bet fraction `n` of remaining bankroll | Dynamic |

### Using Strategies

```bash
# Martingale with 10 APE base bet
apechurch-cli play ape-strong 10 --cover 50 --loop --bet-strategy martingale

# Martingale with safety cap
apechurch-cli play roulette 10 RED --loop --bet-strategy martingale --max-bet 100

# Fibonacci on blackjack
apechurch-cli blackjack 5 --auto --loop --bet-strategy fibonacci --max-games 50

# Bankroll fraction sizing (no explicit wager amount)
apechurch-cli play roulette --bet RED --loop --bankroll 500 --bet-strategy bankroll-fraction=0.09 --max-bet 100 --min-bet 5
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
--min-bet <ape>    # Floor dynamic bankroll-fraction bets
--stop-loss <ape>  # Stop if balance drops too low
```

**Recommended:** Always use `--max-bet` with progressive strategies.

`bankroll-fraction=<n>` requires `--bankroll`/`--max-loss` or `--stop-loss`, conflicts with an explicit starting wager, and sizes each new play from the remaining bankroll.

---

## Blackjack ✔︎

Interactive H17 blackjack with optional auto-play. The dealer hits soft 17.

**CLI aliases:** `bj`
**ABI verified:** `true`

### Quick Play (Auto)

```bash
apechurch-cli blackjack 10 --auto              # Single game, simple auto-play
apechurch-cli blackjack 10 --auto best         # Single game, exact-EV auto-play
apechurch-cli blackjack 10 --auto max          # Exact-EV auto-play with larger solver budget
apechurch-cli blackjack 10 --solver max        # Manual play with worker and simple suggestions
apechurch-cli blackjack 10 --auto best --solver-max-states 100000
apechurch-cli blackjack 10 --auto best --solver-timeout-ms 3000
apechurch-cli blackjack 10 --auto --loop       # Continuous auto-play
apechurch-cli blackjack 10 --auto --loop --max-games 20 --bet-strategy martingale
apechurch-cli bj 10 --auto --loop --max-games 20 # Using alias
```

### Interactive Play

```bash
apechurch-cli blackjack 10   # Prompts for each decision
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

`--auto` enables automatic play for decision-heavy hands:
- Considers your hand value and dealer context
- Chooses actions such as hit, stand, double, split, insurance, and surrender when available
- Models the live dealer-hits-soft-17 rule surface
- Supports loop mode and betting strategies

`--auto best` runs the exact-EV search in a worker capped by
`--solver-max-states <n>` and `--solver-timeout-ms <ms>`. Defaults are `50000`
recursive player states and `5000` ms; either guard falls back to simple mode.
`--auto max` uses the same solver with larger defaults: `150000` states and
`30000` ms. Raise the state cap for branchy hands only when you can tolerate
the extra CPU, and lower the timeout when loop latency matters more than exact
completion.

`--solver [simple|best|max]` enables manual suggestions without executing an
action. Bare `--solver` defaults to `best`; `best` and `max` show both the worker
choice and the simple-strategy choice for comparison.

Use `apechurch-cli help auto` for advanced auto-play modes and pacing controls.

### Managing Games

```bash
apechurch-cli blackjack resume     # Resume unfinished game
apechurch-cli blackjack status     # Check active games
apechurch-cli blackjack clear      # Clear stuck games
```

---

## Video Poker

Jacks or Better video poker with interactive and auto-play flows. Ape Church calls this game `Gimboz Poker`; the CLI command is `video-poker` and the alias `gimboz-poker` is supported too.

### Quick Play (Auto)

```bash
apechurch-cli video-poker 10 --auto              # Single game, simple mode
apechurch-cli video-poker 10 --solver           # Show best-EV hold suggestion
apechurch-cli video-poker 10 --auto --loop       # Continuous
apechurch-cli vp 10 --auto --loop --max-games 50 # Using alias
```

### Bet Amounts

Video poker uses fixed denominations: **1, 5, 10, 25, 50, 100 APE**

### Hand Rankings

| Hand | Payout |
|------|--------|
| Royal Flush | 250x base (+ progressive jackpot at max bet) |
| Straight Flush | 50x |
| Four of a Kind | 25x |
| Full House | 9x |
| Flush | 6x |
| Straight | 4x |
| Three of a Kind | 3x |
| Two Pair | 2x |
| Jacks or Better | 1x |

### Auto & Solver Behavior

- `--auto`
  - Enables automatic play for the draw and hold phases
  - Works with loop mode and betting strategies
- `--solver`
  - Leaves play interactive
  - Shows the best-EV hold suggestion before you choose cards to keep

Use `apechurch-cli help auto` for advanced auto-play modes and pacing controls.

### Display Modes

- `--display full`
  - Uses the boxed ASCII table renderer for video poker
- `--display simple`
  - Keeps the compact text renderer

### Managing Games

```bash
apechurch-cli video-poker resume   # Resume unfinished game
apechurch-cli video-poker status   # Check active games
apechurch-cli video-poker clear    # Clear stuck games
apechurch-cli video-poker payouts  # Show payout table
```

**CLI aliases:** `vp`, `gimboz-poker`

---

## Commands Reference

### Core Commands

| Command | Description |
|---------|-------------|
| `apechurch-cli install` | Setup wallet and register |
| `apechurch-cli status` | Check balance, address, settings |
| `apechurch-cli play [game] [amount]` | Play a game |
| `apechurch-cli blackjack|bj <amount>` | Play blackjack |
| `apechurch-cli video-poker <amount>` | Play video poker |
| `apechurch-cli wallet download [address]` | Download supported on-chain history into local cache |
| `apechurch-cli games` | List all games |
| `apechurch-cli game <name>` | Detailed game info |
| `apechurch-cli history [address]` | Read cached history, recent games, and history stats |
| `apechurch-cli pause` | Stop autonomous play |
| `apechurch-cli continue` | Resume play |

### Wallet Commands

| Command | Description |
|---------|-------------|
| `apechurch-cli wallet status` | Check encrypted wallet status |
| `apechurch-cli wallet --list` | List locally available wallet addresses |
| `apechurch-cli wallet new` | Create and select a new wallet |
| `apechurch-cli wallet select [address]` | Switch to a stored wallet |
| `apechurch-cli wallet download [address]` | Download supported on-chain history into local cache |
| `apechurch-cli wallet password` | Re-encrypt the local wallet with a new password |
| `apechurch-cli send APE <amount> <address>` | Send APE |
| `apechurch-cli send GP <amount> <address>` | Send Gimbo Points |

### Profile Commands

| Command | Description |
|---------|-------------|
| `apechurch-cli profile show` | View settings |
| `apechurch-cli profile set --persona <type>` | Change play style |
| `apechurch-cli profile set --gp-ape <points>` | Set the wallet-specific current GP/APE rate |
| `apechurch-cli profile set --no-gp-ape` | Clear the wallet-specific current GP/APE rate |
| `apechurch-cli register --username <name>` | Change username |

### Global Flags

| Flag | Description |
|------|-------------|
| `--json` | Machine-readable JSON output |
| `--loop` | Continuous play mode |
| `--delay <sec>` | Delay between games |
| `--gp-ape <points>` | Override the current run's GP per APE rate |
| `--solver` | Show the best-EV hold suggestion in interactive video poker |
| `--target <ape>` | Stop at target balance |
| `--stop-loss <ape>` | Stop at loss limit |
| `--max-games <n>` | Stop after N games |
| `--bet-strategy <name>` | Betting strategy, including `bankroll-fraction=<0..1>` |
| `--max-bet <ape>` | Maximum bet cap |
| `--min-bet <ape>` | Minimum bet floor |

---

## JSON Output Schemas

All commands support `--json` for machine-readable output. Samples below are abridged to the stable fields most agents typically consume.

### Status Response

```json
{
  "address": "0x1234...abcd",
  "balance": "52.4500",
  "available_ape": "51.4500",
  "gas_reserve_ape": "1.0000",
  "gp_balance": "150",
  "house_balance": "0.0000",
  "paused": false,
  "persona": "balanced",
  "username": "MY_AGENT",
  "gp_rate": {
    "base_gp_per_ape": 5,
    "current_gp_per_ape": 5,
    "effective_gp_per_ape": 5,
    "source": "default",
    "source_label": "default"
  },
  "can_play": true,
  "unfinished_games": []
}
```

### Play Response

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
    "payout_ape": "19.50",
    "won": true,
    "pnl_ape": "9.500000"
  }
}
```

### Wallet Download Response

```json
{
  "history": {
    "version": 2,
    "wallet": "0x1234...abcd",
    "chain_id": 33139,
    "last_synced_block": "35300000",
    "last_download_on": "2026-03-29T12:00:00.000Z",
    "games": [
      {
        "contract": "0x...",
        "game_key": "bear-dice",
        "game_id": "1137230...",
        "play_tx": "0xabc123...",
        "settlement_tx": "0xdef456...",
        "block_number": "35299999",
        "timestamp": 1710000000000,
        "wager_wei": "5000000000000000000",
        "payout_wei": "0",
        "contract_fee_wei": "100000000000000000",
        "gas_fee_wei": "20000000000000000",
        "gp_received_raw": "25",
        "gp_source": "receipt",
        "settled": true,
        "last_sync_on": "2026-03-29T12:00:00.000Z",
        "last_sync_msg": "ok"
      }
    ]
  },
  "stats": {
    "total_saved_games": 14,
    "games": 11,
    "unsynced_games": 3,
    "contract_fees_paid_ape": "0.75528483626624",
    "gas_paid_ape": "0.50494651956676",
    "gross_result_ape": "-500.60836035",
    "net_result_ape": "-500.867599619911120003",
    "win_rate": 63.6,
    "rtp": 11.1,
    "total_wape_received_ape": "35",
    "total_gp_received_display": "175"
  },
  "sync": {
    "wallet": "0x1234...abcd",
    "file_path": "/Users/me/.apechurch-cli/history/0x1234...abcd_history.json",
    "from_block": "35000000",
    "to_block": "35300000",
    "latest_block": "35300000",
    "downloaded_games": 11,
    "new_games": 11,
    "saved_games": 14,
    "missing_transaction_metadata": 0,
    "unsupported_saved_games": 3
  }
}
```

### History Response

```json
{
  "wallet": "0x1234...abcd",
  "history_file": "/Users/me/.apechurch-cli/history/0x1234...abcd_history.json",
  "meta": {
    "version": 2,
    "chain_id": 33139,
    "last_synced_block": "35300000",
    "last_download_on": "2026-03-29T12:00:00.000Z"
  },
  "stats": {
    "total_saved_games": 14,
    "games": 11,
    "unsynced_games": 3,
    "contract_fees_paid_ape": "0.75528483626624",
    "gas_paid_ape": "0.50494651956676",
    "gross_result_ape": "-500.60836035",
    "net_result_ape": "-500.867599619911120003",
    "win_rate": 63.6,
    "rtp": 11.1,
    "current_wape_balance_ape": "32306.9125",
    "total_wape_received_ape": "35",
    "current_gp_balance_display": "188777",
    "total_gp_received_display": "175"
  },
  "game_stats": [
    {
      "game": "Bear-A-Dice",
      "games_played": 6,
      "net_profit_ape": "-32.5200",
      "net_profit_complete": true,
      "wins": 2,
      "pushes": 0,
      "losses": 4,
      "win_rate": 33.33,
      "rtp": 45.8,
      "unfinished_games": 0,
      "unfinished_game_ids": []
    }
  ],
  "breakdown": [
    {
      "game": "Bear-A-Dice",
      "games": 6,
      "contract_fees_paid_ape": "0.4",
      "gas_paid_ape": "0.12",
      "net_result_ape": "-32.52"
    }
  ],
  "sync": null,
  "games": [
    {
      "game": "Bear-A-Dice",
      "gameId": "1137230...",
      "settled": true,
      "wager_ape": "5",
      "payout_ape": "0",
      "contract_fee_ape": "0.1",
      "gas_fee_ape": "0.02"
    }
  ]
}
```

### Error Response

```json
{
  "error": "Insufficient balance. Available: 5.00 APE"
}
```

---

## Agent Play Patterns

### Pattern 1: Simple Loop with Safety

Best for hands-off autonomous play.

```bash
apechurch-cli play --loop --target 200 --stop-loss 50 --max-games 500
```

The agent will:
- Play random games with balanced strategy
- Stop if balance reaches 200 APE (profit target)
- Stop if balance drops to 50 APE (loss limit)
- Stop after 500 games maximum

### Pattern 2: Specific Game Grinding

Target a specific game with fixed parameters.

```bash
apechurch-cli play ape-strong 10 --cover 50 --loop --target 150 --stop-loss 80
```

### Pattern 3: Martingale Recovery

Progressive betting to recover losses.

```bash
apechurch-cli play roulette 5 RED --loop --bet-strategy martingale --max-bet 50 --stop-loss 20
```

**Important:** Always set `--max-bet` with martingale to prevent exponential loss.

### Pattern 4: Session-Based Play

Play a fixed number of games per session.

```bash
apechurch-cli play --loop --max-games 20
```

### Pattern 5: Blackjack Grinding

Auto-play blackjack in loop mode.

```bash
apechurch-cli blackjack 10 --auto --loop --max-games 50 --target 200
```

### Pattern 6: Check Before Play

Always verify state before starting:

```bash
# Check if can play
apechurch-cli status --json | jq '.can_play'

# Check balance
apechurch-cli status --json | jq '.available_ape'

# Then play
apechurch-cli play --loop --max-games 10
```

### Pattern 7: Handle Pause/Continue

```bash
# Human says "stop gambling"
apechurch-cli pause

# Human says "you can play again"
apechurch-cli continue

# Check state
apechurch-cli status --json | jq '.paused'
```

---

## Costs & Limits

### Transaction Costs

| Cost | Amount | Notes |
|------|--------|-------|
| Gas per game | ~0.02-0.2 APE | Varies by game complexity |
| VRF fee | ~0.01-0.1 APE | Randomness oracle cost |
| Gas reserve | 1 APE | Always kept in wallet |

### Minimum Bets

| Game | Minimum |
|------|---------|
| Most games | 1 APE |
| Video Poker | 1 APE (fixed denominations) |

### Recommended Starting Balance

- **Minimum:** 20 APE
- **Recommended:** 50+ APE
- **For martingale:** 100+ APE

---

## Security

Your selected encrypted wallet is tracked by `~/.apechurch-cli/wallets/current.json`, which points to `wallets/<address>.json`. Encrypted wallet material lives in the address-specific entry; if that entry is a symlink, normal filesystem resolution applies.

⚠️ **CRITICAL:**
- Never share your private key
- Never paste it into prompts or third-party tools
- The CLI handles all signing locally
- Consider using `wallet password` periodically to rotate local password protection

---

## Updates

```bash
npm update -g @n0ther/apechurch-cli
apechurch-cli --version
```

---

## Links

- **Website:** https://ape.church
- **Games:** https://ape.church/games
- **GitHub:** https://github.com/ape-church/agent-skills
- **npm:** https://www.npmjs.com/package/@n0ther/apechurch-cli

---

*Built for apes, agents, and degens.* 🦍

---
> Source: [n0ther-coder/apechurch-cli](https://github.com/n0ther-coder/apechurch-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
