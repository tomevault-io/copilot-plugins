## olympus-addon

> - Keep changes focused on the requested task and follow nearby Lua patterns.

# Working on Olympus

## Scope and context

- Keep changes focused on the requested task and follow nearby Lua patterns.
- Read `README.md` for addon behaviour and development commands. Runtime code is
  in `Olympus/`; `Olympus/Olympus.toc` declares supported interfaces and load order.
- Use the existing offline harness in `tests/run.lua` and fixtures in
  `tests/fixtures/` when adding coverage.

## Tests for changes

- For every bug fix, add a regression test that fails for the original bug and
  passes with the fix. Verify this against the pre-fix code or by temporarily
  removing the fix in a scratch copy, when practical.
- For new or changed behaviour, add or update tests covering the expected result
  and relevant failure cases. Check whether existing coverage already exercises it.
- Exercise the actual addon functions and assert observable outcomes. Avoid
  duplicating the implementation inside a test.
- Change assertions or mocks to reflect intended behaviour or verified client
  behaviour, and explain those changes. Do not weaken them just to make a test pass.
- If the offline harness cannot reproduce a bug, explain that limitation and give
  concrete in-game reproduction steps and expected results in the PR.
- For documentation-only changes, validate referenced paths, commands and claims.

## Checks

For code or test changes, run `bash scripts/check.sh` from the repository root when
that script is present. Otherwise run the commands below. Both routes require
LuaJIT and Bash.

```sh
luajit tests/run.lua
bash scripts/lint-globals.sh
```

The lint script checks for globals that share a name with a local in the same file.
See `README.md` for packaging and deployment commands when relevant to the task.

## Client compatibility and handoff

- Follow existing client compatibility guards and verify any newly used WoW API
  against the supported client. A method provided by a mock does not establish
  that the real client provides it.
- Offline tests use simulated WoW APIs and UI frames. Include specific in-game
  checks for client-dependent behaviour such as chat routing, UI interaction and
  communication between players.
- Report the commands run and their results, any checks skipped and why, and the
  in-game verification still needed. Claim an in-game check passed only if observed.

---
> Source: [dnl-gentile/olympus-addon](https://github.com/dnl-gentile/olympus-addon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-27 -->
