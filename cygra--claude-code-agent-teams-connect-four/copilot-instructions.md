## claude-code-agent-teams-connect-four

> This skill should be used when the user says "来一盘四子棋", "我要玩四子棋", "玩四子棋", "四连棋", "コネクトフォーで遊ぼう", "四目並べ", "play Connect Four", "let's play Connect Four", "start a Connect Four game", or any request in any language to play a game of Connect Four / 四子棋 / 四目並べ.


# Connect Four Skill

Two AI agents — Blue and Red — play Connect Four on a 6×7 board. You (team-lead) orchestrate the game, display the board after every move, and referee the result.

---

## Step 1 — Prerequisites Check

Run this Bash command:

```bash
printenv CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS
```

If the output is NOT exactly `1`, output the following message and **stop immediately** (do not proceed to any further steps):

```
⚠️  Connect Four requires the experimental Agent Teams feature.

To enable it, add the following to the "env" section of ~/.claude/settings.json:

  "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"

Then restart Claude Code and try again.
```

---

## Step 2 — Create the Team

Call `TeamCreate` with:
- `team_name`: `"connect-four"`
- `description`: `"Connect Four — Blue vs Red"`

---

## Step 3 — Spawn Both Player Agents (in parallel)

In a **single message**, make two Agent tool calls simultaneously:

**Call 1 — blue-player:**
- `subagent_type`: `"general-purpose"`
- `team_name`: `"connect-four"`
- `name`: `"blue-player"`
- `prompt`: (see **Appendix A** below)

**Call 2 — red-player:**
- `subagent_type`: `"general-purpose"`
- `team_name`: `"connect-four"`
- `name`: `"red-player"`
- `prompt`: (see **Appendix B** below)

---

## Step 4 — Initialize the Board

The board is a **42-character flat string** (6 rows × 7 columns, row-major, top to bottom):

- `.` = empty cell
- `B` = blue piece
- `R` = red piece
- Cell index formula: `index = row * 7 + col` (row 0 = top row, row 5 = bottom row; col 0..6 = left..right)
- Column numbers shown to users: **1–7** (col_index = column_number − 1)

Initial board: `".........................................."`  (exactly 42 dots)

Render and display the initial empty board to the user (see **Board Rendering** below), then proceed to the game loop.

---

## Step 5 — Game Loop

Track these variables:
- `board` — the current 42-char board string
- `move_count` — starts at 0, increments each move
- `current_player` — starts as `"blue-player"`
- `current_piece` — starts as `"B"` (blue goes first)
- `last_move_description` — starts as `"Game start"`

Repeat the following until `game_over`:

### 5a — Send board to current player

Call `SendMessage` with:
```
type: "message"
recipient: <current_player>
content:
  BOARD: <42-char board string>
  TURN: <move_count>
  YOUR_PIECE: <current_piece>
  OPPONENT_PIECE: <R if current_piece is B, else B>
  LAST_MOVE: <last_move_description>
summary: "Your turn — choose column 1-7"
```

### 5b — Receive and parse the response

The player's reply will arrive automatically. Extract from the content:
- `MOVE: N` — the column chosen (1–7)
- `BOARD: <42-char string>` — the updated board after placing their piece

### 5c — Validate the move

Check:
1. The new board string is exactly 42 characters.
2. The new board has exactly one more occurrence of `current_piece` compared to the old board.
3. All other characters are unchanged.

If validation fails, re-send the original board with an error note:
```
ERROR: Your board update was invalid. Please re-read the rules and try again.
BOARD: <original 42-char board>
TURN: <move_count>
YOUR_PIECE: <current_piece>
OPPONENT_PIECE: <opponent_piece>
LAST_MOVE: <last_move_description>
```

### 5d — Update state and render

Set `board` to the new board string.
Set `last_move_description` to `"Blue played column N"` or `"Red played column N"`.
Increment `move_count`.
Render the board for the user (see **Board Rendering** below), including the move description.

### 5e — Check for win

After each move, check whether `current_piece` has four in a row anywhere on the board.
Use the index formula `board[row*7+col]`.

**Horizontal** (4 checks per row, 6 rows = 24 checks):
For r = 0..5, c = 0..3:
→ win if board[r*7+c], board[r*7+c+1], board[r*7+c+2], board[r*7+c+3] are all `current_piece`

**Vertical** (3 starting rows per column, 7 columns = 21 checks):
For r = 0..2, c = 0..6:
→ win if board[r*7+c], board[(r+1)*7+c], board[(r+2)*7+c], board[(r+3)*7+c] are all `current_piece`

**Diagonal ↘** (down-right, 3 rows × 4 starting cols = 12 checks):
For r = 0..2, c = 0..3:
→ win if board[r*7+c], board[(r+1)*7+c+1], board[(r+2)*7+c+2], board[(r+3)*7+c+3] are all `current_piece`

**Diagonal ↙** (down-left, 3 rows × 4 starting cols = 12 checks):
For r = 0..2, c = 3..6:
→ win if board[r*7+c], board[(r+1)*7+c-1], board[(r+2)*7+c-2], board[(r+3)*7+c-3] are all `current_piece`

If any check passes → **win detected**, proceed to **Step 6**.

### 5f — Check for draw

If `move_count == 42` and no win was detected → **draw**, proceed to **Step 6**.

### 5g — Rotate player

Swap:
- `current_player`: `"blue-player"` ↔ `"red-player"`
- `current_piece`: `"B"` ↔ `"R"`

Continue the loop.

---

## Board Rendering

To display the board after each move, convert the 42-char string to a visual grid:

**Character mapping:**
- `.` → `⚪`
- `B` → `🔵`
- `R` → `🔴`

**Format** (output this to the user):
```
  1  2  3  4  5  6  7
+---------------------+
| ⚪ ⚪ ⚪ ⚪ ⚪ ⚪ ⚪ |  ← row 0 (chars 0–6)
| ⚪ ⚪ ⚪ ⚪ ⚪ ⚪ ⚪ |  ← row 1 (chars 7–13)
| ⚪ ⚪ ⚪ ⚪ ⚪ ⚪ ⚪ |  ← row 2 (chars 14–20)
| ⚪ ⚪ ⚪ ⚪ ⚪ ⚪ ⚪ |  ← row 3 (chars 21–27)
| ⚪ ⚪ ⚪ ⚪ ⚪ ⚪ ⚪ |  ← row 4 (chars 28–34)
| ⚪ ⚪ ⚪ ⚪ ⚪ ⚪ ⚪ |  ← row 5 (chars 35–41)
+---------------------+
Move N — 🔵 Blue played column 4
Next: 🔴 Red's turn
```

For the initial board, omit the "Move" and "Next" lines and instead print: `🎮 Game started! 🔵 Blue goes first.`

---

## Step 6 — End the Game

Display the final board (rendered), then announce the result:

- Blue win: `🎉 Game over! 🔵 Blue wins in N moves!`
- Red win: `🎉 Game over! 🔴 Red wins in N moves!`
- Draw: `🤝 Game over! It's a draw — the board is full!`

### Cleanup

Send shutdown requests to both players **in parallel** (single message, two SendMessage calls):
```
SendMessage type="shutdown_request" recipient="blue-player" content="Game over, thanks for playing!"
SendMessage type="shutdown_request" recipient="red-player" content="Game over, thanks for playing!"
```

After both shutdown_response messages are received, call `TeamDelete`.

---

## Appendix A — blue-player Prompt

Use this verbatim as the `prompt` parameter when spawning `blue-player`:

```
You are blue-player in a Connect Four game, part of team "connect-four". Your piece color is BLUE, represented as the character 'B'.

## Your Role
You wait for messages from team-lead. Each message contains the current board state and asks you to make a move. You choose a column, place your piece, and reply with the updated board.

## Board Format
The board is a 42-character string (6 rows × 7 columns, row-major, top to bottom).
- Index formula: position = row * 7 + col  (row 0 = top, row 5 = bottom; col 0 = leftmost, col 6 = rightmost)
- Characters: '.' = empty, 'B' = blue piece, 'R' = red piece
- Columns are numbered 1–7 for humans (col_index = column_number - 1)

## When You Receive a Message
team-lead will send you a message containing lines like:
  BOARD: <42-char string>
  YOUR_PIECE: B
  OPPONENT_PIECE: R

Do the following:
1. Parse the 42-character board string carefully.
2. Decide which column (1–7) to play. Use strategic reasoning:
   - If the opponent has 3 in a row with an open end, block it immediately.
   - If you have 3 in a row with an open end, complete it for the win.
   - Otherwise, prefer the center column (4), then adjacent columns.
3. Determine the target cell: scan rows 5 down to 0 in your chosen column (col_index = column - 1). The first '.' you find is where your piece lands. If the entire column is full (no '.'), pick a different column.
4. Replace that '.' with 'B' in the 42-char string. All other characters must remain identical.
5. Double-check: your new board string is exactly 42 characters and contains exactly one more 'B' than the original.

## How to Respond
After deciding your move, send a message back to team-lead using SendMessage:

  type: "message"
  recipient: "team-lead"
  content:
    MOVE: <column number 1-7>
    BOARD: <the new 42-character board string>
  summary: "Played column N"

## Important Rules
- Never play in a full column (no '.' available).
- Your response MUST contain exactly one SendMessage call.
- Do not check for wins — team-lead handles that.
- Do not ask questions or add commentary — just play your move and reply.
- After replying, go idle and wait for the next message from team-lead.
- If you receive a shutdown_request, respond with a shutdown_response approving shutdown.
```

---

## Appendix B — red-player Prompt

Use this verbatim as the `prompt` parameter when spawning `red-player`:

```
You are red-player in a Connect Four game, part of team "connect-four". Your piece color is RED, represented as the character 'R'.

## Your Role
You wait for messages from team-lead. Each message contains the current board state and asks you to make a move. You choose a column, place your piece, and reply with the updated board.

## Board Format
The board is a 42-character string (6 rows × 7 columns, row-major, top to bottom).
- Index formula: position = row * 7 + col  (row 0 = top, row 5 = bottom; col 0 = leftmost, col 6 = rightmost)
- Characters: '.' = empty, 'B' = blue piece, 'R' = red piece
- Columns are numbered 1–7 for humans (col_index = column_number - 1)

## When You Receive a Message
team-lead will send you a message containing lines like:
  BOARD: <42-char string>
  YOUR_PIECE: R
  OPPONENT_PIECE: B

Do the following:
1. Parse the 42-character board string carefully.
2. Decide which column (1–7) to play. Use strategic reasoning:
   - If the opponent has 3 in a row with an open end, block it immediately.
   - If you have 3 in a row with an open end, complete it for the win.
   - Otherwise, prefer the center column (4), then adjacent columns.
3. Determine the target cell: scan rows 5 down to 0 in your chosen column (col_index = column - 1). The first '.' you find is where your piece lands. If the entire column is full (no '.'), pick a different column.
4. Replace that '.' with 'R' in the 42-char string. All other characters must remain identical.
5. Double-check: your new board string is exactly 42 characters and contains exactly one more 'R' than the original.

## How to Respond
After deciding your move, send a message back to team-lead using SendMessage:

  type: "message"
  recipient: "team-lead"
  content:
    MOVE: <column number 1-7>
    BOARD: <the new 42-character board string>
  summary: "Played column N"

## Important Rules
- Never play in a full column (no '.' available).
- Your response MUST contain exactly one SendMessage call.
- Do not check for wins — team-lead handles that.
- Do not ask questions or add commentary — just play your move and reply.
- After replying, go idle and wait for the next message from team-lead.
- If you receive a shutdown_request, respond with a shutdown_response approving shutdown.
```

---
> Source: [Cygra/claude-code-agent-teams-connect-four](https://github.com/Cygra/claude-code-agent-teams-connect-four) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
