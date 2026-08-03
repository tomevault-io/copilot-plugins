## yorushiro

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

```bash
./gradlew bootJar                    # Build fat jar
java -jar build/libs/KanadeBot-1.0-SNAPSHOT.jar

./gradlew runPreview                 # Render JrrpBoard preview PNGs locally
./gradlew runCommitPreview           # Render GitHub commit card preview
./gradlew runGithubInfoPreview       # Render GitHub repo info card preview
./gradlew verifyConfigSchema         # Verify ConfigManager schema auto-fill
```

Tests are disabled (`tasks.test.enabled = false`). During development, run `KanadeBotApplication.main` directly from the IDE.

No lint/format tools are configured.

## Architecture

**Entry point**: `KanadeBotApplication.main` — enables Java2D hardware acceleration (Metal on macOS, OpenGL otherwise), then boots Spring Boot.

**Core**: `KanadeBot` is a Shiro `BotPlugin` (`@Component`, `@Shiro`). On `@PostConstruct` it initializes all managers and registers commands. `onGroupMessage` delegates to `CommandManager.process()`.

### Command system

- `ICommand` interface: `val data: CommandData` + `fun process(bot, event, args)`
- `CommandData` holds name, description, usage, aliases, and `restricted: Boolean` (for admin-only badge in help card)
- Registration is manual in `KanadeBot.initialize()` via `CommandManager.register(...)`
- `CommandManager` (object singleton) holds `commands: Map<name, ICommand>` and `aliases: Map<alias, name>`
- Two trigger modes: `.command` prefix or `@bot command` (OneBot CQ code parsing)
- `AdminCheck.kt` utilities: `isPrivilegedUser(event)` returns true for group admins/owner OR bot admins listed in `kanade.json#botAdmins`

### Managers (all `object` singletons)

| Manager | Responsibility |
|---|---|
| `ConfigManager` | Read/write `kanade.json` (deepSeekApiKey, githubToken, botAdmins). Auto-fills missing keys on startup. |
| `JrrpManager` | Daily 0–100 fortune per user. Persisted to `jrrp.json` (user values) and `jrrp_groups.json` (group participation). Global value per user; leaderboard per-group. |
| `GitHubManager` | Repo commit subscription. 60s poll loop with coroutines. Tracks last known SHA per repo, pushes new commits as rendered PNGs to subscribed groups. Persisted to `github_subs.json`. Content compliance check via DeepSeek on first poll. |

### Image renderers (all `object` singletons, return `String` = base64 PNG)

- `HelpImageRenderer` — command list card with fingerprint-based memory cache in `HelpCommand`
- `GitHubRepoInfoRenderer` — repo overview (stars, forks, language, latest commit)
- `GitHubCommitInfoRenderer` — single commit card (sha, title, author, stats)
- `JrrpBoardRenderer` — daily fortune leaderboard table

All use Java2D (Material 3 dark palette), no external image libraries. Font stack: PingFang SC → Source Han Sans SC → Noto Sans CJK SC → fallback chains to Sans Serif.

### API / IO utilities

- `GitHubApi` — shared `java.net.http.HttpClient` wrapper for GitHub REST v3. Reads `githubToken` from config for auth.
- `DeepSeekClient` — content compliance check via DeepSeek Chat API (`deepseek-chat` model, JSON response format). Called before showing/subscribing to repos. Returns null (skip) if no API key configured.
- `AvatarProvider` — QQ avatar fetcher with disk cache (`resource/avatar/{userId}.png`)

### Config files

- `src/main/resources/application.yml` — Shiro WebSocket connection (url, access-token)
- `kanade.json` (working dir, auto-generated) — runtime config
- `github_subs.json` (working dir, auto-generated) — subscription state
- `jrrp.json` / `jrrp_groups.json` (working dir, auto-generated) — fortune state

### Key patterns

- Messages sent as base64 PNG: `MsgUtils.builder().img("base64://$base64").build()` for image cards, or `Bot.replyGroupMsg(event, text)` for plain text (extension in `ExtendsFunction.kt`)
- Long-running work (GitHub API calls) is launched on `GlobalScope.launch` to avoid blocking the Shiro WebSocket thread
- All JSON uses Fastjson2 (`JSON.parseObject`, `JSONObject`, `JSONArray`)
- `ConfigManager.getConfig()` is the single source of runtime config; GitHub/DeepSeek clients read tokens from it

---
> Source: [NightcordSekai/Yorushiro](https://github.com/NightcordSekai/Yorushiro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
