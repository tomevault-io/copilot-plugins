## portfolio-cv-site

> Standardy dostępności: kontrast, focus-visible, skip-links, pełna obsługa klawiaturą; aria-label i tekst dla elementów interaktywnych; aria-busy/live przy interakcjach; testy z jest-axe/axe-core i Playwright snapshoty.


# 50 — Dostępność (WCAG 2.2)

**Wymagane:**
- Kontrast, focus-visible, skip-links, pełna obsługa klawiaturą.
- Łącza i przyciski mają czytelne nazwy dostępności (`aria-label` lub tekst).
- Interakcje asynchroniczne → `aria-busy`, `aria-live` gdzie stosowne.

**Testy**
- **jest-axe/axe-core** dla komponentów.
- **Playwright**: snapshoty a11y i nawigacja klawiaturą.

**Snippety**: `snippets/testing-a11y.md`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Piotr-Pod)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Piotr-Pod)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
