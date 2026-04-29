## marathonminimalist

> This rule summarizes conventions and best practices for writing UI components in this codebase, based on the ZeroToShipped documentation.

# UI Component Conventions (ZeroToShipped)

This rule summarizes conventions and best practices for writing UI components in this codebase, based on the ZeroToShipped documentation.

---

## 1. **Component Structure & Organization**

- **Route Groups & Private Folders:**
  - Use folders like `(landing)`, `(app)`, `(auth)` for route grouping. These do not affect the URL path.
  - Use `_components` and other `_`-prefixed folders for private, non-route code (e.g., colocated UI components, hooks, utils). See [Folder Structure](mdc:zerotoshipped-docs.md.txt).

- **Component Location:**
  - Place shared or global components in `src/components`.
  - Place route-specific or segment-specific components in the relevant `_components` folder within that route group.

---

## 2. **Kitze UI Integration**

- **Provider Setup:**
  - Wrap your app (or relevant parts) with `KitzeUIProvider` (usually via `KitzeUIProviders` in `src/components/core/KitzeUIProviders.tsx`).
  - Pass an `isMobile` boolean prop, determined via media queries (e.g., `useMediaQueries`).

- **Hooks:**
  - Use `useKitzeUI()` to access the `isMobile` flag in components.
  - Use `useDialog()` for programmatic dialogs/modals.
  - Use `useConfirm()` and `useConfirmDelete()` for confirmation dialogs.

- **Component Responsiveness:**
  - Kitze UI components (e.g., `SegmentedControl`, `SimpleDialog`, `SimpleDropdownMenu`, `SimpleSelect`, `SimpleTooltip`) automatically adapt to mobile/desktop based on `isMobile`.
  - Use the `mobileView` prop where available to control mobile behavior.

- **Component Usage:**
  - Use Kitze UI components directly. They are shadcn/ui compatible and modifiable.
  - Add new components via the shadcn/ui CLI or by copying into `src/components`.

---

## 3. **File Uploads (UploadThing)**

- **Enablement:**
  - Set `NEXT_PUBLIC_ENABLE_UPLOADTHING` to `true` to enable UploadThing features.

- **Component Usage:**
  - Use `FormFieldUploadThingImage` for single image uploads in forms (integrates with react-hook-form).
  - Use `UploadThingUploadSingleImage` for standalone upload UIs.

- **Form Integration:**
  - Store image state as `{ id: string, key: string } | null` in your form schema.
  - Use the `endpoint` prop (e.g., `imageUploader`) to specify the upload route.

---

## 4. **Forms**

- Use `react-hook-form` for all forms.
- Use custom form field components (e.g., `FormFieldInput`, `FormFieldAdvancedSelect`, `FormFieldCheckbox`, `FormFieldSwitch`, `FormFieldTextarea`, `FormFieldUploadThingImage`).
- Integrate file/image uploads using the provided UploadThing form field component.

---

## 5. **General UI/UX Conventions**

- **Loading States:**
  - Use a dedicated loading spinner component (e.g., `Spinner.tsx`).
- **Buttons:**
  - Use a custom button component (e.g., `CustomButton.tsx`) that supports icons, loading, and extra props.
- **Dialogs/Modals:**
  - Use the dialog manager (`useDialog`) for opening components in dialogs.
- **Confirmation:**
  - Use `useConfirm()` and `useConfirmDelete()` for confirmation dialogs.
- **Segmented Controls:**
  - Use the provided `SegmentedControl` component; do not reimplement from scratch.
- **Responsiveness:**
  - Use `useKitzeUI()` for device detection when conditional logic is needed beyond Tailwind's media queries.

---

## 6. **Environment Variables in UI**

- Only expose variables prefixed with `NEXT_PUBLIC_` to the client.
- Use `clientEnv` from `src/env/client.ts` for accessing validated client-side env variables.

---

## 7. **References**

- [Kitze UI Docs](mdc:zerotoshipped-docs.md.txt)
- [Folder Structure](mdc:zerotoshipped-docs.md.txt)
- [UploadThing Integration](mdc:zerotoshipped-docs.md.txt)

---

**Follow these conventions to ensure consistency, maintainability, and best practices when writing UI components in this codebase.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/macphers) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
