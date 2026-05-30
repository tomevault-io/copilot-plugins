## electron-live-annotations

> You are helping a user add live annotation mode to their Electron product. Do not treat this as a generic package install. The point is to let a human point at the real Electron UI and leave a receipt that future agents can map back to source code and product decisions.

# Agent Guide: Implementing electron-live-annotations

You are helping a user add live annotation mode to their Electron product. Do not treat this as a generic package install. The point is to let a human point at the real Electron UI and leave a receipt that future agents can map back to source code and product decisions.

## Product Goal

The user should be able to:

1. open the real Electron app;
2. toggle annotation mode;
3. click the exact UI element that needs feedback;
4. write a short note;
5. save a local receipt with element identity and screenshot evidence;
6. hand that receipt to an agent without forcing the agent to reconstruct what was clicked.

The relief sentence is:

> "The agent can now see exactly what I meant in the native app."

## Discovery Gate Before Editing

Before changing files, inspect the host app and answer these questions in your notes:

- Electron topology: where are the main, preload, and renderer entrypoints?
- Security posture: is `contextIsolation` enabled, and what preload bridge pattern is already used?
- UI framework: React, Vue, Svelte, Solid, Lit, vanilla, or mixed?
- Build system: electron-vite, Vite, webpack, Forge, Builder, custom, or another runner?
- Selector conventions: does the app already use `data-testid`, `data-test`, `data-qa`, `data-cy`, or product-specific `data-*` attributes?
- Review workflow: where should annotation receipts be read later, and who acts on them?
- Storage/privacy posture: is local `app.getPath("userData")` acceptable, or does the product need a custom path?
- Remote-content posture: can any renderer route load arbitrary web content? If yes, gate or disable capture for those pages before exposing the preload API.
- Product truth boundary: are annotations only local review aids, or should a separate command promote them into issue/review state?

If those answers are unknown, inspect the repo first. Do not paste a generic integration.

## Implementation Plan

1. Install the package.

```bash
pnpm add electron-live-annotations
```

2. Main process: register the capture IPC after the app is ready.

```ts
import { registerAnnotationCaptureIpc } from "electron-live-annotations/main";

app.whenReady().then(() => {
  registerAnnotationCaptureIpc({
    shouldCapture(_request, event) {
      const url = event.sender.getURL();
      return url.startsWith("app://") || url.startsWith("file://");
    },
  });
});
```

3. Preload: expose the safe bridge.

```ts
import { contextBridge, ipcRenderer } from "electron";
import { exposeAnnotationPreloadApi } from "electron-live-annotations/preload";

exposeAnnotationPreloadApi(contextBridge, ipcRenderer);
```

4. Renderer: create the controller from the product shell, route root, or dev/debug panel that persists for the whole app session.

```ts
import { createLiveAnnotationController } from "electron-live-annotations/renderer";

const annotations = createLiveAnnotationController({
  stableAttributeNames: ["data-testid", "data-product-id"],
  stableAttributePrefixes: ["data-product-"],
});
```

5. Add stable selectors to product surfaces that humans will annotate. Prefer semantic, product-owned selectors over component-library internals.

6. Document where receipts land for the host app, usually:

```text
<app userData>/annotations/<sessionId>/manifest.jsonl
```

## Framework Placement

React:

- Create the controller in a root effect in `App.tsx`, route shell, or debug provider.
- Destroy it on unmount.

Vue:

- Create the controller in `onMounted` inside the app shell.
- Destroy it in `onBeforeUnmount`.

Svelte:

- Create the controller in `onMount`.
- Return the destroy function.

Vanilla:

- Create it after DOM boot and keep the returned controller.

The overlay is plain DOM. Do not wrap it in framework component state unless the product needs custom UI.

## Agent Reconciliation Workflow

When asked to act on annotations:

1. Locate the latest `manifest.jsonl`.
2. Read each row.
3. Prefer target identity in this order:
   - stable product selector such as `data-testid`;
   - configured product `data-*` selector;
   - nearest stable ancestor;
   - CSS selector;
   - XPath only as a brittle fallback.
4. Search the repo for the stable selector.
5. Open the cropped element image when visual judgment matters.
6. Convert each note into a requirement row:
   - actor/stakeholder;
   - scenario;
   - claim;
   - proof surface;
   - forbidden proof substitution;
   - expected receipt.
7. Implement the smallest product fix.
8. Verify with tests plus real Electron proof when the change touches native behavior.

## Product Truth Boundary

Annotation receipts are local review artifacts. They are not automatically:

- issue tracker truth;
- approval truth;
- mission state;
- customer feedback truth;
- telemetry proof;
- durable audit logs.

If the host product wants any of those, add an explicit command or workflow that imports annotation receipts into that product system. Do not silently treat a screenshot or note as canonical truth.

## Proof Checklist

Before claiming the integration is complete:

- `Cmd+.` or `Ctrl+.` toggles annotation mode without stealing input focus from text fields.
- Hover outlines a real renderer element.
- Clicking the overlay itself does not create an annotation.
- Saving writes an element PNG.
- Saving appends one row to `manifest.jsonl`.
- The manifest row includes route, viewport, rect, stable selector candidates, verdict, note, and image paths.
- The app still runs with `contextIsolation` enabled if it used that before.
- The implementation does not upload screenshots unless the user explicitly requested that product behavior.
- Remote or untrusted web content cannot call the capture bridge.

## Anti-Patterns

- Do not implement this as a browser-only script for an Electron product and claim native proof.
- Do not claim annotation receipts alone prove IPC behavior, file dialogs, auth callbacks, packaged boundaries, updater flows, or filesystem state.
- Do not rely on XPath as primary identity.
- Do not couple the package to React, Vue, Svelte, or a design system.
- Do not hide where screenshots are written.
- Do not add a second IPC bridge if the app already has a bridge convention.
- Do not route screenshots to external services by default.
- Do not expose the capture bridge to arbitrary remote pages.
- Do not ask the user to review machine-checkable facts manually.

## Suggested User-Facing Copy

Use product language, not implementation language:

- Good: "Capture annotation"
- Good: "What should change here?"
- Good: "Saved to local annotations"
- Avoid: "IPC capture payload"
- Avoid: "DOM selector manifest"
- Avoid: "renderer process debug artifact"

---
> Source: [danialhasan/electron-live-annotations](https://github.com/danialhasan/electron-live-annotations) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
