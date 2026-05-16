## chrome-extension-meshy-ai-easy-download

> - **manifest v3** chrome extension

# working with this extension

## architecture

- **manifest v3** chrome extension
- **content script** (`content.js`) - button injection, storage, downloads
- **main world script** (`content-main.js`) - fetch/XHR interceptor (runs in page context)
- **glb2stl converter** (`glb2stl.js`) - client-side GLB to STL conversion

## key files

- `manifest.json` - extension config
- `content.js` - button injection, storage, downloads
- `content-main.js` - fetch/XHR interceptor (MAIN world)
- `glb2stl.js` - lightweight GLB parser and STL generator
- `know-how/manifest-v3-api-interception.md` - technical docs
- `scripts/` - build and icon conversion scripts

## development workflow

1. make changes to extension files
2. reload extension at `chrome://extensions`
3. open devtools console (F12)
4. navigate to https://www.meshy.ai/workspace
5. watch for `[meshy-dl]` logs in console
6. test button injection and downloads

## critical implementation details

### fetch interceptor (content.js)
- must run at `document_start` to catch all API calls
- must clone response before reading (responses can only be read once)
- must return original response to avoid breaking the page

### dom injection (content.js)
- anchors on SVG path: `path[d^="M4.933 6.272h1.06V2.938"]`
- walks up to wrapper: `div.relative.inline-flex`
- injects buttons as siblings to wrapper

### api response structure
currently expects:
```javascript
result?.result?.generate?.modelUrl
result?.name || result?.args?.draft?.prompt
```

if meshy.ai changes their API, update content.js lines 48-54

## testing before release

1. test on fresh chrome profile
2. verify buttons appear on multiple models
3. test GLB download
4. test STL download and conversion
5. check file sizes are correct
6. verify no console errors

## known limitations

- intercepts both fetch and XHR but depends on specific API paths
- relies on specific SVG path selector (brittle if UI changes)
- STL conversion is client-side (can be slow for large models)

## if buttons stop appearing

1. check console for `[meshy-dl]` error logs
2. inspect DOM for expected SVG path and wrapper structure
3. check API response structure in network tab

---
> Source: [Gyvastis/chrome-extension-meshy.ai-easy-download](https://github.com/Gyvastis/chrome-extension-meshy.ai-easy-download) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
