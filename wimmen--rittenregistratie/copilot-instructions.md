## rittenregistratie

> The Xelflow web project is a web application that uses Xelflow Server Flow Engine to create a web application.

# Project Overview

The Xelflow web project is a web application that uses Xelflow Server Flow Engine to create a web application.
The Xelflow Server Flow Engine runs on Azure using Azure Web App.
The Xelflow Server Flow Engine uses Azure SQL Database to store data and flows.
Every webapplication uses a schema in the database.
The Xelflow web project runs on Azure Static Web Apps.

This is a Xelflow web project divided into several distinct parts:

- **src**: The frontend application (Vanilla HTML/CSS/JS).
- **api**: The backend API (Azure Functions Node.js v4).
- **flows**: Business logic defined in JSON flows.
- **tables**: Database schema defined in JSON tables.
- **tables/migrations**: Database migrations defined in JSON files.

## Development

- Run `npm start` in the root to start the Static Web Apps (SWA) emulator.
- Run `npm run sync-flows` to sync flows to the server (uses `flows/sync.js`).
- Run `npm run sync-tables` to sync tables to the server (uses `tables/migrations/sync.js`).

## General Guidelines

- **Simplicity**: Prefer simple, readable solutions. API is minimal, Logic is in Flows.
- **Consistency**: Follow existing naming conventions and file structures.
- **Absolute Paths**: Always use absolute paths when referencing files.

---
> Source: [Wimmen/rittenregistratie](https://github.com/Wimmen/rittenregistratie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
