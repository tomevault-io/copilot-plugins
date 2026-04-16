## out-of-the-box

> Sip Happens game concept, coding standards, and marketing guidelines


# Sip Happens - Social Drinking Game

## Game Concept

**Sip Happens** is a social drinking card game with multiple game modes, each themed around a specific drink. Simple gameplay, maximum fun.

### Core Mechanics

- **Card-based gameplay**: Draw a card, follow the prompt, sip if applicable
- **Category = Drink theme**: Each game mode is named after a beverage
- **Difficulty levels**: Cards have varying intensity/complexity
- **Special rule**: Drink less if you're drinking the featured beverage for that category
- **Customization**: Users can create custom cards and mix categories

### Game Categories (Drink Themes)

| Category          | Drink Theme             |
| ----------------- | ----------------------- |
| Dating            | Love on the Rocks       |
| Never Have I Ever | Champagne & Plot Twists |
| Would You Rather  | Booze Roulette          |
| Truth or Drink    | Truth & Tonic           |
| Pictionary Adult  | Tipsy Doodles           |
| Most Likely       | Shots Fired             |
| Deep Talk         | Rum & Reflection        |
| Hot Topics        | Tequila & Taboos        |
| Buzzed            | BrandyStorm             |

## Coding Standards

### Keep It DRY and Simple

```typescript
// ❌ BAD - Repetitive category definitions
const category1 = { id: 1, name: "Love on the Rocks" };
const category2 = { id: 2, name: "Champagne & Plot Twists" };

// ✅ GOOD - Centralized data structure
const CATEGORIES = [
  { id: "dating", name: "Love on the Rocks", icon: "🍸" },
  { id: "never-have-i-ever", name: "Champagne & Plot Twists", icon: "🥂" },
];
```

### Component Reusability

```typescript
// ❌ BAD - Separate components for similar cards
<DatingCard />
<TruthCard />
<WouldYouRatherCard />

// ✅ GOOD - Single configurable component
<GameCard category={category} />
```

### Data Structure Consistency

All game JSON files should follow the same structure:

```typescript
{
  "id": "category-name",
  "name": "Drink Theme Name",
  "description": "Brief description",
  "difficulty": "easy" | "medium" | "hard",
  "cards": [
    { "id": string, "text": string, "difficulty": string }
  ]
}
```

## Marketing Message

**Tone**: Fun, playful, social, light-hearted
**Target**: Young adults 21+ looking for party entertainment
**Key phrases**:

- "Sip Happens" (playful on "Shit Happens")
- "Where every card is a cocktail of fun"
- "Simple rules, unforgettable nights"
- "Your drink, your game, your way"

### Copy Guidelines

- Keep language conversational and inclusive
- Emphasize the social bonding aspect
- Highlight simplicity and customization
- Use drink-related puns when appropriate
- Always remind to drink responsibly

```typescript
// ❌ BAD - Boring, generic
description: "A card game with questions";

// ✅ GOOD - On-brand, engaging
description: "Mix drinks, shuffle cards, and let the fun pour out with your crew";
```

## Architecture Principles

1. **Single source of truth**: Category data in `src/data/categories.ts`
2. **Lazy loading**: Load game cards only when category is selected
3. **Type safety**: Use TypeScript interfaces for all game data
4. **Performance**: Minimize re-renders, memoize expensive operations
5. **Accessibility**: Ensure all interactive elements are keyboard-navigable

## Feature Development Checklist

When adding new features:

- [ ] Does it keep the gameplay simple?
- [ ] Is it reusable across categories?
- [ ] Does it align with the marketing message?
- [ ] Is the code DRY (no duplication)?
- [ ] Does it enhance the social experience?

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Lovable-ghk21nznz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
