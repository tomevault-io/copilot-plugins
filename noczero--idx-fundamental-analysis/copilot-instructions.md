## idx-fundamental-analysis

> Generates a Svelte Playground link with the provided code.


# Overview

IDX Fundamental Analysis project aims to retrieve and analyse fundamental stock data of companies listed on the
Indonesian Stock Exchange (IDX).

# Runtime Version

- Python 3.14
- Node v24

# Stack

- API: FastAPI (under ./app/api)
- UI: Svelte (under ./app/ui)

# Database

- DB: SQLite
- ORM: SQLAlchemy

# Code Convention

- use asynchronus process instead of synchronus.
- global import is priority instead of local import.
- do not overcode with try and except.

## Svelte UI pattern:

- components: PascalCase.svelte
- routes: SvelteKit defaults (+page.svelte, etc.), route folders kebab-case
- Lib modules (clients, services, types): camelCase.ts

# Layer Architecture

This project uses workflow like this:

- view/controller: router api (`./app/api/routers`)
- data transmit/dto: schema (`./schemas`)
- business logic/ service+impl: service (`./services`)
- data access/dao/mapper: repository (`./repositories`)
- model/entity: model (`./db/models`)

# Representations

- Represents data structure as @dataclass inside `./schemas`
- Represents external system class on `./providers`
- Represents db session in `./db/session.py`. Use `get_session` as context manager.
- Represents builders for outputing the result in the excel or spreadsheet.
- Represents business logic inside service class on `./services`

# Tests

- Always generate test case for code you generate except for UI.
- Create test on `./tests/` folder.
- Make sure the test is pass using pytest.

# Constraints

- always define asynchronus function in router
- Put the docs or unecessary file you generated under `./tmp`

# Command Line

- nvm use first for UI CLI.
- Use .venv first for api CLI.

# MCPs

You are able to use the Svelte MCP server, where you have access to comprehensive Svelte 5 and SvelteKit documentation. Here's how to use the available tools effectively:

## Available MCP Tools:

### 1. list-sections

Use this FIRST to discover all available documentation sections. Returns a structured list with titles, use_cases, and paths.
When asked about Svelte or SvelteKit topics, ALWAYS use this tool at the start of the chat to find relevant sections.

### 2. get-documentation

Retrieves full documentation content for specific sections. Accepts single or multiple sections.
After calling the list-sections tool, you MUST analyze the returned documentation sections (especially the use_cases field) and then use the get-documentation tool to fetch ALL documentation sections that are relevant for the user's task.

### 3. svelte-autofixer

Analyzes Svelte code and returns issues and suggestions.
You MUST use this tool whenever writing Svelte code before sending it to the user. Keep calling it until no issues or suggestions are returned.

### 4. playground-link

Generates a Svelte Playground link with the provided code.
After completing the code, ask the user if they want a playground link. Only call this tool after user confirmation and NEVER if code was written to files in their project.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/noczero) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
