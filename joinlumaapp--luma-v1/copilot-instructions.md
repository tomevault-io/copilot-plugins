## luma-v1

> LUMA is a compatibility-first social discovery platform that combines meaningful matching with social features. It merges the best of Instagram (social feed), Tinder (card swipe discovery), and Omegle (random live video) into a single dating & friendship app.

# LUMA V1 -- Project Instructions for Claude

## Project Overview
LUMA is a compatibility-first social discovery platform that combines meaningful matching with social features. It merges the best of Instagram (social feed), Tinder (card swipe discovery), and Omegle (random live video) into a single dating & friendship app.

**Core Philosophy:** "Gercek uyum icin kendin ol." -- Compatibility is the foundation. Every feature serves the goal of connecting compatible people.

### Locked V1 Architecture
- **5 Main Tabs**: Akis, Kesfet, Canli, Eslesme, Profil
- **3 Packages**: Ucretsiz, Premium, Supreme (NO Gold, Pro, or Reserved)
- **20+5 Questions**: 20 uyum (mandatory) + 5 kisilik (optional)
- **4 Relationship Types**: Takip, Arkadas, Eslesme, Super Begeni
- **5 Hedefler**: Evlenmek, Iliski bulmak, Sohbet/Arkadas, Kulturleri ogrenmek, Dunyayi gezmek
- **Jeton Economy**: In-app currency for premium actions
- **Uyum Score Range**: 47-97% (90+ = Super Uyum)
- **Photos**: Min 2, Max 9

## Tech Stack
- **Mobile**: React Native + TypeScript (apps/mobile/)
- **Backend**: Node.js + NestJS + TypeScript (apps/backend/)
- **Database**: PostgreSQL + Redis + Elasticsearch
- **Shared Types**: @luma/shared (packages/shared/)
- **Infrastructure**: AWS (Terraform in infrastructure/)
- **CI/CD**: GitHub Actions (.github/workflows/)
- **Real-time**: WebSocket (Socket.io) for messaging, live video, notifications
- **Video**: WebRTC for Canli (live) random video matching
- **Push Notifications**: Firebase Cloud Messaging (FCM)
- **Ads**: Google AdMob (rewarded ads for free users)

## Language Rules
- All code, comments, and technical documentation: **English**
- All user-facing strings and responses to the founder: **Turkish**
- Agent prompts are in English, output always in Turkish

## Code Standards
- TypeScript strict mode everywhere
- No `any` types
- All API endpoints must have input validation
- All business logic must have unit tests
- Follow NestJS module pattern for backend
- Follow React Navigation + Zustand for mobile
- Use Prisma ORM for database operations
- All shared types go in `packages/shared/`
- All API routes defined in `packages/shared/src/constants/api.ts`
- All WebSocket events defined in `packages/shared/src/constants/api.ts`

## Design Direction
- **Reference**: Bumpy dating app (animations, gradients, modern UI)
- Smooth card swipe transitions in Kesfet
- Konfeti/kalp animasyonu on match
- Gradient backgrounds (pink-to-peach light theme, purple-to-dark premium theme)
- Soft, rounded UI elements
- Micro-animations on interactions (like, follow, boost)

---

## APP ARCHITECTURE -- 5 Main Tabs

### Tab 1: Akis (Feed)
Instagram-like social feed with stories and posts.

**Top Section -- Stories:**
- Horizontal scrollable story circles
- First circle: user's own story with "+" to add new
- Only stories from matched/followed users visible
- Stories expire after 24 hours
- Story creation limits by package (Ucretsiz: 1/gun, Premium: 5/gun, Supreme: Sinirsiz)
- Supreme users' stories appear first (Hikaye Onde Gosterim)

**Post Creation:**
- "Gonderini paylas..." input area
- 3 post types: Fotograf, Video, Yazi
- Each post shows: user avatar, name, verification badge, city, time ago, hedef tag

**Feed Tabs:**
- **Populer**: Posts from nearby users sorted by distance + uyum score + engagement. Visible to everyone.
- **Takip**: Posts only from users you follow.

**Interactions on posts:**
- Begen (like with heart animation)
- Yorum (comment)
- Takip et (follow from post)
- Profili ziyaret et (visit profile)

### Tab 2: Kesfet (Discover)
Card-based swipe system for finding compatible people.

**Card Display:**
- Full-screen profile cards with photos
- Shows: name, age, city, uyum yuzdesi (compatibility %), hedef tag, kisilik tipi tag
- Swipe right = Begen, Swipe left = Pas gec
- Swipe up = Super Begeni (costs jeton)

**Top Right Controls:**
- Boost button (profilini one cikar)
- Filter button (yas, mesafe, cinsiyet, gelismis filtreler)

**Matching Logic:**
- Karsilikli begeni (mutual like) = Eslesme -> mesajlasma acilir
- Tek tarafli begeni -> Eslesme bolumunde "Begeniler" tabinda blurlu gorunur
- Begeni gorunurlugu: Ucretsiz (1-2 blurlu), Premium (sinirli net), Supreme (sinirsiz net)

**Filters (package-tiered):**
- Ucretsiz: Yas araligi, cinsiyet, mesafe (temel)
- Premium: + Ilgi alanlari, egitim, yasam tarzi
- Supreme: Tum filtreler acik + gelismis kombinasyonlar

### Tab 3: Canli (Live)
Omegle-style random video matching for instant connections.

**How It Works:**
1. User taps "Baglan"
2. System finds a compatible user (based on uyum score + filters)
3. Live video chat begins (jeton consumed)
4. At end of chat, options appear: Takip Et, Begen, Sonraki
5. Karsilikli takip = Arkadas olur

**Jeton Cost:**
- Each Canli session costs jeton
- Ucretsiz users get limited daily sessions
- Supreme users: unlimited

**Important:** Voice/video calls between matched/friended users happen in the MESSAGING section, NOT in Canli tab. Canli is ONLY for random discovery.

### Tab 4: Eslesme (Matches)
Central hub for all connections and communications.

**5 Scrollable Sub-Tabs:**
1. **Eslesmeler** -- Grid of matched users (mutual likes from Kesfet)
2. **Mesajlar** -- Chat list with matched/friended users (text, voice call, video call, selam gonder)
3. **Begeniler** -- Users who liked you (blurlu for free, limited for premium, full for supreme)
4. **Takipciler** -- Users who follow you (blurlu for free, limited for premium, full for supreme)
5. **Kim Gordu** -- Users who viewed your profile (package-gated)

**Messaging Features:**
- Text messages
- Voice call (between matched/friended users, costs jeton for free users)
- Video call (between matched/friended users, costs jeton for free users)
- Selam Gonder (icebreaker message, costs jeton)
- Okundu Bilgisi (read receipts -- Premium+)
- Buz Kirici Oyunlar (icebreaker mini-games)

### Tab 5: Profil (Profile)
User profile management and settings.

**Profile View:**
- Cover photo, Name, Age, Verification badge, Uyum yuzdesi badge, City
- Kisilik tipi tag (e.g., "Acik Fikirli")
- Stats: Gonderi count, Takipci count, Takip count
- Buttons: "Duzenle" + package upgrade CTA
- Profil Gucu bar (% completion indicator)
- "Profilini One Cikar" (Boost -- 24 saat 10x gorunurluk)

**Hakkimda Section (grid cards):**
Yas, Cinsiyet, Sehir, Is, Egitim, Cocuk, Sigara, Boy, Spor, Burc, Evcil Hayvan, Alkol

**Gamification Section:**
- **Kasif** -- Daily missions (e.g., "5 profili kesfet" -> earn jeton). Timer for next mission.
- **Bu Haftanin Yildizlari** -- Weekly leaderboard: En Cok Begenilen, En Cok Mesaj, En Uyumlu. Resets every Monday.

**Profili Duzenle (Edit Profile):**
- Kisilik Tipi: "Kisilik Testini Coz" (5 fun questions, 1 minute)
- Profil Videosu: 10-30 second video
- Fotograflar: Min 2, max 9 (3x3 grid). First photo = Ana (main profile photo)
- Hedefim: 5 options (see Locked Architecture above)
- Hakkimda Daha Fazlasi: Kilo, Cinsel Yonelim, Burc, Egzersiz, Egitim Seviyesi, Medeni Durum, Cocuklar, Icki, Sigara, Evcil Hayvanlar, Din, Degerler
- Ilgi Alanlari: Select up to 15
- Prompt'larim: Max 3 profile prompts
- Sevdigin Mekanlar: Max 8 favorite places

**Settings:** Account, Notification preferences, Privacy, Package management, Help & support, Logout

---

## RELATIONSHIP SYSTEM

4 types of user connections:

| Type | How It Happens | Result |
|------|---------------|--------|
| **Takip** (Follow) | One-way from Akis, profile, or Canli | See their posts in Takip tab |
| **Arkadas** (Friend) | Mutual follow (both follow each other) | Enhanced visibility, messaging |
| **Eslesme** (Match) | Mutual like in Kesfet | Full messaging + voice/video call |
| **Super Begeni** | Jeton-powered like in Kesfet | Notifies other user, priority visibility |

---

## AUTHENTICATION SYSTEM

**Login Methods:**
- Telefon + OTP (primary method)
- Google Sign-In (planned)
- Apple Sign-In (planned -- required for iOS App Store)

**Onboarding Flow (after auth):**
1. Basic info (name, birthday, gender)
2. Photo upload (min 2 required)
3. Uyum Analizi (20 questions -- MANDATORY)
4. Hedef selection
5. City/location permission
6. Kisilik Testi (5 questions -- OPTIONAL, can skip)
7. Welcome to app

---

## COMPATIBILITY SYSTEM (Test System)

### Uyum Analizi (Compatibility Analysis)
- **20 questions**, mandatory during onboarding
- 4 answer options per question (NOT Likert scale)
- Progress bar shows completion (Soru 1/20, 2/20, etc.)
- "Atla" (Skip) option available but discouraged
- Results calculate **uyum yuzdesi** (compatibility %) between users (range: 47-97%)
- 90+ = **Super Uyum** with special UI treatment (glow, badge, konfeti)
- This is the CORE matching algorithm -- all recommendations depend on it
- Questions organized across 8 psychological categories: lifestyle, values, communication style, future plans, personality traits, relationship approach, social habits, conflict resolution

### Kisilik Testi (Personality Quiz)
- **5 questions**, optional
- Quick and fun format (under 1 minute)
- Results assign a **kisilik tipi** tag shown on profile
- Example tags: "Acik Fikirli", "Lider ve kararli", "Sessiz ve derin", "Eglenceli ve enerjik", "Mantikli ve analitik"
- Does NOT affect matching algorithm, purely cosmetic/social

---

## PACKAGE SYSTEM

**CRITICAL RULE: No feature is fully locked. Every feature is accessible to ALL users -- only QUANTITY and LIMITS differ by package.**

### 3 Packages:

| Feature | Ucretsiz (0 TL) | Premium (499 TL/ay) | Supreme (1.199 TL/ay) |
|---------|-----------------|---------------------|----------------------|
| Swipe (Kesfet) | Limited/day | More/day | Unlimited |
| Story creation | 1/gun | 5/gun | Unlimited |
| Begeni gorunurlugu | 1-2 blurlu | Sinirli net | Unlimited net |
| Kim Gordu | Limited | Expanded | Full |
| Filtreler | Basic | Advanced | All + combinations |
| Boost | Purchase only | 4/ay included | Unlimited |
| Jeton/ay | 0 | 250 | 1000 |
| Okundu bilgisi | No | Yes | Yes |
| Hikaye onde gosterim | No | No | Yes |
| Reklam | Yes (AdMob) | No | No |
| Gunun Eslesmesi | 1/hafta | 1/gun | 3/gun |
| Badge | -- | -- | "En Populer" |

### Jeton (Token) Economy
In-app currency for premium actions across all packages.

**What costs jeton:**
- Selam Gonder (icebreaker message)
- Super Begeni (special like in Kesfet)
- Boost (24h profile visibility 10x)
- Canli sessions (random video chat)
- Voice/video calls (free users only)

**Purchase packages:**
- 79,99 TL (base amount)
- 199,99 TL (500 jeton -- "EN POPULER")
- 349,99 TL (1000 jeton)

**Monthly allocation:**
- Ucretsiz: 0 jeton
- Premium: 250 jeton/ay
- Supreme: 1000 jeton/ay

**Earn jeton for free:**
- Kasif daily missions (e.g., "5 profili kesfet" -> +5 jeton)
- Watching rewarded ads (free users only)

### Boost System
- "24 saat boyunca profilini one cikar ve 10x daha fazla gorunurluk kazan"
- Boost packages (jeton cost): 1 boost (120), 5 boost (500), 10 boost (900), 20 boost (1500)
- Supreme: Unlimited boost
- Premium: 4 boost/ay included

---

## NEW FEATURES (V1)

### 1. Gunun Eslesmesi (Daily Match)
AI-powered daily recommendation based on uyum score + ilgi alanlari + mesafe.
- Ucretsiz: 1/hafta, Premium: 1/gun, Supreme: 3/gun

### 2. Ortak Mekan Onerisi (Mutual Place Suggestion)
When two users match, system checks their "Sevdigin Mekanlar" lists and suggests a meeting place if overlap exists.

### 3. Mood Status (Anlik Ruh Hali)
Quick status on profile: "Sohbete acigim", "Bugun sessizim", "Bulusmaya varim", "Kafede takiliyorum".
Active users with mood status get priority in feeds.

### 4. Buz Kirici Oyunlar (Icebreaker Games)
Mini-games to start conversations after matching:
- "2 Dogru 1 Yanlis"
- Quick question prompts
- "Bu mu O mu?" (This or That)

### 5. Haftalik Uyum Raporu (Weekly Compatibility Report)
Weekly summary: compatibility stats, most active day, profile views, gamification boost.

---

## NOTIFICATION SYSTEM (3-Tier)

**Kritik (Always push, cannot disable):**
- Yeni eslesme, Yeni mesaj, Super begeni aldin, Canli eslesme bulundu

**Onemli (Default ON, user can disable):**
- Yeni takipci, Begeni aldin, Arkadaslik olustu, Uyum sonucu hazir, Gunun eslesmesi hazir

**Dusuk Oncelik (Default OFF, user can enable):**
- Hikaye goruntuleme, Gonderi etkilesimi, Kasif gorevi hatirlatma, Boost suresi bitti, Haftalik rapor hazir

---

## GAMIFICATION

### Kasif (Explorer) Missions
- Daily missions that reward jeton (e.g., "5 profili kesfet" -> +5 jeton)
- Progress bar + timer for next mission, resets daily

### Bu Haftanin Yildizlari (Weekly Stars)
- 3 categories: En Cok Begenilen, En Cok Mesaj, En Uyumlu
- Resets every Monday, visible on profile

### Profil Gucu (Profile Strength)
- Percentage indicator showing profile completion
- Higher profile strength = better visibility in feeds

---

## AD SYSTEM
- Google AdMob integration for free users only
- Rewarded ads: watch ad to earn jeton or unlock temporary access
- Ads NEVER shown to Premium or Supreme users (Reklamsiz Deneyim)
- Ad placements: between feed posts, before Canli sessions, after kesfet card stack ends

---

## ARCHITECTURE RULES
- Follow the 5-tab structure above -- do not invent new tabs or sections
- Package system: NEVER fully lock a feature, only limit quantities
- **3 packages ONLY**: Ucretsiz, Premium, Supreme (NO Gold, Pro, or Reserved -- those are REMOVED)
- **Uyum Analizi**: exactly 20 questions, 4 options each (NOT Likert 1-5)
- **Kisilik Testi**: exactly 5 questions
- **Hedef options**: exactly 5 (Evlenmek, Iliski bulmak, Sohbet/Arkadas, Kulturleri ogrenmek, Dunyayi gezmek)
- **Uyum score range**: 47-97% (90+ = Super Uyum)
- **Photos**: min 2, max 9
- **Ilgi Alanlari**: max 15
- **Prompt'larim**: max 3
- **Sevdigin Mekanlar**: max 8
- **Profile Video**: 10-30 seconds
- **NO Room concept** -- the old Compatibility Room system is completely removed
- **NO 45-question system** -- replaced by 20+5
- **NO intention tags** -- replaced by 5 Hedefler

---

## Available Agents
See `.claude/agents/` for department-specific agents.

## Available Skills
See `.claude/skills/` for quick-action slash commands.

## Reference Documents
- `monetization.md` -- Detailed package comparison table & jeton pricing
- `algorithm_spec.md` -- Compatibility algorithm specification (8 categories, score formulas)
- `data_model.md` -- Database schema and entity models
- `vision.md` -- App vision and mission
- `roadmap.md` -- Development roadmap (8 phases, 16 weeks)
- `progress.md` -- Current progress tracking
- `decisions.md` -- Architectural decisions log
- `debug.md` -- Current bug tracking and deploy status

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joinlumaapp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
