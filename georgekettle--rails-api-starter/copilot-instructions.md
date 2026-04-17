## rails-api-starter

> name: Gems to be used in this project

---
name: Gems to be used in this project
type: always
description: A list of which gems will be used in this project
---

Only add gems if they are absolutely necessary. The fewer dependencies, the better.

Below are some of the gems we love working with.

gems:
  - gem: rails
    version: '~> 8.0.0'
    reason: Core framework for building the API.

  - gem: sqlite3
    reason: Lightweight database for development and testing.

  - gem: puma
    reason: Fast, concurrent web server for serving Rack applications.

  - gem: rack-cors
    reason: Enables Cross-Origin Resource Sharing, required for APIs accessed by browsers.

  - gem: dotenv-rails
    groups: ['development', 'test']
    reason: Loads environment variables from .env files for local development and testing.

  - gem: minitest
    reason: Default testing framework in Rails, fast and simple.

  - gem: pagy
    reason: Minimal and fast pagination gem.

  - gem: stripe
    reason: Integration with Stripe API for handling payments.

  - gem: letter_opener
    groups: ['development']
    reason: Opens sent emails in the browser instead of sending them.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/georgekettle) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
