## rubyzip

> Instructions for coding agents (and other automated contributors) working in

# AGENTS.md

Instructions for coding agents (and other automated contributors) working in
this repository.

## What this project is

Rubyzip is a pure-Ruby library for reading and writing zip files. It exposes both a high-level, `File`-like API (`Zip::File`) for working with zip archives as regular files, and lower-level streaming APIs (`Zip::InputStream`, `Zip::OutputStream`) for reading/writing zip data as a stream without needing random access to the underlying storage.

Key areas of the codebase, under `lib/zip/`:

- `file.rb`, `entry.rb` — the high-level archive API.
- `input_stream.rb`, `output_stream.rb` — streaming read/write API.
- `central_directory.rb`, `entry.rb` — zip format metadata handling.
- `extra_field/` — parsers for the various "extra field" records a zip entry
  can carry (Unix permissions, Unicode paths, Zip64 sizes, etc).
- `crypto/` — traditional (weak, legacy) zip encryption support and more modern AES decryption.
- `filesystem/` — an adapter that lets `Zip::File` be used as a drop-in for
  parts of Ruby's `File`/`Dir` API.
- `ioextras/` — shared IO helper mixins used by the streaming classes.

The library has no runtime dependencies outside the Ruby standard library. It supports Ruby >= 3.0, plus JRuby and TruffleRuby (see `.github/workflows/tests.yml` for the exact CI matrix).

## Setting up

```shell
bundle install
```

This installs the development dependencies declared in `rubyzip.gemspec` (minitest, rubocop, rdoc, simplecov, etc). There's also a `:benchmark` group in the `Gemfile` for the scripts under `benchmark/`, which isn't needed for normal development — CI runs with `BUNDLE_WITHOUT: benchmark`.

If you are testing with JRuby or TruffleRuby use the following to avoid issues with installing dependencies:

```shell
BUNDLE_WITHOUT=benchmark bundle install
```

And then use `BUNDLE_WITHOUT=benchmark` before any `bundle exec` commands.

## Testing

Tests are written with Minitest and live under `test/`, mirroring the structure of `lib/zip/`. Test file names end in `_test.rb`.

Run the full suite with Rake (this is also the default Rake task):

```shell
bundle exec rake test
# or simply:
bundle exec rake
```

Run a single test file directly:

```
bundle exec ruby -Ilib -Itest test/entry_test.rb
```

Notes:

- `test/gentestfiles.rb` and helpers under `test/helpers/` generate fixture zip files used by many tests; you generally don't need to touch these unless you're adding a new kind of fixture.
- `test/data/` holds static test fixtures (including some historic/malformed zip files) and is excluded from Rubocop.
- Some Zip64 tests are slow and are skipped unless `FULL_ZIP64_TEST=1` is set in the environment. Set it if you're changing Zip64-related code. It is always set in CI tests.
- Coverage is collected via SimpleCov automatically whenever the suite is run through the Rake task (`test.framework = 'require "simplecov"'` in `Rakefile`); results are written to `coverage/`.
- When fixing a bug, add or extend a test that reproduces it before writing the fix, and make sure the new test fails without the fix.

## Linting

This project uses Rubocop, configured in `.rubocop.yml` (with legacy exceptions tracked in `.rubocop_todo.yml`).

```shell
bundle exec rubocop
```

To auto-correct what's safely fixable:

```shell
bundle exec rubocop -a
```

Guidelines:

- Always ask before auto-correcting Rubocop failures.
- Don't add new blanket exclusions to `.rubocop_todo.yml` for new code — that file exists to grandfather in pre-existing style debt, not to excuse new violations. Fix the code, or, if a rule genuinely shouldn't apply, add a narrowly-scoped exception directly in `.rubocop.yml` with a comment explaining why (follow the existing style of comments there). `Layout/LineLength` is capped at 100 for `lib/`, but is uncapped for `test/**/*.rb`.
- CI runs `bundle exec rubocop` as a separate required job (`.github/workflows/lint.yml`), so lint failures block merges just like test failures.

## Documentation

RDoc comments (markdown-flavoured) document the public API. If you change or add public methods on `Zip::File`, `Zip::Entry`, `Zip::InputStream`, or `Zip::OutputStream`, update their RDoc comments accordingly. You can build the docs locally with:

```shell
bundle exec rake rdoc
```

If you need to force all RDoc comments to be rebuilt, use:

```shell
bundle exec rake rerdoc
```

## Other conventions

- All Ruby files start with `# frozen_string_literal: true`.
- In Markdown files, allow paragraph text and text in bulleted lists to wrap. Do not insert line-breaks to artificially format the raw Markdown file.
- Keep the public API changes backwards-compatible where possible; this is a widely-used library. Breaking API changes are called out prominently in `README.md` and `Changelog.md` (see the "Updating to version 3.0" section of the README for an example of the level of detail expected).
- Do not add entries to the `Changelog.md` file. This is built by hand at release time.
- This projects adheres to the principles of Semantic Versioning (see the summary section at https://semver.org)
- Don't bump the version in `lib/zip/version.rb` yourself unless specifically asked to — releases are cut deliberately. Do raise a note if you think that a change requires more than just a patch-level version number bump.
- Security issues should never be discussed in public issues/PRs; the README asks that they be emailed directly to the maintainer instead. If you notice something that looks like a security vulnerability while working in this repo, flag it to the user rather than opening a public PR describing it.

---
> Source: [rubyzip/rubyzip](https://github.com/rubyzip/rubyzip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
