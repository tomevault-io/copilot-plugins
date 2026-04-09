## claude-code-clipboard

> When providing code updates or modifications, please follow these guidelines to ensure compatibility with the user's VSCode extension:

# Instructions for Claude: Code Update Format

When providing code updates or modifications, please follow these guidelines to ensure compatibility with the user's VSCode extension:

0. Please format each code update using this structure in a separate view. This is very important. If there are 4 updates, there should be 4 separate views with code updates.

1. Use XML-like tags to structure your code updates. The main tag is `<code-update>`.

2. Each `<code-update>` tag should have the following attributes:
   - `action`: Specifies the type of update ("replace", "insert", or "delete")
   - `context`: Provides location context (e.g., "function calculateTotal", "class User")
   - `subcontext` (optional): For more specific targeting (e.g., "method __init__", "first occurrence")

3. Use one `<code-update>` tag for each distinct code change.

4. Provide clear, unique context for each update to aid in accurate matching.

5. Include complete code blocks or functions when replacing to ensure proper context.

6. Use the `<explanation>` tag to provide reasoning or additional information about the change.

7. For deletions, include a small portion of the code to be deleted in the tag body for verification.

8. Order multiple updates logically, typically from top to bottom of the file.

9. Use consistent indentation within code blocks.

10. Do not specify line numbers. Instead, rely on descriptive contexts for locating code.

11. Provide the code update snippet in a separate view, formatted as Markdown. This allows for easier testing and application of the update.


Example format:

```xml
<code-update action="replace" context="function calculateTotal">
def calculate_total(items):
    return sum(item.price for item in items if item.in_stock)
</code-update>

<explanation>
Added a condition to only sum prices of in-stock items.
</explanation>

<code-update action="insert" context="class ShoppingCart" subcontext="after method add_item">
def remove_item(self, item):
    if item in self.items:
        self.items.remove(item)
</code-update>

<explanation>
Added a method to remove items from the shopping cart.
</explanation>
```

Please format all code updates using this structure in a separate view. If you encounter any ambiguities or need to provide multiple options, please explain this in an `<explanation>` tag within the code block.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mister-scott)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mister-scott)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
