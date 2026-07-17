## enjoining

> A client-side adult video streaming and library management platform built with Vite + Vanilla JavaScript ES Modules, HTML5, CSS3, and Bootstrap 5. No traditional backend — all data is driven by JSON metadata files and managed client-side. The platform is intended as a **controlled, private-access environment** and is not publicly available.

# GitHub Copilot Instructions — JOI Game / Adult Video Platform

## Project Overview

A client-side adult video streaming and library management platform built with Vite + Vanilla JavaScript ES Modules, HTML5, CSS3, and Bootstrap 5. No traditional backend — all data is driven by JSON metadata files and managed client-side. The platform is intended as a **controlled, private-access environment** and is not publicly available.

---

## Tech Stack

| Layer       | Technology                                              |
| ----------- | ------------------------------------------------------- |
| Build Tool  | Vite (ES Modules, HMR)                                  |
| Language    | Vanilla JavaScript (ESM, no TypeScript)                 |
| UI          | HTML5, CSS3, Bootstrap 5                                |
| Media       | HTML5 `<video>`, Web Audio API                        |
| Data        | JSON files (metadata, user config, library)             |
| Permissions | Client-side role system via sessionStorage/localStorage |
| Downloads   | yt-dlp (external CLI), native `<a download>`links     |

---

## Project Structure Conventions

```
/public
  /videos          → Local video files
  /assets          → Thumbnails, posters, icons
  video-metadata.json → Master video library manifest

/src
  /modules         → ES module files (one responsibility each)
  /components      → Reusable UI builders (pure functions returning DOM elements)
  /styles          → CSS per-component or per-page
  /data            → JSON loaders and schema validators
  /auth            → Role/permission logic
  main.js          → Vite entry point
  router.js        → Client-side routing (hash or History API)

/urls.txt          → Plaintext list of external URLs for yt-dlp batch downloads
```

---

## Coding Style & Patterns

* **Always use ES Modules** — `import/export`, never `require()` or `<script>` globals.
* **No frameworks** — do NOT suggest React, Vue, or Angular. Vanilla JS only.
* **No TypeScript** — plain `.js` files only.
* **Bootstrap for layout** — use Bootstrap 5 grid, utility classes, modals, and cards. Do NOT reinvent layout from scratch.
* **Functional style preferred** — prefer pure functions and module-level state over classes where practical.
* **DOM manipulation** — use `document.createElement`, `insertAdjacentHTML`, or template literals. Avoid `innerHTML` with unsanitized user input.
* **Async/await** — always use `async/await` over `.then()` chains for fetch and file reads.
* **Event delegation** — attach events to parent containers where lists are dynamic.

---

## JSON Metadata Schema

All videos are described by entries in `video-metadata.json`. Always follow this schema:

```json
{
  "id": "uuid-v4-string",
  "title": "Video Title",
  "description": "Optional description",
  "filename": "video-file.mp4",
  "url_external": "https://...",
  "thumbnail": "/assets/thumb.jpg",
  "duration": 1234,
  "tags": ["category1", "tag2"],
  "performers": ["Name One", "Name Two"],
  "platform": "PornHub | XVideos | Local | PMVHaven",
  "download_urls": {
    "1080p": "https://...",
    "720p":  "https://...",
    "480p":  "https://..."
  },
  "added_at": "ISO8601 timestamp",
  "visible_to": ["admin", "member"],
  "uploaded_by": "username"
}
```

* **Never skip `id`, `title`, `visible_to`, or `added_at`** — these are required.
* `visible_to` is an array of role names that can see this entry.
* When generating functions that read metadata, always validate these fields before use.

---

## User Roles & Permissions System

The platform uses a **client-side role system** stored in `sessionStorage`. There is no server-side auth — this is a controlled private environment.

### Roles (in ascending access level):

| Role       | Access                                                    |
| ---------- | --------------------------------------------------------- |
| `guest`  | View public-tagged content only, no downloads             |
| `member` | View member + public content, single-quality download     |
| `admin`  | Full library access, multi-quality downloads, JSON upload |

### Pattern to follow:

```js
// src/auth/permissions.js
export function getCurrentUser() {
  return JSON.parse(sessionStorage.getItem('currentUser')) ?? { role: 'guest' };
}

export function can(action, user = getCurrentUser()) {
  const rules = {
    guest:  ['view:public'],
    member: ['view:public', 'view:member', 'download:single'],
    admin:  ['view:public', 'view:member', 'view:admin',
             'download:single', 'download:multi', 'upload:json', 'manage:library'],
  };
  return rules[user.role]?.includes(action) ?? false;
}
```

* Always call `can('action')` before rendering gated UI elements.
* Never expose admin controls to non-admin roles in the DOM — conditionally render, don't just hide with CSS.

---

## Video Library & Categorization

* The library is loaded from `video-metadata.json` at startup and cached in a module-level variable.
* Filtering, sorting, and searching are always done **in-memory** on the cached array — no re-fetching.
* Categories and tags come directly from the metadata `tags` array — no hardcoded category list.

```js
// Example: filter by tag and role visibility
export function getVisibleVideos(tag = null, user = getCurrentUser()) {
  return library
    .filter(v => v.visible_to.includes(user.role))
    .filter(v => !tag || v.tags.includes(tag));
}
```

---

## JSON Metadata Upload (Admin Only)

* Admins can upload a `.json` file to add new entries to the library.
* Always **validate the schema** before merging — reject entries missing required fields.
* Use `FileReader` API to read the uploaded file client-side.
* Merge into the existing in-memory library array and persist to `localStorage` as a temporary override (pending a real save mechanism).

```js
// Pattern for JSON upload handler
input.addEventListener('change', async (e) => {
  if (!can('upload:json')) return;
  const file = e.target.files[0];
  const text = await file.text();
  const entries = JSON.parse(text);
  const valid = entries.filter(validateMetadataEntry);
  mergeIntoLibrary(valid);
});
```

---

## Download System

* **Single-quality download** : renders an `<a href="..." download>` link for the default quality.
* **Multi-quality download** (admin/member): renders a dropdown of available qualities from `download_urls`.
* For external URLs, link directly — yt-dlp handles the actual download externally via `urls.txt`.
* Never auto-trigger downloads — always require a user click.

```js
export function renderDownloadOptions(video, user) {
  if (!can('download:single', user)) return '';
  if (can('download:multi', user) && video.download_urls) {
    return Object.entries(video.download_urls)
      .map(([q, url]) => `<a class="btn btn-sm btn-outline-light" href="${url}" download>${q}</a>`)
      .join('');
  }
  return `<a class="btn btn-sm btn-primary" href="${video.url_external}" download>Download</a>`;
}
```

---

## What Copilot Should NOT Do

* Do NOT suggest server-side code (Node.js routes, Express, PHP, etc.) unless explicitly asked.
* Do NOT use `var` — always `const` or `let`.
* Do NOT use jQuery — native DOM APIs only.
* Do NOT suggest authentication libraries — the permission system is custom and client-side.
* Do NOT use `eval()` or `Function()` constructor.
* Do NOT `innerHTML` with raw user-controlled strings — always sanitize or use `textContent`.
* Do NOT suggest moving to React or TypeScript — this is intentionally Vanilla JS.

---

## Naming Conventions

| Thing       | Convention                       | Example                            |
| ----------- | -------------------------------- | ---------------------------------- |
| JS files    | camelCase                        | `videoPlayer.js`                 |
| Functions   | camelCase, verb-first            | `loadLibrary()`,`renderCard()` |
| CSS classes | Bootstrap utilities + kebab-case | `video-card`,`tag-badge`       |
| JSON keys   | snake_case                       | `visible_to`,`added_at`        |
| Constants   | UPPER_SNAKE_CASE                 | `MAX_RESULTS`,`DEFAULT_ROLE`   |

---

## Context Notes for Copilot

* This platform is adult content — suggestions involving content filtering, age-gating UI, or content warnings are welcome and appropriate.
* The project is in a **private/controlled access environment** — not a public-facing site yet.
* yt-dlp is used as an external CLI tool. The JS codebase only manages `urls.txt` content and triggers batch instructions — it does not wrap yt-dlp in a Node.js child process.
* Web Audio API may be used for audio visualization or sound effects on the player page — keep this in mind when suggesting player-related code.

---
> Source: [JacobPillai/Enjoining](https://github.com/JacobPillai/Enjoining) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
