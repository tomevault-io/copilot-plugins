## mathcommander

> The battle screen control panel uses CSS `transform: scale()` to maintain alignment between the PNG overlay and CRT elements across different screen sizes. The panel is designed at 896px (max-w-4xl) and scales down proportionally on narrower screens. This eliminates the need for per-breakpoint pixel adjustments.

# Space Math Commander - Project Guide

## Battle Screen Panel Scaling

The battle screen control panel uses CSS `transform: scale()` to maintain alignment between the PNG overlay and CRT elements across different screen sizes. The panel is designed at 896px (max-w-4xl) and scales down proportionally on narrower screens. This eliminates the need for per-breakpoint pixel adjustments.

## Quick Links

| Topic | Documentation |
|-------|---------------|
| Audio System | `docs/AUDIO-SYSTEM.md` - volumes, playback locations, architecture |

## Important Guidelines

### Version Numbers
When making changes that will be deployed, increment the version number in TWO places:
1. `package.json` - the `version` field
2. `src/pages/StartScreen.tsx` - the displayed version string (search for "INDUSTRIAL BETA")

Use semantic versioning:
- Patch (0.5.x) for bug fixes
- Minor (0.x.0) for new features
- Include the version in git commit messages, e.g., `fix: Description (v0.5.8)`

### Reverting Failed Fixes
If a fix does not work and you are asked to revert it, you must remove ALL code traces of that fix unless they added additional functionality. Ask if unsure.

## Design System Architecture

This project uses a centralized design system with tokens, theme plumbing, and Tailwind integration.

### Quick Reference

| What | Where |
|------|-------|
| Colors | `src/styles/tokens/colors.ts` |
| Typography | `src/styles/tokens/typography.ts` |
| Effects (shadows/glows) | `src/styles/tokens/effects.ts` |
| Asset paths | `src/styles/tokens/assets.ts` |
| Theme runtime | `src/styles/theme/index.ts` |
| Tailwind plugin | `src/styles/plugins/designSystem.ts` |
| Global CSS | `src/index.css` |
| Tailwind config | `tailwind.config.ts` |

### Asset Locations

| Asset Type | Path | Notes |
|------------|------|-------|
| Audio | `public/assets/audio/` | Music, SFX, speech |
| Backgrounds | `public/assets/images/backgrounds/` | Space backgrounds, star layers |
| Planets | `public/assets/images/planets/` | Animated (60-frame) and static |
| Characters | `public/assets/images/characters/` | Commander, aliens |
| Ships | `public/assets/images/ships/` | Boss ship |
| UI Elements | `public/assets/images/ui/` | Badges, panels |
| Video | `public/assets/video/` | Intro cinematic |
| Battle Sprites | `public/assets/helianthus/` | ShooterFull, Landscapes (legacy) |

### Adding New Assets

1. Place file in appropriate `public/assets/images/` subfolder
2. Add path to `src/styles/tokens/assets.ts` (for type-safe imports)
3. Import from tokens in components:
   ```typescript
   import { assets } from '@/styles/tokens/assets';
   // Use: assets.backgrounds.spaceDark
   ```

### Adding New Animations

Animations are now consolidated. The theme exports keyframes that Tailwind consumes.

**For CSS animations:**
Add to `src/styles/theme/index.ts` in the `keyframes` and `animation` sections.

**Existing animations:**
- `animate-slideDown` / `animate-slideInFromRight` - Ship entrances
- `animate-hover` / `animate-hoverSmall` - Floating effect
- `animate-shake` - Shake effect
- `animate-slideOut` / `animate-flyOutRight` - Exit animations
- `animate-parallaxSlow/Medium/Fast` - Star parallax
- `animate-fadeIn` - Fade in
- `animate-starPop` - Star earned effect
- `animate-xpBounce` - XP display bounce

### Color System

Colors are defined in `src/styles/tokens/colors.ts` and exposed via Tailwind:

```
bg-space-black    (#050510) - Main app background
bg-space-dark     (#0B0B1E) - Darker areas
bg-space-light    (#1A1A35) - Lighter space areas

bg-industrial-dark      (#181825) - Panel backgrounds
bg-industrial-metal     (#4A4A5A) - Metallic accents
bg-industrial-blue      (#2A2A40) - Blue tinted panels
bg-industrial-highlight (#6E6E80) - Highlights

text-brand-primary   (#2563EB) - Primary blue
text-brand-secondary (#00F0FF) - Cyan accent
text-brand-accent    (#FFE500) - Yellow/gold
text-brand-hazard    (#E5C000) - Warning yellow
text-brand-danger    (#FF2A2A) - Red/error
text-brand-success   (#00FF9D) - Green/success
```

### Typography

Single font: `font-pixel` ("Press Start 2P")

Semantic classes (defined in Tailwind plugin):
- `.text-display` - Large scores (60px)
- `.text-title` - Screen titles (36px desktop, 24px mobile)
- `.text-heading` - Section heads (24px desktop, 20px mobile)
- `.text-subheading` - Minor heads (18px)
- `.text-body` / `.text-body-lg` - Body text
- `.text-label` - Form labels (12px uppercase)
- `.text-caption` - Helper text (10px)
- `.text-micro` - Tiny labels (11px)
- `.text-mono` - Numbers/equations (20px)
- `.text-dialogue` - Character speech
- `.text-speaker` - "[COMMANDER]" labels

### Utility Classes

Custom utilities from the design system plugin:
- `.pixel-border` - Hard-edged pixel borders
- `.panel-industrial` - Industrial panel style
- `.bg-hazard` - Hazard stripe pattern
- `.btn-retro` - Retro button base

### Reusable UI Components

Located in `src/components/ui/`:

- `Header` - Consistent header with back button, title, right content
- `PixelButton` - Main button component
- `PixelCard` - Card/panel component
- `CRTDialogueBox` - Green CRT terminal style dialogue
- `DialogueBox` - General dialogue component

### Footer Pattern

All screens use consistent footer styling:
```tsx
<div className="bg-gray-900/80 border-4 border-gray-700 p-4 max-w-xl w-full">
  {/* Content */}
</div>
```

### iOS Safe Area

CSS variables available:
```css
var(--safe-area-top)
var(--safe-area-bottom)
var(--safe-area-left)
var(--safe-area-right)
```

Use in components:
```tsx
style={{ paddingBottom: 'var(--safe-area-bottom)' }}
// or
style={{ paddingBottom: 'env(safe-area-inset-bottom)' }}
```

### Audio System

Located in `src/audio/`:
- `audioEngine.ts` - Main audio engine (Web Audio API + iOS HTML5 fallback)
- `sounds.ts` - Sound definitions (music, SFX, ambience)
- `speechSounds.ts` - Voice line definitions (166 total)

iOS uses HTML5 Audio fallback (see global CLAUDE.md for details).

### Project Structure

```
src/
├── audio/              # Audio engine and sound definitions
├── components/
│   ├── game/           # Game-specific components (AnimatedPlanet, SpaceBackground)
│   └── ui/             # Reusable UI components
├── data/               # Game data (campaign, narrative)
├── hooks/              # React hooks (battle, audio)
├── pages/              # Screen components
├── styles/
│   ├── tokens/         # Design tokens (colors, typography, effects, assets)
│   ├── theme/          # Theme runtime (CSS variables, Tailwind extend)
│   └── plugins/        # Tailwind plugins
├── types/              # TypeScript types
└── utils/              # Utility functions
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Oggie110)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Oggie110)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
