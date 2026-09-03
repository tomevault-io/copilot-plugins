## ovh-api

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`ovh-api` is a **machine-generated** Ruby transport gem covering the entire OVH REST API
(70 top-level resources). It is the thin HTTP layer only — business ergonomics belong in a
separate hand-written wrapper (`ovh-client`). Read `README.md` for the full rationale; it is
the source of truth for the pipeline and the auth story.

## The one invariant that governs everything

`lib/**` is 100% generated output. **Never hand-edit anything under `lib/`** — changes are
wiped on the next `mise run generate` (the task does `rm -rf lib` first). To change generated
code, change its *source*: the conversion pipeline, the OVH schemas, or the generator fork.

Hand-maintained files (protected by `.openapi-generator-ignore`, safe to edit): `mise.toml`,
`scripts/**`, `versions/**`, `README.md`, `.rubocop.yml`, `.github/**`, this file.

## Build pipeline

OVH publishes legacy **Swagger 1.2**, which openapi-generator cannot ingest. The pipeline
bridges it deterministically:

```
versions/raw/*.json      pinned OVH 1.2 declarations (70 resources, committed for reproducibility)
  └─ scripts/convert.mjs   union-merge colliding models → swagger-converter (1.2→2.0)
     │                     → 3 structural fixups → swagger2openapi (2.0→3.0)
  └─ versions/ovh.oas3.json
  └─ openapi-generator (-g ruby-nextgen)  → lib/**
```

`versions/raw/*.json` are committed, so the build works **without** hitting OVH. Only
`mise run fetch` re-downloads them.

`versions/ovh.oas3.json`, `scripts/node_modules/` and `scripts/package-lock.json` are
git-ignored — so `versions/**` is hand-maintained *except* the generated spec. On a fresh
clone, `mise run convert` must run before `mise run generate`. See *What is not committed*
in `README.md` for the reproducibility caveat this creates.

### Generator dependency

The generator is `ruby-nextgen`, which **ships in the published openapi-generator release**
pinned as `OPENAPI_GENERATOR_VERSION` in `mise.toml` (`7.24.0`). There is no fork and no
local Maven build — earlier revisions of this file claimed the generator was named
`ruby-idiomatic` and lived on an unmerged fork branch; that is obsolete (the name survives
only in `ruby-nextgen`'s own README template, upstream).

`mise run generate` resolves the generator from `${OPENAPI_GENERATOR_CMD:-openapi-generator}`
— Homebrew locally, the Maven Central jar in CI — and **cross-checks its reported version
against the pin**, failing rather than silently generating different code. Bump the pin in
`mise.toml` to move to a newer generator; it is the only place the version lives.

## Commands

Toolchain is pinned via `mise.toml` (java 21, ruby 3.3, node 22).

```bash
mise run build         # convert + generate + format (the full regeneration)
mise run convert       # versions/raw → versions/ovh.oas3.json only
mise run generate      # spec → lib/** only (requires the fork jar + ovh.oas3.json)
mise run fetch         # refresh pinned OVH schemas (only when deliberately re-pinning)
mise run dev:deps      # bundle install
mise run dev:spec      # bundle exec rspec  (adds --format documentation on a TTY)
```

Tests / lint (direct):

```bash
bundle exec rspec                        # full generated spec suite
bundle exec rspec spec/api/cloud_spec.rb # a single resource's specs
bundle exec rubocop                      # lint (CI gate)
rubocop -A lib                           # the `format` step: auto-correct generated code
```

CI (`.github/workflows/ci.yml`) runs rubocop once and rspec across Ruby 3.0–4.0 + jruby +
truffleruby. It does **not** regenerate — it lints and tests the committed `lib/**`.

## Runtime shape of the generated gem

- `Ovh::Api::Client` — entry point; one memoized accessor per resource (`client.cloud`,
  `client.domain`, …). Each returns an `Ovh::Api::Api::<Resource>` instance.
- `Api::<Resource>` classes (`lib/ovh-api/api/**`) — one method per endpoint; each delegates to
  `@connection.call(:VERB, path, ...)`. Path params are `gsub`-interpolated with
  `ERB::Util.url_encode`.
- `Connection` / `Configuration` — Faraday-based; middleware stack is configurable via the
  `Client.new { |config| ... }` block. Zeitwerk autoloads everything from `lib/ovh-api.rb`.

**Authentication is not generated.** OVH signs each request (`X-Ovh-Signature = $1$` +
SHA1 over app_secret+consumer_key+method+url+body+timestamp) — a per-request signature, not an
OpenAPI security scheme. It is injected as a Faraday middleware through the config block. See
the *Authentication* section of `README.md` for the reference middleware.

---
> Source: [jbox-web/ovh-api](https://github.com/jbox-web/ovh-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
