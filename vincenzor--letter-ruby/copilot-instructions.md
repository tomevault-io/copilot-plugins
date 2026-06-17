## letter-ruby

> Integrate letter.app (product email automation) into a Ruby app using the official letterapp gem. Install with bundler, read the API key from the environment, then identify users and track events so onboarding sequences can fire.


## Install

```bash
bundle add letterapp
# or: gem install letterapp
```

rubygems: https://rubygems.org/gems/letterapp
github: https://github.com/vincenzor/letter-ruby
website: https://letter.app

---

## Core Workflow

The job is to wire Letter into the current Ruby project. Do NOT paste an API
key into source or chat - read it from `LETTER_API_KEY` in the environment
(`.env`). The Letter CLI (`npx @letterapp/cli`) provisions the key securely.

1. **Install** - `bundle add letterapp`.
2. **Client** - create one shared client that reads `LETTER_API_KEY`.
3. **Identify** - call `identify` where users sign up or log in.
4. **Track** - call `track` on the 2-3 actions that matter most.

```ruby
require "letterapp"

letter = Letterapp::Client.new(api_key: ENV["LETTER_API_KEY"])

letter.identify(user_id: user.id, email: user.email, traits: { plan: user.plan })
letter.track(user_id: user.id, event: "Signed Up")

letter.close # flush before the process exits
```

In Rails, create the client once in an initializer
(`config/initializers/letter.rb`) and reuse it.

---

## Essential snippets

```ruby
# Long-running server: enqueue, auto-batched, flush at exit (at_exit hook).
letter = Letterapp::Client.new(api_key: ENV["LETTER_API_KEY"])

# Self-hosted / local instance:
letter = Letterapp::Client.new(api_key: ENV["LETTER_API_KEY"], base_url: ENV.fetch("LETTER_BASE_URL", "https://api.letter.app"))

# Serverless: no background time to flush.
letter = Letterapp::Client.new(api_key: ENV["LETTER_API_KEY"], flush_at: 1)
letter.track(user_id: uid, event: "Checkout Started")
letter.flush

# B2B: associate a user with an account/workspace.
letter.group(user_id: uid, account_id: workspace.id, name: workspace.name)
```

---

## Common Gotchas

1. **Never inline the API key.** Read `ENV["LETTER_API_KEY"]`; keep it in
   `.env`, out of source control.
2. **identify needs an email** (keyword `email:` or inside `traits`).
3. **Flush before exit.** On a long-running server call `letter.close` at
   shutdown (an `at_exit` hook also runs it); in serverless set `flush_at: 1`
   and call `letter.flush` per request. Otherwise queued events can be lost.
4. **Stable user_id.** Use your primary key, the same value everywhere, so
   identify and track line up to one contact.
5. **base_url only for self-host/local.** Omit in production; it defaults to
   `https://api.letter.app`.

---

## Quick Reference

```ruby
Letterapp::Client.new(api_key:, base_url:, flush_at: 50, flush_interval: 0.1, max_retries: 3, open_timeout: 10, read_timeout: 10, on_error: nil)
letter.identify(user_id:, email: nil, traits: nil, timezone: nil, timestamp: nil, message_id: nil)
letter.group(user_id:, account_id:, name: nil, traits: nil, timestamp: nil, message_id: nil)
letter.track(user_id:, event:, properties: nil, timestamp: nil, message_id: nil)
letter.flush
letter.close
```

---
> Source: [vincenzor/letter-ruby](https://github.com/vincenzor/letter-ruby) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
