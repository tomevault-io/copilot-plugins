## ae-mcp

> generateScript: (params) => {

need to use ES3 for communication in generated scripts with AE

## ES3 Compatibility Rules for After Effects Scripts

### Critical Requirements
1. **NO template literals in generated ExtendScript**: Never use backticks (`) or `${expression}` syntax in ExtendScript code
2. **Use string concatenation pattern**: Build scripts using a `script` variable with += concatenation
3. **Escape quotes properly**: Use `\\\"` for quotes inside generated error messages (4 backslashes in the source code)
4. **No ES6+ features in ExtendScript**: No arrow functions, destructuring, const/let, spread operators, or other modern JS features

### Generator Function Pattern (REQUIRED)
All generator functions MUST follow this pattern:

```javascript
generateScript: (params) => {
  let script = '';
  script += 'var comp = app.project.itemByID(' + params.compId + ');\n';
  script += 'if (!comp) {\n';
  script += '  throw new Error("Composition not found");\n';
  script += '}\n\n';
  
  // Conditional code blocks
  if (params.someCondition) {
    script += 'layer.enabled = true;\n';
  }
  
  // String values with proper escaping
  script += 'layer.name = "' + helpers.escapeString(params.name) + '";\n';
  
  // Error messages with escaped quotes
  script += 'throw new Error("Property \\\\"" + propName + "\\\\" not found");\n';
  
  return script;
}
```

### Common Mistakes to Avoid

#### ❌ WRONG - Template Literals:
```javascript
generateScript: (params) => `
  var layer = comp.layer(${params.index});
  ${params.visible ? `layer.enabled = true;` : ''}
`
```

#### ✅ CORRECT - String Concatenation:
```javascript
generateScript: (params) => {
  let script = '';
  script += 'var layer = comp.layer(' + params.index + ');\n';
  if (params.visible) {
    script += 'layer.enabled = true;\n';
  }
  return script;
}
```

#### ❌ WRONG - Incorrect Quote Escaping:
```javascript
script += '  throw new Error("Property \"" + name + "\" not found");\n';
```

#### ✅ CORRECT - Proper Quote Escaping:
```javascript
script += '  throw new Error("Property \\\\"" + name + "\\\\" not found");\n';
```

### ES3 Feature Restrictions
The following JavaScript features are NOT available in After Effects ExtendScript:
- Template literals (backticks)
- Arrow functions
- const/let (use var only)
- Destructuring assignment
- Spread/rest operators (...)
- Default parameters
- for...of loops
- Array methods like .map(), .filter(), .reduce()
- Promise/async/await
- Classes (use prototype instead)
- Computed property names
- Symbol type

### Testing Generated Scripts
Always verify that generated ExtendScript code:
1. Contains no backticks or ${} syntax
2. Uses only var declarations
3. Has properly escaped quotes in strings
4. Uses ES3-compatible loops and conditionals
5. Handles arrays with for loops, not modern methods

### Common After Effects Script Pitfalls

#### Time Remapping
Before accessing the Time Remap property, you MUST enable it first:
```javascript
// WRONG - This will throw "property is hidden" error
var timeRemap = layer.property("Time Remap");

// CORRECT - Enable time remapping first
layer.timeRemapEnabled = true;
var timeRemap = layer.property("Time Remap");
```

Complete example for execute_script:
```javascript
var comp = app.project.itemByID(931);
if (!comp) {
  throw new Error("Composition not found");
}

var layer = comp.layer(2);
if (!layer) {
  throw new Error("Layer not found");
}

// IMPORTANT: Enable time remapping BEFORE accessing the property
if (layer instanceof AVLayer) {
  layer.timeRemapEnabled = true;
  
  // Now you can safely access Time Remap
  var timeRemap = layer.property("Time Remap");
  
  // Clear existing keyframes
  while (timeRemap.numKeys > 0) {
    timeRemap.removeKey(1);
  }
  
  // Set keyframes: 0 seconds maps to 0, 2 seconds maps to 4 (2x speed)
  timeRemap.setValueAtTime(0, 0);
  timeRemap.setValueAtTime(2, 4);
  
  // Adjust layer timing
  layer.outPoint = 2;
}

"Time remapping applied successfully";
```

**Better approach:** Use the `animate_time_remap` tool instead, which handles all of this automatically.

#### Effect Names
Some effects have been renamed in newer versions:
- "Fast Blur" → "Fast Box Blur" (use the applyEffect tool which handles this automatically)
- Check effect availability with `effects.canAddProperty(effectName)` before adding

# important-instruction-reminders
Do what has been asked; nothing more, nothing less.
NEVER create files unless they're absolutely necessary for achieving your goal.
ALWAYS prefer editing an existing file to creating a new one.
NEVER proactively create documentation files (*.md) or README files. Only create documentation files if explicitly requested by the User.

---
> Source: [p10q/ae-mcp](https://github.com/p10q/ae-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
