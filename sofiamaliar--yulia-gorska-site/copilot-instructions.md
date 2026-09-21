## yulia-gorska-site

> Landing page for an online course by Yulia Gorska — cosmetologist, aesthetic medicine specialist, owner of 2 clinics in Chisinau, Moldova. 13 years of practice. The course covers weight loss, nutrition, body procedures, skin care during weight loss — a comprehensive system, not a single-topic program.

# CLAUDE.md — dr.gorska Course Website

## Project Overview
Landing page for an online course by Yulia Gorska — cosmetologist, aesthetic medicine specialist, owner of 2 clinics in Chisinau, Moldova. 13 years of practice. The course covers weight loss, nutrition, body procedures, skin care during weight loss — a comprehensive system, not a single-topic program.

## Site Type
Single-page landing (no personal cabinet, no catalog, no clinic booking). Phase 1 only. Clinic bookings handled via a separate Instagram account (@honest_beauty_clinic_md).

## Page Structure (top to bottom)
1. Hero — strong hook headline + subheading + primary CTA button
2. For whom — target audience description (women 28–45, want to lose weight smartly, understand their body)
3. What's inside — course modules/program breakdown
4. About the author — Yulia Gorska, 13 years practice, 2 clinics in Chisinau, aesthetic medicine expert
5. Results / testimonials — client outcomes
6. Pricing — course price (€9.99–€20), CTA button → Lava.top payment
7. FAQ — common questions

## Payment
Via **Lava.top** (payment aggregator, handles taxes automatically). Clients primarily from Moldova, Russia, Ukraine, and Russian-speaking diaspora in Europe. Lava.top supports all these regions.

## Forms / Leads
No booking form on the site. All leads via Instagram DM or Telegram. If a "notify me" form is added later, it should send to a Telegram bot (to be built separately).

## Brand Identity

### Color Palette
```
--color-primary: #53161D;  /* burgundy — main accent */
--color-deep:    #30050E;  /* dark background */
--color-black:   #1E100F;  /* near-black text */
--color-cream:   #FFFBF0;  /* light background / text on dark */
--color-warm:    #AA9F95;  /* warm grey — secondary text */
--color-light:   #F5F0E8;  /* light secondary background */
```

### Typography
- Headlines / display: **Cormorant Garamond** (Google Fonts, serif, italic for accents)
- Body / UI: **Montserrat** (Google Fonts, weight 300 and 400)

### Visual Style
- Primary reference: **buly1803.com** — dark, ornamental, antique luxury
- Secondary reference: **violetgrey.com** — editorial, magazine layout, long text strips
- Dark luxury aesthetic: deep burgundy backgrounds + cream text
- Large serif headlines (Cormorant Garamond), italic for emotional phrases
- Ornamental dividers between sections (thin lines, decorative elements)
- Alternating dark/light sections — not all dark, not all light
- Editorial feel: oversized typography, generous whitespace
- Photography: large, full-width, moody
- NO pink, NO pastel, NO generic beauty-salon aesthetic
- NO gradients, NO neon, NO rounded "bubble" buttons
- Buttons: thin border, rectangular, uppercase Montserrat 300

## Tech Stack
- **HTML + CSS + vanilla JS** — no framework
- **No database** — fully static site
- **Deploy: Vercel** — free, fast, auto-deploy from GitHub
- Fonts: Google Fonts CDN
- Payment: Lava.top embed button or redirect link (no backend needed)

## Commands
- Open `index.html` in browser — local preview
- `git push` → Vercel auto-deploys

## Future Phases (do not build now)
- Telegram bot for purchase notifications and student support
- Course platform integration (Telegram channel or external platform)
- Possible expansion to multi-page with separate course pages

## Tone & Copy Language
- **Language:** Russian (primary)
- **Tone:** warm expert, confident, no fluff, no medical claims
- DO NOT use: "as a doctor", "medically proven", "clinical"
- DO USE: "years of practice", "my clients", "I've seen this work"
- Audience pain points: want to lose weight without exhaustion, understand why procedures work, stop wasting money on wrong treatments

## Author Bio (for About section)
Юлия Горская. Специалист эстетической медицины. 13 лет практики. Владелец 2 клиник в Кишинёве (Honest Beauty — ул. Куза Водэ 23 и ул. Надежды Русо 14). Работает с лицом, телом, аппаратным массажем. Честные рекомендации. Без давления и навязывания.

## Course Description (for modules section)
The course covers:
- Как читать собственные анализы и находить дефициты, которые блокируют похудение
- Строить питание без подсчёта калорий каждый день
- Совмещать процедуры, спорт и уход за кожей так, чтобы они усиливали друг друга
- Как худеть без потери качества кожи
- Современные методы: что работает, что — маркетинг

## Instagram / Social
- Personal: @dr.gorska
- Clinic: @honest_beauty_clinic_md
- Telegram channel: to be created (link TBD)

## What NOT to Build
- Appointment booking (clinic handles this separately)
- Price list for procedures (clinic account handles this)
- Personal cabinet or login system
- Multi-language support (Russian only for now)

---
> Source: [sofiamaliar/yulia-gorska-site](https://github.com/sofiamaliar/yulia-gorska-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
