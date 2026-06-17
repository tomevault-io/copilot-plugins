## onyx

> >


# Synthenis Onyx OSS — A Brain Made of LEGO Blocks

## What

A reasoning engine where each "skill" is a tiny self-contained module that
knows when it's relevant and what it would suggest. The engine combines all
registered skills into a unified ranking and adapts to each user's clicks
and dismissals over time.

## Why It Feels Intelligent

A single rule like "if no avatar, suggest adding one" is dumb.
Twenty rules combined with weighted signals, time-of-day awareness, account-age
adjustment, and per-user feedback decay **feels intelligent** because:

- It rarely repeats itself
- It learns what the user clicks vs ignores (with 14-day half-life decay)
- It adapts: morning vs night different suggestions
- Every suggestion includes a `reason` field for transparency

## API

```javascript
SynOnyx.init({user, lang, service})
SynOnyx.setUser(user) / setLang(lang) / setSignal(name, value)
SynOnyx.getSignal(name, default)

SynOnyx.registerSkill({...})
SynOnyx.registerSkills([...])
SynOnyx.unregisterSkill(id)

SynOnyx.suggest('home', {limit: 3})
SynOnyx.recordAction(id, 'click'|'dismiss'|'complete')

SynOnyx.explain('home')   // debug
SynOnyx.reset()
```

## Skill Schema

```javascript
SynOnyx.registerSkill({
  id: 'unique_id',                    // required, globally unique
  surfaces: ['home', 'settings'],     // optional, '*' for everywhere
  score(ctx) { return 0..1; },        // required
  render(ctx) {                       // required
    return {
      text: 'Suggestion text',
      icon: 'icon-name',              // optional
      action: { type: 'navigate', section: 'foo' },
      reason: 'why_this_fired'        // optional
    };
  }
});
```

## How AI Should Add New Skills

When a user asks "add a suggestion that does X":

1. Pick a globally unique snake_case `id`
2. Choose appropriate `surfaces`
3. Write `score(ctx)` returning 0 when irrelevant, 0..1 when relevant
   - Use `SynOnyx.getSignal()` to read context
   - Combine multiple signals when possible
4. Write `render(ctx)` returning `{text, icon, action, reason}`
5. Save to a NEW file in `skills/` — do not modify shipped files unless asked
6. Add a `<script>` tag to the HTML page

### Example: "Suggest enabling dark mode at night"

```javascript
SynOnyx.registerSkill({
  id: 'enable_dark_mode',
  surfaces: ['home', 'settings'],
  score(ctx) {
    if (SynOnyx.getSignal('darkMode')) return 0;
    const tod = SynOnyx.getSignal('timeOfDay');
    return tod === 'night' ? 0.85 : 0;
  },
  render() {
    return {
      text: 'Switch to dark mode',
      icon: 'moon',
      action: { type: 'callback', fn: 'toggleDark' },
      reason: 'time_of_day_night'
    };
  }
});
```

## Built-in Signals

| Signal | Type | Source |
|---|---|---|
| `plan` | string | `user.plan` |
| `accountAgeDays` | number | derived from `user.createdAt` |
| `timeOfDay` | string | derived |
| `hour` | number | 0-23 |
| `lang` | string | active language |

Set custom signals:
```javascript
SynOnyx.setSignal('messagesUsed', 47);
```

## Scoring Formula

```
final = clamp(
  base                              // skill.score(ctx) ∈ [0,1]
  + tanh(weightScore * 0.3) * 0.4   // per-user feedback adjustment
  , 0, 1
)
```

Weights decay with 14-day half-life. Below 0.04 they are GC'd.

## Privacy

- All weights live in `localStorage` under `syn_onyx_v2`
- No external API calls
- No telemetry transmitted
- `SynOnyx.reset()` purges everything

## Versioning

Semantic versioning. Major version bumps may change skill schema.
Always check `SynOnyx.VERSION`.

---
> Source: [synthenis/onyx](https://github.com/synthenis/onyx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
