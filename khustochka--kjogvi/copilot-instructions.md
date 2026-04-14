## kjogvi

> **Kjógvi** is a Phoenix v1.8 umbrella project for bird observation tracking with ornithological taxonomy management. It combines birding/observation records with a comprehensive taxonomy system via the [Ornithologue](./apps/ornithologue/) library.

# Kjógvi Codebase Instructions for AI Agents

## Project Overview

**Kjógvi** is a Phoenix v1.8 umbrella project for bird observation tracking with ornithological taxonomy management. It combines birding/observation records with a comprehensive taxonomy system via the [Ornithologue](./apps/ornithologue/) library.

### Architecture
- **Umbrella structure** with 4 apps in `/apps`:
  - `kjogvi`: Core business logic, data models, and contexts (Ecto repos: `Kjogvi.Repo`, `Kjogvi.OrnithoRepo`)
  - `kjogvi_web`: Phoenix 1.8 LiveView web interface (main app)
  - `ornithologue`: Ornithological taxonomy library (reusable package)
  - `ornitho_web`: Taxonomy dashboard UI (composable into `kjogvi_web`)
- **Dual repositories**: Main app repo + separate OrnithoRepo for taxonomy data
- **Single-user mode** by default (configurable via `Kjogvi.Config.with_single_user/1` macro)

## Critical Development Workflows

```bash
# Setup
mix setup                           # Install deps in all apps
docker compose up -d                # Start PostgreSQL (port 5498)

# Development
iex -S mix phx.server              # Start with IEx shell

# Testing & Quality
mix lint.fix                       # Auto-fix formatting + run lint (run before committing)
mix lint                           # Full linting: credo, dialyzer, xref cycles
MIX_ENV=test mix ecto.setup       # Create test database
mix test                           # Run tests with coverage

# Database
mix ecto.create                    # Create databases
mix ecto.migrate                   # Run migrations on main repo
mix ecto.migrate -r Kjogvi.OrnithoRepo  # Run migrations on taxonomy repo
```

## Project-Specific Patterns

- **Router**: Routes in `:require_authenticated_user` pipeline; use `:as` option for scopes. Single-user mode via `Kjogvi.Config.with_single_user/1`
- **Forms**: Use `to_form(changeset)` in LiveView, access via `@form[:field]` in templates. Never pass `@changeset` to templates
- **Components**: Use `<.icon>`, `<.input>`, `<.link>` components
- **Icons**: there are both Heroicons and FontAwesome icons
- **Birding Data** ([kjogvi/lib/birding.ex](./apps/kjogvi/lib/kjogvi/birding.ex)): `Kjogvi.Birding` context, `Card`/`Location` models with privacy settings (`is_private`, `is_patch`)
- **Taxonomy** ([ornithologue](./apps/ornithologue/)): Use `Kjogvi.OrnithoRepo`. Mounted at `/taxonomy` with `ornitho_web` macro
- **Streams**: Use `stream(socket, :items, list)` + `phx-update="stream"`. Not enumerable—refetch and reset to filter. Track counts separately
- **CSS**: Tailwind v4 with new import syntax (no config). Never use `@apply`. Import JS into `app.js`, not inline `<script>` tags
- **Testing**: `Phoenix.LiveViewTest` + `LazyHTML`. Assert elements by ID, not HTML. Test outcomes, not implementation

## Key File References

| File | Purpose |
|------|---------|
| [mix.exs](./mix.exs) | Umbrella config, development aliases |
| [config/config.exs](./config/config.exs) | Ornithologue & Tailwind setup |
| [apps/kjogvi_web/router.ex](./apps/kjogvi_web/lib/kjogvi_web/router.ex) | Main router, auth pipelines |
| [apps/kjogvi/lib/kjogvi/birding.ex](./apps/kjogvi/lib/kjogvi/birding.ex) | Birding context & card logic |
| [apps/kjogvi/lib/kjogvi/geo/location.ex](./apps/kjogvi/lib/kjogvi/geo/location.ex) | Location model with privacy |

## Essential Libraries

- **Req**: HTTP client (preferred over httpoison, tesla)
- **Phoenix LiveView** with streams (not regular lists)
- **Ecto** with dual repos (`Kjogvi.Repo`, `Kjogvi.OrnithoRepo`)
- **Tailwind v4**: No config, new import syntax
- **LazyHTML**: Testing assertions

## Common Pitfalls

1. Missing `@current_scope` in routes—use correct `:require_authenticated_user` pipeline
2. Passing `@changeset` to templates—use `@form` from `to_form/2`
3. Using lists instead of streams—causes memory bloat
4. Raw Heroicons/FontAwesome icons instead of `<.icon>` component  
5. Mixing `Kjogvi.Repo` vs `Kjogvi.OrnithoRepo` — migrations run separately

## Notes for AI Agents

- Add and update tests for all code changes before committing
- Update module and function documentation when making changes
- Run `mix lint.fix` to auto-fix formatting + linting before commits
- Single-user mode is default (wrap multi-user routes in `Kjogvi.Config.with_multiuser/1`)
- Coordinate changes across `kjogvi` and `ornithologue` apps when adding features
- Examine existing LiveViews in `apps/kjogvi_web/lib/kjogvi_web/live/` for patterns
- When designing frontend, always make it responsive (check on smaller screen sizes)
- Be mindful of how it will present on text-based browsers (e.g. lynx) and for screen readers
- When creating a bunch of homogenous elements (e.g. filters), implement them using `<ul>` and `<li>`, even if they are not rendered visually as a list.
<!-- usage-rules-end -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/khustochka) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
