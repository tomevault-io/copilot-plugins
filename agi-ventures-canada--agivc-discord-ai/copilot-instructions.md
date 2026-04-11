## agivc-discord-ai

> **Always use `bun`** for package management and script execution:

# AGENTS.md - Discord AI Bot

## Package Manager

**Always use `bun`** for package management and script execution:

```bash
bun install
bun add <package>
bun run dev
bun run test
```

## Running Commands

When asked to run shell commands, run them automatically without asking for confirmation.

## Testing

**Always update tests when modifying code.** Tests live in the `/tests` directory, mirroring the `src/` structure.

- When adding new functions or modules, add corresponding tests
- When modifying existing code, update related tests to cover the changes
- When fixing bugs, add a test that would have caught the bug
- **Always run `bun run test` after writing code** to verify all tests pass

Test files follow the pattern `tests/<path>/<module>.test.ts`.

When adding a new user-facing bot capability, also add a manual way to test it from Discord.

- Prefer a slash command for manual verification
- For cron and automation features, make sure they can be triggered via `/test_cron <job-id>` or a dedicated test slash command
- If you add or change slash commands, update `scripts/register-commands.ts`
- If you add or change slash commands, update `tests/lib/discord/commands.test.ts`

## Cron Architecture

Cron in this repo is **code-defined and Redis-backed**.

- App cron definitions live in `src/config/crons.ts`
- Cron execution logic lives in `src/lib/cron/executor.ts`
- Cron scheduling state and locking live in `src/lib/cron/index.ts`
- The cron entrypoint is `src/cron/run.ts`, invoked by `bun run cron:tick`

Production behavior:

- `agivc-discord-ai` is the long-running bot/web service
- `agivc-discord-ai-cron` is a separate Railway cron service
- Railway only wakes the app scheduler on `*/5 * * * *`
- Redis stores cron state like `nextRunAt` and prevents duplicate runs with locks

Do not regress this back to a pure in-process scheduler in production.

- In-process cron is only a fallback when Redis is absent or `ENABLE_IN_PROCESS_CRONS=true`
- Do not create one Railway cron per configured app job unless the architecture is being intentionally changed
- If you change production deploy behavior, keep `railway.web.toml` and `railway.cron.toml` in sync with the intended service setup
- If you add or change cron definitions, update the cron tests too

The cron service needs its own Railway variables. At minimum it requires:

- `DISCORD_BOT_TOKEN`
- `DISCORD_PUBLIC_KEY`
- `DISCORD_GUILD_ID`
- `REDIS_URL`
- `AI_GATEWAY_API_KEY`

Depending on the configured jobs, it may also require:

- `TAVILY_API_KEY`
- `LUMA_API_KEY`
- `LUMA_CALENDAR_ID`

When the Railway cron deployment check is failing but CI is green, treat it as an ops issue first.

- Check `railway service status -a -e production`
- Check `railway deployment list -s agivc-discord-ai-cron -e production`
- Check `railway logs -s agivc-discord-ai-cron -e production --deployment --latest --lines 200`
- Compare env var presence between `agivc-discord-ai` and `agivc-discord-ai-cron`
- The previous real failure here was a missing `DISCORD_PUBLIC_KEY` on the cron service
- Verify the cron service is still attached to `/railway.cron.toml`; the Railway CLI may not expose the config-file binding directly, so use the dashboard or Public API when needed
- After fixing envs or config drift, redeploy `agivc-discord-ai-cron` and verify the Railway check turns green
- Never paste secret values from Railway env var output into chat

For the full step-by-step operational workflow, use the repo-local skill at `.agents/skills/railway-cron-ops/`.

## AI Models

**Never change user-specified AI model names** unless explicitly told to do so. The current model is `openai/gpt-5.1`.

## Commit Conventions

Follow conventional commit format for all commit messages:

- `feat:` - New features
- `fix:` - Bug fixes
- `docs:` - Documentation changes
- `style:` - Code style changes (formatting, whitespace)
- `refactor:` - Code refactoring without changing functionality
- `test:` - Adding or updating tests
- `chore:` - Maintenance tasks, dependency updates
- `perf:` - Performance improvements
- `ci:` - CI/CD configuration changes

Example: `feat: add user authentication endpoint`

## Code Style & Patterns

### General Guidelines

- **No inline comments** - Keep code clean and self-documenting
- Use TypeScript with ES modules (extensionless imports, bundled with tsup)
- Export functions and classes, not default exports
- Use `interface` for object shapes, `type` for unions/aliases

### Dependencies & Stack

- **Runtime**: Node.js with TypeScript
- **Discord**: discord.js v14
- **AI**: Vercel AI SDK 6 Beta (`ai@beta`, `@ai-sdk/openai@beta`) with ToolLoopAgent
- **HTTP**: Hono framework with @hono/node-server
- **Config**: dotenv for environment variables

### Coding Patterns

- Use `const` by default, `let` when reassignment is needed
- Async/await for all asynchronous operations
- Early returns for guard clauses
- Type imports with `import type` when only importing types
- String literals for simple types, template literals for interpolation

### Environment Variables Gotcha

**ES modules hoist imports before any code runs.** This means `dotenv.config()` executes _after_ all module-level code in imported files.

**Bad** - reads env var at module load (before dotenv):

```typescript
const API_KEY = process.env.API_KEY; // Always undefined!

export function doThing() {
  return API_KEY;
}
```

**Good** - reads env var at runtime (after dotenv):

```typescript
export function doThing() {
  const apiKey = process.env.API_KEY; // Works!
  return apiKey;
}
```

Always read `process.env` inside functions, not at module scope.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AGI-Ventures-Canada)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AGI-Ventures-Canada)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
