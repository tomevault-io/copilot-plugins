## todo-swe-1-5

> This rule enforces the use of Bun as the package manager for this project. All package management operations should be performed using Bun commands rather than npm or yarn.


This rule enforces the use of Bun as the package manager for this project. All package management operations should be performed using Bun commands rather than npm or yarn.

**Commands to use:**
- `bun install` instead of `npm install` or `yarn install`
- `bun add <package>` instead of `npm install <package>` or `yarn add <package>`
- `bun run <script>` instead of `npm run <script>` or `yarn <script>`
- `bun test` instead of `npm test` or `yarn test`

**Why Bun?**
- Faster installation and execution
- Built-in TypeScript support
- Compatible with npm packages
- Smaller memory footprint
- First-class support in this project template

**Note:** This rule is enforced by the project configuration and will be checked automatically during development.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lassestilvang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
