## luna

> Implement the requested change.

<task>
Implement the requested change.
</task>

These the instructions for implementing code changes in this project.
Do not deviate from these instructions and structure defined here.
If you cannot implement something because the instructions and structure doesn't allow reaching a working implementation, STOP IMPLEMENTING RIGHT AWAY. Tell me what changes you wanted to implement that go outside the instructions and structure here, why you needed them, and suggest how we can improve the instructions and strcuture here.
Even if the instructions are very limiting (cause we are now building them as we go), DO NOT deviate, and follow the plan to improve the instructions.
<important>
YOU CANNOT DO ANYTHING OTHER THAN WHATS LISTED HERE
</important>

# Pages

## Creating a new page

* Create a `.astro` file at `src/pages/` at the path matching the route of the page.

## Page Code

* Always use a [layout](#layouts)
* The basic structure of an astro page code is:
```astro
---
import SomeLayout from "@/layouts/<layout path>";
// ... more imports as needed

// Pages are rendered at built time. Add this export to make the page server side rendered.
// export const prerender = false;

// possibly async code that loads data for rendering
// ...
---
<SomeLayout {...maybeLayoutProps}>
    <!-- HTML elements and other Astro Components -->
</SomeLayout>
```
* Page code can only have 

# Layouts

<!-- listDir("src/layouts") -->
- BaseLayout.astro
<!-- end -->

## Using `BaseLayout`

```astro
---
import BaseLayout from "@/layouts/BaseLayout.astro";
---
<BaseLayout>
    <!-- elements here will go into a slot inside <head> -->
    <Fragment slot="body">
        <!-- elements here will go into a slot inside <body> -->
    </Fragment>
</BaseLayout>
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/netanelgilad)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/netanelgilad)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
