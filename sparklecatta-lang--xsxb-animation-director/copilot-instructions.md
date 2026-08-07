## xsxb-animation-director

> Keep this repository code-only. Install dependencies, import user-authorized local animation assets, verify the real WebApp, and never commit or upload source media or generated asset binaries.

# XSXB Animation Director Agent Instructions

## Goal

Keep this repository code-only. Install dependencies, import user-authorized local animation assets, verify the real WebApp, and never commit or upload source media or generated asset binaries.

## Non-negotiable boundaries

- Never add GLB, GLTF, FBX, Blend, OBJ, ZIP, Unity package, video, user model, texture set, or downloaded marketplace content to Git.
- Treat `public/local-assets/`, `asset-downloads/`, `assets/`, `blender-projects/`, `reports/`, `tools/`, `unity-plugin/`, and `.runtime/` as local/private workspaces.
- Keep the original downloaded files unchanged. Convert into new local outputs.
- Check the source license before processing. A free price does not imply permission to redistribute.
- Unity Asset Store content may be used only from the user's own acquired copy and must never be redistributed through this repository.
- Do not promise one-pass retargeting. Iterate and verify visible results with the user.

## Start or repair the WebApp

1. Detect Node.js and npm. Install a current Node.js LTS release when missing.
2. Run `npm ci` when `package-lock.json` is present; otherwise run `npm install`.
3. Run `npm test` and `npm run build`.
4. Start with `npm run dev` or the Windows launcher.
5. Verify the page loads without bundled animation assets. An empty action library is valid.

Do not ask the user to manually configure Node.js, Blender, Unity paths, environment variables, or manifest JSON. Discover or install what is needed. Ask the user only for account login/license acceptance or a choice that materially changes the result.

## Import a downloaded animation pack

1. Record the author, exact pack name, source page/store name, current license, and the user-provided source path.
2. Inspect the archive or Unity project without modifying the source.
3. Identify the animated model, skeleton, clips, frame rate, root motion, materials, and embedded props.
4. Prefer a self-contained GLB for browser playback. FBX is supported when external texture paths remain valid, but GLB is the default handoff.
5. Preserve the pack's own rig and materials unless compatibility has been visually verified.
6. Write converted outputs only under `public/local-assets/packs/<pack-id>/`.
7. Create or update the ignored file `public/local-assets/animation-packs.json` using `public/local-assets/animation-packs.example.json` and `docs/asset-pack-manifest.md`.
8. Run `npm run assets:validate`.
9. Start the real WebApp and verify every acceptance gate below.

## Retarget a user model

1. Preserve the supplied model, textures, original rig, materials, and embedded props.
2. Work on a copy under the ignored local asset workspace.
3. Inspect rest pose and bone axes before transferring animation.
4. Iterate on semantic bone mapping, rest-pose alignment, skin weights, root motion, grounding, and prop sockets.
5. Use the WebApp bone/skin editor when useful, but bake or document any correction required for repeatable playback.
6. Keep each attempted result recoverable. Do not overwrite the supplied source.
7. State clearly when a result remains partial or visually unverified.

## Browser acceptance gates

- The pack appears under the correct author and pack name.
- Every listed clip previews on the intended character.
- Dragging a card inserts it into the timeline.
- Timeline reorder, speed, left trim, and right trim work.
- Materials and embedded props remain correct.
- The browser console has no asset-loading or animation errors for the imported pack.
- A non-empty MP4 export completes.
- Eight-direction export produces transparent PNG files, eight direction folders, and a valid `manifest.json`.

## Before committing code

Run:

```powershell
npm test
npm run build
npm run release:check
```

Review `git status` and `git diff --cached --name-only`. If any downloaded or generated media appears, stop and remove it from the commit without deleting the user's local copy.

---
> Source: [sparklecatta-lang/XSXB-animation-director](https://github.com/sparklecatta-lang/XSXB-animation-director) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-05 -->
