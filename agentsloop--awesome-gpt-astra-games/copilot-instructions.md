## awesome-gpt-astra-games

> - Read [Scraping skills](wiki/scraping-skills.md) before selecting a skill for web or GitHub collection work.

# Project instructions

## Wiki

- Read [Scraping skills](wiki/scraping-skills.md) before selecting a skill for web or GitHub collection work.
- Read [Screenshot ratings](wiki/screenshot-ratings.md) before rating or ranking game images.
- Read [Search experiments](wiki/search-experiments.md) before repeating a discovery pass.
- Read [Fresh model discovery](wiki/fresh-model-discovery.md) before searching model-specific branches or commit frontiers.

## Game collections

- Add every verified game collection or awesome list you find to `awesomelists.md`.
- Record its source link, counted game entries, count method, and verification date.
- Keep collection repositories out of `games.json` unless they also contain a playable game.

## AI game generators

- Add AI game generators encountered during other research to `ai-game-generators.md` after verifying a primary source claims playable-game creation.
- Record the tool link, creation capability, access or export limits, primary evidence link, and verification date.
- Keep generator-only tools out of `games.json` unless their repository also contains a playable game.

## Game assets and resources

- Add verified game assets, asset libraries, and production tools encountered during other research to `awesome-game-resources.md`.
- Record the source link, use, license or access limits, primary evidence link, and verification date.
- Keep resource-only repositories out of `games.json`.

## Model attribution

- Include Claude Sonnet 5.5 and GPT-6.1 Sol with explicit source or creator attribution.
- Keep GPT-6.1 Sol separate from GPT-6 Astra. Distinguish authoring from runtime use and evaluation.

## Browser ports

- Read [Browser-decompiled games](browser-decompiled-games.md) before adding browser ports built from decompilations, reverse engineering, or reimplemented engines.
- Separate decomp ports from source releases, engine reimplementations, emulators, and reconstructions. Keep this catalog outside the AI-made `games.json` dataset.

## Game link collection

- Save every verified GitHub game repository link in `games.json`.
- Add a record immediately after confirming that the repository exists and contains a game or playable game prototype.
- Keep one record per repository.
- Use the repository URL as the stable identifier.
- Record the game name, repository URL, evidence URL, technology, verification status, and verification date.
- Record `added_to_repo_on` as the UTC date the repository first appears in `games.json`; preserve it when updating a record.
- Do not add prompt-only projects, catalogs, skills, or repositories without a game.
- When Reddit or X leads to a GitHub game, add the exact post URL and the repository URL to that game’s `discovery_sources` array; preserve all existing sources.
- Keep `games.json` valid JSON.
- Update existing records instead of creating duplicates.
- Mirror every verified README link refresh into the matching `games.json` record in the same change.
- Keep `live_demo_url` as the backward-compatible primary URL; store every additional verified playable URL in `live_demo_urls` and deduplicate the arrays.
- Store verified YouTube gameplay links in a `youtube_urls` array and keep every verified screenshot link in `screenshot_urls`.
- Reject dead, placeholder, documentation, development-only, and unrelated author links before recording them.
- Run `node scripts/validate-games.mjs` after every dataset edit and keep the note generator aligned with every media field.

## Git delivery

- Verify and commit every completed change.
- Regenerate `profile/README.md` with `node scripts/generate-organization-profile.mjs` after changing the root README.
- Push every completed commit to `origin`, `astra`, and `org-profile` (`https://github.com/AgentsLoop/.github.git`) before reporting completion.
- Use the default `gh` account for read-only GitHub searches; do not specify an account or override `GH_TOKEN`. Inspect `~/.config/gh/hosts.yml` and `git remote -v` before remote writes. Use the intended account and ask if it is ambiguous.

---
> Source: [AgentsLoop/awesome-gpt-astra-games](https://github.com/AgentsLoop/awesome-gpt-astra-games) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
