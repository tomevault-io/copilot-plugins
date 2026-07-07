## linecode

> LineCode is a React Native TypeScript app. App entry points live in `index.js` and `App.tsx`; native projects are under `android/` and `ios/`. Main application code is in `src/`: `components/` for UI, `screens/` for route-level views, `hooks/` for reusable state logic, `services/` for storage and AI providers, `mcp/` for tool execution, `theme/` for colors, and `types/` for shared TypeScript contracts. Static prompt and image assets live in `src/assets/` and `res/`. Tests currently live in `__tests__/`.

# Repository Guidelines

## Project Structure & Module Organization

LineCode is a React Native TypeScript app. App entry points live in `index.js` and `App.tsx`; native projects are under `android/` and `ios/`. Main application code is in `src/`: `components/` for UI, `screens/` for route-level views, `hooks/` for reusable state logic, `services/` for storage and AI providers, `mcp/` for tool execution, `theme/` for colors, and `types/` for shared TypeScript contracts. Static prompt and image assets live in `src/assets/` and `res/`. Tests currently live in `__tests__/`.

## Build, Test, and Development Commands

- `npm start`: builds the system prompt first, then starts Metro.
- `npm run android`: builds the prompt and runs the Android app.
- `npm run ios`: builds the prompt and runs the iOS app.
- `npm run build-prompt`: regenerates `src/constants/prompt.ts` from prompt assets.
- `npm run build-hot-update -- --changelog "<summary>"`: syncs versions, builds the prompt, and generates `dist/hot-update/base.zip`, `base.txt`, and `base-{hotUpdateVersionCode}.txt` whose contents are JSON. For APK updates, pass `--apk-update` with `--remote-apk <apk>` and/or `--local-apk <apk>`; the script emits XOR-encrypted `base-remote.enc` / `base-local.enc` files for upload.
- `npm test`: runs Jest with the React Native preset.
- `npm run lint`: runs ESLint across the repository.

Use Node `>=22.11.0`, as declared in `package.json`.

Manage app, native, and hot update versions only through `version.json`; run `npm run sync-version` or a build command after changing it. When generating a hot update, increment `hotUpdateVersionCode` and provide a meaningful changelog with `--changelog` or `--changelog-file` that describes the actual change. For APK updates, also increment the native APK version fields as appropriate and upload the generated `.enc` APK packages through the hot update manager. Do not leave the default `Hot update <version>` text, because it is shown to users and is not useful for testing or release notes.

## Coding Style & Naming Conventions

Use TypeScript and React function components. Follow the existing two-space indentation, single quotes, and semicolon style enforced by ESLint/Prettier config. Name components in PascalCase, hooks as `useSomething`, services as `SomethingService`, and shared interfaces/types in `src/types`. Prefer existing UI primitives and theme tokens from `src/constants/theme.ts` and `src/theme/` over inline styling.

## Testing Guidelines

Jest is configured through `jest.config.js` with `@react-native/jest-preset`. Add tests under `__tests__/` or beside modules using `*.test.ts` / `*.test.tsx`. Focus tests on stateful hooks, service behavior, message processing, and tool execution edge cases. Run `npm test` before submitting changes that affect logic.

## Commit & Pull Request Guidelines

Recent commits use Conventional Commit style, often with scopes, for example `feat(agent): ...`, `feat(mcp): ...`, and `feat: ...`. Prefer concise messages in the form `type(scope): summary`, using `feat`, `fix`, `refactor`, `test`, or `docs`.

Pull requests should include a short description, key files changed, test results, and screenshots or recordings for UI changes. Mention any native Android/iOS changes, storage migrations, or AI provider behavior changes explicitly.

## Security & Configuration Tips

Do not commit API keys, local logs, generated APKs, or private workspace data. Treat AI provider settings, filesystem tools, and MCP permissions as sensitive surfaces; document any behavior that writes files, starts servers, or sends reasoning/history back to model APIs.

---
> Source: [LangLang03/LineCode](https://github.com/LangLang03/LineCode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-06 -->
