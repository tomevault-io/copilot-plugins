## ultima-headless-cli

> A headless Ultima Online client that speaks the full UO network protocol with no window, no rendering, and no audio. Designed to be controlled by an agent (human or AI) through a CLI or daemon interface.

# ClassicUO Headless Client — Agent Guide

A headless Ultima Online client that speaks the full UO network protocol with no window, no rendering, and no audio. Designed to be controlled by an agent (human or AI) through a CLI or daemon interface.

---

## Quick Start

### 1. Configure credentials

Edit `headless/.env`:

```env
UO_HOST=login.uorenaissance.com
UO_PORT=2593
UO_USER=youraccount
UO_PASS=yourpassword
UO_VERSION=5.0.8.3
UO_SERVER=0        # auto-select server index (comment out to pick manually)
UO_CHAR=0          # auto-select character index (comment out to pick manually)
```

### 2. Build

```bash
cd headless
dotnet build
```

### 3. Run — Interactive mode

```bash
dotnet run
```

You get a `cuo>` prompt. The client auto-connects and logs in using `.env`. Type `help` for all commands.

### 4. Run — Daemon mode (recommended for agents)

```bash
dotnet run -- --daemon &
```

- Commands go to: `echo "command" >> /tmp/cuocmd`
- All output goes to: `tail -f /tmp/cuolog`
- The session stays alive. Send commands at any time without restarting.

```bash
# Send a command
echo "status" >> /tmp/cuocmd

# Watch output live
tail -f /tmp/cuolog

# Read last N lines
tail -20 /tmp/cuolog
```

---

## Command Reference

### Login flow (manual, if UO_SERVER / UO_CHAR not set)

| Command | Description |
|---------|-------------|
| `connect <host> <port> <user> <pass>` | Connect to a server |
| `servers` | List available shards after connecting |
| `server <index>` | Select a shard by index |
| `chars` | List character slots |
| `char <index>` | Enter the world with that character |

---

### Character info

| Command | Aliases | Description |
|---------|---------|-------------|
| `status` | `stat` | Full stats: HP, mana, stamina, STR/DEX/INT, gold |
| `pos` | `position` | Current X, Y, Z, direction, map index |
| `world` | `info` | Login state, map, mobile/item counts |
| `skills` | `sk` | All skills with value and lock status (↑up / ↓dn / ■locked) |
| `inv` | `inventory`, `equip` | Equipped items (by layer) + backpack contents |

---

### Movement

| Command | Description |
|---------|-------------|
| `walk <dir>` | Walk one tile. Directions: `n s e w ne nw se sw` (or full names) |
| `walk <dir> run` | Run instead of walk |

**Examples:**
```
walk n
walk ne run
walk southwest
```

Position updates automatically after each confirmed step.

---

### Combat

| Command | Aliases | Description |
|---------|---------|-------------|
| `war` | `warmode` | Toggle war/peace mode |
| `attack <serial>` | `atk`, `a` | Attack a specific mobile. **Only works on grey/red targets.** Blue (Innocent) targets are refused. |
| `attacknearest` | `an` | Attack nearest grey/red mobile with clear **line of sight** |
| `nearest` | `nearestmob` | Show the closest mobile (name, serial, distance) |

**Important:** `attacknearest` (`an`) automatically:
1. Filters to CanBeAttacked / Criminal / Enemy / Murderer only
2. Checks line of sight via map data
3. Skips targets behind walls

---

### Interaction

| Command | Aliases | Description |
|---------|---------|-------------|
| `click <serial>` | | Single-click (request name/status bar) |
| `use <serial>` | `dclick` | Double-click (open container, use item, open door) |
| `wear <serial> <layer_hex>` | | Equip an item to a body layer (lift + wear) |
| `unequip <serial>` | `topack` | Move an equipped item into the backpack |

**Layer hex values:**
```
01 = One-handed weapon    02 = Two-handed weapon (bows go here)
03 = Shoes                04 = Pants / Legs
05 = Shirt / Chest        06 = Hat / Helm
07 = Gloves               08 = Ring
09 = Talisman             0A = Neck
0B = Ring (alt)           0C = Earrings
0D = Cloak                0E = Belt
0F = Tunic                10 = Robe
11 = Boots                12 = Skirt
13 = Robe (alt)           14 = Misc
15 = Backpack             16 = Mount
```

**Example — equip crossbow:**
```
use 401F10E1        # open backpack to load inventory
inv                 # find crossbow serial
unequip 401F10E4   # remove melee weapon from slot 1
wear 401F10E6 02   # equip crossbow to two-handed slot
```

---

### Speech

| Command | Aliases | Description |
|---------|---------|-------------|
| `say <text>` | `s` | Say in public chat |
| `yell <text>` | `y` | Yell (heard further away) |
| `emote <text>` | `em` | `*emote*` action |
| `whisper <text>` | `wh` | Whisper (private, close range) |

Server commands use the same `say` interface:
```
say [help
say [skills
say [young
say [where
```

---

### World / nearby entities

| Command | Aliases | Description |
|---------|---------|-------------|
| `mobiles [range]` | `mobs`, `m` | List nearby mobiles with name, serial, notoriety, HP, distance. Default range: 18 |
| `items [range]` | `i` | List nearby ground items with serial, graphic, amount |
| `nearest` | | Closest mobile |

**Notoriety meanings:**
- `Innocent` — blue, attacking makes you criminal
- `CanBeAttacked` — grey, safe to attack
- `Criminal` — grey, already flagged
- `Enemy` — orange, attackable
- `Murderer` — red, attackable
- `Invulnerable` — cannot be harmed

---

### Map & pathfinding

Requires map data to be loaded first.

| Command | Description |
|---------|-------------|
| `loadmap [path]` | Load UO map files. Default path: `UO_MAP_DIR` from `.env`, or `uodata/` |
| `canwalk <x> <y>` | Check if a tile is walkable |
| `path <tx> <ty>` | Find A* path from current position to target. Shows list of directions |
| `los` | Line-of-sight check to all nearby mobiles. Shows YES/NO per target |

**Example workflow:**
```
loadmap
los                         # see which nearby mobs have clear LOS
path 1500 1700              # get directions to walk there
canwalk 4413 1155           # verify a tile is walkable
```

---

### Debugging / logging

| Command | Description |
|---------|-------------|
| `verbose` | Enable trace-level logging (shows all packets sent/received) |
| `quiet` | Suppress info-level logs |
| `disconnect` | Disconnect from server |
| `quit` / `exit` | Exit the client |

---

## Daemon mode workflow (agent usage)

```bash
# Start daemon (stays alive, logs everything)
cd headless
dotnet run -- --daemon &

# Wait for login (~15s), then interact
sleep 16

# Load map data for pathfinding and LOS
echo "loadmap" >> /tmp/cuocmd
sleep 10

# Check state
echo "status" >> /tmp/cuocmd && sleep 0.5 && tail -5 /tmp/cuolog
echo "skills" >> /tmp/cuocmd && sleep 0.5 && tail -20 /tmp/cuolog

# Check line of sight before attacking
echo "los" >> /tmp/cuocmd && sleep 0.5 && tail -20 /tmp/cuolog

# Attack nearest grey/red target with clear LOS
echo "an" >> /tmp/cuocmd && sleep 0.5 && tail -3 /tmp/cuolog

# Walk toward mobs
echo "walk n" >> /tmp/cuocmd && sleep 0.5

# Check for skill gains (printed automatically when they happen)
grep "SKILL" /tmp/cuolog | tail -10
```

---

## Game state in real time

The daemon logs key events automatically:

```
[INFO ] [SKILL] Archery: 2.1 ↑        ← skill gain
[INFO ] War mode: True                  ← war mode change

System: Welcome to UO Renaissance...   ← server messages
passenger: Hello!                       ← speech heard in range
*** Entered the world! ***              ← character entered world
```

Monitor with:
```bash
tail -f /tmp/cuolog                     # live stream
grep "SKILL" /tmp/cuolog               # all skill gains
grep "System:" /tmp/cuolog             # server messages
grep "Archery" /tmp/cuolog             # archery-specific events
```

---

## Architecture overview

```
headless/
├── .env                    # credentials (gitignored)
├── Program.cs              # CLI + daemon loop
├── HeadlessClient.cs       # public API: walk, attack, say, etc.
│
├── Network/
│   ├── UoClient.cs         # TCP connection, send/recv loop
│   ├── PacketsTable.cs     # packet ID → length table
│   ├── CircularBuffer.cs   # byte ring buffer
│   ├── Huffman.cs          # server→client decompression
│   └── Encryption/         # Blowfish / Twofish / LoginCrypt
│
├── Packets/
│   ├── IncomingPackets.cs  # packet router + all handlers
│   └── OutgoingPackets.cs  # packet builders (attack, walk, say…)
│
├── World/
│   └── GameWorld.cs        # PlayerInfo, MobileInfo, ItemInfo, SkillInfo
│
└── Map/
    ├── MapData.cs          # terrain + statics loader, walkability, LOS
    ├── TileData.cs         # tiledata.mul parser + TileFlag enum
    ├── Pathfinder.cs       # A* pathfinder
    ├── MapManager.cs       # singleton façade
    └── MobNames.cs         # body graphic ID → mob name dictionary
```

### Key packet handlers

| Packet | Direction | Description |
|--------|-----------|-------------|
| `0x1B` | ← server | Enter world (login confirm) |
| `0xA8` | ← server | Server list |
| `0x8C` | ← server | Relay to game server |
| `0xA9` | ← server | Character list |
| `0x20` | ← server | Player position update |
| `0x77` | ← server | Mobile moved |
| `0x78` | ← server | Object/mobile appeared |
| `0x11` | ← server | Status bar (HP/stats) |
| `0x2D` | ← server | Mobile attributes (HP/mana/stam) |
| `0x3A` | ← server | Skill update |
| `0x2E` | ← server | Item equipped on mobile |
| `0x3C` | ← server | Container contents |
| `0x25` | ← server | Single item added to container |
| `0x1C` | ← server | ASCII speech |
| `0xAE` | ← server | Unicode speech |
| `0x21` | ← server | Walk denied (resync position) |
| `0x22` | ← server | Walk confirmed (update position) |
| `0x02` | → server | Walk request |
| `0x05` | → server | Attack request |
| `0xAD` | → server | Speech |
| `0x06` | → server | Double-click |
| `0x09` | → server | Single-click |
| `0x07` | → server | Pick up item |
| `0x08` | → server | Drop item |
| `0x13` | → server | Wear/equip item |
| `0x72` | → server | Toggle war mode |

---

## Serials

All UO entities have a unique 32-bit serial number shown as hex:

```
< 0x40000000  →  mobile (player or NPC)
≥ 0x40000000  →  item
```

Serials appear in `mobiles` and `items` output as `[XXXXXXXX]`. Use them with `attack`, `click`, `use`, `wear`, `unequip`.

```bash
# Example: attack the first mobile in the list
echo "mobiles" >> /tmp/cuocmd
sleep 1
# See: [00012F0D] Dog  →  then:
echo "attack 12F0D" >> /tmp/cuocmd
```

---

## Notes for agents

- **Always check LOS before attacking** with ranged weapons (`los` command or `an` which checks automatically)
- **Skill lock status** shown in `skills` output: `↑up` = gaining, `■locked` = cannot gain
- **Crossbow swing timer** is ~6 seconds — don't send attacks faster than that
- **Encryption** is off for UO Renaissance (`useEncryption = false` in HeadlessClient.cs)
- **Map data** must be loaded with `loadmap` for pathfinding and LOS to work
- **Daemon PID**: kill with `pkill -f cuoheadless`

---
> Source: [johnpwrs/ultima-headless-cli](https://github.com/johnpwrs/ultima-headless-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
