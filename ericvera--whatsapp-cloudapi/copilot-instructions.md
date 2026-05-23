## tests-commands

> Use yarn instead of npm for all test commands

# Rules

## Use yarn for test commands
When running test commands, always use `yarn` instead of `npm`.

# Examples

## Valid
```bash
yarn test
cd functions && yarn test
yarn test src/utils/usage/updateDailyUsage.test.ts
```

## Invalid
```bash
npm test
cd functions && npm test
npm test src/utils/usage/updateDailyUsage.test.ts
```

---
> Source: [ericvera/whatsapp-cloudapi](https://github.com/ericvera/whatsapp-cloudapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
