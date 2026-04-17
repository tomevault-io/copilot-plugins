## mathkids-pwa

> MathKids is a kids math learning PWA (Progressive Web App) built as a **single HTML file** with inline CSS/JS. Target audience: children ages 3-10.

# MathKids - Claude Instructions

## Project Overview
MathKids is a kids math learning PWA (Progressive Web App) built as a **single HTML file** with inline CSS/JS. Target audience: children ages 3-10.

## Architecture
- **Single file**: `mathkids-v3.html` is the source of truth (~2900 lines)
- **PWA output**: `www/index.html` is generated from source via `python3 generate_pwa.py`
- **Capacitor**: Android wrapper in `android/` folder
- **Hosting**: GitHub Pages at `https://segadg.github.io/mathkids-pwa/`

## Tech Stack
- Vanilla HTML/CSS/JS only - NO frameworks
- Lottie-web for character animations (CDN)
- Google Fonts (Baloo 2, Nunito)
- Web Speech API for audio feedback
- localStorage for all data persistence
- Capacitor 6 for Android build

## Key Patterns

### File Structure
```
mathkids-v3.html          # Source file (edit this)
generate_pwa.py           # Transforms source -> www/index.html
www/                      # PWA output (don't edit directly)
  index.html              # Generated from source
  manifest.json           # PWA manifest
  sw.js                   # Service worker
  icons/                  # App icons
  assets/                 # 2D game icons (PNG, transparent bg)
android/                  # Capacitor Android project
```

### How to Deploy
```bash
python3 generate_pwa.py                    # Regenerate PWA
npx cap sync                               # Sync to Android
git add -A && git commit -m "msg" && git push
git subtree split --prefix www -b tmp && git push origin tmp:gh-pages --force && git branch -D tmp
```

### Source File Structure (mathkids-v3.html)
1. **Lines 1-25**: Lottie JSON data (9 characters, very large)
2. **Lines 26-400**: CSS styles
3. **Lines 400-800**: HTML screens (onboarding, home, levelmap, game, result, leaderboard, stats, parent)
4. **Lines 800-1000**: Language strings (id, en, ms, fr, ja)
5. **Lines 1000-1350**: Character config, generators, T translations, coin system, subscription
6. **Lines 1350-2000**: LEVELS data (9 categories, 250 levels total)
7. **Lines 2000-2900**: Game logic, UI functions, speech, modals, init

### Important Conventions
- **Language**: Global `lang` variable ('id','en','ms','fr','ja'). All UI text uses `S(key)` helper
- **Currency**: Follows country via `getCur()`, NOT language
- **Audio**: Follows language via `LANG_VOICE_MAP[lang]`
- **Null safety**: Always null-check DOM elements in `goHome()` since it's called before screens render
- **Question generators**: Return `{text, answer}` or `{text, answer, options:[4 unique strings]}`
- **Translations in generators**: Use `t('key')` helper with `T` object for multilingual question text

### localStorage Keys
- `mk_profile` - User profile (name, age, char, lang, country)
- `mk_progress` - Level completion & stars
- `mk_stars` - Total stars count
- `mk_stats` - Gameplay statistics (per-day, per-category)
- `mk_coins` - Coin balance
- `mk_owned_chars` - Purchased characters array
- `mk_streak`, `mk_last_date` - Login streak
- `mk_perf_[catId]` - Adaptive difficulty (last 5 answers)
- `mk_trial_end` - Free trial end date
- `mk_subscribed` - Subscription status
- `mk_parent_pin` - Parent zone PIN (default: 1234)
- `mk_sound` - Sound on/off

### Monetization Model
- **Free forever**: count, add, sub categories
- **Premium** (subscription): mul, div, frac, shape, time, money
- **3-day free trial** on first install
- **Coins**: Earned via daily login (1/day), streaks (bonus), daily target (3 coins)
- **Character shop**: Characters cost 0/5/10 coins, permanently owned

### Common Pitfalls
- Editing `www/index.html` directly will be overwritten by `generate_pwa.py`
- `generate_pwa.py` removes phone simulator wrapper, notch, adds PWA meta tags
- DOM elements may not exist when `goHome()` is called from `finishOnboard()` - always null-check
- Lottie data is huge (~900KB) - don't try to read/parse entire file at once
- `manifest.json` uses relative paths (`"start_url": "."`) for GitHub Pages compatibility
- All question answer options must be unique (use `_uniqueOpts` helper for text answers)

### Categories & Level Counts
| Category | Levels | Sub-levels | Premium |
|----------|--------|------------|---------|
| Counting | 10 | 30 | Free |
| Addition | 30 | 90 | Free |
| Subtraction | 30 | 90 | Free |
| Multiplication | 30 | 90 | Premium |
| Division | 30 | 90 | Premium |
| Fractions | 30 | 90 | Premium |
| Shapes | 30 | 90 | Premium |
| Time | 30 | 90 | Premium |
| Money | 30 | 90 | Premium |

### How to Operate
1. Read error messages fully before attempting fixes
2. Always edit `mathkids-v3.html`, never `www/index.html` directly
3. Read only the relevant section of the file, not the entire 3000+ lines
4. Run `python3 generate_pwa.py` after changes to regenerate PWA output

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/segaDG) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
