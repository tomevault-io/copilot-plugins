## deeperseeker-app

> This is AI chat app called DeeperSeeker.

This is AI chat app called DeeperSeeker.

Here are some pieces of information you need to know about this project:
• React Native v0.76 is used as the framework with Expo SDK v52 on top of it
• Expo Router is used for routing with file based routing in the `./app` directory
• The database is a local SQLite database in `./utils/Database.ts`
• The components are in `./components` directory
• Auth is setup with Clerk
• Use `bun` instead of `npm` and `bunx` instead of `npx` for all package management and execution
• Common commands are in the scripts section of the `package.json` file
  • To build the project fresh on the iOS simulator use `bun ios` and make sure to prebuild the project first with `bun prebuild` since this is an expo-dev-client project
  • To start the project without building run `bun start`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ansh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
