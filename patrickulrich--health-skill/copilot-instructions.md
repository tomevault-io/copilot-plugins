## health-skill

> Comprehensive fitness and diet tracking with natural language meal logging, workout logging, food image analysis, automatic calorie/macro calculation, daily and weekly health summaries integrating Fitbit data, hydration tracking, saved meal shortcuts, exercise standardization (60+ exercises), progressive overload tracking with PR history, workout templates, recovery monitoring, natural language workout queries, and adaptive training suggestions. Use for logging meals ("I ate X for lunch") or workouts ("I did 34 situps"), querying PRs ("What is my bench PR?"), daily/weekly summaries, or health tracking queries.


# Fitness & Diet Management

## Coaching Philosophy

1. **Consistency over perfection** -- A logged imperfect day is more valuable than an untracked perfect one. Encourage logging even when choices aren't ideal.
2. **Progress the weakest link** -- Identify the one area that will yield the most improvement and focus coaching notes there rather than listing everything at once.
3. **Recovery is training** -- Sleep, rest days, and stress management are as important as workouts. Flag poor recovery patterns proactively.
4. **Small sustainable changes > dramatic overhauls** -- Recommend incremental adjustments (e.g., add one serving of protein per day) rather than complete diet rewrites.
5. **Data informs, doesn't dictate** -- Use trends and averages for coaching, not single-day outliers. One bad day doesn't erase a good week.
6. **Celebrate wins, don't just flag problems** -- Lead with strengths in coach's notes before areas for improvement. Positive reinforcement builds habit adherence.

## User Goals

All thresholds and targets are configurable via `config.json` under the `GOALS` section. When weight/height/age are set, calorie targets are calculated automatically using the Mifflin-St Jeor equation with activity multipliers.

| Setting | Default | Description |
|---------|---------|-------------|
| `goal_type` | maintenance | One of: maintenance, weight_loss, muscle_gain |
| `weight_kg` | null | Body weight in kg (enables protein target calculation) |
| `height_cm` | null | Height in cm (enables calorie target calculation) |
| `age` | null | Age in years (enables calorie target calculation) |
| `sex` | male | male or female (affects BMR calculation) |
| `activity_level` | moderate | sedentary, light, moderate, active, very_active |
| `protein_per_kg` | 0.8 | Protein target multiplier per kg body weight |
| `calorie_target` | null | Manual override for daily calorie target |
| `sodium_limit_mg` | 2300 | Daily sodium limit in mg |
| `fiber_target_g` | 38 | Daily fiber target in grams |
| `step_target` | 10000 | Daily step target |
| `sleep_target_h` | 7.0 | Daily sleep target in hours |

## Dietary Profile

Personalization for allergies, dietary restrictions, health conditions, and food preferences. Configured via `config.json` under the `DIETARY_PROFILE` section.

| Setting | Default | Description |
|---------|---------|-------------|
| `allergies` | [] | Food allergies (e.g., peanuts, shellfish, dairy, gluten) |
| `dietary_restrictions` | [] | Dietary restrictions (e.g., vegetarian, vegan, keto) |
| `dislikes` | [] | Foods to avoid in meal suggestions |
| `cuisine_preferences` | [] | Preferred cuisines (e.g., italian, mexican, asian) |
| `health_conditions` | [] | Health conditions affecting diet (e.g., diabetes, hypertension) |
| `cooking_skill` | null | Cooking skill level: basic, intermediate, advanced |
| `budget` | null | Food budget: budget, moderate, premium |
| `meal_timing` | null | Typical meal schedule |
| `meal_variety` | "balanced" | Recommendation style: explore, balanced, or consistent |
| `notes` | "" | Additional dietary notes |

**Gradual learning:** Preferences are learned over time through coach note prompts. Safety-critical items (allergies, health conditions) are asked after the first meal log. Other preferences are prompted gradually:
- Allergies + health conditions: after 1st meal (safety)
- Dietary restrictions: after 5th meal (coaching)
- Dislikes: after 10th meal (comfort)
- Cuisine preferences: after 15th meal or 1st meal plan request (planning)
- Cooking skill / budget: on 1st meal plan request (planning)
- Meal timing: after 20th meal (optimization)

**Quick setup:** Say "let's configure" to set all preferences at once.

**CLI:**
- `python3 scripts/dietary_profile.py --show` — View current profile
- `python3 scripts/dietary_profile.py --set allergies "peanuts,shellfish"` — Set a preference
- `python3 scripts/dietary_profile.py --next-prompt` — Get next gradual learning prompt
- `python3 scripts/dietary_profile.py --full-setup` — Get all unset preference prompts

## Core Capabilities

This skill manages personal health data through eight integrated workflows:

### 1. Natural Language Meal Logging
Process natural language food descriptions and automatically log to daily diet file with calorie/macro estimates using multi-source food database (750,000+ foods from local SQLite, OpenNutrition, and USDA API). Beverages are automatically tracked for hydration. Saved meal shortcuts are expanded before parsing.

**Triggers:**
- "I ate/had [food] for [meal]"
- "Just finished [food] for [meal]"
- "For lunch today I had [food]"

**Process:**
1. Expand saved meal shortcuts (see Saved Meals below)
2. Parse meal type (breakfast, lunch, dinner, snack)
3. Extract food items and quantities
4. Look up in food database (see references/food-database.md)
5. Calculate calories/macros using scripts/calculate_macros.py
6. Track beverages for hydration
7. Update today's diet log with meal details
8. Update daily totals (including hydration count)
9. Add coach's note if relevant

**Allergy Warnings:** When allergens are detected in a meal, warnings are displayed in the terminal and logged to the diet file. Warnings are based on the user's configured allergies and the allergen map (`allergen_map.json`). Keyword matches (direct food name) and contextual matches (broader dishes like "pad thai" for peanuts) are both checked.

**Example user input:** "I had a chicken breast and some rice for lunch"

**Expected log format:**
```markdown
### Lunch (~2:30 PM)
- Chicken breast (200g)
- White rice (1 cup)
  - Est. calories: ~450
  - Macros: ~40g protein, ~45g carbs, ~8g fat
  - Hydration: 1 beverage(s)
```

### 2. Natural Language Workout Logging
Process natural language exercise descriptions and automatically log to daily fitness file with workout details (type, volume, duration, intensity). Exercise names are automatically normalized via the exercise database (see Exercise Standardization below). Saved workout templates are expanded before parsing. PRs are recorded automatically. Consult references/workout-programming.md for exercise science context.

**Triggers:**
- "I did [number] [exercise]"
- "Just completed [number] [exercise]"
- "Finished [workout type]"
- "Went to the gym"
- "Did [exercise] today"
- "Completed [number] sets of [exercise]"
- "[template name]" (e.g., "push day", "pf express circuit")
- "3 sets of bench press, 3 sets of shoulder press" (comma-separated)

**Supported exercises:** 60+ exercises in `exercise_aliases.json` covering:
- Bodyweight: push-ups, pull-ups, dips, sit-ups, crunches, planks, squats, lunges, burpees
- Cardio: treadmill/running, elliptical, stationary bike, recumbent bike, ARC trainer, stair climber, rowing machine
- Free weights: bench press, squat, deadlift, overhead press, barbell row, DB exercises
- Machines (Planet Fitness): chest press, lat pulldown, shoulder press, leg press, leg extension, leg curl, bicep curl, tricep extension, ab crunch, seated row, and more
- Smith machine: squat, bench, row, OHP, lunge, calf raise
- Functional: kettlebell swing, battle ropes, resistance band
- Flexibility: stretching, yoga, mobility work

**Process:**
1. Expand saved workout templates (see Saved Workouts below)
2. Normalize exercise names via exercise database
3. Detect exercise type and workout category
4. Extract exercise name, sets, reps, weight, duration, distance
5. Split comma/and-separated exercises for multi-exercise parsing
6. Create/append to today's workout log
7. Calculate volume (sets x reps x weight) for resistance training
8. Log intensity (light, moderate, hard, failure)
9. Record PRs to pr_history.json (announces new PRs)

### 3. Food Image Analysis
Analyze food images and log meals with visual estimation.

**Process:**
1. Save image to $WORKSPACE/diet/images/YYYY-MM-DD-meal-#.jpg
2. Use vision model to identify foods and estimate portions
3. Log meal with calorie/macro estimates
4. Reference image path in diet log

### 4. Daily Health Summary Generation
Generate comprehensive daily health report combining Fitbit data, diet logs, and workout logs. All thresholds come from GOALS config. Consult references/nutrition-targets.md for coaching context.

**Triggers:**
- Nightly cron job (11:35 PM)
- User requests daily summary
- Message: "Run daily health summary"

**Process:**
1. Read Fitbit data from $WORKSPACE/fitness/fitbit/YYYY-MM-DD.json
2. Read diet log from $WORKSPACE/diet/YYYY-MM-DD.md
3. Read workout log from $WORKSPACE/fitness/YYYY-MM-DD.md
4. Generate summary including:
   - Diet overview (calories, macros, sodium, fiber, hydration, meals)
   - Fitness overview (steps, heart rate, sleep, weight, calories burned)
   - Workout overview (volume, intensity, session count)
   - Net balance (calories consumed - burned, calorie target if configured)
   - Coach's notes and tomorrow's focus

### 5. Weekly Health Summary
Generate weekly trend analysis comparing current week to previous week. Includes exercise progression (PRs, trends, stalled lifts), recovery notes, and adaptive plan suggestions.

**Triggers:**
- User requests weekly summary
- Message: "Run weekly summary"
- Weekly cron (optional)

**Process:**
1. Collect daily data for Mon-Sun of the target week
2. Calculate averages (calories, protein, carbs, fat, steps, sleep, hydration)
3. Count consistency (days with meals, workouts, Fitbit data)
4. Sum weekly totals (resistance volume, cardio minutes, total steps)
5. Compare to previous week for trend arrows (up/down/stable)
6. Generate exercise progression section (PRs, 4-week trends, stalled lifts)
7. Generate recovery notes (neglected muscles, insufficient recovery)
8. Generate weekly coach notes (trends, consistency, next week focus)
9. Generate adaptive plan suggestions (deload, scheduling, recovery, fatigue)

**Script:** `python3 scripts/generate_weekly_summary.py [YYYY-MM-DD]`

### 6. Hydration Tracking
Beverages are automatically detected during meal logging and tracked as hydration data.

**Tracked beverages:** water, coffee, tea, soda, juice, smoothie, beer, wine, milk, lemonade, sparkling

**Process:**
1. During meal analysis, beverages are identified from food items
2. Beverage count is logged per meal in the diet file
3. Daily totals include hydration count
4. Daily summary shows hydration in Diet Overview
5. Coach notes praise good hydration (6+) and flag low hydration (<4)

### 7. Workout Query
Answer natural language questions about workout history, PRs, trends, and Fitbit metrics.

**Triggers:**
- "What is my [exercise] PR?"
- "When did I last do [exercise]?"
- "How many times did I [exercise] this week?"
- "How has my sleep trended this month?"

**Process:**
1. Classify query type (pr, last_workout, count, trend, summary)
2. Normalize exercise name via exercise database
3. Look up data from pr_history.json or Fitbit files
4. Return human-readable answer

**Script:** `python3 scripts/query_history.py 'your question'`

### 8. Meal Planner
Suggest meals based on remaining macros, user dietary profile, and preferences. Uses curated meal templates (~60 meals) with optional TheMealDB API enrichment (300+ recipes).

**Triggers:**
- "Plan my dinner"
- "What should I eat?"
- "Suggest a meal"
- "Meal plan"
- "What should I cook?"

**Process:**
1. Calculate remaining macro budget (calories, protein, carbs, fat, sodium)
2. Infer meal type from time of day (or use specified type)
3. Load meal templates from `meal_templates.json`
4. Filter by allergens, dietary restrictions, dislikes, cooking skill, budget, season, difficulty
5. Score templates by macro fit (calorie + protein), sodium safety, cuisine preference, variety, and randomness
6. Return top suggestions with macro fill percentages
7. Optionally enrich with TheMealDB API results

**Variety modes** (`meal_variety` in DIETARY_PROFILE):
- **explore**: Aggressively suggests new cuisines and ingredients you haven't tried recently
- **balanced** (default): Mix of familiar favorites and new suggestions
- **consistent**: Recommends meals matching your established eating patterns and preferred cuisines

**Scoring:** Uses 10 weighted components (calorie_fit, protein_fit, sodium_ok, cuisine_bonus, cuisine_diverse, novelty_bonus, repetition_penalty, familiarity_bonus, pattern_match, random_factor) with weights selected by variety mode. All weights sum to 1.0.

**Filter relaxation:** If too few results, filters are relaxed in order: budget → cooking_skill → seasons → difficulty → dislikes. Allergens and dietary restrictions are NEVER relaxed.

**CLI:**
- `python3 scripts/meal_planner.py` — Suggest for next meal
- `python3 scripts/meal_planner.py --type dinner` — Suggest dinner
- `python3 scripts/meal_planner.py --count 3` — Top 3 suggestions
- `python3 scripts/meal_planner.py --remaining` — Show remaining macros

**Output format:**
```
Remaining today: ~800 cal, 45g protein, 100g carbs, 30g fat

Suggested meals for dinner:

1. Grilled Chicken with Rice and Broccoli
   ~520 cal | 45g protein | 52g carbs | 10g fat
   Prep: 25 min | Skill: Basic | American
   Fills 65% of remaining calories, 100% of remaining protein
```

## Exercise Standardization

All exercise names are normalized through `exercise_aliases.json`, which maps common names and abbreviations to canonical exercise names. This ensures consistent tracking across workouts.

**File:** `exercise_aliases.json` (in skill root, ~60 exercises)

Each entry contains:
- `canonical`: The standard name used for tracking (e.g., "Bench Press")
- `aliases`: Common names and abbreviations (e.g., "bench", "flat bench", "bb bench")
- `muscle_groups`: Muscles targeted (e.g., ["chest", "shoulders", "triceps"])
- `type`: Exercise classification (compound, isolation, bodyweight, cardio)

**Adding custom exercises:** Edit `exercise_aliases.json` and add a new entry following the existing format. The database is loaded once at import time; call `exercise_db.reload_db()` to refresh.

**Planet Fitness 30-Minute Express Circuit:** The 10 circuit stations are all mapped in the database. Use the "pf express circuit" saved workout template to log all 10 at once.

## Saved Workouts

Saved workouts are shortcuts for common workout routines stored in `saved_workouts.json`. When a saved workout name is used as input, it is expanded to the full exercise list before parsing.

**File:** `saved_workouts.json` (in skill root)
```json
{
  "push day": "3 sets of chest press machine, 3 sets of incline press machine, ...",
  "pull day": "3 sets of lat pulldown machine, 3 sets of seated row machine, ...",
  "leg day": "4 sets of smith machine squats, 3 sets of leg press machine, ...",
  "pf express circuit": "seated row, leg press, leg curl, ab crunch, ..."
}
```

**Usage:**
- `python3 scripts/log_workout.py "push day"` -- expands and logs all exercises
- Save new templates: `python3 scripts/log_workout.py --save "template name" "exercise description"`

## Saved Meals

Saved meals are shortcuts for frequently eaten meals stored in `saved_meals.json`. When a saved meal name appears in meal text, it is expanded to the full description before parsing.

**File:** `saved_meals.json` (in skill root)
```json
{
  "my usual breakfast": "2 eggs and toast with coffee",
  "post workout shake": "protein shake with banana and almond milk"
}
```

**Usage:**
- "I had my usual breakfast" expands to "I had 2 eggs and toast with coffee"
- Save new shortcuts: `python3 scripts/calculate_macros.py --save "meal name" "food description"`

## Diet Log Format Standard

**CRITICAL:** Always follow this exact format so the daily summary script can parse macros correctly.

### Meal Entry Format
```markdown
### {MealType} (~{Time})
- {Food item 1}
- {Food item 2}
  - Est. calories: ~{calories}
  - Macros: ~{protein}g protein, ~{carbs}g carbs, ~{fat}g fat
  - Sodium: ~{sodium}mg (optional)
  - Fiber: ~{fiber}g (optional)
```

### Daily Totals Format (REQUIRED)
```markdown
## Daily Totals
- Calories: ~{total} kcal
- Protein: ~{total}g
- Carbs: ~{total}g
- Fat: ~{total}g
- Fiber: ~{total}g (optional)
- Hydration: {count} beverages (optional)
```

**IMPORTANT:**
- Use exact format: `- Calories: ~950 kcal` (NOT "Total calories" or ranges like "900-1,000")
- Single values only, no ranges
- The summary script regex matches these exact patterns
- When logging manually, calculate midpoint of any range and use single number

## File Structure

**Health summaries:** $WORKSPACE/summaries/YYYY-MM-DD.md
- Combined daily health report (diet + workout + Fitbit + coach notes)
- Auto-generated by `generate_daily_summary.py`

**Diet logs:** $WORKSPACE/diet/YYYY-MM-DD.md
- Meals with timestamps
- Calorie/macro estimates
- Hydration tracking
- Daily totals (including beverage count)

**Workout logs:** $WORKSPACE/fitness/YYYY-MM-DD.md
- Workout sessions with timestamps
- Exercise details (name, sets, reps, weight, volume)
- Cardio sessions (type, duration, distance)
- Intensity tracking

**Fitness data:** $WORKSPACE/fitness/fitbit/YYYY-MM-DD.json
- Comprehensive Fitbit data (see Tracked Metrics below for full list)
- All raw API responses preserved for future analysis

**Fitbit sync script:** `scripts/fitbit-integration/fetch-fitbit.sh`
- Run manually: `./scripts/fitbit-integration/fetch-fitbit.sh [YYYY-MM-DD]`
- Auto-sync: Cron job at 11:50 PM daily

**Food images:** $WORKSPACE/diet/images/YYYY-MM-DD-meal-#.jpg

**Configuration:** config.json (GOALS, DIETARY_PROFILE, data sources, paths), `.env` (API keys, secrets)

**Saved meals:** saved_meals.json (meal shortcuts)

**Saved workouts:** saved_workouts.json (workout template shortcuts)

**Exercise database:** exercise_aliases.json (60+ exercises with aliases and muscle groups)

**PR history:** pr_history.json (per-exercise PR and session history, auto-generated)

**Allergen map:** allergen_map.json (14 allergens with food keywords and contextual matches)

**Meal templates:** meal_templates.json (~60 curated meals with macros, allergens, and tags)

**Dietary profile state:** dietary_profile_state.json (internal gradual learning state, auto-generated)

## Tracked Metrics

**Diet:**
- Calories (total and per meal)
- Protein (grams, with configurable g/kg target)
- Carbohydrates (grams)
- Fat (grams)
- Sodium (milligrams, with configurable daily limit)
- Fiber (grams, with configurable daily target)
- Hydration (beverage count)

**Fitness (from Fitbit):**

*Activity:*
- Steps (with configurable target)
- Distance (km)
- Floors climbed
- Elevation (meters)

*Active Minutes:*
- Sedentary minutes
- Lightly active minutes
- Fairly active minutes
- Very active minutes

*Heart:*
- Resting heart rate
- Heart rate intraday (per-minute breakdown)
- Heart Rate Variability (HRV) - daily RMSSD

*Sleep:*
- Duration (with configurable target)
- Sleep efficiency/score (if available)

*Body Composition:*
- Weight
- BMI
- Body fat percentage

*Energy:*
- Calories burned

**Workouts (manual logging):**
- Resistance training volume (lbs moved: sets x reps x weight)
- Cardio time (minutes)
- Workout sessions (per day)
- Workout intensity (light, moderate, hard, failure)
- Personal records (weight PR, volume PR per exercise)
- Exercise progression trends (improving, stalled, declining)
- Muscle group recovery tracking (neglected, insufficient recovery)

## Reference Materials

Consult these references when providing coaching advice:

| Reference | When to consult |
|-----------|-----------------|
| `references/nutrition-targets.md` | Giving diet advice, assessing macros, setting calorie targets, meal timing |
| `references/workout-programming.md` | Analyzing workout volume, suggesting exercises, programming splits, deloads |
| `references/recovery.md` | Poor sleep patterns, overtraining signs, rest day advice, stress management |
| `references/food-database.md` | Quick lookup of common food macros |
| `references/macro-guidelines.md` | General macro ratio guidelines |
| `references/summary-template.md` | Daily summary format reference |

## Database Setup

The food lookup system uses a **multi-source architecture** -- all enabled sources are queried simultaneously and results are merged by relevance. Configure sources via `FOOD_SOURCES` in config.json (default: `["local_db", "opennutrition", "usda_api"]`). Sources that are unavailable are silently skipped.

| Source | Config key | Type | Size |
|--------|-----------|------|------|
| `local_db` | `DB_PATH` | SQLite (ComprehensiveFoodDatabase) | ~450K foods |
| `opennutrition` | `OPENNUTRITION_DB_PATH` | SQLite (imported via `scripts/import_opennutrition.py`) | ~300K foods |
| `usda_api` | `USDA_API_KEY` | REST API (USDA FoodData Central) | Live queries |

Setup:
- **local_db**: Run `python3 scripts/import_local_db.py` (requires `megatools`, downloads ~1.4 GB from Mega.nz)
- **opennutrition**: Run `python3 scripts/import_opennutrition.py` (downloads 282MB, imports ~300K foods into `data/opennutrition.sqlite`)
- **usda_api**: Register at https://fdc.nal.usda.gov/api-key-signup, set `HEALTH_SKILL_USDA_API_KEY` in `.env`

## Resources

**scripts/calculate_macros.py** - Parse natural language and calculate macros from food database (with hydration tracking and saved meals)
**scripts/log_workout.py** - Parse natural language workouts, expand templates, normalize names, record PRs
**scripts/generate_daily_summary.py** - Generate comprehensive daily health reports (diet + workout + Fitbit + PRs)
**scripts/generate_weekly_summary.py** - Generate weekly trend analysis with exercise progression and adaptive plans
**scripts/exercise_db.py** - Exercise name normalization and muscle group lookup
**scripts/progressive_overload.py** - PR tracking, stall detection, and progression trends
**scripts/recovery_tracking.py** - Muscle group recovery monitoring
**scripts/query_history.py** - Natural language workout query engine
**scripts/regenerate_summary.py** - Regenerate summary for specific date
**scripts/dietary_profile.py** - Dietary profile management and gradual preference learning
**scripts/allergy_checker.py** - Allergen detection during meal logging
**scripts/meal_history.py** - Meal history analysis, ingredient-based cuisine detection, and caching
**scripts/meal_planner.py** - Meal suggestion engine with scoring, filtering, and seasonal awareness
**scripts/themealdb.py** - Optional TheMealDB API client for recipe enrichment

---
> Source: [patrickulrich/health-skill](https://github.com/patrickulrich/health-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
