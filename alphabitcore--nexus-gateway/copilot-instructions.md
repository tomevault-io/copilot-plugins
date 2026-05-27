## i18n-mandatory

> All user-visible UI text uses t('namespace:section.key') from react-i18next


# i18n mandatory (binding)

You are editing UI code. **Every user-visible string in JSX must go through `t('namespace:section.key')`** from `react-i18next`. Never hardcode English strings in components.

Surfaces this rule covers:

- Labels, titles, descriptions.
- Placeholders.
- Button text.
- Table headers.
- Error messages.
- Empty states.
- Tooltips.

## How to add a new key

1. Add to **all three locale files**:
   - `packages/control-plane-ui/src/i18n/locales/en/pages.json` (or `common.json` / `nav.json`).
   - `packages/control-plane-ui/src/i18n/locales/zh/...`.
   - `packages/control-plane-ui/src/i18n/locales/es/...`.
2. Copy to `packages/control-plane-ui/public/locales/{en,zh,es}/...`.
3. Verify key counts match across all 3 locales:

```bash
npm run check:i18n
```

CI gate. Pre-merge required.

## Technical terms

Stay in English across all locales: `API`, `SSO`, `Provider`, `Model`, `Agent`, `Device`, `Hook`, `Token`, `mTLS`, `OAuth`, `PKCE`, `JWT`, etc.

## Forbidden

```tsx
<button>Save</button>                // ❌ hardcoded
<input placeholder="Email" />        // ❌ hardcoded
<h1>Dashboard</h1>                   // ❌ hardcoded
```

## Correct

```tsx
<button>{t('pages:settings.save')}</button>
<input placeholder={t('common:fields.email')} />
<h1>{t('pages:dashboard.title')}</h1>
```

Skipping this rule requires **explicit user approval** in chat.

---
> Source: [AlphaBitCore/nexus-gateway](https://github.com/AlphaBitCore/nexus-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
