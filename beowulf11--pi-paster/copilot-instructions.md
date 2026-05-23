## pi-paster

> <!--VITE PLUS START-->

<!--VITE PLUS START-->

# Using Vite+, the Unified Toolchain for the Web

This project is using Vite+, a unified toolchain built on top of Vite, Rolldown, Vitest, tsdown, Oxlint, Oxfmt, and Vite Task. Vite+ wraps runtime management, package management, and frontend tooling in a single global CLI called `vp`. Vite+ is distinct from Vite, and it invokes Vite through `vp dev` and `vp build`. Run `vp help` to print a list of commands and `vp <command> --help` for information about a specific command.

Docs are local at `node_modules/vite-plus/docs` or online at https://viteplus.dev/guide/.

## Review Checklist

- [ ] Run `vp install` after pulling remote changes and before getting started.
- [ ] Run `vp check` and `vp test` to format, lint, type check and test changes.
- [ ] Check if there are `vite.config.ts` tasks or `package.json` scripts necessary for validation, run via `vp run <script>`.

<!--VITE PLUS END-->

# pi-paster Project Guide

## What this project is

`pi-paster` is a pi extension package. It makes pasted, drag-dropped, or clipboard-provided images behave like first-class image attachments in pi interactive mode.

Target behavior:

1. User pastes or drag-drops an image path, or uses pi's image clipboard paste keybinding.
2. The editor replaces the raw path/image with a placeholder like `[#image 1]`.
3. The extension stores the image payload immediately in memory.
4. On submit, the user text is sent with placeholders preserved and matching image blocks attached to the same user turn.
5. Submitted attachments render back in chat history with an `Attached [#image N] <path>` label and image preview.

## Product behavior notes

- Placeholder format is `[#image 1]`, `[#image 2]`, etc. Keep the space after `image`.
- Attach each referenced placeholder at most once per submitted message, ordered by first placeholder occurrence in the text.
- If a placeholder is deleted before submit, do not attach that image.
- If a placeholder is duplicated in text, attach the image once.
- Read image payloads immediately on paste/drop/clipboard; later file deletion should not matter.
- Attachment state is in-memory and resets on session lifecycle changes.
- Supported image formats: PNG, JPEG, WebP, GIF. Detect by magic bytes, not extension only.
- Max image file size is 10 MB.
- Leave unsupported/non-image paste text unchanged. Warn only for oversized images.
- Do not add slash commands or LLM-callable tools; the extension should work automatically.

## Important files

- `src/index.ts` — extension entrypoint and event wiring.
- `src/config.ts` — public configuration types/defaults.
- `src/editor.ts` — custom editor integration, paste handling, clipboard insertion, atomic placeholder deletion.
- `src/image-utils.ts` — image path parsing, MIME detection, image loading, and image-content generation.
- `src/preview.ts` — submitted-image and cursor-preview render components.
- `src/store.ts` — in-memory attachment store and placeholder allocation.
- `src/clipboard.ts` — macOS clipboard image reader.
- `src/terminal-input.ts` — fallback terminal input handler used when the custom editor is disabled.
- `tests/` — Vitest tests run through Vite+.
- `package.json` — package metadata, npm publishing metadata, peer dependencies, and `pi.extensions` manifest.
- `docs/preview.png` — package gallery/README preview image.
- `README.md` — user-facing docs.

## How to run it

Install dependencies:

```bash
vp install
```

Run checks:

```bash
vp check
vp test run
```

Build package output:

```bash
vp run build
```

Try the extension locally in pi:

```bash
pi -e .
```

## Implementation notes

- The extension entrypoint is `src/index.ts` and must default-export a function receiving `ExtensionAPI`.
- Also export `createPaster(config)` so users can configure the extension from a wrapper extension.
- Pi discovers this package through `package.json`:

  ```json
  {
    "pi": {
      "extensions": ["./src/index.ts"]
    }
  }
  ```

- Keep pi core packages used by the extension as peer dependencies for consumers:
  - `@earendil-works/pi-coding-agent`
  - `@earendil-works/pi-tui`
- Use `CustomEditor` for editor customization so pi app keybindings continue to work.
- If `customEditor.enabled` is false, do not install the custom editor; use the terminal input fallback for bracketed paste/drop paths.
- Do not override private pi editor methods. Intercept paste/control sequences through public editor APIs such as `handleInput()` and `insertTextAtCursor()`.
- Use `pi.on("input", ...)` to transform submitted text and attach image content.
- Keep image parsing and MIME detection helper functions small and unit-testable.
- Real images must not render inside pi-tui overlays; overlay compositing can corrupt terminal image escape sequences. Use normal widget/custom render flow instead.

## Configuration contract

Default config enables all editor integrations:

```ts
createPaster({
  customEditor: {
    enabled: true,
    showImagePreview: true,
    deletePlaceholderAsBlock: true,
  },
});
```

Behavior:

- `customEditor.enabled: false` keeps pi's default editor and disables cursor previews, atomic deletion, and paster's clipboard-image handler.
- `customEditor.showImagePreview: false` keeps the custom editor but disables the above-editor cursor image preview.
- `customEditor.deletePlaceholderAsBlock: false` keeps the custom editor but lets placeholders delete as normal text.

## Publishing notes

- npm package name: `pi-paster`.
- Keep `keywords` including `pi-package` so the pi package gallery can discover it.
- Keep `pi.image` pointing at the published preview image, e.g. `https://unpkg.com/pi-paster@<version>/docs/preview.png`.
- Before publishing, run:

```bash
vp check
vp test run
vp run build
npm pack --dry-run
```

- Publish with:

```bash
npm publish
```

NPM may require browser/OTP authentication.

## Testing guidance

- Put fast unit tests under `tests/` and import test helpers from `vite-plus/test`.
- Prefer unit tests for parsing, path resolution, MIME detection, attachment ordering, and placeholder matching.
- Use manual pi testing for TUI behavior that is difficult to automate:
  - normal text paste still works
  - image path paste becomes `[#image N]`
  - multiple image paths preserve order
  - deleting a placeholder prevents attachment
  - pi keybindings still work
  - cursor preview appears only when the cursor is inside the placeholder
  - clipboard image paste works via pi's image paste keybinding on macOS

Before handing off changes, run:

```bash
vp check
vp test run
```

If implementation changed package output or publish metadata, also run:

```bash
vp run build
npm pack --dry-run
```

## Documentation references

When working on pi extension APIs, consult the local pi docs first:

- Extensions: `/Users/beowulf/.vite-plus/packages/@earendil-works/pi-coding-agent/lib/node_modules/@earendil-works/pi-coding-agent/docs/extensions.md`
- TUI/custom editor APIs: `/Users/beowulf/.vite-plus/packages/@earendil-works/pi-coding-agent/lib/node_modules/@earendil-works/pi-coding-agent/docs/tui.md`
- Package manifests: `/Users/beowulf/.vite-plus/packages/@earendil-works/pi-coding-agent/lib/node_modules/@earendil-works/pi-coding-agent/docs/packages.md`
- Extension examples: `/Users/beowulf/.vite-plus/packages/@earendil-works/pi-coding-agent/lib/node_modules/@earendil-works/pi-coding-agent/examples/extensions/`

For Vite+ commands/config, use local docs in `node_modules/vite-plus/docs` or https://viteplus.dev/guide/.

---
> Source: [beowulf11/pi-paster](https://github.com/beowulf11/pi-paster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
