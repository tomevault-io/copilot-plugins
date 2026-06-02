## cli

> The cli is written in TypeScript and uses Node.js. It is hosted in `./cli` directory. This directory also

The cli is written in TypeScript and uses Node.js. It is hosted in `./cli` directory. This directory also 
contains the vscode extension to interact with the flapi server.

The vscode extension and the cli should both share the same API client, harmonizing communication between the
backend config service, authentication with the config service token and building and resolving of URLs.

Both vscode extension and cli should be always interact with the flapi server using the config service, never
directly via files.
    

---
> Source: [DataZooDE/flapi](https://github.com/DataZooDE/flapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
