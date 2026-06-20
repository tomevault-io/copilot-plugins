## mobile-fullstack-skill

> >


# Mobile App UI/UX Design Skill

This skill produces **pixel-quality, production-grade mobile app UIs** that look like they came from
a top-tier product studio. Reference apps: **Airbnb** (trust + clarity), **Spotify** (personalization
+ emotional peaks), **Duolingo** (delight + feedback loops), **Revolut** (tactile + premium),
**Phantom** (polish = trust in high-stakes domains).

---

## Core Philosophy

Great mobile UI is about **intentionality** — not flashiness. Before any layout or code, answer:

1. **What is the user trying to accomplish?** Reduce friction to that goal.
2. **How should this make the user feel?** Trust, delight, confidence, calm?
3. **What is the ONE thing they should notice first?** Visual hierarchy must answer this.

---

## Step 0 — Load the design playbook first

Before writing any code, read the reference that matches the app type:

| File | Contains |
|---|---|
| `references/visual-system.md` | Colors per industry, typography scale, spacing tokens, elevation, motion |
| `references/components.md` | Production-ready CSS for every component |
| `references/industry-patterns.md` | Screen layouts for finance, AI, health, e-commerce, social |
| `references/industry-conventions.md` | Emotional design, Peak-End rule, Spotify/Duolingo/Revolut lessons |

Read **all four** before starting.

---

## Step 1 — Understand the context

Extract or infer before designing:

1. **App type / industry** — finance, health, AI, e-commerce, social, productivity, crypto
2. **Screen(s) requested** — onboarding, dashboard, detail, auth, settings, chat
3. **User stage** — new (guided), returning (routine), power user (dense info)
4. **Platform feel** — iOS-leaning, Material/Android, or neutral cross-platform
5. **Mood / brand** — dark/light, vibrant/muted, playful/serious, minimal/rich

If ambiguous, **commit to an opinionated direction and state it**. Good designers make decisions.

---

## Step 2 — Choose the right output format

| Request | Output |
|---|---|
| Single screen / component | HTML artifact — single file, 390px mobile viewport |
| Multi-screen flow | HTML with tab/step navigation between screens |
| React component | JSX artifact with Tailwind or styled-components |
| Android / KMP / React Native | Platform-specific code (see Step 10) |

**Default:** Single HTML artifact at 390px (iPhone 15 viewport), wrapped in a phone shell with
status bar and bottom nav for full visual fidelity.

---

## Step 3 — Apply the visual system (non-negotiable)

### 3.1 Color (60/30/10 Rule)

| Share | Role |
|---|---|
| 60% | Neutral base — backgrounds, surfaces |
| 30% | Complementary — text, dark elements |
| 10% | Accent — CTAs, key indicators, icons |

Every color as a **CSS variable**. Use opacity for text hierarchy: 100% headings, 80% body,
60% secondary. Use accent at 5% opacity for secondary buttons. Match shadow color to background
hue — never pure black shadows on colored backgrounds.

**Industry palettes:**

```css
/* Finance / Banking — trust, stability */
:root {
  --bg: #F5F5F5; --surface: #FFFFFF; --accent: #1A56DB; --accent2: #1041A8;
  --accent-soft: #EBF0FF; --text: #111827; --text2: #6B7280; --text3: #9CA3AF;
  --success: #10B981; --error: #EF4444; --divider: rgba(0,0,0,0.07);
  --shadow: 0 1px 3px rgba(0,0,0,0.06), 0 4px 12px rgba(0,0,0,0.04);
}

/* Health / Fitness — energy, calm */
:root {
  --bg: #F0FAF4; --surface: #FFFFFF; --accent: #16A34A;
  --accent-soft: #DCFCE7; --highlight: #0EA5E9;
}

/* AI / Productivity — dark default */
:root {
  --bg: #0A0A0F; --surface: #16161E; --surface2: #1E1E2A;
  --accent: #7C3AED; --accent-soft: #2D1F5E;
  --text: #F8F8FC; --text2: #9CA3AF; --neon: #A78BFA;
}

/* Social / Lifestyle */
:root { --bg: #FAFAFA; --surface: #FFFFFF; --accent: #EC4899; --accent-soft: #FCE7F3; }

/* E-commerce */
:root {
  --bg: #F9FAFB; --surface: #FFFFFF; --accent: #F97316;
  --accent-soft: #FFF7ED; --badge: #EF4444;
}

/* Crypto / Web3 — futuristic, neon */
:root {
  --bg: #0D0D1A; --surface: #16162A; --accent: #9945FF;
  --accent-soft: #1E1040; --neon: #19FB9B; --text: #F8F8FC;
}
```

**Dark mode mappings:**

| Light token | Dark value |
|---|---|
| #FFFFFF surface | #1C1C1E |
| #F2F2F7 bg | #000000 |
| #111827 text | #F9FAFB |
| #6B7280 secondary | #9CA3AF |
| rgba(0,0,0,0.06) divider | rgba(255,255,255,0.08) |

---

### 3.2 Typography

**Max 4 sizes. Max 2–3 weights.**

| Token | Size | Weight | Use |
|---|---|---|---|
| display | 32–40px | 700 | Hero moments, balance totals, big stats |
| h1 | 24px | 700 | Screen titles |
| title | 20px | 600 | Section headers |
| body | 15px | 400 | Main content |
| label | 13px | 500 | Input labels, button text |
| caption | 12px | 400 | Meta, timestamps, helper text |

- Letter-spacing: **-0.5px** on display/h1, **-0.3px** on title, 0 on body+
- Line-height: **1.3–1.5**
- Use `font-variant-numeric: tabular-nums` for all prices, stats, and large numbers
- System fonts: `-apple-system, BlinkMacSystemFont, 'Segoe UI', Helvetica, sans-serif`

**Anti-pattern:** Never use more than 2 font families. Never bold everything — create hierarchy
with size, weight, AND opacity together.

---

### 3.3 Spacing (8pt grid — always)

| Token | px | Use |
|---|---|---|
| xs | 4 | Icon gaps, tight chips |
| sm | 8 | Between related elements |
| md | 12 | Input padding, compact cards |
| base | 16 | Standard padding |
| lg | 24 | Section gaps |
| xl | 32 | Between major sections |
| 2xl | 48 | Top/bottom breathing room |
| 3xl | 64–96 | Section vertical padding |

**Never use arbitrary values like 11px, 17px, or 23px.**

Relationship rule: if related elements are 16px apart, the gap to the next group is 2× = 32px.
Card internal padding: 24–32px. Larger text needs larger spacing.

---

### 3.4 Elevation & surfaces

| Level | CSS | Use |
|---|---|---|
| 0 | none | Backgrounds |
| 1 | `0 1px 3px rgba(0,0,0,0.06), 0 4px 12px rgba(0,0,0,0.04)` | Cards |
| 2 | `0 4px 16px rgba(0,0,0,0.08), 0 8px 32px rgba(0,0,0,0.06)` | Floating |
| 3 | `0 -4px 32px rgba(0,0,0,0.12)` | Bottom sheets |
| 4 | `0 24px 48px rgba(0,0,0,0.16)` | Modals |

Corner radii: 4px (chips), 8px (tags), 12px (inputs/buttons), 16px (cards), 24px (sheets)

---

### 3.5 Motion tokens

| Duration | Use |
|---|---|
| 100ms | Icon swaps, color changes |
| 150ms | Button press, tap feedback |
| 250ms | Panel slides, tab switches |
| 350ms | Screen transitions |
| 500ms | Onboarding, success states |

Easing: `ease-out` entering, `ease-in` exiting,
`cubic-bezier(0.34,1.56,0.64,1)` for spring popups.

```css
transition: transform 0.15s ease, opacity 0.2s ease, background-color 0.15s ease;
```

---

## Step 4 — Build the layout structure

Start from the phone shell, fill inward:

```html
<div class="phone">
  <div class="status-bar">
    <span>9:41</span>
    <div style="display:flex;gap:6px;align-items:center;font-size:11px">
      <span>●●●●</span><span>WiFi</span><span>🔋</span>
    </div>
  </div>
  <div class="screen">
    <!-- Hero / Header -->
    <!-- Content sections -->
    <!-- height:8px spacer at end -->
  </div>
  <nav class="bottom-nav">...</nav>
</div>
```

**Thumb zone:** All primary actions MUST live in the **bottom 40%** of the screen.

**F-pattern:** Most important content top-left. Reduce interaction cost — expose content directly,
not hidden behind extra taps.

---

## Step 5 — Component standards

### 5.1 Buttons

```css
.btn {
  border-radius: 14px; padding: 15px 24px; font-size: 16px; font-weight: 600;
  border: none; cursor: pointer; width: 100%;
  transition: transform 0.12s ease, filter 0.12s ease;
}
.btn-primary {
  background: var(--accent); color: #fff;
  box-shadow: inset 0 1px 0 rgba(255,255,255,0.15);
}
.btn-primary:active { transform: scale(0.97); filter: brightness(0.88); }
.btn-secondary { background: var(--accent-soft); color: var(--accent); }
.btn-outline { background: transparent; color: var(--accent); border: 1.5px solid var(--accent); }
.btn-ghost { background: transparent; color: var(--text2); }
.btn-loading { opacity: 0.7; pointer-events: none; }

.spinner {
  width: 18px; height: 18px; border: 2px solid rgba(255,255,255,0.3);
  border-top-color: #fff; border-radius: 50%;
  animation: spin 0.6s linear infinite;
}
@keyframes spin { to { transform: rotate(360deg); } }
```

Rules: **one primary per screen**. Disabled: opacity 0.4.

---

### 5.2 Input fields

```html
<div class="input-group">
  <label class="input-label">Email</label>
  <div class="input-wrapper">
    <span class="input-icon">✉️</span>
    <input type="email" class="input" placeholder="you@example.com">
  </div>
  <span class="input-helper">We will never share your email</span>
</div>
```

```css
.input-label { font-size: 13px; font-weight: 500; color: var(--text); margin-bottom: 6px; display: block; }
.input-wrapper { position: relative; }
.input {
  width: 100%; padding: 14px 16px 14px 40px; border-radius: 12px;
  border: 1.5px solid var(--divider); background: var(--surface);
  font-size: 15px; color: var(--text); outline: none; transition: border-color 0.15s;
}
.input:focus { border-color: var(--accent); }
.input-icon { position: absolute; left: 14px; top: 50%; transform: translateY(-50%); font-size: 16px; }
.input-error { border-color: var(--error) !important; }
.input-error-msg { color: var(--error); font-size: 12px; margin-top: 5px; }
```

Human errors: "That email is not registered" — not "Invalid credentials".
Label above input (not placeholder-only). Show/hide toggle for passwords.

---

### 5.3 Cards

```css
.card {
  background: var(--surface); border-radius: 16px; padding: 16px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.06), 0 4px 12px rgba(0,0,0,0.04);
}

/* Hero metric card */
.hero-card {
  background: linear-gradient(135deg, var(--accent), var(--accent2));
  border-radius: 20px; padding: 20px 20px 24px;
  color: #fff; position: relative; overflow: hidden;
}
.hero-card::after {
  content: ""; position: absolute; right: -40px; top: -40px;
  width: 200px; height: 200px; border-radius: 50%;
  background: rgba(255,255,255,0.05);
}

/* Glow variant for AI/crypto */
.card-glow {
  background: var(--surface); border-radius: 16px; padding: 16px;
  box-shadow: 0 0 0 1px rgba(124,58,237,0.15), 0 4px 24px rgba(124,58,237,0.12);
}
```

---

### 5.4 List items

```css
.list-item {
  display: flex; align-items: center; padding: 14px 16px;
  gap: 12px; border-bottom: 1px solid var(--divider);
  background: var(--surface); transition: background 0.1s;
}
.list-item:active { background: var(--bg); }
.list-icon {
  width: 40px; height: 40px; border-radius: 10px;
  background: var(--accent-soft); display: flex;
  align-items: center; justify-content: center;
  font-size: 18px; flex-shrink: 0;
}
.list-title { font-size: 15px; font-weight: 500; color: var(--text); }
.list-subtitle { font-size: 12px; color: var(--text2); margin-top: 2px; }
.chevron { color: var(--text3); font-size: 18px; }
```

---

### 5.5 Bottom navigation

```css
.bottom-nav {
  position: absolute; bottom: 0; left: 0; right: 0; height: 84px;
  background: var(--surface); border-top: 1px solid var(--divider);
  display: flex; padding-bottom: 20px;
}
.nav-item {
  flex: 1; display: flex; flex-direction: column; align-items: center;
  justify-content: center; gap: 2px; font-size: 10px; font-weight: 500;
  color: var(--text3); cursor: pointer; transition: color 0.15s;
}
.nav-item.active { color: var(--accent); }
.nav-icon { font-size: 22px; transition: transform 0.15s; }
.nav-item.active .nav-icon { transform: scale(1.1); }
```

3–5 tabs max. Never icons without labels.

---

### 5.6 Feedback components

```css
/* Toast */
.toast {
  position: fixed; bottom: 96px; left: 16px; right: 16px;
  padding: 14px 16px; border-radius: 12px; font-size: 14px; font-weight: 500;
  animation: slideUp 0.25s ease-out;
}
.toast-success { background: #111827; color: #fff; }
.toast-error { background: var(--error); color: #fff; }
@keyframes slideUp {
  from { transform: translateY(20px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}

/* Skeleton loader */
.skeleton {
  background: linear-gradient(90deg, var(--divider) 25%, rgba(0,0,0,0.02) 50%, var(--divider) 75%);
  background-size: 200% 100%; animation: shimmer 1.5s infinite; border-radius: 6px;
}
@keyframes shimmer {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}

/* Empty state */
.empty-state {
  display: flex; flex-direction: column; align-items: center;
  padding: 48px 32px; text-align: center; gap: 8px;
}
.empty-icon { font-size: 48px; margin-bottom: 8px; }
```

Use **skeletons** over spinners. Empty states must include: icon + friendly message + CTA.

---

## Step 6 — Screen-specific patterns

### 6.1 Onboarding (Airbnb / Duolingo pattern)

1. **Splash** — full-bleed gradient, single bold headline, 2-line sub-headline, CTA + ghost "Sign in"
2. **Feature highlights** (2–3 screens) — illustration top 50%, benefit title, one sentence, progress dots
3. **Permission** — centered icon, friendly ask, "Allow" primary + "Not now" ghost
4. **Sign-up** — minimal fields, social login at top, or single personalization question

```css
.onboarding-screen {
  min-height: 100%; display: flex; flex-direction: column;
  background: linear-gradient(160deg, var(--accent) 0%, #4338CA 100%);
}
.onboarding-content {
  background: var(--surface); border-radius: 32px 32px 0 0; padding: 32px 24px 40px;
}
.dot { width: 6px; height: 6px; border-radius: 50%; background: var(--divider); }
.dot.active { width: 20px; background: var(--accent); border-radius: 3px; transition: width 0.25s ease; }
```

Always allow skipping. Show real UI previews. Never trap users.

---

### 6.2 Finance / Banking dashboard

```
[ Hero card: gradient, balance in 36–40px display font ]
  [ Change pill: arrow + percent + amount today ]
  [ Quick actions: Send | Request | Pay | Top Up ]

[ Horizontal card scroll: bank cards with gradient + masked number ]

[ "Recent Activity" ]
  [ Merchant icon 40px | Name + Category | Amount + Date ]
  income = success green | expense = text-primary (NOT red)
  [ "See all" link ]

[ Spending insights: horizontal progress bars vs budget ]

[ Bottom nav: Home | Analytics | Transfer | Accounts | Profile ]
```

Security signals: lock icon, "Secured by..." badge. **Never red for regular expenses.**

---

### 6.3 Auth / Login

```
[ Logo centered — top third ]
[ Apple | Google social login (border buttons) ]
[ Divider: "or continue with email" ]
[ Email input ]
[ Password input with show/hide toggle ]
[ "Forgot password?" right-aligned ]
[ "Sign In" primary CTA full-width ]
[ "No account? Sign up" at bottom ]
```

---

### 6.4 Chat / AI assistant

```
[ App bar: back | Model name | settings ]
[ Message thread (scrollable, padding-bottom 100px) ]
  AI: left bubble, surface bg, rounded 18/18/18/4px, avatar 28px, timestamp
  User: right bubble, accent bg, rounded 18/4/18/18px

[ Input bar — pinned bottom ]
  [ + attach ] [ text field "Ask anything..." ] [ send ]
  [ Quick chips: Summarize | Explain | Translate | Rewrite ]
```

AI apps default to **dark mode** (`--bg: #0A0A0F`). Typing indicator: 3 animated dots.
Code blocks: monospace, dark bg, copy button.

---

### 6.5 Health / Fitness dashboard

```
[ App bar: icon | "Thursday, May 9" | analytics ]
[ Goal ring card: SVG progress circle + "73% to your goal" ]
[ Stats 2x2 grid: Steps | Calories | Active min | Heart rate ]
[ "Today Activity": workout cards ]
[ Weekly trend bar chart ]
[ Bottom nav: Home | Workouts | Nutrition | Sleep | Profile ]
```

Streaks: flame + amber (#F59E0B). Empathetic copy — never guilt users.

---

### 6.6 Settings screen (universal)

```
[ App bar: back + "Settings" ]
[ Profile: avatar 60px | Name | Email | Edit ]
[ Section "Preferences": Notifications toggle | Dark mode toggle | Language ]
[ Section "Account": Privacy | Billing | Delete account (error color) ]
[ Footer: Version 1.4.2 ]
```

Section labels: small-caps gray. Destructive action always at bottom in error color.

---

### 6.7 Detail screen

```
[ App bar: back arrow + title + optional overflow ]
[ Hero image or hero card ]
[ Structured info in labeled sections ]
[ Spacer ]
[ Sticky bottom: primary action button full-width ]
```

---

## Step 7 — Emotional design (Peak-End Rule)

The brain compresses an experience into two moments: the **peak** (most intense) and the
**end** (last impression). Disney parks: 70% of visitors return because those moments were
intentional — not because every moment was flawless.

**Design ONE peak moment per flow:**
- Duolingo: character animation + cheer on streak hit
- Spotify Wrapped: shareable personal identity insight
- Revolut: 3D card flip on card creation

```css
@keyframes celebrate {
  0% { transform: scale(0.8); opacity: 0; }
  60% { transform: scale(1.1); }
  100% { transform: scale(1); opacity: 1; }
}
.success-icon { animation: celebrate 0.5s cubic-bezier(0.34,1.56,0.64,1) forwards; }
```

**Design the ending:** summary card + progress affirmation + gentle next-step nudge.

**Reduce negative peaks:** Loading = tips or animation (not dead time). Errors = helpful copy +
recovery action. Long forms = progress indicator.

**Three strategic principles from Spotify:**
1. **Trojan Horse** — wrap complex tech in familiar UI patterns users already know
2. **Vanity Mirror** — celebrate who users are, not what they did in your app
3. **Comfort Trap** — consistency as a competitive moat; predictable patterns create habits

---

## Step 8 — Smart patterns

### By user stage

| Stage | Approach |
|---|---|
| New | Simple welcome, guided setup, minimal options, large tap targets |
| Returning | Personalized content, routine-focused, progress indicators |
| Power | Dense info, advanced stats, optimization tools, shortcuts |

### Search screens
Never blank. Always show: recent searches + popular/trending + personalized recommendations.

### Status / tracking
Open with confident status. Humanize with photos + names. Visual timelines, not text date lists.

### Selection over manual input
Tappable options for common choices. Include emoji/icons alongside. Always provide "Other" fallback.

---

## Step 9 — Polish checklist

**Visual:**
- [ ] All spacing on 8pt grid (no arbitrary values)?
- [ ] Max 4 text sizes, max 3 font weights?
- [ ] All cards consistent radius and shadow?
- [ ] Single primary CTA per screen?
- [ ] Realistic domain content (no lorem ipsum)?
- [ ] Status bar and 84px bottom safe area respected?
- [ ] Income = success color, expense = text-primary, overdraft = error?

**UX:**
- [ ] Primary action in bottom 40% (thumb zone)?
- [ ] All tap targets >= 44x44pt?
- [ ] Empty states have illustration + message + CTA?
- [ ] Loading uses skeletons not spinners?
- [ ] Error messages human and specific?

**Emotional:**
- [ ] ONE intentional peak moment designed?
- [ ] Ending/confirmation screen feels complete?
- [ ] Success states feel rewarding (animation + copy)?
- [ ] Negative state copy is empathetic, not robotic?

**Anti-patterns to avoid:**
- Overusing gradients/blur unless you can truly execute it
- Pure gray/black shadows on colored backgrounds
- More than 3 font weights
- Random spacing values (11px, 17px, 23px)
- CTAs above the thumb zone
- Making labels bigger than values
- Sliders for frequent or precise data entry
- Generic empty states with no guidance

---

## Step 10 — Implementation by platform

### 10.1 Android Studio — Jetpack Compose (Kotlin)

```kotlin
object AppTheme {
  object Colors {
    val Accent = Color(0xFF1A56DB)
    val AccentSoft = Color(0xFFEBF0FF)
    val Surface = Color(0xFFFFFFFF)
    val Bg = Color(0xFFF2F2F7)
    val TextPrimary = Color(0xFF111827)
    val TextSecondary = Color(0xFF6B7280)
    val Success = Color(0xFF10B981)
    val Error = Color(0xFFEF4444)
  }
  object Spacing {
    val xs = 4.dp; val sm = 8.dp; val md = 12.dp
    val base = 16.dp; val lg = 24.dp; val xl = 32.dp
  }
  object Radius {
    val input = 12.dp; val card = 16.dp; val btn = 14.dp; val sheet = 24.dp
  }
}

@Composable
fun PrimaryButton(text: String, onClick: () -> Unit, isLoading: Boolean = false) {
  Button(
    onClick = onClick,
    modifier = Modifier.fillMaxWidth().height(52.dp),
    shape = RoundedCornerShape(AppTheme.Radius.btn),
    colors = ButtonDefaults.buttonColors(containerColor = AppTheme.Colors.Accent),
    enabled = !isLoading
  ) {
    if (isLoading)
      CircularProgressIndicator(color = Color.White, modifier = Modifier.size(20.dp))
    else
      Text(text, fontSize = 16.sp, fontWeight = FontWeight.SemiBold)
  }
}

@Composable
fun AppCard(content: @Composable () -> Unit) {
  Card(
    modifier = Modifier.fillMaxWidth(),
    shape = RoundedCornerShape(AppTheme.Radius.card),
    colors = CardDefaults.cardColors(containerColor = AppTheme.Colors.Surface),
    elevation = CardDefaults.cardElevation(defaultElevation = 2.dp)
  ) {
    Box(modifier = Modifier.padding(AppTheme.Spacing.base)) { content() }
  }
}
```

Key practices: Use `MaterialTheme` with custom `colorScheme`, `typography`, `shapes`.
Use `@Preview` to iterate fast. State hoisting to ViewModels. `LazyColumn`/`LazyRow` for lists.

---

### 10.2 Kotlin Multiplatform Mobile (KMP)

```kotlin
// shared/src/commonMain/kotlin/theme/Tokens.kt
object DesignTokens {
  object Color {
    const val ACCENT = 0xFF1A56DB
    const val SUCCESS = 0xFF10B981
    const val ERROR = 0xFFEF4444
  }
  object Spacing { const val BASE = 16; const val LG = 24; const val XL = 32 }
  object Radius { const val CARD = 16; const val BTN = 14; const val INPUT = 12 }
}
```

- **Share:** models, API clients, validation, formatters, business rules
- **Platform-specific:** Compose on Android, SwiftUI on iOS
- Keep token names identical on both platforms

---

### 10.3 React + Tailwind CSS

```javascript
// tailwind.config.js
module.exports = {
  theme: { extend: {
    colors: {
      accent: '#1A56DB', 'accent-soft': '#EBF0FF',
      surface: '#FFFFFF', bg: '#F2F2F7',
      success: '#10B981', error: '#EF4444',
      text: { primary: '#111827', secondary: '#6B7280', tertiary: '#9CA3AF' }
    },
    borderRadius: { card: '16px', btn: '14px', input: '12px' },
    boxShadow: { card: '0 1px 3px rgba(0,0,0,0.06), 0 4px 12px rgba(0,0,0,0.04)' }
  }}
}
```

```jsx
export function PrimaryButton({ children, onClick, isLoading }) {
  return (
    <button
      onClick={onClick} disabled={isLoading}
      className="w-full py-4 px-6 bg-accent text-white text-base font-semibold
                 rounded-btn active:scale-[0.97] active:brightness-90
                 transition-all duration-150 disabled:opacity-40">
      {isLoading ? <span className="animate-spin">⟳</span> : children}
    </button>
  )
}

export function Card({ children, className = '' }) {
  return (
    <div className={`bg-surface rounded-card shadow-card p-4 ${className}`}>
      {children}
    </div>
  )
}
```

Use `dark:` variants for dark mode. Lucide React for icons, Recharts for data viz.

---

### 10.4 React Native + Expo

```typescript
// theme.ts
export const theme = {
  colors: {
    accent: '#1A56DB', accentSoft: '#EBF0FF', surface: '#FFFFFF', bg: '#F2F2F7',
    text: '#111827', text2: '#6B7280', text3: '#9CA3AF',
    success: '#10B981', error: '#EF4444', divider: 'rgba(0,0,0,0.07)'
  },
  spacing: { xs: 4, sm: 8, md: 12, base: 16, lg: 24, xl: 32 },
  radius: { input: 12, card: 16, sheet: 24, btn: 14 },
  shadow: {
    card: {
      shadowColor: '#000', shadowOffset: { width: 0, height: 2 },
      shadowOpacity: 0.06, shadowRadius: 8, elevation: 3
    }
  }
}
```

```jsx
import { Pressable, Text, StyleSheet } from 'react-native'
import { theme } from './theme'

export function PrimaryButton({ title, onPress }) {
  return (
    <Pressable
      onPress={onPress}
      style={({ pressed }) => [styles.btn, pressed && styles.pressed]}>
      <Text style={styles.label}>{title}</Text>
    </Pressable>
  )
}

const styles = StyleSheet.create({
  btn: {
    backgroundColor: theme.colors.accent, borderRadius: theme.radius.btn,
    paddingVertical: 15, alignItems: 'center', width: '100%'
  },
  pressed: { opacity: 0.85, transform: [{ scale: 0.97 }] },
  label: { color: '#fff', fontSize: 16, fontWeight: '600' }
})
```

Tools: `@react-navigation/native` for routing, `@expo/vector-icons` for icons,
`expo-font` for custom fonts, `react-native-reanimated` for gesture/spring animations.

---

### 10.5 React Native + Expo + NativeWind (Tailwind-style)

```jsx
import { Pressable, Text, View } from 'react-native'

export function PrimaryButton({ title, onPress }) {
  return (
    <Pressable
      onPress={onPress}
      className="w-full py-4 bg-accent rounded-btn items-center active:opacity-80">
      <Text className="text-white text-base font-semibold">{title}</Text>
    </Pressable>
  )
}

export function Card({ children, className }) {
  return (
    <View className={`bg-surface rounded-card p-4 shadow-card ${className}`}>
      {children}
    </View>
  )
}
```

Extend `tailwind.config.js` with the same token set as the web version for cross-platform consistency.

---

## Step 11 — HTML artifact boilerplate

Use as starting boilerplate. Swap palette per industry (see Step 3.1).

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>App Name</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    :root {
      --bg: #F2F2F7; --surface: #FFFFFF; --surface2: #F8F8FC;
      --accent: #1A56DB; --accent2: #1041A8; --accent-soft: #EBF0FF;
      --text: #0D0D14; --text2: #6B7280; --text3: #9CA3AF;
      --divider: rgba(0,0,0,0.07); --success: #10B981; --error: #EF4444; --warning: #F59E0B;
      --shadow: 0 1px 3px rgba(0,0,0,0.06), 0 4px 12px rgba(0,0,0,0.04);
      --radius-card: 16px; --radius-btn: 14px; --radius-input: 12px;
    }
    body {
      background: #D1D5DB; display: flex; justify-content: center;
      padding: 24px 0; min-height: 100vh;
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
    }
    .phone {
      width: 390px; min-height: 844px; background: var(--bg); border-radius: 48px;
      overflow: hidden; position: relative;
      box-shadow: 0 32px 80px rgba(0,0,0,0.22), 0 0 0 1px rgba(0,0,0,0.06);
    }
    .status-bar {
      height: 50px; background: var(--surface); display: flex;
      align-items: center; justify-content: space-between;
      padding: 14px 24px 0; font-size: 12px; font-weight: 600; color: var(--text);
    }
    .screen {
      height: calc(844px - 50px); overflow-y: auto;
      padding-bottom: 84px; scrollbar-width: none;
    }
    .screen::-webkit-scrollbar { display: none; }
    .bottom-nav {
      position: absolute; bottom: 0; left: 0; right: 0; height: 84px;
      background: var(--surface); border-top: 1px solid var(--divider);
      display: flex; padding-bottom: 20px;
    }
    .nav-item {
      flex: 1; display: flex; flex-direction: column; align-items: center;
      justify-content: center; gap: 2px; font-size: 10px; font-weight: 500;
      color: var(--text3); cursor: pointer; transition: color 0.15s;
    }
    .nav-item.active { color: var(--accent); }
    .nav-icon { font-size: 22px; transition: transform 0.15s; }
    .nav-item.active .nav-icon { transform: scale(1.1); }
    .card {
      background: var(--surface); border-radius: var(--radius-card);
      padding: 16px; box-shadow: var(--shadow);
    }
    .btn {
      display: block; width: 100%; padding: 15px 24px; font-size: 16px;
      font-weight: 600; border: none; border-radius: var(--radius-btn);
      cursor: pointer; text-align: center;
      transition: transform 0.12s ease, filter 0.12s ease;
    }
    .btn-primary {
      background: var(--accent); color: #fff;
      box-shadow: inset 0 1px 0 rgba(255,255,255,0.15);
    }
    .btn-primary:active { transform: scale(0.97); filter: brightness(0.88); }
    .section-header {
      display: flex; justify-content: space-between; align-items: center;
      padding: 20px 16px 10px;
    }
    .section-title { font-size: 17px; font-weight: 600; color: var(--text); }
    .section-link { font-size: 13px; color: var(--accent); font-weight: 500; }
    @keyframes slideUp {
      from { transform: translateY(20px); opacity: 0; }
      to { transform: translateY(0); opacity: 1; }
    }
    @keyframes celebrate {
      0% { transform: scale(0.8); opacity: 0; }
      60% { transform: scale(1.1); }
      100% { transform: scale(1); opacity: 1; }
    }
    .skeleton {
      background: linear-gradient(90deg, var(--divider) 25%, rgba(0,0,0,0.02) 50%, var(--divider) 75%);
      background-size: 200% 100%; animation: shimmer 1.5s infinite; border-radius: 6px;
    }
    @keyframes shimmer { 0% { background-position: 200% 0; } 100% { background-position: -200% 0; } }
  </style>
</head>
<body>
  <div class="phone">
    <div class="status-bar">
      <span>9:41</span>
      <div style="display:flex;gap:6px;align-items:center;font-size:11px">
        <span>●●●●</span><span>WiFi</span><span>🔋</span>
      </div>
    </div>
    <div class="screen">
      <!-- SCREEN CONTENT HERE -->
    </div>
    <nav class="bottom-nav">
      <div class="nav-item active"><span class="nav-icon">🏠</span><span>Home</span></div>
      <div class="nav-item"><span class="nav-icon">🔍</span><span>Explore</span></div>
      <div class="nav-item"><span class="nav-icon">＋</span><span>New</span></div>
      <div class="nav-item"><span class="nav-icon">🔔</span><span>Alerts</span></div>
      <div class="nav-item"><span class="nav-icon">👤</span><span>Profile</span></div>
    </nav>
  </div>
</body>
</html>
```

---

## Quality bar

Ask: *Could this be mistaken for a screenshot from a real App Store app?*

If no — iterate on spacing, color consistency, typography, content realism, and emotional peak.

Open in a browser and squint. Does visual hierarchy hold? Does the dominant element draw the eye
correctly? Is there breathing room? Does it feel smooth and intentional?

> Benchmark: **Airbnb** (trust + clarity) · **Spotify** (personalization + peak moments)
> · **Duolingo** (delight + feedback loops) · **Revolut** (tactile premium) · **Phantom** (polish
> = trust in high-stakes) · **Headspace** (calm + minimal)

---
> Source: [sisovin/mobile-fullstack-skill](https://github.com/sisovin/mobile-fullstack-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
