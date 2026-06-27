## picmark

> PicMark is being built for the user's father, whose core need is local image annotation for practical tasks such as marking parking spaces in an underground garage image. The app should feel closer to WPS 图片 / a lightweight local image viewer with editable annotations than to a screenshot tool or a heavy design editor.

# PicMark Agent Notes

## Product Context

PicMark is being built for the user's father, whose core need is local image annotation for practical tasks such as marking parking spaces in an underground garage image. The app should feel closer to WPS 图片 / a lightweight local image viewer with editable annotations than to a screenshot tool or a heavy design editor.

## Target Hardware And Performance Budget

- Treat the real target machine as Windows 7 era / low-end office hardware.
- Concrete benchmark to keep in mind: 13th-gen Intel i3 class machine, not a high-end workstation.
- Features must stay responsive on large local folders and large images.
- Avoid synchronous full-folder scans, full-thumbnail generation, or expensive per-frame work on the UI thread.
- Prefer lazy loading, small visible windows of data, caps, caches, and opt-in heavy features.
- If a feature becomes heavy, make it optional or remove it rather than forcing the app into a bloated image manager.

## Product Principles

- Local first, offline first, no account, no upload.
- Open existing local images directly, annotate continuously, return later through `.picmark` projects.
- Editing must preserve the original image until explicit export/overwrite.
- UI should stay simple enough for non-technical family users.
- Do not copy screenshot-product assumptions when the task is ongoing annotation of existing images.

## Local Test App Handling

- The user's local PicMark process is always treated as a disposable test session.
- If PicMark blocks rebuilding, copying, or replacing the executable, close/kill PicMark directly without asking first.
- Do not assume the currently opened PicMark image/project contains important unsaved work during development.

## UI Rules

- Popup menus must be compact, left-aligned, and sized to their content.
- Do not make menu items look like large centered cards.
- Avoid strange empty space in menus; reduce popup width before adding visual padding.
- Menu button text should align left unless there is a specific icon-only reason.

---
> Source: [Tsang12140/picmark](https://github.com/Tsang12140/picmark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
