## tika

> - build in ruby. minimize external dependencies. prefer re-implementing simple logic internally versus using an external library. ask for approval before using non-standard ruby libraries.

# Requirements:

- build in ruby. minimize external dependencies. prefer re-implementing simple logic internally versus using an external library. ask for approval before using non-standard ruby libraries.
- always share your plan before coding anything.
- always create or update spec files before coding anything.
- provide 80% test coverage, focusing on exercising the new core logic being developed, with use cases that test for edge cases.
- provide integration testing that focus on the core customer flows throughout the experince. Mock up external services and data files when necessary.
- always validate changes by making sure all tests run cleanly and the code complies without errors. explain warnings to me when you see them and suggest any helpful changes based on those errors.

## Validation commands

Run the full test suite (provisions sample site, runs all unit + integration tests, checks 80% coverage):
```sh
bundle exec rake test
```

Check Ruby syntax across all library files without executing (catches parse errors):
```sh
find lib -name '*.rb' | xargs ruby -c
```

Check for load errors — missing requires, undefined constants, bad method calls on load (closest to a compile check in Ruby):
```sh
bundle exec ruby -I lib -e "require 'tika'"
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nbuggia) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
