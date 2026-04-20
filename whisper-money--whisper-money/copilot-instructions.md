## whisper-money

> - Make sure that CI steps are passing (`.github/workflows/ci.yml`).

# After each change
- Make sure that CI steps are passing (`.github/workflows/ci.yml`).
  - bun install --frozen-lockfile
  - composer install --no-interaction --prefer-dist --optimize-autoloader
  - bun run build
  - ./vendor/bin/pest
  - vendor/bin/pint --test
  - bun run format
  - bun run lint

---
> Source: [whisper-money/whisper-money](https://github.com/whisper-money/whisper-money) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
