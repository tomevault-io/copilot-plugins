## reaction-game

> Auto-add copyright footer with green heart to HTML files


# Copyright Footer Rule

When working with HTML files, automatically ensure the copyright footer exists:

1. Check if a `<footer>` element already exists in the HTML
2. If a footer exists:
   - Replace its content with: `Made with <span class="heart">♥</span> © James Geiger 2026`
   - Ensure it has the `class="footer"` attribute
3. If no footer exists:
   - Add a `<footer class="footer">` element after the `<main>` content (or before `</body>` if no `<main>` exists)
   - Include the text: `Made with <span class="heart">♥</span> © James Geiger 2026`
4. Add CSS to color the heart green using the project's success/accent color

## HTML Structure

```html
<footer class="footer">
  Made with <span class="heart">♥</span> © James Geiger 2026
</footer>
```

## CSS Styling

```css
.footer {
  text-align: center;
  padding: 24px;
  color: var(--text-secondary, #94a3b8);
  font-size: 0.86rem;
}

.heart {
  color: var(--success, #22c55e);
}
```

Adapt the CSS variable names to match the existing project theme if needed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thegeigerux) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
