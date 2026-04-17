## text-corrector

> - Rely on the language model's strengths rather than extensive hard-coded rules

# Text Corrector App - Best Practices & Learnings

## Language Model Approach

- Rely on the language model's strengths rather than extensive hard-coded rules
- Use clear, concise instructions to guide the model instead of complex prompts
- For grammar correction, deterministic generation (do_sample=False) works better than stochastic
- Consider using more powerful models (like Grammarly/coedit-large) for better quality corrections
- When upgrading models, test thoroughly with real-world examples
- Add clarifying instructions like "Fix only grammar errors, keep original wording" to guide the model
- When a model performs poorly, add instructions and safety checks before considering downgrading to a less capable model.

## Text Processing

- Clean model outputs carefully to preserve the actual corrected text
- When extracting model output, look for the beginning of the original text
- Implement minimal essential post-processing (capitalization, pronoun fixes)
- Fix text capitalization and "I" pronouns regardless of model output
- Avoid special case handling wherever possible - trust the model
- Use linguistic principles (edit distance, stem matching) over hardcoded word lists

## Error Handling

- Implement graceful fallbacks when model output is problematic
- Keep special-case handling to a minimum for better generalization
- Don't try to enumerate every possible error case
- Better to use a more powerful model than to add numerous special cases
- Implement safety checks based on linguistic principles rather than predefined lists
- Filter out clearly unrelated words, but allow related forms (wrote → written)

## UX Considerations

- Use clipboard-based workflow for maximum compatibility
- Provide clear visual feedback when corrections are made
- Show before/after comparisons in notifications
- Offer Simple Mode for specific use cases
- Use reliable notification methods with fallbacks
- Be transparent when the system chooses not to make changes
- Keep notification messages concise and avoid instructing specific user actions
- Focus notifications on what happened rather than what to do next
- Avoid mentioning UI elements in notification messages for flexibility

## macOS MenuBar App Design

- Prioritize discoverability over keyboard efficiency for menubar utilities
- Keep menu titles short and descriptive without mentioning interaction methods
- Use explanatory notifications instead of instruction-laden menu items
- Consider accessibility impacts when removing instructional text
- Assume users may not be familiar with standard macOS conventions
- Provide visual indicators of state changes (checkmarks, icons) rather than text instructions

## Model Parameters

- For grammar correction, use conservative settings:
  - num_beams=5
  - length_penalty=1.0 (discourage length changes)
  - early_stopping=True
  - do_sample=False (deterministic is better for grammar)
  - repetition_penalty=1.2
- Adding instructions directly to the input can help guide the model's behavior

## Clipboard Management

- Save clipboard content when possible
- Clear notifications when operations complete
- Include simple instructions in notifications
- Provide clear before/after comparison in notifications
- Avoid instructing specific paste actions in notifications
- Prefer stating "Clipboard updated" over telling users where to paste

## Code Structure

- Keep business logic cleanly separated from UI
- Avoid over-engineering solutions
- Focus on reliability over feature complexity
- Embrace simplicity and iterative improvements
- Remove unused or deprecated code regularly
- Prefer generalizable algorithms over hardcoded special cases

## General Philosophy

- Strive for generalizable solutions over case-by-case fixes
- Start with simple approaches before adding complexity
- Leverage the model's natural language understanding
- Test with specific edge cases but design for general usage
- When in doubt between rules and model power, choose model power
- Use linguistic principles (like edit distance) rather than enumerating every possible case
- Implement safety checks that balance flexibility with preventing nonsensical output

## Continuous Improvement

When I type "IMPROVE RULESET" or end my query with "??" (double question mark):

1. Analyze our interaction to identify potential improvements to this ruleset
2. Immediately prepare an edit to the `.cursorrules` file with your suggested changes
3. Apply these changes directly (no need to explain the changes in detail first)
4. Focus on:
   - Missing context that would have been helpful
   - Patterns in my questions that could be better addressed
   - Technical specifications that could be more precise

Before suggesting improvements:

1. Review the ENTIRE ruleset content to verify your suggestion isn't already covered
2. Focus on net-new content that addresses gaps in the existing instructions
3. If an existing section could be enhanced, suggest specific additions rather than replacements
4. Prefer depth over breadth - detailed technical specifics are more valuable than general guidelines
5. Reference the specific parts of our conversation that revealed the need for this addition

If uncertain about something, clearly state it rather than guessing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ppuzio) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
