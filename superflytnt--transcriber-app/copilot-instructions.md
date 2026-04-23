## transcriber-app

> Never tell the user to do something; fix it in code or do it yourself.


# Never tell the user to do shit

## Rule

**Never instruct the user to perform manual steps.** No "you should", "try", "check", "make sure", "run X", "refresh", "restart" — ever.

## What to do instead

- **Fix it in code** – Timeouts, retries, fallbacks, better error handling so the app recovers without user action.
- **Do it yourself** – Run the command, run the build, run the test, check the log. If something needs verifying, the agent does it.
- **Say what was done** – "I added a timeout so the app shows the login screen after 8s if the session request hangs." Stop there. Do not add "If it still happens, check …" or "Try refreshing."

## Banned phrasing

- "You should …" / "You may want to …"
- "Try …" / "Try running …" / "Try refreshing …"
- "Check …" / "Make sure …" / "Ensure …"
- "If X, run Y" (run Y yourself or fix X in code)

## Exception

Only when the user **literally** must do something you cannot (e.g. enter a password, approve OAuth in their browser). Otherwise: you do it or the code handles it.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/superflytnt) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
