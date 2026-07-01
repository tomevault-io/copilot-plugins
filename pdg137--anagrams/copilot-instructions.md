## anagrams

> This is an implementation of the game "anagrams" in Ruby on

This is an implementation of the game "anagrams" in Ruby on
Rails. It's a personal experimental project to help me learn modern
Javascript techniques such as Websockets, ES6, Import maps, and Turbo.

Stack: Ruby on Rails with postgres/sqlite.
Dependencies: installed via nix (no need to run "bundle exec").
Development: test-driven development using rspec/Capybara.

Philosophy:

- minimalist self-explanatory code
- get content (HTML) right before presentation (CSS)
- use small methods
- use standard technology features over custom implementations
- do not add defensive conditional code, especially when untested
- comment only unusual parts; better to make the code itself descriptive
- refactor often
- test visible behavior of site, NOT hidden HTML attributes

Structure:

User commands in app/channels/chat_channel.rb
Game logic in app/models/game.rb
Main controller logic in app/controllers/games_controller.rb

Game rules:

The game is represented by a log that shows, one line at a time,
exactly what happened in the game.

The first line is the list of starting letters, like AAABBCDEFG.

A command like "Alice+A" means player "Alice" flipped letter "A",
making it visible.

A command like "Bob:COWRY" means player "Bob" formed word "COWRY",
taking letters from the visible set and possibly combining them with
words owned by other players (stealing).

The log can be processed at any time to update the game state.

---
> Source: [pdg137/anagrams](https://github.com/pdg137/anagrams) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-30 -->
