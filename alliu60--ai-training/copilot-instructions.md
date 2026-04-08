## ai-training

> Please convert the attached PDF file into a Chinese English two language version React component and strictly adhere to the following requirements:

Please convert the attached PDF file into a Chinese English two language version React component and strictly adhere to the following requirements:

1. **Add at the beginning of the file**: `"use client"`
2. **Character escaping rules** (CRITICAL - this is the most important and must be strictly followed):
    - Replace `'` with `&rsquo;` or `&apos;` in all JSX text content
    - Replace `"` with `&ldquo;` and `&rdquo;` in all JSX text content
    - Replace `&` with `&amp;` in all JSX text content
    - **Special note**: Quotes in JavaScript strings (inside `{}`, function parameters, etc.) do NOT need to be escaped, only JSX text content needs to be escaped
    - **Examples**:
        - ❌ `<p>Don't do this</p>`
        - ✅ `<p>Don&rsquo;t do this</p>`
        - ❌ `<p>"Hello world"</p>`
        - ✅ `<p>&ldquo;Hello world&rdquo;</p>`
        - ✅ `<button onClick={() => alert("This is fine")}>Click</button>` (JS strings are fine)
3. **Technical requirements**:
    - Use modern React function components with TypeScript
    - Use Tailwind CSS for styling (convert all inline styles)
    - Maintain responsive design with proper breakpoints
    - Retain all animations and interactive effects using CSS classes
    - Ensure all links work with proper `target="_blank" rel="noopener noreferrer"`
    - Convert all media embeds to proper React syntax
4. **Code quality**:
    - Use semantic HTML tags (`<main>`, `<section>`, `<article>`, `<nav>`, etc.)
    - Add appropriate ARIA labels for accessibility (`aria-label`, `role`, etc.)
    - Ensure all images have meaningful `alt` attributes
    - Use proper TypeScript interfaces for props and state
    - Add proper event handler types
5. **Critical checks before completion**:
    - ✅ Scan ALL text content for unescaped `'`, `"`, `&` characters
    - ✅ Verify all `className` attributes are properly set
    - ✅ Ensure event handlers use React syntax (`onClick`, `onChange`, etc.)
    - ✅ Check that component exports properly (`export default ComponentName`)
    - ✅ Validate that all JSX is properly closed
    - ✅ Ensure no HTML entities appear in JavaScript strings (only in JSX text)

**Pay special attention to character escaping rules in point 2, as this is the #1 cause of build failures.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Alliu60)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Alliu60)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
