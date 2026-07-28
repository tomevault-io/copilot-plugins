## kvcache-view

> This file provides guidance to Claude Code (claude.ai/code) when working with

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## IMPORTANT: Before Committing

1. **ALWAYS run `make check-commits`** to ensure commit messages follow these guidelines.
   - If check fails, run `make fix-commits` to automatically correct the format.

2. **ALWAYS run `make check-format`** to ensure HTML/JS code is properly formatted.
   - If check fails, run `make format` to automatically format the code.
   - Uses Prettier with project-specific settings (4-space tabs, single quotes, no semicolons).

## One commit per change

As with the Linux kernel, this project prefers commits to be atomic and to
the point. We don't want spell fixes to be blended in with code changes.
Spell fixes should go into separate commits. When in doubt, just don't do
any spell fixes unless asked explicitly to do that.

## Use the Signed-off-by tag

We want to use the Signed-off-by tag which embodies the application of the
Developer Certificate or Origin. Use the git configured user name and email
for the Signed-off-by tag (check with `git config user.name` and
`git config user.email`).

## AI Collaboration

This project is a collaboration between multiple AI assistants. Each AI
should use its own `Generated-by` tag to identify its contributions.

For example:
- `Generated-by: Claude AI`
- `Generated-by: ChatGPT Codex`
- `Generated-by: Gemini`

The `GEMINI.md` file is a symlink to this file to reflect this
collaborative environment.

**CRITICAL FORMATTING RULE**: The `Generated-by:` tag MUST be immediately
followed by the "Signed-off-by:" tag with NO empty lines between them.
These two lines must be consecutive.

Correct format:
```
Subject line

Detailed description of changes...

Generated-by: Gemini
Signed-off-by: Your Name <email@example.com>
```

## Avoid Shopping cart lists

Generative AI seems to like to make commit logs long itemized lists of things
it did. This is stupid. This should be avoided. It is creating very silly
commit logs. Use plain english and get to the point. Be as clear a possible
and get to the point of not what you want to communicate, but rather what
will make a reviewer easily understand what the heck you are implementing.

You should *think* hard about your commit log, always.

## Project-Specific Guidelines for kvcache-view

### Visual Consistency
When modifying the visualization:
- Maintain consistent color schemes (purple for weights, colors for sequences, red for saved memory)
- Keep animation speeds synchronized across related elements
- Ensure visual elements scale properly with different GPU memory sizes

### Performance Considerations
- Use requestAnimationFrame for smooth animations
- Batch DOM updates when possible
- Be mindful of canvas redraw frequency - the visualization runs continuously

### Memory Calculations
- Double-check memory formulas against the LMCache calculator
- Account for different data types (FP16, BF16, INT8, FP32)
- Remember that batch size multiplies memory requirements
- Flash Attention savings grow quadratically with sequence length

### UI Layout Principles
- Info panels should maintain fixed sizes to prevent jitter
- Use min-height CSS properties to reserve space for dynamic content
- Check for overlapping elements when features are enabled/disabled
- Auto-hide elements when screen space is insufficient

### Testing Checklist
Before committing visualization changes:
1. Test with different batch sizes (1, 2, 4, 8, 16)
2. Toggle all optimization features (CB, Paged, Flash)
3. Verify at multiple context lengths
4. Check different GPU models
5. Ensure no JavaScript console errors
6. Verify mobile responsiveness still works

### Common Pitfalls to Avoid
- **Variable name collisions**: Check for duplicate const declarations (e.g., tileSize)
- **Memory calculation errors**: Verify calculations match expected GPU capacity
- **Visual overlap**: Test with all possible UI element combinations
- **Animation performance**: Don't create too many particles or frequent redraws
- **Box sizing changes**: Use consistent heights to prevent UI elements jumping

### Commit Message Examples for This Project

Good:
```
fix: prevent info panel resize when Flash Attention toggled

The info panel would grow when Flash Attention was enabled due to
additional text being added. Fixed by using min-height containers
to reserve space regardless of content.
```

Bad:
```
fix: updated visualization.js with the following changes:
- modified updateInfoPanel function
- added min-height to div elements
- changed textContent to innerHTML
- adjusted Flash Attention display logic
```

### Mathematical Accuracy
When implementing memory calculations:
- KV cache = 2 * layers * (hidden_size / heads) * heads * dtype_bytes * sequence_length * batch_size
- Attention matrix (without Flash) = sequence_length² * dtype_bytes * batch_size
- Always validate against the LMCache calculator formulas

### Visual Debugging Tips
- Use console.log sparingly but strategically for state changes
- The browser's Performance tab can help identify rendering bottlenecks
- Test with browser zoom at different levels
- Check Canvas context save/restore pairs are balanced

---
> Source: [LMCache/kvcache-view](https://github.com/LMCache/kvcache-view) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
