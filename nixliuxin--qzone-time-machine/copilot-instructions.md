## no-external-resources

> No external resource loading in the viewer


# Zero External Resources

The viewer MUST be fully self-contained and work offline via `file://` protocol.

## Rules

1. **No external URLs in rendered HTML/components.** All images, avatars, emojis,
   fonts, and scripts must be downloaded during backup and referenced locally.
2. **Avatars**: Downloaded to `media/avatars/{uin}.jpg` (QQ) and
   `media/avatars/{uin}_qz.jpg` (QZone). Frontend uses `./media/avatars/` paths.
3. **QQ Emojis**: Downloaded to `media/emoji/` during backup. Rendered via local paths.
4. **Photos/Videos**: Already downloaded to `media/albums/` or `media/messages/` etc.
5. **Navigation links** (e.g. `<a href="https://user.qzone.qq.com/...">`) that only
   activate on user click are acceptable — they do not load external resources passively.

## Rationale

- Users may view archives years later when QQ CDN URLs have expired.
- Archives must work in air-gapped / offline environments.
- No tracking or privacy leakage from opening the viewer.

---
> Source: [nixliuxin/QZone-Time-Machine](https://github.com/nixliuxin/QZone-Time-Machine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
