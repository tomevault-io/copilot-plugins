## leah-test-10-yottagraph

> Apply when working with user preferences, settings persistence, usePrefsStore, the Pref class, KV storage, or app namespacing (NUXT_PUBLIC_APP_ID).


# User Preferences

Preferences are persisted to Upstash Redis (KV) via `usePrefsStore()` from
`~/composables/usePrefsStore.ts`, backed by `KVPrefsStore` which calls
`/api/kv/*` endpoints. The KV store is provisioned automatically during
project creation and connected via Vercel env vars (`KV_REST_API_URL`,
`KV_REST_API_TOKEN`).

## Two-Tier Namespacing

Preferences are scoped by the app ID set in `NUXT_PUBLIC_APP_ID`:

- **App-specific**: `/users/{userId}/apps/{appId}/settings/general`
- **Global (cross-app)**: `/users/{userId}/global/settings/general`

## The Pref Class

`Pref<T>` is a reactive wrapper that auto-syncs to KV. Defined in
`usePrefsStore.ts`.

```typescript
const myPref = new Pref<string>(docPath, 'fieldName', 'defaultValue');
await myPref.initialize();

myPref.r.value; // reactive ref (use in templates)
myPref.v;       // getter shorthand
myPref.set('new value'); // persists to KV
```

Values are JSON-serialized. The `Pref` sets up a watcher after
`initialize()` so any change to `.r` auto-persists.

## usePrefsStore()

```typescript
const { readDoc, listDocuments, deleteCollection } = usePrefsStore();
```

The backing store auto-initializes on first use — no need to call
`initializePrefsStore()` manually.

## Local Development

When KV credentials (`KV_REST_API_URL`, `KV_REST_API_TOKEN`) are not set
(e.g. local dev without Vercel env vars), the KV server routes return
`undefined` for reads and silently skip writes. Prefs will work with
their default values but won't persist across page refreshes.

For full local persistence, copy the KV credentials from your Vercel
project settings into your local `.env` file (or use the Portal's
"Get .env file" feature).

## Direct API Alternative

If you prefer not to use the `Pref<T>` class, you can call the KV
routes directly:

```typescript
// Read
const value = await $fetch('/api/kv/read', {
  params: { docPath: '/users/abc/settings', fieldName: 'theme' }
});

// Write
await $fetch('/api/kv/write', {
  method: 'POST',
  body: { docPath: '/users/abc/settings', fieldName: 'theme', value: '"dark"' }
});
```

## Feature-Scoped Preferences

Features should namespace preferences under the app's prefix:

```typescript
function useMyFeaturePrefs() {
    const { appId } = useRuntimeConfig().public;
    const { userId } = useUserState();
    const path = `/users/${userId.value}/apps/${appId}/features/my-feature`;

    const myPref = new Pref<boolean>(path, 'enabled', true);
    return { myPref };
}
```

## KV Architecture

- **Server**: `server/utils/redis.ts` initializes the Upstash Redis client
  from `KV_REST_API_URL` and `KV_REST_API_TOKEN` env vars
- **API routes**: `server/api/kv/` (read, write, delete, documents)
- **Client store**: `utils/kvPrefsStore.ts` implements `PrefsStore` by
  calling the KV API routes
- **Key format**: `prefs:users:{userId}:apps:{appId}:settings:general`
  (doc-style paths converted to colon-separated Redis keys)

## Local Development Without KV

When KV credentials (`KV_REST_API_URL`, `KV_REST_API_TOKEN`) aren't configured,
`Pref<T>` still works with its default value but won't persist across page
refreshes. For local dev, use `localStorage` directly as a lightweight
alternative:

```typescript
const saved = localStorage.getItem('watchlist');
const watchlist = ref<string[]>(saved ? JSON.parse(saved) : []);
watch(watchlist, (val) => localStorage.setItem('watchlist', JSON.stringify(val)), { deep: true });
```

Use `Pref<T>` for production persistence and `localStorage` for local-only
development when KV isn't available.

## Scope Guidance

| App-specific | Global |
|---|---|
| Layout prefs, favorites | Language |
| Watchlists, feature settings | Accessibility |
| Feature-specific settings | Timezone, notifications |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Lovelace-AI) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
