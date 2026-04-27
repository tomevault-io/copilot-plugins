## artistmixer

> - **Runtime/UI**: React 19 + TypeScript, Vite 7, SWC plugin.


# ArtistMixer Frontend – Conventions and Rules

### 1) Technologies and Tooling

- **Runtime/UI**: React 19 + TypeScript, Vite 7, SWC plugin.
- **Routing**: `react-router-dom@7` with basename `/ArtistMixer`.
- **State**: Jotai (with `atomWithStorage` for persistence).
- **UI**: MUI v7 with Emotion.
- **HTTP**: `fetch` via centralized `apiClient` with Bearer token.
- **Testing**: Vitest + Testing Library (jsdom), setup in `src/setupTests.ts`.
- **Linting**: ESLint v9 with TypeScript + React Hooks configs.
- **Aliases**: `@app`, `@features`, `@pages`, `@shared`, `@state` (Vite + TS config).

Examples:

```9:16:src/app/App.tsx
    <AuthProvider>
      <BrowserRouter basename="/ArtistMixer">
        <Routes>
          <Route path="/" element={<PlaylistMixerPage />} />
          <Route path="/other" element={<OtherSpotifyPage />} />
        </Routes>
      </BrowserRouter>
    </AuthProvider>
```
```8:16:vite.config.ts
  base: '/ArtistMixer/',
  resolve: {
    alias: {
      '@app': path.resolve(__dirname, 'src/app'),
      '@features': path.resolve(__dirname, 'src/features'),
      '@pages': path.resolve(__dirname, 'src/pages'),
      '@shared': path.resolve(__dirname, 'src/shared'),
      '@state': path.resolve(__dirname, 'src/state'),
    }
  },
```
```16:21:vitest.config.ts
  test: {
    environment: 'jsdom',
    setupFiles: ['src/setupTests.ts'],
    globals: true,
  },
```

### 2) Project Slicing and Layout

- **Feature-first** in `src/features/<feature>`:
  - `components/` UI for that feature
  - `services/` network/API for that feature
  - `hooks/` feature hooks
  - `types/` only if feature-specific; otherwise use `src/shared/types`
  - `index.ts` re-exports public surface
- **Pages**: Route containers in `src/pages` export via `src/pages/index.ts`.
- **App shell**: `src/app` for top-level app and tests.
- **Shared**: Cross-feature `components/`, `constants/`, `types/`, `utils/`.
- **State**: Jotai atoms/selectors in `src/state` (cross-feature client state).

Example feature layout:

- `src/features/selectArtists/components/...`
- `src/features/generatePlaylist/services/createSpotifyPlaylist.ts`
```1:4:src/pages/index.ts
export { PlaylistMixerPage } from './PlaylistMixerPage';
export { OtherSpotifyPage } from './OtherSpotifyPage';
```


### 3) Routing Rules

- Use `<BrowserRouter basename="/ArtistMixer">` (already in `App.tsx`).
- Add routes by exporting components in `src/pages` and adding `<Route>` in `App.tsx`.
- For tests, use `<MemoryRouter>`.
```8:16:src/app/App.test.tsx
  const { container } = render(
    <AuthProvider>
      <MemoryRouter>
        <PlaylistMixerPage />
      </MemoryRouter>
    </AuthProvider>
  );
```


### 4) State Management Rules (Jotai)

- Cross-feature persisted state uses `atomWithStorage` in `src/state`.
- Derivations use `atom(get => ...)`, write-only use `atom(null, (get, set, arg) => ...)`.
- Use feature components/hooks to read/write with `useAtom`/`useSetAtom`.
```5:17:src/state/playlistAtoms.ts
export const draftPlaylistAtom = atomWithStorage<PlaylistContract>(
  'draftPlaylist',
  new Playlist()
);
...
export const clearPlaylistAtom = atom(null, (_get, set) => {
  set(draftPlaylistAtom, new Playlist());
});
```
```7:15:src/state/selectedArtistsAtoms.ts
export const selectedArtistsAtom = atomWithStorage<Artist[]>(
  'selectedArtists',
  []
);
export const selectedArtistIdsAtom = atom((get) => {
  const artists = get(selectedArtistsAtom);
  return new Set(artists.map((a) => a.id));
});
```


### 5) Persistence Management

- **Local (client)**: persisted via `atomWithStorage` with keys `selectedArtists`, `draftPlaylist`.
- **Remote**: Spotify-backed API via AWS API Gateway base URL; include `Authorization: Bearer <token>` header via `apiClient`.
- Store `refreshToken` in `localStorage` (see auth service) and exchange for new access tokens.
```18:35:src/shared/utils/apiClient.ts
export const apiClient = async (
  endpoint: string,
  token: string,
  options: FetchOptions = {}
): Promise<Response> => {
  const { headers = {}, ...restOptions } = options;
  const response = await fetch(`${BASE_URL}${endpoint}`, {
    ...restOptions,
    headers: {
      'Content-Type': 'application/json',
      Authorization: `Bearer ${token}`,
      ...headers,
    },
  });
  return response;
};
```


### 6) API and Services Rules

- Put feature API calls in `src/features/<feature>/services`.
- Always call via `apiGet`/`apiPost` and pass `token` from `useAuth()`.
- Build query params with `URLSearchParams`; send JSON bodies; handle `response.ok`.
- Do not mutate server responses; copy/normalize (see playlist mapping).
```24:33:src/features/generatePlaylist/components/GeneratePlaylistControl.tsx
  apiGet('/playlists/random', token, {
    artists: selectedArtists.map((a) => a.id).join(','),
    playlistSize: playlistSize.toString(),
  })
    .then((res) => res.json())
```
```4:13:src/features/generatePlaylist/services/createSpotifyPlaylist.ts
export const createSpotifyPlaylist = async (
  name: string,
  playlist: PlaylistContract,
  token: string,
) => {
  const trackUris = playlist.songs.map((song) => song.uri);
  const response = await apiPost('/playlists/random', token, trackUris, { name });
  ...
}
```


### 7) Auth Rules

- Wrap app with `AuthProvider`; consume auth via `useAuth()`.
- OAuth flow: `getCode()` redirects to Spotify; backend exchanges `code` for `token` and `refreshToken`.
- `AuthProvider` reads `code` from URL or `refreshToken` from `localStorage` and sets `token`.
- On login/refresh failure, set `error` and stop authenticating.
```27:35:src/features/auth/services/spotifyAuth.ts
export const getToken = async (code: string): Promise<string> => {
  const res = await fetch(`${ENDPOINTS.AUTHENTICATE}?code=${code}`);
  const data = await res.json();
  localStorage.setItem('refreshToken', data.refreshToken);
  return data.token;
};
```
```15:27:src/features/auth/providers/AuthProvider.tsx
useEffect(() => {
  const code = new URLSearchParams(globalThis.location.search).get('code') ?? '';
  const refreshToken = localStorage.getItem('refreshToken');
  ...
}, [isAuthenticated, isAuthenticating, token, error]);
```


### 8) Entity Model Rules

- Prefer `interface` for shapes; use `src/shared/types` for cross-feature entities.
- Current core types: `Artist`, `Song`, `PlaylistContract`, `TrackListItem`, `SearchArtist`.
- Use a thin class only when encapsulating default construction (`Playlist`).
```6:14:src/shared/types/playlist.ts
export class Playlist implements PlaylistContract {
  version: string;
  songs: Song[];
  constructor() {
    this.version = '0.2';
    this.songs = [];
  }
}
```


### 9) UI and Styling Rules (MUI)

- Use MUI components and `sx` prop for styles; prefer responsive props.
- Use `GridLegacy` (Grid2) as established; keep alignment/spacing consistent.
- Use semantic roles and labels for accessibility (e.g., `role="search"`).
```68:92:src/features/selectArtists/components/Search/SearchForm.tsx
  return (
    <form
      role="search"
      onSubmit={handleSubmit}
      onKeyDown={(e) => e.key === 'Enter' && handleSubmit(e)}
    >
      <Grid2 container spacing={2} alignItems="center" justifyContent="end">
        ...
      </Grid2>
    </form>
  );
```


### 10) Testing Rules (Vitest + Testing Library)

- Co-locate tests as `*.test.tsx` next to components/pages.
- Use `MemoryRouter` for route components and wrap in `AuthProvider` when token context is needed.
- Use DOM queries via `screen.*`; add `@testing-library/jest-dom` matchers from setup.
- Keep tests unit-focused; mock `fetch` for service calls when necessary.
```1:6:src/setupTests.ts
import '@testing-library/jest-dom/vitest'
// jest-dom adds custom jest matchers ...
import '@testing-library/jest-dom';
```


### 11) TypeScript and Linting Rules

- TS is strict; avoid `any`; define types in `shared/types` or feature `types/`.
- Unused locals/params are disallowed; remove or prefix with `_` if needed.
- React Hooks rules enforced; follow dependency arrays.
```11:17:tsconfig.app.json
  "moduleResolution": "bundler",
  "noEmit": true,
  "jsx": "react-jsx",
  "strict": true,
  "noUnusedLocals": true,
  "noUnusedParameters": true,
```


### 12) Environment & Deployment Rules

- Vite base and router basename are `/ArtistMixer/` for GitHub Pages.
- Spotify Client ID via `import.meta.env.VITE_SPOTIFY_CLIENT_ID`.
- Do not hardcode secrets; only use public env vars prefixed with `VITE_`.

### 13) Adding a New Feature (Checklist)

1. Create `src/features/<feature>/{components,services,hooks,types}` and `index.ts` re-exports.
2. Define/consume types from `src/shared/types` when cross-cutting; otherwise feature `types/`.
3. Add any cross-feature state to `src/state` via `atomWithStorage` when you need persistence.
4. Implement services using `apiGet`/`apiPost` and `useAuth().token` for Authorization.
5. Build UI with MUI; keep spacing and accessibility consistent; use `sx` prop.
6. If route-level, add a page in `src/pages` and register a `<Route>` in `App.tsx`.
7. Write `*.test.tsx` co-located tests; wrap with providers (`AuthProvider`, `MemoryRouter`).
8. Run `yarn lint` and `yarn test` before committing.

### 14) Do/Don’t Summary

- Do: use aliases, central `apiClient`, Jotai atoms for state, Testing Library for tests, MUI for UI.
- Don’t: call `fetch` directly outside services, mutate server responses in-place, introduce Redux/Zustand, add global state outside `src/state`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cryosis7) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
