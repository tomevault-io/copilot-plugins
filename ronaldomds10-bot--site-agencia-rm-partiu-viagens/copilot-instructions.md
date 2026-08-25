## site-agencia-rm-partiu-viagens

> This is a static single-page website for RM Partiu Viagens. The project uses plain HTML, inline CSS, inline JavaScript, Google Fonts, anchor navigation, WhatsApp links, Instagram links, CSS media queries, gradients, animations, and one local JPG asset.

# Repository Guidelines

## Project Structure & Technologies

This is a static single-page website for RM Partiu Viagens. The project uses plain HTML, inline CSS, inline JavaScript, Google Fonts, anchor navigation, WhatsApp links, Instagram links, CSS media queries, gradients, animations, and one local JPG asset.

- `index.html`: all page markup, styles, interactions, responsive rules, and navigation.
- `Post instagram de dicas para viajar com o pet11.jpg`: local image asset.
- `README.md`: short project description.
- `.agents/`: currently empty.

There is no npm package, framework, build tool, backend, database, or automated test suite.

## Run & Test Commands

Open the site directly in a browser:

```powershell
cmd /c start index.html
```

Inspect repository files:

```powershell
Get-ChildItem -Force
```

Search important links and actions:

```powershell
Select-String -Path index.html -Pattern "wa.me|instagram|onclick|href="
```

Check pending notes:

```powershell
Select-String -Path index.html -Pattern "TODO|FIXME"
```

## Code Organization & Style

Keep the current single-file structure unless the owner authorizes a broader refactor. Organize changes inside the existing sections: navigation, hero, destinations, benefits, steps, testimonials, Instagram, CTA, footer, floating WhatsApp assistant, and script.

Use descriptive class names matching the existing style, such as `hero-content`, `dest-card`, and `wa-bot-bubble`. Keep repeated brand colors in `:root`. Preserve semantic anchors and section IDs used by navigation.

## Responsiveness Rules

Maintain layouts for mobile, tablet, and desktop. Mobile changes must work below `600px`; destination cards already switch to one column below `700px`. Tablet layouts should remain readable between `701px` and `1024px`. Desktop layouts should preserve the multi-column grids and centered page rhythm.

After responsive edits, verify there is no horizontal scroll, clipped text, overlapping buttons, broken sticky navigation, or unreadable CTA content.

## Brand & Official Links

Preserve the RM Partiu Viagens visual identity: orange, gold, dark brown/black tones, premium travel tone, Playfair Display headings, Montserrat body text, rounded CTAs, gradients, and WhatsApp green for contact actions.

Do not modify, redraw, rename, replace, or recreate the RM logo without explicit authorization. Keep the official WhatsApp number `5511987569836` and Instagram URL `https://www.instagram.com/rmpartiuviagens` unless the owner provides a new official link.

## Testing Requirements

After every change, manually test all buttons, links, forms if added, anchor navigation, WhatsApp actions, Instagram actions, close behavior for the floating assistant, and scrolling. Capture screenshots for visual or responsive changes.

## Publishing & Pull Requests

Do not publish, deploy, push, or release any change without explicit owner authorization. Commits and pull requests should summarize changed sections, list manual tests performed, and include screenshots for visual updates.

---
> Source: [ronaldomds10-bot/Site-Agencia-RM-Partiu-Viagens](https://github.com/ronaldomds10-bot/Site-Agencia-RM-Partiu-Viagens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
