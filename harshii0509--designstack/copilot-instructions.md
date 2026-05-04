## ds-principles

> designStack four principles — always-on behavioral rules for Cursor


# designStack — Design Principles

Apply these four principles to every conversation that involves visual, layout, or CSS changes.

## 1. Think in references, not descriptions

When a user says something vague like "make it look better", "clean it up", or "make it more modern" — **ask for a reference** before doing anything. A URL, a screenshot, or a specific element ("the button on the checkout page") is worth a thousand adjectives.

Don't interpret vague adjectives. Ask: "Can you show me what you mean, or point to a specific element?"

## 2. Summarize before changing

Before editing any UI file, CSS, or component, write **one sentence** describing exactly what you're about to change and why. Wait for the user to say yes before making the change.

Example:
> "I'm going to change the button background from `#333` to `#000` to match your brand color. OK?"

## 3. Read the Design Bible before touching visuals

If `design/DESIGN-BIBLE.md` exists in this project, **read it before making any visual change**. Apply its color, font, spacing, and component rules. If a value you're about to use isn't in the Bible, say so.

If the Bible doesn't exist, mention it once:
> "I don't see a Design Bible for this project. Run `/ds:context` in Claude Code to set one up — it makes every future session design-aware."

## 4. Flag drift, don't silently fix

If you notice a value that doesn't match the Design Bible (wrong color, off-brand font size, inconsistent spacing), **flag it** before changing it. Don't fix it silently — the user needs to know it was wrong so they can decide whether to fix everything or just this instance.

---

*These rules are from [designStack](https://github.com/harshii0509/designStack) — the AI design companion for designers building their own products.*

---
> Source: [harshii0509/designStack](https://github.com/harshii0509/designStack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
