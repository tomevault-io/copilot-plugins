## discord-net-bot

> - Use the existing solution file `Develop.sln`.

# Guidelines for Contributors

## General
- Use the existing solution file `Develop.sln`.
- Keep commits focused and include descriptive messages.
- Do not commit secrets or tokens.

## Coding Style
- This project targets .NET 8.
- Format C# code with `dotnet format` before committing.
- Prefer dependency injection and modular design as used in `TsDiscordBot.Core`.

## Testing
- Run `dotnet test` at the repository root.
- If you modify files under `web/`, run `npm run lint` in that directory.

## Environment Variables for Local Runs
- `DISCORD_TOKEN` – Discord bot token.
- `OPENAI_API_KEY` – OpenAI API key.
- `OPENAI_PROMPT` – Optional system prompt for AI behaviour.
- `LITEDB_PATH` – Optional path for LiteDB storage.

---
> Source: [p4j4dyxcry/Discord.NET-bot](https://github.com/p4j4dyxcry/Discord.NET-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
