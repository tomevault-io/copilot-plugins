## dice-roller-skill

> Roll one or more six-sided dice and show the result in an animated visual webview.


# Dice Roller

## Examples

- "抛个骰子"
- "掷一下骰子"
- "roll a die"
- "roll 2 dice"
- "帮我抛三个骰子"

## Instructions

When the user asks to roll, throw, or toss dice, call the `run_js` tool with the following exact parameters:

- script name: index.html
- data: A JSON string with the following fields:
  - dice: Number. Optional. How many six-sided dice to roll. Use 1 if the user does not specify a count. Clamp to the range 1 to 6.

After the tool returns, briefly tell the user the rolled value or values. Do not invent a result yourself.

---
> Source: [GACN/dice-roller-skill](https://github.com/GACN/dice-roller-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
