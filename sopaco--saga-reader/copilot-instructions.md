## saga-reader

> > This file defines the working rules for AI Agents in this project. The project knowledge base lives in `.ai-context/`.

# AGENTS.md — Saga Reader Project Guidelines

> This file defines the working rules for AI Agents in this project. The project knowledge base lives in `.ai-context/`.

---

## 1. Session Startup Flow

At the beginning of every session, read in order:

1. **`.ai-context/SKILL.md`** — get the project overview and key paths
2. **`.ai-context/references/PROJECT-ESSENCE.md`** — understand project positioning and core constraints
3. Read selectively based on task type:
   - Cross-component development → `ARCHITECTURE.md`
   - Modifying design patterns → `DECISIONS.md`
   - Troubleshooting → `DYNAMICS.md`

---

## 2. Project Structure & Working Conventions

### Tech Stack

| Layer | Technology | Notes |
|-------|-----------|-------|
| Desktop framework | Tauri 2.x | Rust backend + Web frontend |
| Frontend | SvelteKit 2.x + Svelte 5 | Static adapter, TailwindCSS styling |
| UI component library | Skeleton 3.x | See `app/src/lib/widgets/` |
| Backend | Rust (edition 2024) | Cargo workspace |
| Database | SQLite + SeaORM | Local storage, see `crates/recorder/` |
| Package manager | Bun (recommended) / npm / pnpm | Bun mirror config in `bunfig.toml` |

### Common Commands

```bash
bun install          # Install dependencies
bun run dev          # Start development (Tauri dev mode)
bun run build        # Build the desktop app
bun run build:macos  # Build macOS Universal
```

### Code Conventions

- **Rust**: Follow `clippy.toml`; use `anyhow` for error handling; use `spdlog-rs` for logging
- **TypeScript/Svelte**: ESLint + Prettier configs under `app/`; strict mode
- **Prompt templates**: Store as `.prompt` files (see `crates/intelligent/src/article_processor/`), not inlined in Rust code
- **i18n**: New user-visible text must be added to `app/src/lib/i18n/locales/zh.json` and `en.json`

---

## 3. Architectural Key Points

- **Single entry point**: All backend capabilities are exposed through the `feed_api_rs::FeaturesAPI` trait — do not call sub-crates directly
- **Frontend call chain**: Svelte component → `app/src/lib/hybrid-apis/` (Tauri command bindings) → `tauri-plugin-feed-api` → `feed_api_rs`
- **Provider enum dispatch**: LLM and search Providers use enums, not `dyn Trait`; adding a new Provider requires modifying the enum and its match arms
- **All user data stays local**: SQLite database, no remote sync; the `recorder` crate handles persistence

---

## 4. External Dependency Lookup: Use DeepWiki MCP

When you encounter an **unfamiliar external dependency**, you **must** use the DeepWiki MCP tools to look it up — do not guess.

### Available Tools

| Tool | Purpose | Example |
|------|---------|---------|
| `deepwiki:read_wiki_structure` | View a repo's documentation structure | `tauri-apps/tauri` |
| `deepwiki:read_wiki_contents` | Read a repo's full documentation | `SeaQL/sea-orm` |
| `deepwiki:ask_question` | Ask a specific question about a repo | `"sveltejs/kit", "How to configure the SvelteKit static adapter?"` |

### When to Use

- **Required**: When modifying behavior that involves third-party library APIs you are not familiar with
- **Recommended**: Tauri plugin development, SeaORM entity definitions, SvelteKit routing/load, and other framework-specific features
- **Not needed**: Standard Rust/TypeScript syntax, the project's own code logic

### DeepWiki Repo Names for Key External Dependencies

| Dependency | DeepWiki repo name |
|-----------|-------------------|
| Tauri | `tauri-apps/tauri` |
| SvelteKit | `sveltejs/kit` |
| Svelte | `sveltejs/svelte` |
| Skeleton UI | `skeletonlabs/skeleton` |
| SeaORM | `SeaQL/sea-orm` |
| reqwest | `seanmonstar/reqwest` |
| tokio | `tokio-rs/tokio` |
| scraper | `causal-agent/scraper` |
| spdlog-rs | `SergioBenitez/spdlog-rs` |
| svelte-i18n | `kaisermann/svelte-i18n` |

### Usage Examples

```
# Want to learn how to register commands in a Tauri 2.x plugin
deepwiki:ask_question(repoName="tauri-apps/tauri", question="How to register custom commands in a Tauri 2.x plugin?")

# Want to learn about SeaORM's DeriveEntityModel and relations
deepwiki:ask_question(repoName="SeaQL/sea-orm", question="How to define an entity with DeriveEntityModel and relations?")
```

---

## 5. New Feature Checklist

When adding a new feature, make sure:

- [ ] Backend logic is exposed through the `FeaturesAPI` trait (`crates/feed_api_rs/src/features/api.rs`)
- [ ] The frontend calls Tauri commands through `hybrid-apis/`, not by importing Rust crates directly
- [ ] If a new LLM/Search Provider is involved, update the corresponding enum and match arms
- [ ] If new user-visible text is involved, update both `zh.json` and `en.json`
- [ ] If non-obvious design decisions are involved, update `.ai-context/references/DECISIONS.md`

---
> Source: [sopaco/saga-reader](https://github.com/sopaco/saga-reader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
