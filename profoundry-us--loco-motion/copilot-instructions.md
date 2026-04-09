## loco-motion

> When implementing components, follow these procedures.

# Component Implementation

When implementing components, follow these procedures.

1. For DaisyUI styling variants like sizes, colors, etc., MUST use CSS classes
   directly rather than adding parameters to the component. For example, use
   `css: "component-primary component-lg"` rather than creating `size:` and
   `color:` parameters.

2. MUST focus on making components that follow the existing patterns in the
   library while keeping them simple and maintainable.

3. For example views, MUST follow the pattern of using `doc_title` with a block
   for the description, and each `doc_example` should include descriptions using
   `doc.with_description` blocks.

4. MUST follow the template structure provided in
   `docs/plans/templates/new_component.md` for any new component implementation
   plan.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/profoundry-us)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/profoundry-us)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
