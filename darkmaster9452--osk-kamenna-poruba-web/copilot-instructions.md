## osk-kamenna-poruba-web

> Webová stránka futbalového klubu OŠK Kamenná Poruba. Frontend je statické HTML/CSS/JS nasadené cez Vercel, backend je Express.js API s Prisma ORM a PostgreSQL (Neon).

# OŠK Kamenná Poruba – Web

Webová stránka futbalového klubu OŠK Kamenná Poruba. Frontend je statické HTML/CSS/JS nasadené cez Vercel, backend je Express.js API s Prisma ORM a PostgreSQL (Neon).

## Štruktúra projektu

```
/                        # Frontend (statické HTML)
  index.html             # Hlavná stránka
  pages/                 # Podstránky (akademia, atim, blog, galeria, matches, ...)
  assets/
    images/              # Obrázky sponzorov a loga (raw GitHub URLs)
    js/
      cloudinary-assets.js  # Načíta obrázky z Cloudinary API a napĺňa [data-cloudinary-id]
      trainings.js

backend/                 # Express.js API
  src/
    server.js / app.js
    routes/              # REST endpointy (players, blog, auth, cloudinary, trainings, ...)
    services/
    middleware/
    config/
  prisma/
    schema.prisma        # Databázová schéma (PostgreSQL/Neon)
    seed.js

api/                     # Vercel serverless proxy → presmerovanie /api/* na backend
vercel.json              # Rewrite: /api/* → /api/proxy, cache pre /assets/images/
```

## Príkazy

### Backend
```bash
cd backend
npm run dev          # Spustí dev server s hot-reload (node --watch)
npm start            # Production (prisma db push + node src/server.js)
npm test             # Jest testy
npm run db:push      # Synchronizuj Prisma schéma
npm run db:seed      # Seed databázy
npm run db:studio    # Prisma Studio UI
```

### Frontend
Žiadny build krok – statické súbory sa servujú priamo. Lokálne stačí Live Server alebo akýkoľvek HTTP server.

## Kľúčové veci

- **Obrázky sponzorov**: uložené v `assets/images/` s Cloudinary suffixom (napr. `kofola_u2pzvx.png`). HTML má `data-cloudinary-id="kofola"` a `src="https://raw.githubusercontent.com/DarkMaster9452/OSK-Kamenna-Poruba-Web/main/assets/images/..."` ako fallback/primárny zdroj. `cloudinary-assets.js` môže src prepísať Cloudinary URL ak je backend dostupný.
- **API base**: Na produkcii `/api`, lokálne `http://localhost:4000/api`. Konfigurovateľné cez `localStorage.setItem('OSK_API_BASE', ...)`.
- **Auth**: JWT cookies, bcryptjs na hash hesiel.
- **Databáza**: PostgreSQL cez Neon (connection string v `.env` backendx).
- **Nasadenie**: Vercel (frontend + proxy), backend nasadený samostatne.

## GitHub repo
`DarkMaster9452/OSK-Kamenna-Poruba-Web` (main branch)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DarkMaster9452)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DarkMaster9452)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
