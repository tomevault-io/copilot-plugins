## i18n-catalog-layers

> Layered i18n catalog merge order for packages/i18n-catalog and app i18n paths


# i18n catalog layers

When editing translation sources or the shared catalog package, follow the **layered merge** model.

## Layout

```
packages/i18n-catalog/
├── shared/{originals,overrides,compiled}/
├── consumer/{originals,overrides,compiled}/
├── management/{originals,overrides,compiled}/
└── mobile/{originals,overrides,compiled}/
```

## Merge order (compile time)

| App | Layers (later wins on key conflict) |
| --- | ----------------------------------- |
| `apps/web` | `shared` → `consumer` |
| `apps/mobile` | `shared` → `consumer` → `mobile/` |
| `apps/management-web` | `shared` → `management/` |

Do **not** duplicate the same key path in multiple layers. CI should reject duplicates.

## Runtime (per platform)

- **Web / management-web:** next-intl — load **compiled** merge output, not raw originals alone.
- **Mobile:** i18next + expo-localization — bundle merged JSON; do not use next-intl.

Share **JSON data**, not i18n libraries. `@podverse/ui` stays string-free (**shared-ui-i18n** rule).

## Rich text

Keys used on mobile must not require next-intl rich tags (`<link>`, etc.). Use plain strings or app-specific markup on RN.

## Related

- [docs/localization/I18N.md](/docs/localization/I18N.md)
- Master plan Track 17 (steps 17.0, 17.8–17.12)
- **i18n-management** rule

---
> Source: [podverse/podverse](https://github.com/podverse/podverse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
