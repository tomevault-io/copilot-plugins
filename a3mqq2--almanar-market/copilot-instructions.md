## almanar-market

> This project is a production-grade Laravel POS and Inventory System.

# Project AI Instructions

This project is a production-grade Laravel POS and Inventory System.

All generated code must strictly follow these rules.

---

## 1. Code Style

- Do NOT write any comments.
- No inline, block, or doc comments.

- Follow PSR-12.
- Clean and readable code only.

---

## 2. UI Design Philosophy

- NO fancy gradients.
- NO neon colors.
- NO glassmorphism.
- NO heavy animations.

- Prefer:
  - Solid colors
  - Neutral palette
  - High contrast
  - Readable fonts

- UI must prioritize usability over aesthetics.

---

## 3. Color Rules

Allowed:

- Light Mode:
  - White / Light Gray backgrounds
  - Dark text

- Dark Mode:
  - Dark Gray / Near Black backgrounds
  - Light text

- One primary color only.
- One danger color.
- One success color.

Forbidden:

- Gradients
- Glows
- Rainbow UI

---

## 4. Dark Mode Support (Mandatory)

- Every page must support Dark Mode.
- No hardcoded colors in HTML or CSS.
- Use CSS variables for all colors.

- Define themes using:

  --bg-primary
  --bg-secondary
  --text-primary
  --text-secondary
  --border-color
  --card-bg
  --table-bg
  --input-bg

- Support:

  - System preference (prefers-color-scheme)
  - Manual toggle
  - Saved user preference

- Dark Mode must:

  - Preserve contrast
  - Keep tables readable
  - Avoid gray-on-gray text
  - Avoid low-opacity text

- No page may break visually in Dark Mode.

---

## 5. Layout Rules

- Grid-based layout.
- Consistent spacing.
- No floating elements.

---

## 6. UX Rules

- No alert() or confirm().
- Use SweetAlert2, Toasts, Modals.

- Predictable behavior only.

---

## 7. Performance

- Fast UI > effects.
- Lazy load when possible.

---

## 8. Accessibility

- WCAG contrast compliance.
- Keyboard navigation.

---

## 9. Tables & Forms

- Tables:
  - Sorting
  - Pagination
  - Sticky headers

- Forms:
  - Clear labels
  - No placeholder-only fields

---

## 10. Icons & Fonts

- Simple icons only.
- System fonts preferred.

---

## 11. Consistency

- One theme system.
- No per-page theming.

---

## 12. Final UI Rule

If the interface looks decorative,
it is wrong.

This is operational software.

Design for long working hours.

## Row Interaction & Actions Policy

- All data tables must support row click navigation.

- Clicking anywhere on a table row must:
  -> Open the detailed view of the record.

- Do NOT add a separate "Actions" column.

- Do NOT use action buttons inside tables.

- The Actions column is forbidden.

- All primary actions must be placed in:

  -> A clean header bar inside the details page.

- The header bar must contain only:

  - Edit
  - Delete
  - Back
  - More (Dropdown if needed)

- Table rows must show hover and focus states.

- Cursor must be pointer on hover.

- No inline buttons inside rows.

- No nested clickable elements inside rows.

---
> Source: [a3mqq2/almanar-market](https://github.com/a3mqq2/almanar-market) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
