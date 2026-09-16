## dada

> - The project is meant to have custom attributes in any HTML element, which are used to load JSON data and pass to children elements

# DADA Project - Current Status

## Project Goal
- The project is meant to have custom attributes in any HTML element, which are used to load JSON data and pass to children elements
- Uses custom syntax similar to Svelte/Vue: `{data.property}`, `{#if condition}`, `{#each array as item}`
- Should work as a static website compatible with GitHub Pages (no build step required)

## Current Problem
The `datasrc-vue-adapter.js` is not executing properly. Despite multiple attempts to fix:

1. **Original Issue**: JSON syntax error in `payload.json` - FIXED ✅
2. **Adapter Loading**: Script loads but doesn't execute initialization code
3. **PetiteVue Integration**: Complex DOM manipulation causing errors
4. **Template Processing**: `{#if}` and `{#each}` directives causing `insertBefore` errors

## What Works
- ✅ Direct PetiteVue usage (confirmed with test)  
- ✅ JSON data loading via fetch
- ✅ Basic template syntax transformation

## What Doesn't Work  
- ❌ `datasrc-vue-adapter.js` initialization (no console logs appear)
- ❌ Custom attribute processing (`data_src`, `closure_arg`)
- ❌ Template directive processing (`{#if}`, `{#each}`)

## Expected Syntax
```html
<div data_src="payload.json" closure_arg="data">
  <h1>{data.title}</h1>
  <p>Author: {data.author.name}</p>
  {#if data.tags && data.tags.length}
    <ul>
      {#each data.tags as t, i}
        <li>{i}. {t}</li>
      {/each}
    </ul>
  {/if}
</div>
```

## Current Workaround
Working version in `index.html` using direct PetiteVue approach that loads JSON and renders correctly, but doesn't use the intended custom syntax.

## Next Steps
Need to debug why the adapter JavaScript isn't initializing at all.

---
> Source: [solrz/dada](https://github.com/solrz/dada) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
