## habit-hooks

> A scope filename is exact, and RuboCop reads its file arguments two other ways before it reads them as names. A filename beginning with `-` is parsed as short options — `-c` among them, which takes the rest as its `--config` value — and an argument containing a `*` is handed to `Dir[]` (`TargetFinder#process_explicit_path`), so a literal star sweeps in every file it matches. `run_rubocop` therefore puts `--` between its flags and the files, and `literal_spelling_of` escapes the glob metacharacters.

# habit-hooks-ruby notes

## Gotchas

### RuboCop reads a file argument as options, then as a glob

A scope filename is exact, and RuboCop reads its file arguments two other ways before it reads them as names. A filename beginning with `-` is parsed as short options — `-c` among them, which takes the rest as its `--config` value — and an argument containing a `*` is handed to `Dir[]` (`TargetFinder#process_explicit_path`), so a literal star sweeps in every file it matches. `run_rubocop` therefore puts `--` between its flags and the files, and `literal_spelling_of` escapes the glob metacharacters.

The escaping is narrower than it looks, and has to be. RuboCop globs only an argument containing a `*`; every other one it takes verbatim, backslashes included, so escaping a `?` in a starless path would name a file that does not exist and the run would die on `Error: No such file or directory`. The other metacharacters are escaped only inside a starred argument, where `Dir[]` is reading the whole thing as a pattern. The literal-star behavioural test runs only where a filesystem allows a `*` in a name — Windows forbids it — so it skips there through `tests/platform_probe.A_FILESYSTEM_THAT_ALLOWS_A_STAR_IN_A_FILENAME`.

### RuboCop's exit code lies when the binstub never reached RuboCop

`ruff_sensor` can trust ruff's exit code. `rubocop` is a RubyGems binstub beginning `#!/usr/bin/env ruby`, so it is only as good as the `ruby` that answers first. Point it at an interpreter it is not installed into (a version manager left off `PATH`, the wrong bundle, macOS's system Ruby 2.6) and it dies in `find_spec_for_exe` with a Ruby traceback and **exit 1** — the code reserved for "I found offences". Judged on the code alone, that is a clean file from a tool that never started, which is the false-clean class.

So the report is the evidence, not the code. `--format json` prints its envelope on every run RuboCop completed, down to `"files": []` when it inspected nothing, so no envelope means no run whatever it exited with. `report()` answers `None` for anything that is not a report, and it requires the `files` key rather than merely valid JSON. `rubocop_crashed()` takes the parsed report as an argument instead of re-deriving it, so the rule lives in one place and `main` cannot drift from what the tests exercise.

Ask this of any wrapped tool reached through an interpreter shim rather than a binary. The cost of getting it wrong is silence, and silence reads as success.

The envelope check has one concern: a cop that *raises exceptions*. By default, RuboCop rescues the exception, reports the crash on stderr, and exits 1 with a valid envelope listing that file's offences as `[]`. This isn't what habit-hooks needs to detect the crash. So `--raise-cop-error` is used to make a RuboCop crash become an `Error:` and exit 2, which habit-hooks interprets as a failed run.

`tests/test_the_sensor_runs_the_rubocop_it_is_handed.py` has to hand `ruby` back on a directory of its own for the same reason. The binstub and its interpreter live in one directory, so taking `rubocop` off `PATH` takes `ruby` with it, and the test would then be proving the crash rather than the lookup.

### RuboCop globs every ancestor directory looking for a gemspec

`TargetRuby` settles which Ruby to parse as by trying, in order, `TargetRubyVersion` in the config, then any `*.gemspec`, then `.ruby-version`. The gemspec step is a `Dir.glob` up every ancestor directory to the filesystem root, and **`.ruby-version` does not prevent it**, because RuboCop looks for the gemspec first. A Rails app, which has no gemspec, therefore sends RuboCop climbing out of the project on every run. A gem stops the climb by having a gemspec; everything else stops it by pinning `TargetRubyVersion`.

That is RuboCop's own behaviour and the sensor reproduces it rather than papering over it, per the precedence rule in the root `AGENTS.md`. It matters for the tests because a suite case runs in a temp dir that may sit under this checkout: a fixture pinning neither climbs through this checkout and out into the home directory, where a sandboxed dev machine denies the glob outright and the sensor fails for a reason that has nothing to do with the case. Every fixture — the scenario sample's `.rubocop.yml` and `tests/installed_projects.ruby_project` — pins `TargetRubyVersion`.

It is the RuboCop counterpart of the root `AGENTS.md`'s `GIT_CEILING_DIRECTORIES` rule and of jscpd's `.gitignore` walk: a wrapped tool that searches upward has to be given a floor, or it finds ours.

### A Rails project's rubocop is the only one that can read its config

A `.rubocop.yml` naming cops from an extension gem is a hard RuboCop error when that gem is not loadable:

```
Error: `Rails/*` has been extracted to the `rubocop-rails` gem.
```

Nearly every Rails repo pins rubocop plus `rubocop-rails` / `rubocop-rspec` / `rubocop-performance` in its `Gemfile`, and those gems load only under the project's own bundle. This is the likeliest way the sensor fails in practice, and it is why the plugin's rubocop detector declares `search_paths = ["bin"]` (`src/habit_hooks_ruby/config.toml`): `bundle binstubs rubocop` writes `bin/rubocop`, and every lookup for that tool — `missing_tools` clearing it, `sensors/named_tools.py` resolving the recipe's `${detector:rubocop}` — searches the project's `bin` ahead of the default path.

The lookup belongs in the core and not in this sensor. `missing_tools` clears a tool by asking `tool_executable`, and `sensors/spawn.py` spawns the file it answered with, so a second answer here would let setup clear a rubocop the run then does not use. And `bin` is deliberately **not** on the default search path (`project_paths.tool_search_path`), only on this detector's: `node_modules/.bin` and `.venv/bin` are directories an install keeps to itself, where a `bin` may hold a project's own scripts, and a script named after a tool should not outrank a real install beside it. Naming the directory on the detector keeps it in play for exactly the tools that live in it.

## Testing

RuboCop behaviour is gated by `scenarios/rubocop/` — the approved-output scenario runs the real tool, so a RuboCop upgrade that changes its output shape fails there. A machine without rubocop skips the scenario visibly instead of failing: the scenario, not the suite, is what needs the tool.

---
> Source: [habit-hooks/habit-hooks](https://github.com/habit-hooks/habit-hooks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
