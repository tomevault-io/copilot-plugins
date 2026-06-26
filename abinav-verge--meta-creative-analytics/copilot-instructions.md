## meta-creative-analytics

> Internal Meta Ads creative analytics dashboard for team use. Hosted on Vercel. No auth required.

# Creative Analytics Dashboard

## Project Overview
Internal Meta Ads creative analytics dashboard for team use. Hosted on Vercel. No auth required.

## Stack
- **Framework**: Next.js (App Router, JavaScript)
- **Styling**: Tailwind CSS + DaisyUI (dark theme default)
- **Charts**: Recharts (line, bar, etc.)
- **Heatmaps**: @nivo/heatmap
- **Data**: Mock JSON initially → Meta Marketing API later

## Folder Structure
```
/app            - Next.js App Router pages and layouts
/components     - Shared React components
/data           - Mock data files (JSON)
/utils          - Helper/utility functions
/lib            - Shared logic, API clients, etc.
```

## Pages
| Route                  | Page Name            | Status      |
|------------------------|----------------------|-------------|
| `/`                    | Dashboard            | Placeholder |
| `/creative-testing`    | Creative Testing     | Placeholder |
| `/reports`             | Reports              | Placeholder |
| `/persona-performance` | Persona Performance  | Placeholder |

## Ad Naming Conventions

### Image Ads
Format: `[Brand][AwarenessLevel][PsychLever][Format][Branded/Raw][FunnelStage][Emotion]_[AudienceSignal]`

Example: `Nike_SolutionAware_SocialProof_Testimonial_Branded_TOF_Inspirational_Aspirational`

| Segment         | Examples                                          |
|-----------------|---------------------------------------------------|
| Brand Name      | Nike, Adidas, etc.                                |
| Awareness Level | Unaware, ProblemAware, SolutionAware, ProductAware, MostAware |
| Psych Lever     | SocialProof, FOMO, Authority, Scarcity, Curiosity |
| Format          | Testimonial, UGC, Static, Carousel                |
| Branded/Raw     | Branded, Raw                                      |
| Funnel Stage    | TOF, MOF, BOF                                     |
| Emotion/Tone    | Inspirational, Humorous, Fear, Urgent             |
| Audience Signal | Aspirational, PainPoint, ValueSeekers             |

### Video Ads
Naming convention TBD.

## Design Guidelines
- Clean, minimal, professional aesthetic
- DaisyUI components throughout
- Dark theme as default (`data-theme="dark"`)
- Avoid heavy decorative elements — data should be front and center

## Development Notes
- Prefer JavaScript over TypeScript for simplicity
- Keep mock data in `/data` as plain `.json` files
- When integrating Meta Marketing API, put API logic in `/lib`
- Use DaisyUI utility classes before writing custom CSS

---
> Source: [abinav-verge/meta-creative-analytics](https://github.com/abinav-verge/meta-creative-analytics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
