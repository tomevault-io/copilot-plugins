## immosquare-cookies

> Rails Engine pour bannière de consentement cookies GDPR.

# CLAUDE.md

Rails Engine pour bannière de consentement cookies GDPR.

## Structure

```
app/views/immosquare-cookies/_consent_banner.html.erb  # Partial principal (ERB + JS inline)
app/assets/stylesheets/immosquare-cookies.scss         # Styles avec CSS custom properties
config/locales/*.yml                                   # Traductions (7 langues)
lib/immosquare-cookies/engine.rb                       # Rails Engine
```

## Implémentation

- **Rendu conditionnel** : la bannière ne s'affiche que si `cookies[cookie_name].blank?` (condition `if` en fin de partial)
- **Suppression cookies** : le JS supprime les cookies sur plusieurs variations de domaine (actuel, `.example.com`, `.example.co.uk`)
- **Paramètres** : utilise `defined?()` pour vérifier les variables locales passées au partial

## Conventions

- JavaScript inline dans l'ERB (zéro dépendance externe)
- SCSS avec `@use "sass:color"`
- Pas de tests (gem minimaliste)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/immosquare) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
