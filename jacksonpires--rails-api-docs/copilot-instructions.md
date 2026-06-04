## rails-api-docs

> Notes for future sessions working on this gem. Does not duplicate the README — focuses on design decisions, gotchas, and how to contribute without breaking things.

# CLAUDE.md

Notes for future sessions working on this gem. Does not duplicate the README — focuses on design decisions, gotchas, and how to contribute without breaking things.

## Architecture in one sentence

`RouteInspector` (routes) + `ControllerInspector` (Prism AST of `params.permit`) + `SchemaInspector` (AR columns) → `Config::Builder` assembles hash → `Config::Appender` merges with existing YAML → `Doc::Renderer` (ERB + `CurlRenderer`) → self-contained HTML.

## How to run

```bash
bundle install
bundle exec rake test           # 91 runs, ~0.1s
```

End-to-end smoke test against a real Rails app:
```bash
bash /tmp/rad_smoke_test.sh     # spins up app in /tmp/rad-host, mounts gem, hits /rails/api-docs
bash /tmp/rad_smoke_build.sh    # exercises rake task + OUTPUT= override + missing-config branch
```

## Conventions (non-obvious)

- **Hyphenated gem name, single `RailsApiDocs` module** (no nesting). Files live in `lib/rails-api-docs/...` (with hyphen).
- **Generator namespace forced** in `init_generator.rb` via `namespace "rails-api-docs:init"`. Without it, Rails would infer `rails_api_docs:init` from the module name.
- **`:init` and `:update` are aliases via inheritance.** `UpdateGenerator < InitGenerator` only overrides the namespace; behavior is identical. Add new flags / logic to InitGenerator only — UpdateGenerator inherits everything (including `class_option`s, which Thor preserves on subclasses).
- **Tests using `Rails::Generators::TestCase`** clash if you define a helper named `render` — use a different name (already tripped on this).
- **Hashes with string keys passed to a method with kwargs**: always wrap them in explicit `{}` — Ruby 3 sometimes interprets them as kwargs.

## Sensitive areas — touch carefully

### `Config::Appender` — append-only preserves user edits
Endpoint identity is `"#{method} #{path}"`. **Existing fields always win** over generated ones. If you change the logic, make sure the tests in `test/config/appender_test.rb` still cover:
- User edits to description/body/examples survive.
- Existing `general_configurations` are never overwritten (but new keys are added).
- Header comments (`#` lines at the top) preserved via `Loader.header`.

**Inline** comments inside the YAML are lost on re-dump — documented limitation in the README.

### `CurlRenderer` shell-escape gsub trap
This line:
```ruby
str.gsub("'") { "'\\''" }
```
**Uses the block form on purpose.** The replacement-string form (`str.gsub("'", "'\\''")`) falls into the `\'` trap — `gsub` interprets it as "post-match reference" and duplicates the content. If you touch this, run `test_escapes_single_quotes_in_body_for_shell` before committing.

### Engine `rake_tasks` / `generators` are class-level macros
They **cannot** live inside `initializer "...".do`. They must be top-level in the Engine class body. Broke this once while reorganizing.

### ERB template uses `<% ... -%>` trim mode
Whitespace control via the trailing `-`. If you add tags, keep the pattern or the HTML will sprout random blank lines.

### `render_field` is the single source of truth for field rendering
Body, params, request headers, response headers, and response schema all flow through `Renderer#render_field`. To add a new per-field attribute (a new badge or meta line), edit only that one method. The template just calls `render_fields(endpoint["body"])` etc. — never expand field markup inline in the ERB.

Order of badges in a field-row matters for visual hierarchy — see the method body for the order (format → in → readonly → writeonly → nullable → required → metas).

### `field["example"]` is the single source of truth for sample values
`CurlRenderer#body_json` and `Renderer#default_response_example` both prefer `field["example"]` over the type-derived `sample_value` fallback. The Builder + BodyInferrer seed `example:` for every inferred field, so by the time the renderer runs, there's almost always a real value to display. If you add a new place that needs a placeholder, follow the same precedence: user example wins; fall back to `RailsApiDocs::SampleValue.for(type)`.

### `RailsApiDocs::SampleValue` is the single source of truth for type→sample
Three consumers (BodyInferrer, Builder for path params, CurlRenderer, Renderer) all call `SampleValue.for(type)`. Adding a new type maps in one place.

### Pragmatic vs verbose YAML scaffold
Default emits the commonly-edited subset (`description`, `example` on every field; minimal `responses["200"]` stub). `--verbose-yaml` (or `RailsApiDocs.configuration.verbose_yaml = true`) emits **every** key with defaults — useful for discoverability, costly in lines per endpoint (~10× larger). The verbose defaults live in `verbose_field_defaults` / `verbose_endpoint_meta` methods (not constants — return fresh hashes per call so `YAML.dump` doesn't emit anchors).

## Filters & extensibility

User-facing filters on `Configuration`:
- `ignored_path_prefixes` (strings)
- `ignored_controllers` (strings **or** regexps)
- `only_controllers` (strings **or** regexps, whitelist)
- `ignored_actions` (strings)
- `api_only` (boolean) — see `JsonRouteDetector` below

`INTERNAL_PREFIXES` in `RouteInspector` is hardcoded — add new entries there when a new Rails core prefix shows up. User filters **stack on top of** these, they don't replace them.

### Shared `controller_matches?` rule

`ignored_controllers` and `only_controllers` share `RouteInspector#controller_matches?`. Rules:

- **Regexp** → `Regexp#match?`.
- **String containing `/`** → exact path match (`"api/v1/users"` matches only that exact controller).
- **String without `/`** → boundary-aware suffix match. `"users"` matches `users` OR `api/v1/users`, but NOT `super_users` (boundary is the `/` separator or full equality).

When both filters apply, blacklist (`ignored_controllers`) wins over whitelist (`only_controllers`).

**Note:** the `ignored_controllers` string-match was changed to suffix-match in this same change (v0.1.0 was never released, so no migration impact). The previous semantics were exact-match.

### CLI parsing of `--only-controllers` (4 forms)

Thor `class_option :only_controllers, type: :array` accepts space-separated natively. The generator's `parse_only_controllers` additionally strips brackets and splits on commas, so all of these work:

```
--only-controllers users posts            # space-separated
--only-controllers=users,posts            # comma-separated
--only-controllers=[users,posts]          # bracketed
--only-controllers=[ users , posts ]      # whitespace tolerated
```

What does NOT work: repeating the flag (`--only-controllers users --only-controllers posts`). Thor's array type takes the last occurrence — always pass all names in one flag.

## `--api-only` flag / `Configuration#api_only`

`Inspectors::JsonRouteDetector` decides per-route. Logic:

1. Parses each controller file via Prism **once** per detector instance (cache keyed by controller path). Same AST feeds two visitors:
   - `SuperclassVisitor` — extracts the parent class constant name (`ActionController::API`, `Api::BaseController`, etc.).
   - `JsonActionVisitor` — walks every `DefNode` and records method names whose body contains `render` with a `json:` key (kwarg or hash-rocket form, inside or outside `respond_to`).
2. Resolves inheritance chain recursively, **with cycle guard** — placeholder cache entry inserted before recursing to prevent infinite loops on pathological inputs.
3. Decision: `:api` controller → all actions true. Otherwise, action-level `render json:` → true. Else → false.

**Strict on `:unknown`** — controller file missing or unresolvable parent → excluded. Per-action detection still runs even when inheritance is unresolvable (file exists but parent comes from a gem).

CLI flag (`--api-only`) wins when both set; `--no-api-only` lets you override config for a single run. `class_option` default is `nil` (not `false`) so we can distinguish "flag omitted" from "flag explicitly off".

## Where to edit what

| To change…                         | File                                                   |
|------------------------------------|--------------------------------------------------------|
| visual / HTML layout               | `lib/rails-api-docs/templates/api_docs.html.erb`       |
| default colors                     | `Config::Builder::DEFAULT_GENERAL` + CSS vars in ERB   |
| field type inference               | `Inspectors::BodyInferrer`                             |
| sample value for type X            | `CurlRenderer#sample_value` and `Renderer#sample_value`|
| new route filters                  | `Inspectors::RouteInspector` + `Configuration`         |
| dev mount behavior                 | `Doc::Responder` (logic) + `engine.rb` (mount)         |

## Known limitations (do NOT "fix" without explicit ask)

- Nested permits (`permit(items: [:name])`): top-level keys only.
- Splat permits (`permit(*ARGS)`): not resolved.
- Inline YAML comments lost on re-runs.
- PATCH and PUT for the same update produce two endpoints (intentional — user can `show: false`).
- `new` and `edit` actions appear by default (user opts out via `ignored_actions`).

## Mockup vs output

`preview/api-docs.html` is the **original static mockup** (visual reference for the design). It is not produced by the gem. Files matching `preview/api-docs-rendered*.html` are real outputs of `Doc::Renderer` for visual comparison.

---
> Source: [jacksonpires/rails-api-docs](https://github.com/jacksonpires/rails-api-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
