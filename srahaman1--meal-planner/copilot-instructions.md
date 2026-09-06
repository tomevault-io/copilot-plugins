## meal-planner

> This file provides guidance to Claude Code and other AI assistants when working with this repository.

# CLAUDE.md

This file provides guidance to Claude Code and other AI assistants when working with this repository.

## Project Overview

**Meal Planner** is an automated meal planning system for households with individualized macro targets and diabetic-friendly recipes.

**Key Features:**
- Automated recipe selection with 2-week cooldown
- Allergen filtering (hard filter - removes recipes)
- Dislike flagging (soft filter - warns but includes recipes)
- Portion calculation based on individual macro targets
- Grocery list generation
- Interactive CLI interface

**Target Users:**
- Households managing diabetes or pre-diabetes
- Families with different macro targets per person
- Batch meal prep on Sundays
- 3 meals/day with rotating recipes

## Architecture

### Core Principles

**YAGNI (You Aren't Gonna Need It):**
- Standard library only (no external dependencies)
- Simple algorithms (greedy selection, proportional allocation)
- POC-focused implementation
- Complex optimizations deferred to future

**DRY (Don't Repeat Yourself):**
- Modular design (selector, calculator, generator)
- Reusable utilities (JSON I/O, date helpers)
- Single source of truth for data (JSON files)

### Tech Stack

- **Python 3.8+** (standard library only)
- **JSON** for data storage (human-readable, easy to edit)
- **Markdown** for recipes (Obsidian-compatible)

### Key Design Decisions

**Why no external dependencies?**
- Faster iteration (no dependency management)
- Easier deployment (just Python + repo)
- Lower barrier to contribution
- Sufficient for POC scope

**Why JSON instead of database?**
- Human-readable and editable
- Version control friendly
- No setup required
- Sufficient for 265 recipes + 3 personas

**Why greedy algorithm instead of optimization?**
- Fast and "good enough" for POC
- Complexity doesn't justify optimization yet
- Can upgrade to constraint solver later if needed

## Project Structure

```
meal-planner/
├── meal_planner/              # Main package
│   ├── cli.py                 # Interactive CLI entry point
│   ├── utils.py               # JSON I/O, date helpers
│   ├── recipe_loader.py       # Allergen/dislike filtering
│   ├── recipe_selector.py     # Selection algorithm
│   ├── portion_calculator.py  # Serving size math
│   ├── plan_generator.py      # Complete plan generation
│   ├── grocery_generator.py   # Grocery list generation
│   └── swap_engine.py         # Find similar recipes
├── data/
│   ├── recipes/               # 265 markdown recipe files
│   ├── recipe-metadata.json   # Complete recipe database
│   ├── macro-profiles.json    # Macro target templates
│   └── personas.json          # Individual household members
├── scripts/                   # Supporting utilities
└── docs/                      # Documentation
```

## Data Model

### Macro Profiles

**Purpose:** Reusable macro target templates

**Structure:**
```json
{
  "id": "weight-loss-high-protein",
  "name": "Weight Loss - High Protein",
  "calories": 1600,
  "protein": 150,
  "carbs": 110,
  "fat": 62,
  "notes": "High protein for satiety"
}
```

**Key Insight:** Macro profiles are templates that personas reference. Multiple personas can share the same profile.

### Personas

**Purpose:** Individual household members with allergies and preferences

**Structure:**
```json
{
  "id": "john",
  "name": "John",
  "macro_profile": "weight-loss-high-protein",
  "allergies": ["peanuts", "shellfish"],
  "dislikes": ["raw-tomatoes"],
  "dietary_restrictions": [],
  "notes": "Pre-diabetic, weight loss goal"
}
```

**Key Distinctions:**
- **Allergies:** Hard filter - recipes completely removed from selection
- **Dislikes:** Soft filter - recipes flagged but still selectable (user can swap)
- **Dietary restrictions:** Not yet implemented

### Recipes

**Metadata:**
```json
{
  "slug": "korean-bbq-chicken-rice-bowls",
  "name": "Korean BBQ Chicken Rice Bowls",
  "category": "meals",
  "calories_per_serving": "580",
  "protein": "52g",
  "carbs": "48g",
  "fat": "22g",
  "key_ingredients": "chicken breast, rice, gochujang"
}
```

**Categories:**
- `breakfast` - Morning meals (26 recipes)
- `meals` - Lunch/dinner recipes (198 recipes)
- `sides` - Side dishes (25 recipes)
- `snacks` - High-protein snacks (16 recipes)

**Source:** Extracted from two high-protein meal prep cookbooks (265 total recipes)

### Meal Plans

**Structure:**
```json
{
  "plan_id": "2026-02-10",
  "week_start": "2026-02-10",
  "week_end": "2026-02-23",
  "recipe_summary": [...],
  "schedule": [...]
}
```

**recipe_summary:** Lists each recipe with total servings needed and breakdown by persona

**schedule:** 14-day daily schedule with portion sizes per persona per meal

## Algorithms

### Recipe Selection

**Algorithm:** Greedy selection with filters

**Process:**
1. Load all 265 recipes
2. **Hard filter:** Remove recipes with allergens (checks `key_ingredients`)
3. **Cooldown filter:** Remove recipes used in last 2 weeks
4. **Soft filter:** Flag recipes with disliked ingredients
5. Split by category (breakfast vs meals)
6. Select first N available (3 breakfasts, 4 meals)
7. Build 14-day schedule (rotate recipes)

**Improvement opportunities:**
- Optimize for macro targets (currently just picks first N)
- Balance protein sources (chicken vs beef vs seafood)
- Vary carb bases (rice vs pasta vs potatoes)
- Consider recipe complexity (prep time)

### Portion Calculation

**Algorithm:** Proportional allocation

**Formula:**
```
meal_allocation = daily_calories / 3  # 33% per meal
portion_size = meal_allocation / recipe_calories_per_serving
```

**Example:**
- Person: 1850 cal/day target
- Meal: Korean BBQ Chicken (580 cal/serving)
- Portion: (1850/3) / 580 = 1.06 servings

**Assumptions:**
- Equal meal distribution (33% each)
- Future: Meal-specific ratios (breakfast 25%, lunch 35%, dinner 40%)

**Trade-offs:**
- Simple and predictable
- Doesn't account for varying meal sizes
- Good enough for POC

### Macro Targeting

**Approach:** Hybrid (daily targets with ±10-15% variance)

**Why not daily precision?**
- Recipes have different macro profiles
- Forcing exact targets = restrictive selection
- Requires heavy reliance on supplemental items

**Why not pure weekly balance?**
- Diabetes management needs consistent carbs
- Large swings = inconsistent energy/hunger
- Example: 80g carbs Monday, 240g Tuesday = bad for blood sugar

**Hybrid approach:**
- Target daily macros but allow variance
- Weekly total = 7× daily target
- Prevents extreme days while allowing flexibility

## Development Workflow

### Testing Strategy

**Current:** Manual testing (POC phase)

**Process:**
1. Run CLI: `python -m meal_planner.cli`
2. Test workflows: generate plan, view plan, swap recipe, view grocery list
3. Verify allergen filtering
4. Check portion calculations
5. Validate macro totals

**Future:** Automated tests
- Unit tests for algorithms
- Integration tests for CLI workflows
- Fixture data for reproducible tests

### Code Style

**Conventions:**
- Type hints on all functions
- Docstrings (Google style)
- Descriptive variable names
- Keep functions focused (single responsibility)

**Example:**
```python
def calculate_portion(
    recipe: Dict,
    persona_name: str,
    meal_allocation: float = 0.33
) -> float:
    """Calculate portion size (in servings) for a persona.

    Args:
        recipe: Recipe dictionary with macros
        persona_name: Name of persona
        meal_allocation: Fraction of daily calories (default 0.33)

    Returns:
        Portion size in servings (e.g., 1.05 servings)
    """
```

### Git Workflow

**Branch naming:** `feature/feature-name` or `fix/bug-description`

**Commit messages:** Conventional commits
```
feat: add fiber tracking to recipe selection
fix: correct portion calculation for zero-carb recipes
docs: update user guide with swap workflow
```

**Co-authored commits:**
```
feat: add swap engine for recipe alternatives

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

## Common Tasks

### Adding New Recipes

1. Create markdown file in `data/recipes/{category}/`
2. Follow naming: `kebab-case.md`
3. Include: name, tags, ingredients, directions, macros
4. Run: `python scripts/rebuild-recipe-metadata.py`
5. Verify: `python scripts/generate-recipe-summary.py`

### Adding New Features

1. Read design doc: `docs/design.md`
2. Check implementation plan: `docs/implementation.md`
3. Create feature branch
4. Implement with TDD approach (if possible)
5. Update user guide: `docs/user-guide.md`
6. Test manually via CLI
7. Commit with descriptive message

### Debugging Issues

**Recipe not showing up?**
- Check allergen filtering (does it contain allergen ingredients?)
- Check cooldown (was it used in last 2 weeks?)
- Check category (breakfast vs meals)

**Portions seem wrong?**
- Verify macro profiles are correct
- Check portion calculation (meal_allocation / recipe_calories)
- Review recipe metadata (calories_per_serving accurate?)

**Grocery list missing ingredients?**
- Note: POC has recipe-based list (not full ingredient parsing)
- Full aggregation is future work
- Use recipe references as shopping guide

## Future Roadmap

### Immediate Next Steps (Post-POC)

1. **Full ingredient aggregation:** Parse recipe ingredients, aggregate quantities
2. **US unit conversions:** Grams → pounds, ml → cups
3. **Swap implementation:** Actually update plan (currently just shows alternatives)
4. **Fiber tracking:** Add fiber as optimization target

### Medium-Term Features

1. **Pantry intelligence:** Track what you have, only list what to buy
2. **Recipe ratings:** Track which recipes family likes
3. **Auto-remove recipes:** If swapped/delayed 3+ times, suggest removal
4. **Prep day scheduler:** Optimize cooking order (parallel tasks)

### Long-Term Vision

1. **Optimal recipe selection:** Constraint solver instead of greedy
2. **Meal-specific macro ratios:** Breakfast 25%, lunch 35%, dinner 40%
3. **Smart substitutions:** Suggest ingredient swaps (e.g., chicken → tofu)
4. **Nutrition analysis:** Vitamins, minerals, fiber tracking

## Health Context

**Important:** This system is designed for diabetes and pre-diabetes management.

**Key requirements:**
- **Consistent carbs:** Daily carb intake should be relatively stable (±15%)
- **High protein:** Promotes satiety and blood sugar control
- **Allergen safety:** Hard filter ensures allergens never appear
- **Portion control:** Individual serving sizes support weight management

**When making changes:**
- Consider impact on blood sugar (carb consistency)
- Maintain high-protein focus (average 49g protein/serving)
- Respect allergen filtering (no false negatives allowed)
- Keep portions flexible (people have different targets)

## Known Limitations

**POC Scope:**
- Greedy recipe selection (not optimized for macros)
- Recipe-based grocery list (no ingredient parsing)
- Manual testing (no automated tests)
- Simple CLI (no GUI)
- No meal-specific ratios (assumes 33% per meal)

**Intentional Trade-offs:**
- Standard library only (limits features but improves accessibility)
- JSON storage (simple but doesn't scale to thousands of recipes)
- Proportional allocation (simple but not perfectly optimal)

## Questions to Ask Before Making Changes

1. **Does this break allergen filtering?** (Critical - health risk)
2. **Does this maintain macro targeting?** (Core feature)
3. **Does this require external dependencies?** (Violates design principle)
4. **Does this add complexity for small gain?** (YAGNI check)
5. **Does this preserve backwards compatibility?** (User data safety)

## Support

**Documentation:**
- Design: `docs/design.md`
- Implementation: `docs/implementation.md`
- User guide: `docs/user-guide.md`
- Project history: `docs/history/`

**For Claude Code:**
- This repo was built with Claude Code during brainstorming and implementation sessions
- All design decisions are documented in `docs/design.md`
- Implementation steps are in `docs/implementation.md`
- Follow existing patterns when extending

---

**Last Updated:** 2026-02-08
**Project Status:** POC Complete, Ready for Production Use

---
> Source: [srahaman1/meal-planner](https://github.com/srahaman1/meal-planner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
