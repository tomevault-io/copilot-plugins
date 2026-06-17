## capacitor-google-auth-skill

> Implement native Google Sign-In for Capacitor mobile apps. Use this skill whenever the user needs Google authentication in a Capacitor app, is dealing with OAuth issues in a WebView or WKWebView, gets a "disallowed_useragent" error from Google, or is building a hybrid mobile app that needs Google login on iOS or Android. Also use when the user mentions @capgo/capacitor-social-login, signInWithIdToken, or native social login in a Capacitor context.


# Native Google Auth in Capacitor Apps

## Why this skill exists

Google blocks OAuth redirect flows inside WKWebView (which Capacitor uses). If you try the standard web OAuth flow in a native Capacitor app, Google returns a `disallowed_useragent` error. No amount of user-agent spoofing or workarounds fixes this — Google uses JS-based detection too.

The solution is a **platform-aware split**: use the native Google Sign-In SDK on mobile to get an ID token, then exchange that token with your backend (Supabase, Firebase, custom server, etc.) via token-based auth like `signInWithIdToken()`.

## Architecture

```
Web:    User → Supabase/backend OAuth redirect → callback → session
Native: User → Native Google SDK → ID token → backend signInWithIdToken() → session
```

The key insight: on native, you never go through a browser redirect. The native SDK presents Google's own sign-in UI, returns an ID token directly, and your backend validates that token to create a session.

## Plugin

Use `@capgo/capacitor-social-login` — this is the actively maintained plugin. Do NOT use the archived `@codetrix-studio/capacitor-google-auth`.

The plugin's major version tracks Capacitor's major version (Capacitor 7 → plugin v7.x, Capacitor 8 → plugin v8.x). Always match the project's Capacitor version.

## Implementation steps

1. **Scan the project** — understand the current auth setup, framework (Next.js, React, Vue, etc.), Capacitor version, and whether Google OAuth already works on web
2. **Install the plugin** — `npm install @capgo/capacitor-social-login` with the correct major version
3. **Initialize the plugin** at app startup with both web and iOS client IDs
4. **Implement platform-aware sign-in** — detect native vs web at runtime and branch:
   - Native: use `SocialLogin.login()` → get ID token → exchange with backend
   - Web: use standard OAuth redirect flow
5. **Guide the user through manual config** they must do themselves:
   - Google Cloud Console: create an iOS OAuth Client ID (Bundle ID must match Xcode project)
   - Backend dashboard (Supabase/Firebase): register the iOS client ID + enable "Skip nonce checks" if using Supabase
   - iOS `Info.plist`: add reversed iOS client ID as a URL scheme
   - Environment variables for both client IDs
6. **Sync native projects** — run `npx cap sync ios` and `npx cap sync android`

For detailed code examples, config values, type gotchas, and a list of approaches that don't work, see [references/implementation.md](references/implementation.md).

## Critical type gotcha

`GoogleLoginResponse` from the plugin is a union type — `idToken` only exists on the `Online` variant. Always narrow before accessing:

```typescript
if (!result || !('idToken' in result) || !result.idToken) {
  throw new Error('No ID token from Google')
}
```

## Dead ends (don't waste time on these)

- OAuth redirect in WKWebView → `disallowed_useragent`
- User-agent spoofing → Google uses JS detection too
- SFSafariViewController via `@capacitor/browser` → can't present with custom SceneDelegate, redirect back via custom URL scheme is unreliable
- `window.location.href` to external OAuth URL → opens Safari but return to app is flaky

---
> Source: [sukibk/capacitor-google-auth-skill](https://github.com/sukibk/capacitor-google-auth-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
