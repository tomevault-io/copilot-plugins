## replicube-deoptimization-rule

> Convert optimized Replicube LUA code into human-readable format by reversing the optimization techniques described in coding.md.

# Replicube Code Deoptimization Rule

## Purpose
Convert optimized Replicube LUA code into human-readable format by reversing the optimization techniques described in coding.md.

## Cursor Rule

When asked to "deoptimize" or "make readable" Replicube LUA code, apply these transformations:

### Convert Ternary-like Returns to If-Else Statements
**From:** `return condition and value or 0`
**To:**
```lua
if condition then
  return value
end
return 0
```

**From:** `return condition and value` (implicit or 0)
**To:**
```lua
if condition then
  return value
end
return 0
```

### Convert Complex Boolean Returns to If-Else
**From:** `return x == y and y == z and 11 or 0`
**To:**
```lua
if x == y and y == z then
  return 11
end
return 0
```

### Extract Repeated Calculations to Variables
**From:** Multiple occurrences of `x*x`. There must be AT MINIMUM 2 occurrences of each case to extract it, else do NOT extract to a variable.
Example:
```lua
if x*x > 2 and y*y > 4 then
  -- some code
end
if x*x > 3 and y*y < 6 then
  -- some other code
end
```
**To:**
```lua
local x_squared = x * x
local y_squared = y * y

if x_squared > 2 and y_squared > 4 then
  -- some code
end
if x_squared > 3 and y_squared < 6 then
  -- some other code
end
```
Example (of non-conversion due to only a single occurrence):
From:
```lua
if x*x > 2 and y*y > 4 then
  -- some code
end
```
To:
```lua
-- code is unchanged
if x*x > 2 and y*y > 4 then
  -- some code
end
```

### Add Local Scope for Readability
**From:** Variables without `local` scope
**To:** Add `local` keyword for better readability (even if it increases instruction count)

### Prefer abs(x) instead of x*x
**From:** `x*x == squared_value_of_x` (optimized absolute value comparison)
**To:** `abs(x) == value_of_x` or explicit absolute value logic

**From:** `x*x + z*z < value` (squared distance calculations)
**To:** Consider if this represents `abs(x) + abs(z) < threshold` (Manhattan distance) or keep as Euclidean distance.

**Note:** When `x*x` appears to be used for absolute value comparisons (especially in small integer ranges), convert back to `abs(x)` for clarity, taking into account to also update the value compared with. This includes local variables (feel free to rename them accordingly). Example:
From:
```lua
local x_squared = x*x
local y_squared = y*y

if x_squared < 4 and y_squared < 4 then
  return PEACH
end
```
to:
```lua
local x_abs = abs(x)
local y_abs = abs(y)

if x_abs < 2 and y_abs < 2 then
  return PEACH
end
```
Another example:
From:
```lua
if y * y > x * x then
  return YELLOW
end
```
To:
```lua
if abs(y) > abs(x) then
  return YELLOW
end
```

### Remove Single-Usage Variables
**From:** A variable that is only used once
**To:** Inlined value of the variable
Example:
From:
```lua
local x_abs = abs(x)
local y_abs = abs(y)

if y_abs > x_abs then
  return YELLOW
end
```
To:
```lua
if abs(y) > abs(x) then
  return YELLOW
end
```

### Add Explicit Return Statements
**From:** Implicit `return 0` (no return statement)
**To:** Explicit `return 0` at the end of functions

### Break Down Complex Conditions
**From:** Long chained conditions in single if statements
**To:** Multiple if-elseif statements or nested conditions with clear variable names

### Add Descriptive Variable Names
**From:** Single letter variables or abbreviated names
**To:** Descriptive variable names that explain their purpose

### Add Comments for Complex Logic
When finding complex logic, add comments explaining:
- What each condition checks
- The purpose of mathematical operations
- The meaning of color values returned

### Replace Color Numbers with Color Names
**From:** Direct numeric returns like `return 4`, `return 11`, etc.
**To:** Replace with descriptive color constants using this mapping:

```
0 EMPTY        8 ORANGE
1 WHITE        9 YELLOW
2 GREY         10 LIGHTGREEN
3 BLACK        11 GREEN
4 PEACH        12 DARKBLUE
5 PINK         13 BLUE
6 PURPLE       14 LIGHTBLUE
7 RED          15 BROWN
8 ORANGE       16 DARKBROWN
```

Do NOT define the colors as constants, assume they were already defined outside.

**Example:**
- `return 4` becomes `return PEACH`
- `return 11` becomes `return GREEN`
- `return 0` becomes `return EMPTY`

**Note:** Only replace direct returns (e.g., `return 7`), not color values used in calculations or variables.

### Format for Readability
- Proper indentation
- Consistent spacing
- Line breaks for complex expressions

## Example Transformations

### Example 1: Pythagorean Theorem (Keep Squared)
**Optimized:**
```lua
return x == y and y == z and 11 or (x*x + y*y == z*z and 5)
```

**Deoptimized:**
```lua
-- Check if all coordinates are equal (diagonal condition)
if x == y and y == z then
  return GREEN -- Perfect diagonal
end

-- Check if coordinates form a right triangle (Pythagorean theorem)
local x_squared = x * x
local y_squared = y * y
local z_squared = z * z

if x_squared + y_squared == z_squared then
  return PINK -- Right triangle pattern
end

return EMPTY -- Transparent/no color
```

### Example 2: Manhattan Distance (Convert to abs)
**Optimized:**
```lua
return y == 2 and x*x + z*z < 5 and 11 or 7
```

**Deoptimized:**
```lua
-- Check for top pattern using Manhattan distance
if y == 2 and abs(x) + abs(z) < 3 then
  return GREEN -- Top pattern
end

return RED -- Default color
```

## Usage
Apply this rule when you need to understand or debug optimized Replicube LUA code by converting it to a more readable format that prioritizes clarity over performance.

---
> Source: [Kartones/replicube-game-solutions](https://github.com/Kartones/replicube-game-solutions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
