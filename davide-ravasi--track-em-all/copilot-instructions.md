## track-em-all

> - CRA uses `process.env.REACT_APP_*`


# Environment Variables: CRA vs Vite

- CRA uses `process.env.REACT_APP_*`
- Vite uses `import.meta.env.VITE_*`

Migration guidance:

- Rename all `REACT_APP_FOO` → `VITE_FOO`
- Replace `process.env.REACT_APP_FOO` → `import.meta.env.VITE_FOO`

Where used (reference):

- [src/utils.tsx](mdc:src/utils.tsx)
- [src/pages/ShowPage/ShowPage.tsx](mdc:src/pages/ShowPage/ShowPage.tsx)
- [src/pages/PersonPage/PersonPage.tsx](mdc:src/pages/PersonPage/PersonPage.tsx)
- [src/pages/EpisodePage/EpisodePage.tsx](mdc:src/pages/EpisodePage/EpisodePage.tsx)
- [src/components/ShowVideo/ShowVideo.tsx](mdc:src/components/ShowVideo/ShowVideo.tsx)
- [src/components/ShowEpisodes/ShowEpisodes.tsx](mdc:src/components/ShowEpisodes/ShowEpisodes.tsx)
- [src/features/auth/authSlice.tsx](mdc:src/features/auth/authSlice.tsx)
- [src/features/auth/authService.js](mdc:src/features/auth/authService.js)
- [src/firebase/firebase.js](mdc:src/firebase/firebase.js)

Note:

- Keep `.env.example` updated and never commit real secrets.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davide-ravasi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
