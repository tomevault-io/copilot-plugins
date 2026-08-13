## rails-named-cache

> Guidance for AI coding agents (and humans) working on `rails_named_cache`.

# AGENTS.md

Guidance for AI coding agents (and humans) working on `rails_named_cache`.

## What this gem is

A Rails gem that lets an app register several `ActiveSupport::Cache::Store` instances by name
and read them back with `Rails.cache(:name)`. That is the entire feature.

The whole runtime is ~150 lines across four files:

| File | Responsibility |
| --- | --- |
| `lib/rails_named_cache.rb` | Public module API, error classes, the process-wide registry constant |
| `lib/rails_named_cache/registry.rb` | Thread-safe name => store map. Owns all state |
| `lib/rails_named_cache/configuration.rb` | Resolves/validates `config.named_cache_stores`. Owns no state |
| `lib/rails_named_cache/logging.rb` | Hit/miss logging: one `cache_read.active_support` subscriber |
| `lib/rails_named_cache/rails_ext.rb` | The single monkey patch: `Rails.cache(name = nil)` |
| `lib/rails_named_cache/railtie.rb` | Boot wiring only |

## Design rules — do not break these

1. **`Rails.cache` with no argument must behave exactly as stock Rails.** An app that never sets
   `config.named_cache_stores` must be bit-for-bit unaffected. `Rails.cache=` must keep working.
2. **Named stores are returned as-is.** No wrapping, no proxying, no decorating. `Rails.cache(:x)`
   returns the identical object that was configured (`be(store)`, not `eq(store)`).
3. **No new caching API.** Anything a user could want — TTLs, namespaces, compression,
   instrumentation, coders — already exists in ActiveSupport. Do not reimplement it here.
4. **One monkey patch, ever.** `RailsExt` prepended onto the `Rails` singleton. If a feature needs
   a second patch, the feature is wrong.
5. **Lookups never fall back.** An unregistered name raises `UnknownStoreError` listing what is
   available. Silent fallback to another store is a data-corruption bug, not a convenience.
6. **`nil` is never a store.** `ActiveSupport::Cache.lookup_store(nil)` hands back a MemoryStore;
   that must stay an explicit `ConfigurationError`.
7. **Configuration resolves everything before registering anything.** A bad entry must leave the
   registry untouched (there is a spec for this).
8. **Observability comes from ActiveSupport::Notifications, never from wrapping a store.** The
   hit/miss logger is a subscriber; if it ever needs the store instance, the answer is still no —
   the payload carries only the class, and that is the price of rule 2.
9. **Names are non-empty symbols or strings, coerced in one place.** `RailsNamedCache.normalize_name`
   is the only `to_sym`; a duplicate name (`"x"` and `:x`) is a `ConfigurationError`, not last-wins.

## Style

- Lazy first. This gem exists because the alternative was five global constants. Keep the diff
  smaller than the problem. No abstraction with one implementation, no config for a value that
  never changes, no scaffolding for a feature nobody asked for.
- Ruby >= 3.3, `# frozen_string_literal: true` on every file, double quotes, 120 columns.
- Rails >= 7.0 is supported, so no API newer than Rails 7.0 in runtime code.
- Every public method gets a YARD comment with `@param`/`@return`/`@raise`. Match the density
  already in the files.
- Runtime dependencies are `activesupport`, `railties` and `concurrent-ruby` (all things a Rails
  app already has). **Adding a runtime dependency needs a very good reason.**

## Testing

```bash
bundle exec rake            # rspec + rubocop, the gate for any change
bundle exec rspec           # specs only
bundle exec rubocop -A      # autocorrect

BUNDLE_GEMFILE=gemfiles/rails_7.0.gemfile bundle install
BUNDLE_GEMFILE=gemfiles/rails_7.0.gemfile bundle exec rspec
```

- `spec/support/test_app.rb` boots one real, tiny `Rails::Application` in a tmpdir for the whole
  suite. A process can hold only one `Rails.application`, so it is memoized and its registry is
  snapshotted immediately after boot. **Do not add a second app boot** — use the snapshot.
- The registry is process-wide state. `spec_helper` clears it after every example; anything a spec
  registers must not leak.
- Specs that touch `Rails.cache=` must restore the previous value (see the `around` hook in
  `rails_ext_spec.rb`).
- New behaviour needs a spec at the level it lives: registry behaviour in `registry_spec.rb`,
  resolution in `configuration_spec.rb`, boot wiring in `railtie_spec.rb`.

## Release

1. Bump `lib/rails_named_cache/version.rb` (SemVer).
2. Move the `Unreleased` entries in `CHANGELOG.md` under the new version + date, update the
   compare links at the bottom.
3. `bundle exec rake` must be green on the full CI matrix (Ruby 3.3, 3.4, 4.0 × Rails 7.0–8.1).
4. `bundle exec rake release`.

## Things deliberately not built

Do not add these without an explicit request — they were considered and skipped:

- L1/L2 layering (`Rails.cache(:memory, fallback: :redis)`). The signature leaves room; the
  feature is not there.
- A `spec/dummy` application directory. The in-memory `TestApp` covers boot integration.
- Per-store instrumentation, stats, or a dashboard. ActiveSupport already instruments.
- Lazy/deferred store instantiation. Stores are built once at boot; that is cheap and predictable.

---
> Source: [igorkasyanchuk/rails_named_cache](https://github.com/igorkasyanchuk/rails_named_cache) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
