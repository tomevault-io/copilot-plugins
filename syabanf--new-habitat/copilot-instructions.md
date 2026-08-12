## ui-interaction-standards

> UI interaction and visual standards for this project


# UI Interaction Standards

- Use the Purchasing/PR pages as the main UI reference for list tables and detail pages. Match their structure, spacing, table layout, card wrappers, and action placement before inventing new styling.
- Tables must follow the Purchase Request list style: card wrapper, padded table container (`px-4`), soft dividers (`border-gray-200/70`), balanced footer padding, subtle hover states, and right-aligned action cells. Avoid custom black table borders.
- Detail pages must follow the Purchasing detail style: proportional header spacing, soft card borders, clean timeline/info cards, and no high-contrast table/divider borders.
- Use soft borders only. Prefer semantic `border-border`, `border-gray-200/70`, `border-gray-200/80`, or soft `border-primary/20` for branded actions.
- Do not introduce black or high-contrast borders/rings in forms, tables, cards, dialogs, dropdowns, or focus states.
- Focus states should be subtle: prefer tokenized `ring-1`, `ring-primary/30`, `border-border`, and soft border colors over thick dark outlines.
- Brand color must come from semantic tokens such as `bg-primary`, `text-primary`, `text-primary-foreground`, and `border-primary`; do not hardcode Wonderland pink in component code.
- Prefer semantic text and surface utilities (`text-muted-foreground`, `text-foreground`, `bg-card`, `bg-muted/50`, `border-border`) before adding raw color values.
- Outline buttons should remain visually light. Destructive secondary actions should use outline styling with soft red borders/text, not heavy filled dark styling unless explicitly requested.
- Every button that triggers an API call or server mutation must show a loading state and be disabled while the request is in progress.
- Every API call or server mutation must provide user feedback with toast success/error messages.
- Toasts must not be duplicated. Use a single source of truth for success feedback after redirects, clear query params after showing redirect toasts, and guard effects with refs or equivalent when needed.
- Dialogs must use the shared `DialogPanel` design system from `@/components/ui/dialog` instead of ad-hoc `DialogContent` padding. Structure: `DialogPanel` → `DialogPanelHeader` (`DialogPanelTitle`, `DialogPanelDescription`) → optional `DialogPanelToolbar` → `DialogPanelBody` → `DialogFooter`. Use `DialogPanelForm` when the modal contains a form. Sizes: `xs` (420px confirm), `sm`, `md`, `lg`, `xl`. Footer is always full-width with `px-6 py-4`, `gap-3`, right-aligned actions on desktop.
- Margins and spacing must be proportional and consistent with nearby reference pages. If unsure, inspect existing project pages before implementing UI.

---
> Source: [syabanf/new-habitat](https://github.com/syabanf/new-habitat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
