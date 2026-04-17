## personal-ai

> @manifest.json = src/manifest.json

@manifest.json = src/manifest.json 
@webpack.config.js = webpack.common.cjs
@autoupdate.mdc = docs/autoupdate.mdc
@autoload = autoupdate.mdc
docs: If code contains a lot of docstrings, surface them to the user.
doc-update: When discussing features described in docs/, Claude should check if discussion contains updates that should be reflected in documentation and offer to update relevant .mdc files.
slides-analyzer: If discussing changes to google_slides_analyzer feature, check if docs/features/google_slides_analyzer.mdc needs to be updated.
app-script-version: When modifying src/scheduled-messages/app-script-template.gs, MUST update APP_SCRIPT_VERSION and APP_SCRIPT_LAST_UPDATED at the top of the file. For bug fixes or minor changes, increment patch version (e.g., 2.0.0 -> 2.0.1). For new features, increment minor version (e.g., 2.0.1 -> 2.1.0). For breaking changes, increment major version (e.g., 2.1.0 -> 3.0.0).
dev-workflow: After modifying TypeScript/JavaScript files in src/, prefer running `npm start` for development verification (webpack watch mode with faster rebuilds). Only use `npm run build` for production builds or when explicitly requested by user.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ee01) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
