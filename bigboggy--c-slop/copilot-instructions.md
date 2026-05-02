## c-slop

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

C-slop is a **token-minimal programming language** for web applications that reduces token count by 75-82% compared to JavaScript. It uses symbols (`@`, `$`, `#`, `>`, `*`) instead of keywords. The repository contains:

1. **Backend Compiler** (`compiler/`) - Compiles `.slop` files to Node.js/Express
2. **Frontend Compiler** (`compiler/frontend/`) - Compiles `.ui` component files to vanilla JavaScript with reactive signals
3. **Documentation Website** (`docs/`) - Docusaurus v3.9.2 site at https://c-slop.dev
4. **Example App** (`compiler/examples/`) - Full-stack sample application

## Commands

### Backend Compiler
```bash
cd compiler
npm install
npm link                    # Install 'cslop' command globally

cslop create my-app         # Create a new project
cslop start                 # Compile frontend + run backend
cslop watch                 # Start with hot reload (WebSocket-based)
cslop app.slop              # Run a .slop file (starts Express server on :3000)
cslop build app.slop -o out.js  # Compile to JavaScript

npm test                    # Run tests (node test/test.js)
```

### Frontend Compiler
```bash
cd compiler/frontend
node cli.js <input-dir> <output-dir>    # Compile .ui files to JS

# Example:
node cli.js ../examples/components ../examples/public
```

### Documentation Site
```bash
cd docs
npm install
npm start                   # Dev server at http://localhost:3000
npm run build               # Generate static site
npm run typecheck           # TypeScript type checking
```

## Architecture

### Backend Compiler (`compiler/src/`)
- `cli.js` - CLI entry point with commands: `create`, `start`, `watch`, `build`, and direct `.slop` execution
- `compiler.js` - Parser that transforms `.slop` syntax to JavaScript/Express code
- `runtime.js` - Express.js wrapper with database abstraction (in-memory or SQLite)
- `index.js` - Module exports

Configuration via `slop.json`:
```json
{
  "database": { "type": "sqlite", "connection": "./dev.db" },
  "server": { "port": 3000 }
}
```

### Frontend Compiler (`compiler/frontend/`)
- `cli.js` - CLI entry point
- `parser.js` - Parses `.ui` component syntax (state, effects, markup, events, loops)
- `codegen.js` - Generates JavaScript with hyperscript DOM and reactive signals
- `router-parser.js` - Parses `router.slop` files for client-side routing
- `router-codegen.js` - Generates router code

### Runtime (`compiler/runtime/`)
- `signals.js` (~1.5KB) - Reactive state: `signal()`, `computed()`, `effect()`
- `dom.js` (~1KB) - Hyperscript helpers: `h()`, `mount()`, `list()`, `navigate()`
- `router.js` (~1KB) - Client-side SPA router with `$route` signal

### SlopUI (`compiler/slopui/`)
- `base.css` - CSS reset, theme variables, utility classes
- `components.css` - Pre-built component styles (buttons, cards, inputs, alerts)
- `theme.js` - Theme CSS generator from `slop.json` config

### Documentation (`docs/`)
- `docusaurus.config.ts` - Site config (dark mode default, blog disabled)
- `docs/` - Markdown documentation content (auto-generates sidebar)
- `src/pages/index.tsx` - Custom homepage
- `src/components/HomepageFeatures/` - Homepage feature cards

## Language Syntax Reference

### Backend (.slop files)
```
*           Route definitions (GET, POST+, PUT^, DELETE-)
@           Database operations
$           Request data ($.id, $.body, $.query)
#           Response (#json, #html, #201, #404)
>           Pipeline operator
?           Query/filter (@users?{active:true})
!           Mutations (@users!$.body for insert)
```

Route examples:
```cslop
*/users > @users > #json                    # GET /users
*/users/:id > @users[$.id] > #json          # GET /users/:id
*/users + @users!$.body > #json             # POST /users
*/users/:id ^ @users[$.id]!$.body > #json   # PUT /users/:id
*/users/:id - @users[$.id]!- > #204         # DELETE /users/:id
```

### Frontend (.ui files)
```
$count:0           State declaration
$doubled := expr   Computed state
~                  Effects (~ fetch("/api") > $data)
<?                 Template separator (state above, markup below)
@{$var}            Reactive interpolation
@@ComponentName    Use component (auto-imports)
@                  Event handlers (button["Click" @ $count++])
?                  Conditionals (? $loading)
:property          Property access in loops (:name, :email)
```

Component example (Main.ui):
```
<?

div.container
  h1["My App"]
  @@Counter
```

Component example (Counter.ui):
```
$count:0

<?

div.counter
  h1["Count: @{$count}"]
  button["+" @ $count++]
  button["-" @ $count--]
```

## Key Files for Understanding the Codebase

- `C-SLOP.md` - Complete language specification
- `compiler/README.md` - Backend compiler documentation
- `compiler/FRONTEND-STATUS.md` - Frontend compiler implementation status
- `compiler/DATABASE.md` - Database configuration guide
- `compiler/examples/` - Example `.slop` and `.ui` files

## Node Version

Requires Node.js >= 20.0

---
> Source: [bigboggy/C-slop](https://github.com/bigboggy/C-slop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
