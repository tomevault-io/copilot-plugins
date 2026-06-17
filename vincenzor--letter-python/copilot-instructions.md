## letter-python

> Integrate letter.app (product email automation) into a Python app using the official letterapp SDK. Install with pip, read the API key from the environment, then identify users and track events so onboarding sequences can fire.


## Install

```bash
pip install letterapp
```

pypi: https://pypi.org/project/letterapp/
github: https://github.com/vincenzor/letter-python
website: https://letter.app

---

## Core Workflow

The job is to wire Letter into the current Python project. Do NOT paste an API
key into source or chat - read it from `LETTER_API_KEY` in the environment
(`.env`). The Letter CLI (`npx @letterapp/cli`) provisions the key securely.

1. **Install** - `pip install letterapp`.
2. **Client** - create one shared client that reads `LETTER_API_KEY`.
3. **Identify** - call `identify` where users sign up or log in.
4. **Track** - call `track` on the 2-3 actions that matter most.

```python
import os
from letterapp import Letter

letter = Letter(api_key=os.environ["LETTER_API_KEY"])

letter.identify(user_id=user.id, email=user.email, traits={"plan": user.plan})
letter.track(user_id=user.id, event="Signed Up")

letter.close()  # flush before the process exits
```

---

## Essential snippets

```python
# Long-running server: enqueue, auto-batched, flush at exit.
letter = Letter(api_key=os.environ["LETTER_API_KEY"])

# Self-hosted / local instance:
letter = Letter(api_key=..., base_url=os.environ.get("LETTER_BASE_URL", "https://api.letter.app"))

# Serverless (Lambda / Cloud Functions): no background time to flush.
letter = Letter(api_key=os.environ["LETTER_API_KEY"], flush_at=1)
letter.track(user_id=uid, event="Checkout Started")
letter.flush()

# Context manager flushes on exit:
with Letter(api_key=os.environ["LETTER_API_KEY"]) as letter:
    letter.track(user_id=uid, event="Workspace Created")

# B2B: associate a user with an account/workspace.
letter.group(user_id=uid, account_id=workspace.id, name=workspace.name)
```

---

## Common Gotchas

1. **Never inline the API key.** Read `os.environ["LETTER_API_KEY"]`; keep it
   in `.env`, out of source control.
2. **identify needs an email** (top-level `email=` or inside `traits`).
3. **Flush before exit.** On a long-running server call `letter.close()` at
   shutdown; in serverless set `flush_at=1` and call `letter.flush()` per
   request. Otherwise queued events can be lost.
4. **Stable user_id.** Use your primary key, the same value everywhere, so
   identify and track line up to one contact.
5. **base_url only for self-host/local.** Omit in production; it defaults to
   `https://api.letter.app`.

---

## Quick Reference

```python
Letter(api_key, base_url=..., flush_at=50, flush_interval=0.1, max_retries=3, timeout=10.0, on_error=None)
letter.identify(user_id, email=None, traits=None, timezone=None, timestamp=None, message_id=None)
letter.group(user_id, account_id, name=None, traits=None, timestamp=None, message_id=None)
letter.track(user_id, event, properties=None, timestamp=None, message_id=None)
letter.flush()
letter.close()
```

---
> Source: [vincenzor/letter-python](https://github.com/vincenzor/letter-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
