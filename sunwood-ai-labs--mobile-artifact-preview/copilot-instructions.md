## mobile-artifact-preview

> Use when exposing local development artifacts to a phone or tablet, previewing Codex or agent outputs from mobile, troubleshooting LAN-accessible viewers, Nextcloud previews, file links, screenshots, generated images, HTML, SVG, JSON, XML, Markdown, PDF, or other artifacts that the user needs to visually confirm from a mobile device.


# Mobile Artifact Preview

## Purpose

Use this skill to make local development artifacts visible from mobile devices and to prove that they render. The deliverable is not only a local path: provide a clickable mobile-accessible link, verify the real display surface when possible, and include the exact local fallback path.

This skill supports workflows such as:

- Showing generated images, screenshots, diagrams, HTML, SVG, Markdown, JSON, XML, CSV, PDF, or draw.io exports to the user.
- Creating or repairing a LAN file viewer, Nextcloud mount, or preview plugin for local project files.
- Verifying mobile layout for generated previews or local web apps.
- Storing evidence screenshots under a project folder so the user can inspect them later.

## Core Rule

When the user asks to show, preview, display, or confirm an artifact, treat the visible mobile-checkable surface as the deliverable.

For every newly created or updated user-checkable artifact, especially generated
images, screenshots, HTML, SVG, JSON/XML previews, PDFs, diagrams, reports, or
other visual outputs, include a clickable URL in the chat whenever a LAN viewer
or Nextcloud route is available. Do not make the user ask separately for the
link.

Do not stop at "file created" or a raw local path. Return:

1. A clickable link if a LAN viewer or Nextcloud route is available.
2. The local filesystem path as fallback.
3. The verification surface used, such as `view_image`, Browser screenshot, Playwright mobile viewport, WebDAV listing, or Nextcloud app check.

For images shown in chat, also call `view_image` on the exact file before saying it was displayed.

When a generated image, repository header, thumbnail, or release banner needs
visible text and the user asked for image generation, do not add that text later
with Pillow, SVG, canvas, or another post-processing overlay unless the user
explicitly approves that production method. Treat generated typography as part
of the deliverable: inspect the image-gen output, retry if the text is wrong,
and record whether the final asset is direct image-gen output or a post-processed
composite.

## Preferred Surfaces

Choose the narrowest surface that fits the artifact:

| Need | Preferred surface |
| --- | --- |
| Image visible in chat | `view_image` on the exact file |
| Mobile-accessible project file | Nextcloud file viewer |
| Mobile web UI or preview app | LAN URL plus mobile viewport screenshot |
| MP4/video visible on phone | Verified video viewer or direct HTTP video endpoint with mobile viewport screenshot |
| Local-only static HTML | Start or reuse a small LAN web server |
| JSON/XML structure | Structured preview or companion `.html` preview |
| Repeated screenshots/evidence | Project-local `evidence/` folder |

## Nextcloud Viewer Package

When the bundled Nextcloud viewer is available, prefer it for project and Codex
artifact links:

```text
Default local URL: http://127.0.0.1:8793/
Phone-facing URL: ${MOBILE_ARTIFACT_NEXTCLOUD_BASE_URL:-https://<tailscale-host>:8443}
Default login: admin / admin
Project mount: ${MOBILE_ARTIFACT_PROJECTS_DIR:-${HOME}/Prj} -> /Project
Codex mount: ${MOBILE_ARTIFACT_CODEX_DIR:-${HOME}/.codex} -> /Codex
Project folder link pattern:
${MOBILE_ARTIFACT_NEXTCLOUD_BASE_URL}/apps/files/files?dir=/Project/<path-under-projects-dir>
```

For links returned to the user, prefer the configured HTTPS base URL. Do not
paste old LAN `http://192.168...:8793` links when HTTPS/Tailscale Serve is
configured. On this Mac, the verified Nextcloud phone-facing base URL is:

```text
https://macbook-pro-von-admin.tail8be30.ts.net:8443
```

The bare `https://macbook-pro-von-admin.tail8be30.ts.net` route may point to a
different preview service, so keep the `:8443` port for Nextcloud links unless
`tailscale serve status` proves the mapping changed.

Bundled implementation source in this repository:

```text
assets/nextcloud-file-viewer
```

Useful checks:

```bash
cd assets/nextcloud-file-viewer
docker compose ps
docker exec -u www-data agent-nextcloud php occ app:list | rg 'structuredviewer|htmlviewer|text|viewer|pdf'
docker exec -u www-data agent-nextcloud php occ files:scan --path='admin/files/Project/<relative-folder>' --shallow
```

For a folder link under the projects directory, return:

```text
${MOBILE_ARTIFACT_NEXTCLOUD_BASE_URL}/apps/files/files?dir=/Project/<relative-folder>
```

## Workflow

1. Identify the artifact path and file type.
2. If the artifact is an image intended for chat, call `view_image` on that exact path.
3. If the artifact is under the configured projects directory, build the Nextcloud `/Project/...` link and shallow-scan the folder if new files may not be indexed.
4. If the artifact is under the configured Codex directory, build the Nextcloud `/Codex/...` link when the mount is available.
5. Include that clickable URL in the same chat response as the generated artifact, before or next to the local path.
6. For HTML/SVG/JSON/XML/PDF or UI work, verify with a real browser surface. Use a mobile viewport for mobile-facing claims.
7. Put persistent proof screenshots in a nearby `evidence/` folder, then include both the evidence folder link and local path.
8. In the final response, include the clickable link first, then local path, then a short verification note.

## MP4 and Video Preview

Do not treat a raw WebDAV URL such as `remote.php/dav/.../*.mp4` as a verified
mobile preview link. On iPhone it may open as a generic downloadable MP4 file
with an external-app prompt instead of an inline playable preview.

For generated MP4/video artifacts, provide one of these verified surfaces:

- a LAN viewer page that renders a `<video controls playsinline>` element; or
- a direct HTTP endpoint that returns `Content-Type: video/mp4`,
  `Accept-Ranges: bytes`, and supports `206 Partial Content` for range
  requests.

Do not promise direct iPhone Photos/camera-roll saving from an insecure LAN
`http://192.168...` page. Browser file sharing with `navigator.share({files})`
requires a secure context such as HTTPS, and ChatGPT/Safari webviews may fall
back to file download only. If the user needs Photos import, provide a verified
HTTPS share page or give an explicit fallback route through the Files app,
AirDrop, or another native iOS import path.

Before reporting a video link as previewable, verify:

```bash
curl -I 'http://<host>/<video-url>'
curl -I -H 'Range: bytes=0-1023' 'http://<host>/<video-url>'
```

The first response should be `200 OK` with `Content-Type: video/mp4`; the second
should be `206 Partial Content` with `Content-Range`. Also open the viewer in a
390x844 mobile viewport and confirm the video element loads metadata such as
duration, width, and height.

## Mobile Verification

Use a realistic phone viewport before claiming a mobile rendering fix:

```text
390x844 or similar portrait viewport
```

Check for:

- Header content not hidden behind browser or Nextcloud viewer chrome.
- No text cut off at the left or right edge.
- No horizontal scrolling unless the file type naturally requires it.
- No left-right wobble while vertically scrolling on a phone. Verify that the
  outer viewer has no horizontal overflow; only intentional inner surfaces such
  as tables or code blocks may scroll sideways.
- Buttons and close/menu controls remain reachable.
- Structured JSON/XML content expands or wraps correctly.

If the user shares a mobile screenshot showing clipping, compare against that failure mode in the next verification pass.

## Structured Data Preview

For JSON and XML in the current Nextcloud setup, use the `structuredviewer` app when available. If the browser still opens the Text app, check that the structured viewer scripts are loaded and the file-list click handler is active.

Useful checks:

```bash
docker exec -u www-data agent-nextcloud php occ app:list | rg 'structuredviewer|text'
docker exec -u www-data agent-nextcloud php occ app:enable structuredviewer
docker exec -u www-data agent-nextcloud php occ upgrade
```

If the phone appears stale, suspect cached JS/CSS. Versioned asset filenames or updated app versions may be needed before retrying.

## Markdown and Nextcloud Text Conflicts

Project and Codex mounts may be intentionally exposed to Nextcloud as
read-only Docker mounts. This is safer for generated artifacts, but Nextcloud
Text still treats `.md` files as editable documents.

If a Markdown file opens with duplicate side-by-side content, overwrite/discard
buttons, or a warning such as "current changes cannot be autosaved", treat it
as a Text app conflict session before suspecting file corruption.

Checks:

```bash
docker inspect agent-nextcloud --format '{{range .Mounts}}{{println .Source "->" .Destination "RW=" .RW}}{{end}}'
docker exec agent-nextcloud sh -lc "grep -n 'apps/text/session\\|file_put_contents failed\\|Read-only file system' /var/www/html/data/nextcloud.log | tail -n 40"
```

If the log shows `Read-only file system` for `/external/prj` or
`/external/codex`, do not click the overwrite option. First confirm the real
file content on the host, then discard the stale Text session or reopen the file
from the latest version.

For article or report drafts that the user only needs to inspect from mobile,
prefer the bundled Markdown viewer for the source `.md` file, or a read-only
rendered `.html` companion when a portable fallback is needed. Avoid opening
source `.md` files in Nextcloud Text when the mount is read-only.

The bundled `structuredviewer` app intercepts `.md` and `.markdown` file-list
clicks and opens them through a read-only Markdown preview. This source-file
preview intentionally preserves raw HTML inside trusted Markdown, so tags such
as `<br>`, `<mark>`, `<details>`, and simple HTML tables render in the mobile
preview instead of being escaped or ignored. Do not use this renderer for
untrusted Markdown.

The source-file Markdown preview should use the project default dark appearance:
near-black navy surfaces, cyan links and borders, warm amber highlights, and
GitHub-like document structure. Match the repository header image palette rather
than falling back to a generic white Markdown page. After changing Markdown
preview CSS, verify the actual `.md` file preview with a mobile viewport
screenshot, not only a generated companion `.html`.
When fixing Markdown list rendering, treat visible list markers as the primary
acceptance point. Do not stop at DOM counts or indentation: verify in the
mobile screenshot that unordered and ordered list markers are visible, and check
computed styles such as `list-style-type` so Nextcloud/global CSS has not reset
markers to `none`.
If iOS Safari or an in-app browser shows a top "Nextcloud / Open" app promotion
above the artifact, treat it as a blocker for mobile preview screenshots. Check
for `<meta name="apple-itunes-app">`; in the bundled Nextcloud viewer, clear
`theming.iTunesAppId` so Safari does not render the Smart App Banner.
When changing Markdown inline rendering, verify README-style linked image
badges such as `[![Validate](...badge.svg)](...)`. Markdown image/link syntax
must be parsed before bare URL auto-linking, otherwise generated anchor
attributes can leak into the visible preview. External SVG badges that cannot
load inside the Nextcloud viewer should fall back to local badge styling instead
of showing broken image icons.
Also verify the Files app folder-top README/workspace preview, not only the
opened `.md` viewer. The folder-top preview is a separate Nextcloud Text
surface, so it needs a mobile screenshot proving raw HTML is not visible,
linked badges render, the preview width is not narrowed by workspace padding,
and the file list does not overlap the preview content.

The bundled structured viewer appearance is configurable through Nextcloud app
config. Use these keys:

```bash
docker exec -u www-data agent-nextcloud php occ config:app:set structuredviewer theme --value=branded_dark
docker exec -u www-data agent-nextcloud php occ config:app:set structuredviewer background_image --value='https://example.local/background.png'
docker exec -u www-data agent-nextcloud php occ config:app:set structuredviewer accent --value='#32c7f4'
docker exec -u www-data agent-nextcloud php occ config:app:set structuredviewer highlight --value='#d98545'
```

For temporary checks, URL query parameters can override the app config:
`sv_theme`, `sv_bg`, `sv_accent`, and `sv_highlight`. Example:
`?sv_bg=https%3A%2F%2Fexample.local%2Fbackground.png&sv_accent=%235ce1ff`.

If the user asks about the overall Nextcloud look, theme the global Nextcloud UI
as well as the opened-file preview. The global UI is controlled by the
Nextcloud `theming` app, while `structuredviewer` controls Markdown/JSON/XML
preview internals. Prefer the bundled reproducible helper:

```bash
cd assets/nextcloud-file-viewer
MOBILE_ARTIFACT_THEME_BACKGROUND_IMAGE="$PWD/../../docs/images/eclipse-grand-wallpaper.png" \
MOBILE_ARTIFACT_THEME_MOBILE_BACKGROUND_IMAGE="$PWD/../../docs/images/eclipse-mobile-wallpaper.png" \
MOBILE_ARTIFACT_THEME_LOGO_IMAGE="$PWD/../../docs/images/nextcloud-custom-logo.png" \
MOBILE_ARTIFACT_THEME_FAVICON_IMAGE="$PWD/../../docs/images/nextcloud-custom-favicon.png" \
scripts/apply-global-theme.sh
```

The helper sets Nextcloud name/slogan/colors/background image, custom logo,
header logo, favicon, and then aligns the structured viewer accent/highlight
colors. Use environment variables such as
`MOBILE_ARTIFACT_THEME_PRIMARY_COLOR`, `MOBILE_ARTIFACT_THEME_BACKGROUND_COLOR`,
`MOBILE_ARTIFACT_THEME_BACKGROUND_IMAGE`, `MOBILE_ARTIFACT_THEME_LOGO_IMAGE`,
`MOBILE_ARTIFACT_THEME_LOGO_HEADER_IMAGE`, and
`MOBILE_ARTIFACT_THEME_FAVICON_IMAGE` when the user wants a custom palette,
background, or logo. It also clears the default user's personal background so
the global background is visible; set `MOBILE_ARTIFACT_SYNC_USER_THEME=0` when
personal appearance settings must be preserved.
Use `MOBILE_ARTIFACT_THEME_MOBILE_BACKGROUND_IMAGE` to set a portrait mobile
background for Files and Viewer pages; the regular background remains the
desktop/wide-screen image.
Files, Viewer, and Dashboard recommendation panels should keep a translucent
glass-style UI so the wallpaper is visible while file names remain readable.
Do not stack translucent backgrounds on both parent containers and child rows:
give the section container the glass background, keep nested rows/items
transparent, and verify computed styles on a mobile viewport before reporting
the theme fixed.

The default custom palette is eclipse-derived and contrast-adjusted for mobile:
background `#070810`, primary `#0ea5d8`, accent `#32c7f4`, highlight `#d98545`.

Do not render a redundant internal Markdown preview title or file name above the
document body. Nextcloud already shows the opened file name in the viewer top
bar, so extra labels such as "Markdown Preview" make the mobile surface feel
duplicated and cramped.

README-style Markdown commonly uses raw HTML for centered logos, badge rows, and
image sizing. The `.md` source-file preview must support local relative images,
centered HTML blocks such as `<div align="center">`, badge SVG/PNG rows, and
regular Markdown images. Verify these patterns with an actual `.md` file opened
through Nextcloud, because generated companion HTML can hide broken source-file
relative paths.

When generating a Markdown companion HTML preview, use the bundled script:

```bash
node assets/nextcloud-file-viewer/scripts/render-markdown-preview.mjs <file.md> --output <preview.html>
```

The companion preview uses the same trusted-Markdown raw HTML behavior.

## Common Failures

- Saying an image was shown without calling `view_image` in the same turn.
- Returning only a local path when the user needs to open it from a phone.
- Forgetting to scan a newly created Nextcloud-mounted folder.
- Verifying a mobile issue only on desktop viewport.
- Treating JSON/XML raw text display as structured preview.
- Treating a Nextcloud Text conflict on read-only `.md` files as Markdown file corruption.
- Escaping raw HTML when creating trusted Markdown companion previews.
- Styling the `.md` source-file preview as a custom panel instead of a GitHub-like rendered Markdown document.
- Forgetting to test README-style centered logos, badges, and relative images in the `.md` source-file preview.
- Saving evidence screenshots outside the project folder, making them hard for the user to find later.

## Response Pattern

Keep the final response short and proof-oriented:

```text
モバイル確認用リンク:
[Nextcloud sample-gallery](https://macbook-pro-von-admin.tail8be30.ts.net:8443/apps/files/files?dir=/Project/nextcloud-file-viewer/sample-gallery)

ローカルパス:
<projects-dir>/nextcloud-file-viewer/sample-gallery

確認:
390x844 のモバイル viewport で表示確認済み。JSON/XML は structuredviewer で構造化表示されています。
```

---
> Source: [Sunwood-ai-labs/mobile-artifact-preview](https://github.com/Sunwood-ai-labs/mobile-artifact-preview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
