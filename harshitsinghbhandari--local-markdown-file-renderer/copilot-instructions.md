## local-markdown-file-renderer

> Build, maintain, and use a local browser-based Markdown renderer that reads arbitrary Markdown files from disk on every refresh. Use this skill whenever the user wants Markdown files to behave like local HTML files in a browser, wants agents to edit Markdown while the browser reloads rendered output, or asks to improve this local Markdown rendering repo.


# Local Markdown File Renderer

Use this skill when working on this repository or recreating its workflow elsewhere. The project exists to make Markdown files feel like HTML files opened in a browser: edit the file on disk, refresh the browser, and see the latest rendered output.

## Product Intent

The user wants a local tool, not a hosted publishing system. Preserve these properties:

- The renderer must accept Markdown files from arbitrary local paths.
- The renderer must accept HTTP(S) Markdown URLs and cache them locally.
- A normal browser refresh must read the latest file contents from disk.
- For remote URLs, a normal browser refresh should try to fetch the latest URL contents and fall back to the cached copy when offline.
- The file path should live in the URL so the browser page is bookmarkable and reloadable.
- The UI should be direct and utilitarian: choose a path, render it, refresh it.
- The server should bind to localhost by default.

## Current Architecture

- `server.js` starts an Express server and exposes `/api/render?file=...`.
- `cli.js` exposes the `mdview` command for daemon control.
- `/api/render` resolves the requested path, reads the file fresh from disk, renders Markdown with `markdown-it`, and returns HTML plus file metadata.
- Remote HTTP(S) URLs are cached under `~/.mdview/cache/<hash>.md`, with URL metadata in `~/.mdview/cache/<hash>.json` and `~/.mdview/cache/index.json`.
- `public/app.js` stores the active file path in the URL, calls `/api/render`, and injects the rendered HTML into the page.
- `public/index.html` and `public/styles.css` provide the browser UI.

## Run Workflow

Install dependencies:

```sh
npm install
```

Render a specific file:

```sh
mdview up /absolute/path/to/file.md --open
```

Render a remote Markdown URL:

```sh
mdview up https://raw.githubusercontent.com/aoagents/ReverbCode/refs/heads/main/README.md --open
```

Stop the daemon:

```sh
mdview down
```

Run the server directly without the daemon:

```sh
npm start -- /absolute/path/to/file.md
```

Open the printed local URL. After editing the Markdown file, refresh the browser page. The app should render the changed contents because the server reads from disk on each request.

## Implementation Guidelines

When changing the app:

- Keep the main refresh behavior server-driven. Do not cache Markdown contents in a way that makes browser refresh stale.
- Avoid adding a database, account system, remote upload flow, or cloud dependency.
- Keep path handling explicit. The tool is local, but error messages should still be clear when paths are missing, unreadable, or directories instead of files.
- Treat remote Markdown as less trusted than local files. Local Markdown may render inline HTML, but remote Markdown should not render raw HTML unless a sanitizer is added.
- Prefer small dependencies with obvious value. Markdown parsing and syntax highlighting are reasonable; broad frameworks are unnecessary unless the UI grows substantially.
- Keep the interface usable on small screens, but optimize primarily for a desktop local-development workflow.

## Verification

After code changes, run:

```sh
node --check server.js
node --check cli.js
node cli.js up README.md
```

Then verify:

- `http://127.0.0.1:5898/` serves the app shell.
- `/api/render?file=<absolute markdown path>` returns rendered HTML.
- `/api/render?file=<encoded http markdown url>` downloads, caches, and returns rendered HTML.
- Editing the Markdown file and calling the render endpoint again returns the changed content.
- `node cli.js status` reports the daemon.
- `node cli.js down` stops the daemon.

Use a temporary Markdown file outside the repo when checking arbitrary-path support.

---
> Source: [harshitsinghbhandari/local-markdown-file-renderer](https://github.com/harshitsinghbhandari/local-markdown-file-renderer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
