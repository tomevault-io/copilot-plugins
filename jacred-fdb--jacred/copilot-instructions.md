## jacred

> - This is the Russian documentation site for [JacRed](https://github.com/jacred-fdb/jacred).

# Documentation project instructions

## About this project

- This is the Russian documentation site for [JacRed](https://github.com/jacred-fdb/jacred).
- The site uses Mintlify. Pages are MDX files with YAML frontmatter.
- Mintlify root is this `docs/` directory: `docs.json`, `.mintignore`, and all MDX live here.
- The parent repository is the product source of truth: application code, `Configuration/AppOptions.cs`, `Configuration/Schema/ConfigSchema.cs`, `Data/crontab`, `Data/init.yaml`, `Data/example.yaml`, and `web/public/openapi.yaml`.
- `openapi/openapi.yaml` and branded images are deployment copies of canonical files under `../web/public/`.
- Update the copies manually when their canonical `web/public` files change. Do not add a synchronization script or workflow.
- Public DeepWiki is not a source of truth. Do not copy Mintlify MDX into `.devin/wiki.json`. That file steers a 30-page English code wiki; operator docs stay on this Mintlify site.
- Use the Mintlify docs MCP server to verify current components and configuration.
- In the Mintlify dashboard the content path must be `/docs`.

## Terminology

- Write the product name as “JacRed”.
- Use “трекер”, “раздача”, “FileDB”, “поисковый API”, and “веб-интерфейс”.
- Keep API names in English: Jackett, Torznab, Prowlarr, OpenAPI.
- Use `apikey` for search access and `devkey` for administrative access.

## Style preferences

- Write in Russian using active voice and second person (“вы”).
- Keep sentences concise. Use one idea per sentence.
- Use sentence case for headings.
- Bold UI labels: нажмите **Настройки**.
- Use code formatting for file names, commands, paths, options, and endpoints.
- Use root-relative internal links without the `/docs/` prefix and without `.mdx`.
- Add `title`, `sidebarTitle`, `description`, and useful `keywords` to pages.
- Prefer Mintlify components when they improve scanning, not for decoration.
- Give every code fence a language. Keep examples runnable against port `9117`.

## Content boundaries

- Never publish real API keys, tokens, passwords, cookies, wallet secrets, or private URLs.
- Treat `../web/public/openapi.yaml`, `../Data/crontab`, `../Configuration/Schema/ConfigSchema.cs`, and `../Infrastructure/Security/` as authoritative.
- Distinguish three default layers. Never mix them silently:
  - C# field initializers in `../Configuration/AppOptions.cs` (used when a key is absent from the runtime file);
  - packaged template `../Data/init.yaml` (empty secrets, conservative sync);
  - full example `../Data/example.yaml` (illustrative values, including a sample `syncapi`).
- Runtime config is CWD `init.yaml` or `init.conf`, not `Data/init.yaml`.
- Document `/dev/*` and destructive maintenance actions as operator-only features with warnings.
- Do not describe retired tracker icons as active trackers.
- Keep generated OpenAPI reference separate from task-focused guides.
- Do not invent defaults. Verify them in source code or the matching config layer.
- Before completion, run `mint validate`, `mint broken-links --check-anchors`, and `mint a11y` from `docs/`.

---
> Source: [jacred-fdb/jacred](https://github.com/jacred-fdb/jacred) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
