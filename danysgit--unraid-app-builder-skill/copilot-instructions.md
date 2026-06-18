## unraid-app-builder-skill

> Build and package self-hosted apps for Unraid with a Docker image and a manual Unraid user-template that integrate cleanly with dockerMan. Use when Codex needs to add or review Unraid support, including icons, WebUI and support links, editable port and appdata mappings, persistent user customizations, or final install/update validation on a real Unraid server.


# Unraid App Builder

Build Unraid support as a packaging workflow, not as an afterthought. Favor a single canonical manual install path that works with dockerMan on the first try.

## Workflow

1. Confirm the app is a good fit for Unraid.
2. Prepare the Docker image for self-hosting.
3. Author one canonical manual Unraid user-template.
4. Validate install, edit, and update behavior on a real Unraid host.

Read [references/unraid-packaging.md](references/unraid-packaging.md) before writing or reviewing the template. Return to it when checking template fields, dockerMan quirks, or the final validation checklist.

## Package The Docker Image

- Keep the container port stable. Prefer one internal HTTP port such as `3000`; let Unraid users customize only the host-side mapping.
- Persist app state under `/config` unless there is a strong reason not to. This matches common Unraid expectations.
- Support `PUID`, `PGID`, and `UMASK` when the app writes files.
- Store generated secrets and temp state under the persistent appdata root, not only in memory.
- Expose a simple health endpoint if the app is a web service.
- Keep the image self-contained for the target workload. Install dependencies such as `ffmpeg` or `yt-dlp` inside the image when the app needs them.

## Author The Unraid Template

Publish only one manual-install template in the repo:

- Path: `templates/my-<AppName>.xml`
- Use that file as the install link in docs.
- Point `<TemplateURL>` back to the same `my-...xml` URL.

Do not publish a second base template path for manual installs unless there is a concrete reason. A single canonical `my-` template avoids dockerMan base-template mismatches.

Include these top-level fields:

- `<Name>`
- `<Description>`
- `<Repository>`
- `<Registry>`
- `<Support>`
- `<Project>`
- `<Overview>`
- `<Category>`
- `<WebUI>`
- `<TemplateURL>`
- `<Icon>`

For the container action menu and app presentation:

- Set `<WebUI>` to `http://[IP]:[PORT:3000]/` or the equivalent internal app port.
- Set `<Icon>` to a stable raw asset URL.
- Provide real support and project links.

For dockerMan compatibility, include both:

- legacy sections: `<Networking>`, `<Data>`, and `<Environment>`
- v2 `<Config ...>` entries for editable fields

This is important. In practice, dockerMan edit/update behavior can be inconsistent if the template only contains one style.

## Expose The Right Editable Fields

Always make these user-editable:

- host web port as a `Type="Port"` config targeting the app's internal port
- appdata path as a `Type="Path"` config targeting `/config`

Usually keep these as advanced variables:

- app title / branding overrides
- auth credentials
- cookie file path
- temp dir
- cleanup windows
- optional secret overrides
- `PUID`, `PGID`, `UMASK`

Prefer clear user-facing descriptions. Describe the host port as the port Unraid uses for WebUI access, not just “port 3000”.

## Preserve User Modifications

Design and test so user changes survive reinstall and update flows:

- Use the `my-...xml` template as the only manual-install artifact in the repo.
- During validation on a live Unraid box, verify that dockerMan writes the chosen host port and appdata path into the local saved template under `/boot/config/plugins/dockerMan/templates-user/`.
- If the Unraid `Update Container` screen disagrees with the running container, inspect the local template files before changing the app. This may be a dockerMan UI quirk rather than an app bug.
- Before concluding packaging is correct, confirm the live container binding and mount paths with `docker inspect`.

## Final Validation

Run an end-to-end Unraid test before calling the packaging done.

Minimum validation:

- install from the repo’s `my-...xml` template on a real Unraid host
- choose a non-default host port
- choose a non-default appdata path
- confirm the Docker list shows the expected host IP and port
- confirm the container uses the chosen appdata path
- confirm the container action menu exposes WebUI and support/project links
- reopen the edit/update screen and verify the chosen port and path still appear
- pull a newer image or simulate an update and verify the settings persist

If any of those fail, fix the packaging before shipping.

## Review Mode

When reviewing an existing Unraid integration, prioritize:

- missing or incorrect `WebUI`, `Icon`, `Support`, `Project`, or `TemplateURL`
- no editable host port mapping
- no editable appdata path
- templates that expose multiple competing manual install paths
- images that store mutable state outside `/config`
- missing `PUID` / `PGID` / `UMASK` support when the app writes files
- docs that point users at a stale or non-canonical template

---
> Source: [danysgit/unraid-app-builder-skill](https://github.com/danysgit/unraid-app-builder-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
