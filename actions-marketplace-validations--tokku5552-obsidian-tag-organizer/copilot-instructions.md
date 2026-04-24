## tokku5552-obsidian-tag-organizer

> This project is a GitHub Action that helps organize tags in Obsidian markdown files. It provides automated tag management and organization capabilities.

# Obsidian Tag Organizer

This project is a GitHub Action that helps organize tags in Obsidian markdown files. It provides automated tag management and organization capabilities.

## To do nessesary
- you should use `yarn`. Don't use `npm`

## Main Components

- [src/index.ts](mdc:src/index.ts): The main entry point of the application
- [src/config.ts](mdc:src/config.ts): Configuration settings and validation
- [src/types.ts](mdc:src/types.ts): TypeScript type definitions
- [action.yml](mdc:action.yml): GitHub Action configuration

## Development Setup

1. The project uses TypeScript and is configured with:
   - ESLint for code linting ([.eslintrc.json](mdc:.eslintrc.json))
   - Prettier for code formatting ([.prettierrc](mdc:.prettierrc))
   - Jest for testing ([jest.config.js](mdc:jest.config.js))

2. Dependencies are managed with Yarn (see [package.json](mdc:package.json))

3. Git hooks are configured with Husky (see [.husky/](mdc:.husky))

## Testing

Tests are located in [src/__tests__/](mdc:src/__tests__) and can be run using:
```bash
yarn test
```

## Contributing

Please refer to [CONTRIBUTING.md](mdc:CONTRIBUTING.md) for contribution guidelines.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/actions-marketplace-validations) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
