## gatheros

> - After any code change, always end the reply with the full command to re-run

# GatherOS — Claude notes

## Conventions

- After any code change, always end the reply with the full command to re-run
  the app so the user can copy/paste. Claude's pushes only land in the user's
  local working tree after a `git pull`, so include that explicitly — and use
  the active feature branch (`claude/scaffold-moodmark-app-f7ET2`) so the
  command keeps working even after a branch swap:

  ```
  cd /Users/brett/GatherOS && git pull origin claude/scaffold-moodmark-app-f7ET2 && npm run dev
  ```

- Always use **sentence case** for CTAs / button labels / headings — not Title
  Case. ("Export library as zip", not "Export Library as Zip".)

---
> Source: [BrettfromDJ/GatherOS](https://github.com/BrettfromDJ/GatherOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
