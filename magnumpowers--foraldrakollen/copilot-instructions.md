## foraldrakollen

> En forskningsbaserad webbresurs för svenska föräldrar som vill hjälpa sina barn (8-13 år) att vara trygga online.

# Föräldrakollen

En forskningsbaserad webbresurs för svenska föräldrar som vill hjälpa sina barn (8-13 år) att vara trygga online.

## Projektöversikt

**Live-sajt:** https://foraldrakollen.nu

**GitHub:** https://github.com/magnumpowers/foraldrakollen

**Skapare:** Magnus Paues och Fredrik Hanefalk ([Veckans AI](https://veckans.ai))

## Tech Stack

- **Framework:** Next.js 16 med App Router
- **Språk:** TypeScript
- **Styling:** Tailwind CSS v4 (med @tailwindcss/postcss)
- **Ikoner:** Lucide React
- **Hosting:** Vercel
- **Domän:** foraldrakollen.nu (DNS via one.com)

## Projektstruktur

```
src/
├── app/
│   ├── page.tsx          # Startsida
│   ├── guider/page.tsx   # Interaktiva steg-för-steg-guider
│   ├── forskning/page.tsx # Forskningssammanfattning med källhänvisningar
│   ├── risker/page.tsx   # Risker och varningar
│   ├── om/page.tsx       # Om Föräldrakollen
│   ├── layout.tsx        # Root layout med fonts och metadata
│   └── globals.css       # Tailwind och globala stilar
├── components/
│   ├── Navigation.tsx    # Huvudnavigation
│   ├── Footer.tsx        # Footer
│   ├── Logo.tsx          # SVG-logotyp
│   └── GuideCard.tsx     # Interaktiva guide-accordion
├── lib/
│   └── guides-data.ts    # All guide-data (7 områden)
public/
├── icon.svg              # Favicon
└── veckans-ai.jpg        # Bild på grundarna
```

## Kommandon

```bash
npm run dev      # Starta utvecklingsserver
npm run build    # Bygga för produktion
npm run lint     # Köra linter
vercel --prod    # Deploya till produktion
```

## De 7 guideområdena

1. **Skärmtid nattetid** - Skydda sömnen
2. **Algoritmiskt innehåll** - TikTok, Reels, Shorts
3. **Okända kontakter** - Blockera meddelanden från främlingar
4. **Platstjänster** - Stäng av onödig spårning
5. **Vuxeninnehåll** - Filtrera och blockera
6. **Appgodkännande** - Kräv förälderns OK för nya appar
7. **Köp i appar** - Förhindra oavsiktliga köp

## Designprinciper

- **Färger:** Primary (teal/grön), Coral (korall/orange), Sand (beige bakgrund)
- **Typsnitt:** Nunito (display/rubriker), Inter (brödtext)
- **Ton:** Trygg, förtroendeingivande, icke-dömande
- **Målgrupp:** Svenska föräldrar med barn 8-13 år

## Källor och referenspublikationer

All statistik på sidan ska ha källhänvisning med direktlänk till publikationen.

| Statistik | Källa | Länk |
|-----------|-------|------|
| "1 av 4 barn utsatts för grooming" | Unga, sex och internet efter #metoo (2021) | [PDF](https://allmannabarnhuset.se/wp-content/uploads/2022/09/Unga_Sex_Och_Internet_Efter_MeToo_2021.pdf) |
| "1-2 tim skärmtid för barn 6-12 år" | Folkhälsomyndigheten (2024) | [PDF](https://www.folkhalsomyndigheten.se/contentassets/201463a976054dde8ad7aa8a47861c0a/rekommendationer-digitala-medier-barns-ungas-medieanvandning.pdf) |
| "60 min dagsgräns på TikTok" | TikTok Newsroom (2023) | [Länk](https://newsroom.tiktok.com/sv-se/nya-funktioner-for-ungdomar-och-familjer-pa-tiktok) |
| "16 år för DM på TikTok" | TikTok Safety | [Länk](https://www.tiktok.com/safety/sv-se/guardians-guide/) |

## Viktiga innehållsriktlinjer

- Innehållet ska vara nyanserat och forskningsbaserat
- Undvik kategoriska påståenden - använd formuleringar som "kan öka risken"
- Inkludera alltid "Vid akut fara - ring 112" vid säkerhetsrelaterat innehåll
- Klarföra att uppgiftsfusk gäller spel/appar, inte skola/vård/myndigheter
- All statistik måste ha källhänvisning med klickbar länk till originalpublikationen
- Ange publikationsår i källhänvisningen

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/magnumpowers)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/magnumpowers)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
