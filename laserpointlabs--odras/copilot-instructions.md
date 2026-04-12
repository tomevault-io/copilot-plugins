## odras

> rule "dark-theme-consistency" {

rule "dark-theme-consistency" {
  description = "Maintain consistent dark theme throughout the application"
  when = "creating or modifying UI components"
  then = "Use the defined dark color palette:
    - Background: #0f172a
    - Panel: #111827
    - Panel-2: #0b1220
    - Text: #e5e7eb
    - Muted: #9ca3af
    - Accent: #60a5fa
    - Border: #1f2937
    - Warning: #f59e0b
    - Success: #10b981
    - Error: #ef4444"
}

rule "cad-like-interface" {
  description = "Maintain CAD-like professional appearance"
  when = "designing UI layouts"
  then = "Follow CAD-style conventions:
    - Use panel-based layouts
    - Implement grid-based positioning
    - Include vertical toolbars (56px width)
    - Use tree views with dashed borders for hierarchy
    - Apply 6-8px border radius for interactive elements"
}

rule "no-decorative-elements" {
  description = "Focus on functionality over decoration"
  when = "adding visual elements"
  then = "Avoid decorative elements and maintain professional appearance:
    - No gradient backgrounds
    - No shadows unless functionally necessary
    - No animations unless they serve a purpose
    - Keep interfaces clean and minimal"
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/laserpointlabs)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/laserpointlabs)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
